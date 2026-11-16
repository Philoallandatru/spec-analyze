# 完整性和数据加密（Integrity and Data Encryption, IDE）

> **一句话总结**：IDE 是 PCIe 6.0 引入的硬件级"链路装甲"，使用 AES-256-GCM 对 PCIe 总线上传输的 TLP 进行线速加密、完整性校验和防重放保护，防止物理探测器和恶意插槽窃取或篡改数据。

如果把 PCIe 总线比作一条高速公路，以前的数据包（TLP）就像敞篷货车——谁在主板上接个物理逻辑分析仪（Interposer）就能把车上的机密数据看得一清二楚，甚至偷偷掉包。IDE 相当于给每辆货车装上军用级防弹车厢（AES-256-GCM 加密）和一次性防伪封条（96-bit MAC + 递增包号），即使路边有人拦截，也既看不懂内容、又无法篡改或重放旧数据包。

---

## 为什么需要 IDE？

随着机密计算（Confidential Computing，如 Intel TDX、AMD SEV-SNP）的普及，CPU 内部内存已经加密，但 CPU 与 GPU、NVMe SSD、网卡之间通过 PCIe 总线传输的数据仍处于"明文裸奔"状态。IDE 填补了这一最后防线，提供三大核心安全保障：

1. **机密性（Confidentiality）**：使用 **AES-256-GCM** 算法加密 TLP 负载（及部分头部信息），防止物理总线嗅探。
2. **完整性（Integrity）**：为每个受保护的 TLP 生成 **96 位（12 字节）MAC 认证标签**，任何 1 比特篡改都会在接收端被硬件立即拦截。
3. **防重放（Replay Protection）**：通过单调递增的 **Packet Number（包号）** 参与 GCM IV 构造与认证，防止攻击者录制并重放历史合法数据包。

---

## IDE 的三种保护模式（加密范围）

```text
┌──────────┐   Link IDE    ┌────────┐   Link IDE    ┌──────────┐
│   CPU    │◄═════════════►│ PCIe   │◄═════════════►│ Endpoint │
│   RC     │               │ Switch │               │ (GPU/SSD)│
└──────────┘               └────────┘               └──────────┘
     ▲                                                    ▲
     └──────────── Selective IDE (端到端按地址/流选择) ─────┘
```

| 模式 | 保护范围 | 核心特点与适用场景 |
| :--- | :--- | :--- |
| **Link IDE** | 相邻两个直连端口之间（逐跳加密） | 对链路上指定 Traffic Class（TC）的所有 TLP 加密；若经过 Switch，Switch 需解密后再用下一跳密钥重新加密 |
| **Selective IDE** | Root Complex 与特定 Endpoint 之间（端到端穿透 Switch） | 仅加密匹配特定**地址范围（Address Association）**或特定 Requester ID 的 TLP；Switch 无需拥有密钥，仅需透传 |
| **Flow-Through IDE** | Selective IDE 的硬件流模式 | 允许中间组件在不解密负载的情况下透传 IDE TLP，是实现端到端机密虚拟机（TEE-I/O）直通的关键 |

---

## IDE 加密机制与 TLP 封装格式

IDE 基于 **AES-256-GCM（Galois/Counter Mode）** 认证加密算法，硬件在事务层/数据链路层边界以流水线方式完成加解密，典型额外延迟仅数十纳秒，对吞吐影响极低。

### 加密前后的 TLP 对比

```text
普通 TLP：
┌──────────────┬──────────────────┬─────────┬──────┐
│  TLP Header  │   Data Payload   │  ECRC   │ LCRC │
└──────────────┴──────────────────┴─────────┴──────┘

IDE 保护的 TLP：
┌──────────────┬──────────────────┬──────────────┬──────┐
│  TLP Header  │    Encrypted     │  96-bit MAC  │ LCRC │
│ + IDE Prefix │  Payload (+PCRC) │   (12 Bytes) │      │
└──────────────┴──────────────────┴──────────────┴──────┘
       ▲                 ▲               ▲
   作为 AAD 参与     AES-256-GCM     接收端校验失败则
   完整性认证(不加密)   流式加密       丢弃并触发 AER 错误
```

- **AAD（附加认证数据）**：未加密的 TLP Header 字段与 IDE Prefix（含 Stream ID、Packet Number 等）都会作为 AAD 参与 MAC 计算，防止攻击者篡改目标地址或路由信息。
- **PCRC（Partial Header / Protected CRC）**：部分模式下提供额外的完整性校验保护。
- **密钥槽位与无缝轮换**：每个 IDE Stream 通常维护两组密钥槽（Key Set 0 与 Key Set 1），支持在不中断业务流量的情况下完成**在线密钥轮换（Key Rotation）**。

---

## IDE Extended Capability 与密钥生命周期

### 1. 配置空间结构（Capability ID `0x0030`）

IDE 能力通过 **IDE Extended Capability（`0x0030`）** 暴露在 Root Port、Switch 端口和 Endpoint 的扩展配置空间中：

| 关键寄存器 | 核心作用 |
| :--- | :--- |
| **IDE Capability Register** | 声明是否支持 `Link IDE`、`Selective IDE`、`Flow-Through IDE`、`PCRC`、`IDE_KM`、支持的流数量（Num Streams）及 Traffic Class 数 |
| **IDE Control Register** | 全局控制位（如 Flow-Through IDE Enable） |
| **Link IDE Stream Control / Status** | 配置每个 Link IDE 流的使能、重放保护（PR Enable）、算法（AES-GCM-256）、TC 位图与 Stream ID |
| **Selective IDE Stream Control & Address Association** | 配置端到端保护的 **4KB 对齐基地址（Base）与上限地址（Limit）**、关联的 Stream ID 及有效位 |

### 2. 密钥管理与激活流程（依赖 DOE + SPDM + IDE_KM）

IDE 本身只负责加解密引擎，**密钥的协商、下发和轮换全部通过 DOE 通道完成**：

1. **身份认证**：Host 通过 **DOE** 运行 **CMA / SPDM** 协议验证设备证书与固件度量，建立加密的 Secured SPDM 会话。
2. **密钥下发（`IDE_KM KEY_PROG`）**：Host 生成随机密钥并通过 `IDE_KM` 协议（封装在 Secured SPDM 中）分别写入通信两端（如 Root Port 和 Endpoint）的指定 Key Set。
3. **激活生效（`IDE_KM KEY_SET_GO`）**：两端确认密钥就绪后，发送 `KEY_SET_GO` 正式启用该 Stream 的硬件加密。
4. **密钥轮换**：当加密数据量或时间达到阈值时，向备用 Key Set 下发新密钥并平滑切换，随后通过 `KEY_SET_STOP` 销毁旧密钥。

---

## 关联与引申概念

- **DOE（Data Object Exchange）**（[doe.md](doe.md)）：IDE 密钥管理协议（`IDE_KM`）的底层传输信箱，没有 DOE 就无法安全配置 IDE 密钥。
- **CMA / SPDM**：在启用 IDE 加密前，必须先通过 SPDM 完成设备身份认证与会话密钥派生，确保"加密通道的对端确实是合法硬件"。
- **TEE-I/O（TDX Connect / SEV-TIO）**：机密计算 I/O 标准，结合 **IDE**（链路加密）与 **ACS/IOMMU**（访问隔离），让机密虚拟机（CVM）能够安全地直通 GPU 或 NVMe 设备。
- **AER（高级错误报告）**（[../error-handling/aer.md](../error-handling/aer.md)）：当 IDE 检测到 MAC 校验失败（疑似篡改）或包号回退（疑似重放攻击）时，会通过 AER 上报不可纠正安全错误。
