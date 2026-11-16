# 数据对象交换（Data Object Exchange, DOE）

> **一句话总结**：DOE 是 PCIe 配置空间里的一个标准化"加密信箱（Mailbox）"，让主机和设备在无需分配 BAR/MMIO 内存的情况下，安全地交换设备认证（SPDM/CMA）和链路加密密钥（IDE_KM）等管理消息。

如果把 PCIe 设备的 BAR 空间比作正式营业厅，那么配置空间就是设备出厂自带的"24 小时自助快递柜"。在设备还没被操作系统分配好内存地址、或者还没建立信任之前，主机就可以通过 **DOE（Data Object Exchange）** 这个标准化快递柜，一格一格地投递和取回认证证书、固件度量值和加密密钥，彻底告别各家厂商五花八门的私有寄存器协议。

---

## 为什么需要 DOE？

在 PCIe 5.0 ECN / PCIe 6.0 引入 DOE 之前，安全认证与密钥管理面临两个痛点：
1. **依赖 BAR 资源与专用驱动**：传统带内管理接口需要占用 MMIO BAR 空间且格式不统一，系统启动早期（Pre-OS/BIOS 阶段）或未加载厂商驱动时难以通用交互。
2. **缺乏标准化安全传输通道**：零信任和机密计算（Confidential Computing）要求标准化运行 **SPDM**、**CMA**、**IDE_KM** 等协议，DOE 正是承载这些协议的统一传输层。

---

## DOE 架构与寄存器模型

DOE 作为一个 **PCIe Extended Capability（Capability ID `0x002E`）** 存在于设备或端口的扩展配置空间中。主机软件与设备硬件通过一套简单的 **Mailbox（邮箱）** 寄存器以 32 位 DWORD 为单位收发数据对象（Data Object）。

| 偏移 | 寄存器名称 | 关键位字段与作用 |
| :--- | :--- | :--- |
| `0x00` | Extended Capability Header | Capability ID = `0x002E`，Version，Next Capability Offset |
| `0x04` | **DOE Capabilities** | Bit 0：`DOE Interrupt Support`（是否支持中断通知）；Bits 11:1：MSI/MSI-X 中断消息号 |
| `0x08` | **DOE Control** | Bit 0：`DOE Abort`（中止并复位邮箱）；Bit 1：`DOE Interrupt Enable`；Bit 31：`DOE Go`（写入 1 通知设备请求已写完，开始处理） |
| `0x0C` | **DOE Status** | Bit 0：`DOE Busy`；Bit 1：`DOE Interrupt Status`；Bit 2：`DOE Error`；Bit 31：`Data Object Ready`（设备已准备好响应数据，可读取） |
| `0x10` | **DOE Write Data Mailbox** | 主机逐个 DWORD 写入请求数据对象（地址固定，硬件内部自动递增缓冲指针） |
| `0x14` | **DOE Read Data Mailbox** | 主机逐个 DWORD 读取响应数据对象（每读一次，硬件自动推进到下一个 DWORD） |

### Mailbox 请求/响应交互流程

```text
Host (OS / Firmware)                                PCIe Device (DOE Engine)
  │                                                            │
  ├── 1. 读 DOE Status 确认 Busy == 0                         │
  ├── 2. 逐个 DWORD 写入 DOE Write Data Mailbox ──────────────▶│ (存入内部请求缓冲)
  ├── 3. 写 DOE Control: 设置 DOE Go = 1 ─────────────────────▶│
  │                                                            │ 4. 解析协议并处理
  │      (轮询 DOE Status 或等待 DOE 中断)                      │    (Busy = 1)
  │◀── 5. 置位 Data Object Ready = 1（可选触发中断） ──────────┤
  │                                                            │
  ├── 6. 读 DOE Read Data Mailbox（首 2 DWORD 含协议头与长度） │
  └── 7. 循环读取剩余 DWORD，直至 Data Object Ready 清零        │
```

> **异常恢复**：若发生超时或 `DOE Error = 1`，主机向 `DOE Control` 写入 `DOE Abort = 1`，等待 `DOE Busy` 清零后即可复位邮箱状态机。

---

## DOE 协议体系与安全信任链

每个 DOE 数据对象的前两个 DWORD 为标准头部，包含 **Vendor ID**（`0x0001` 代表 PCI-SIG）、**Data Object Type（Protocol ID）** 以及 **Length**（以 DWORD 计）。

### PCI-SIG 标准 DOE 协议类型

| Vendor ID | Data Object Type | 协议名称 | 核心作用 |
| :--- | :--- | :--- | :--- |
| `0x0001` | `0x00` | **DOE Discovery** | 协议发现：查询该 DOE 实例支持哪些协议（链表式遍历 `Index`） |
| `0x0001` | `0x01` | **CMA / SPDM** | 组件度量与认证（明文 SPDM 消息传输） |
| `0x0001` | `0x02` | **Secured CMA / SPDM** | 基于已建立 SPDM 安全会话的加密消息传输 |
| `0x0001` | `0x05` | **IDE_KM** | IDE 密钥管理：查询、下发、激活与切换 AES-GCM 链路密钥 |
| 其他 | 自定义 | **Vendor Specific** | 厂商自定义的带内配置或诊断协议 |

### 端到端硬件安全信任链（DOE + CMA/SPDM + IDE_KM + IDE）

在机密计算服务器中，DOE 是整个 PCIe 硬件信任链的底座：

```text
1. 【协议发现 (DOE Discovery)】
   Host 通过 DOE (Type 0x00) 查询设备："你支持哪些安全协议？"
   Device 返回支持 CMA/SPDM (0x01)、Secured SPDM (0x02) 和 IDE_KM (0x05)。
       │
       ▼
2. 【身份认证与固件度量 (CMA / SPDM)】
   Host 通过 DOE 运行 DMTF SPDM 握手：
   - 协商密码套件（Version / Algorithms）
   - 获取设备 X.509 证书链并挑战签名（Challenge-Response，防假冒硬件）
   - 获取设备固件哈希度量值（Measurements，防固件被篡改）
   - 通过 ECDH 派生会话密钥，升级为 Secured SPDM 安全通道
       │
       ▼
3. 【密钥协商与下发 (IDE_KM over Secured SPDM)】
   在加密的 SPDM 会话保护下，Host 通过 IDE_KM 协议向设备与 Root Port 下发 AES-256-GCM 密钥与 IV，
   并发送 KEY_SET_GO 激活硬件加密引擎。
       │
       ▼
4. 【链路数据保护 (IDE)】
   硬件 IDE 引擎接管后续所有 TLP 的线速加密与 96-bit MAC 完整性校验。
```

---

## 常用检查命令

在 Linux 内核（5.18+ 已原生支持 `pci_doe` 框架）中，可通过 `lspci` 和内核日志确认设备是否暴露 DOE 能力：

```bash
# 查看设备扩展配置空间中的 Data Object Exchange (DOE) 能力
lspci -vvv -s 01:00.0 | grep -A 5 "Data Object Exchange"

# 查看内核 DOE / CMA / SPDM 枚举日志
dmesg | grep -iE "doe|spdm|cma"
```

---

## 关联与引申概念

- **CMA（Component Measurement and Authentication）**：PCI-SIG 定义的组件度量与认证规范，规定了如何将 SPDM 映射到 PCIe/CXL 设备上，用于验证硬件真伪与固件完整性。
- **SPDM（Security Protocol and Data Model）**：由 DMTF 制定的通用硬件安全协议，负责证书链验证、随机数挑战、固件度量和会话密钥协商，通常运行在 PCIe DOE 之上。
- **IDE（Integrity and Data Encryption）与 IDE_KM**（[ide.md](ide.md)）：IDE 负责对 PCIe 链路上的 TLP 进行硬件级 AES-256-GCM 加密与防篡改校验，而 IDE_KM 密钥管理消息正是通过 DOE（常经 Secured SPDM 保护）下发给设备。
- **PCIe 扩展配置空间**（[../configuration/extended-capabilities.md](../configuration/extended-capabilities.md)）：DOE 位于 `0x100` 以上的扩展配置空间（Capability ID `0x002E`），无需消耗任何 BAR MMIO 资源。
