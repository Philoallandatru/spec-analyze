# 设备认证与链路加密：SPDM / CMA / DOE / IDE

传统服务器默认“插在主板 PCIe 槽上的硬件就是可信的”，但在机密计算（Confidential Computing）与零信任数据中心中，必须防范**恶意固件篡改**与**物理总线探针（Interposer）窃听**。为此，DMTF 与 PCI-SIG 联合构建了 **DOE + SPDM/CMA + IDE** 三位一体的硬件安全防线 `[DSP0274 / PCI-SIG CMA / IDE]`。

可以把这套流程想成**重要访客进入机密实验室的三道关卡**：
1. **DOE（门口对讲机）**：PCIe 配置空间里的标准邮箱，用来传话。
2. **SPDM / CMA（查验身份证与体检）**：主机构建加密挑战，核对设备数字证书并校验固件哈希（Measurement），确认没被植入木马。
3. **IDE（全程加密通道）**：验明正身后，双方协商密钥并开启硬件 **AES-GCM**，给整条 PCIe 线缆上的每个 TLP 报文加密加签。

---

## 四大核心规范的分工

| 规范 / 机制 | 归属组织 | 核心职责 |
| :--- | :--- | :--- |
| **`DOE`（Data Object Exchange）** | PCI-SIG | PCIe 扩展能力（Extended Capability），提供标准化配置空间邮箱，用于在 OS/驱动尚未就绪前收发安全协议报文 |
| **`SPDM`（DSP0274）** | DMTF | 定义设备身份证书链读取、非对称挑战签名（Challenge）与固件度量（Measurements）以及会话密钥派生 |
| **`CMA`（Component Measurement & Auth）** | PCI-SIG | 规定 PCIe 设备如何将 **SPDM** 报文装进 **DOE**（带内）或 **MCTP**（带外 `Type 05h`）执行认证 |
| **`IDE`（Integrity & Data Encryption）** | PCI-SIG | 在 PCIe 事务层/数据链路层之间注入硬件级 **AES-GCM** 加密与防重放保护（Link IDE 或 Selective IDE） |

---

## 启动期认证与链路加密时序

```text
  主机 (Root Complex / TDISP)                         PCIe / CXL 设备
        │                                                    │
        ├── 1. 枚举发现 DOE Extended Capability ────────────▶│
        ├── 2. SPDM GET_VERSION / GET_CAPABILITIES ─────────▶│ (通过 DOE 邮箱)
        ├── 3. SPDM GET_CERTIFICATE / CHALLENGE ────────────▶│ (验证设备硬件根证书)
        ├── 4. SPDM GET_MEASUREMENTS ───────────────────────▶│ (校验设备固件/配置哈希)
        ├── 5. SPDM KEY_EXCHANGE ──▶ 派生 AES-GCM 密钥 ──────▶│
        └── 6. 启用 PCIe IDE Stream ════════════════════════▶│ (此后 TLP 全程加密防篡改)
```

```bash
# 查看 PCIe 设备是否暴露了 DOE (Data Object Exchange) 或 IDE 扩展能力
lspci -vvv -s 0000:01:00.0 | grep -iE "Data Object Exchange|IDE|CMA"
```

---

## 关联与引申概念

- **MCTP 带外传输**：BMC 如何通过 MCTP（`Message Type 05h`）在带外执行 SPDM 认证，详见 [MCTP 传输层与 PCIe VDM / SMBus 封装](../nvme-mi/mctp-transport-and-vdm.md)。
- **CXL 安全**：CXL 3.x 机密计算与内存加密同样依赖 DOE/SPDM/IDE，详见 [CXL 3.x 架构与协议复用](../host-system/cxl-architecture-overview.md)。
