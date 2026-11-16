# MCTP 传输层与 PCIe VDM / SMBus 封装

**MCTP（Management Component Transport Protocol，DMTF DSP0236）** 是服务器内部 BMC 与各子系统（网卡、GPU、NVMe SSD）之间通用的带外管理“快递协议”。它把上层管理协议（如 NVMe-MI、PLDM、SPDM）打包成统一的包格式，再分别绑定到 **SMBus/I2C、I3C 或 PCIe VDM** 等物理总线上 `[DSP0236 §7]`。

可以把 MCTP 想成**标准集装箱**：
- **箱体结构**（MCTP Transport Header）全球统一，标明发件人/收件人（`EID`）以及第几箱（`SOM`/`EOM`/`PktSeq`）。
- **运输车辆**可以切换：走 **SMBus/I2C** 就像卡车走乡间小路（限重 **64 字节 MTU**，必须拆成很多小箱）；走 **PCIe VDM** 就像走高速公路（带宽高、无需额外拉管理线）。

---

## MCTP 通用包头与分片重组

每个 MCTP 数据包都携带 **4 字节 MCTP Transport Header** 与 **1 字节 Message Type** `[DSP0236 §8.1]`：

| 字段 | 长度 | 核心作用 |
| :--- | :--- | :--- |
| **`Destination EID` / `Source EID`** | 各 8-bit | **Endpoint ID**：MCTP 网络中的逻辑地址（类似 IP 地址，由 Bus Owner/BMC 动态分配） |
| **`SOM` / `EOM`** | 各 1-bit | **Start / End of Message**：单包消息 `SOM=1, EOM=1`；多包分片时首包 `SOM=1, EOM=0`，尾包 `SOM=0, EOM=1` |
| **`PktSeq`** | 2-bit | 分片序号（`0..3` 循环），接收端据此检测丢包或乱序 |
| **`TO`（Tag Owner）+ `MsgTag`** | 1+3 bit | 标识请求方（`TO=1`）与响应方（`TO=0`），用于把多条并发请求的响应对号入座 |
| **`IC` + `Message Type`** | 1+7 bit | 位于首包载荷第 1 字节；**`04h` 固定代表 NVMe-MI**（`00h` 为 MCTP Control，`05h` 为 SPDM） |

---

## 两种主流物理绑定对比：PCIe VDM vs SMBus/I3C

| 对比维度 | MCTP over PCIe VDM（DSP0238） | MCTP over SMBus / I2C（DSP0237） |
| :--- | :--- | :--- |
| **物理通道** | 复用 PCIe 差分信号线（无需额外走线） | 背板独立 2 线制 `SMBCLK` / `SMBDATA`（或高速 I3C） |
| **封装载体** | **PCIe Type 1 VDM TLP**，Vendor ID 固定为 DMTF 的 **`0x1AB4`**，VDM Code = `0000b` | SMBus Block Write，Command Code 固定为 `0Fh` |
| **基线 MTU** | 最小支持 64B，通常可协商至 256B / 512B+ | **基线严格限制为 64 字节**（含包头，大数据必须频繁分片） |
| **校验机制** | 依赖 PCIe 数据链路层 LCRC 与事务层 ECRC | 帧尾强制附带 1 字节 **`PEC`（Packet Error Code, CRC-8）** |
| **可用时机** | 仅在 PCIe 链路训练进入 **L0** 且上电后可用 | 只要 **`3.3Vaux`** 辅助供电存在即可通信（**即使主 12V 掉电或 OS 死机**） |

> **关键规则（`0x1AB4`）**：在 PCIe 协议分析仪抓包时，凡是 Vendor ID 为 `0x1AB4` 的 VDM TLP 即为 DMTF MCTP 流量；若设备收到非 `0x1AB4` 或不合法路由的 VDM，必须静默丢弃 `[DSP0238 §6.1]`。

```text
MCTP over PCIe VDM 帧结构：
[ PCIe TLP Header (Type 1 VDM, VID=0x1AB4) ] ──▶ [ MCTP Header (EID, SOM/EOM, Tag) ] ──▶ [ MsgType=04h + NVMe-MI ]

MCTP over SMBus 帧结构：
[ Slave Addr | Cmd=0Fh | ByteCount | SrcAddr ] ──▶ [ MCTP Header ] ──▶ [ MsgType=04h + NVMe-MI ] ──▶ [ PEC (CRC-8) ]
```

---

## 关联与引申概念

- **PCIe VDM（Vendor Defined Message）**：PCIe 事务层允许厂商/标准组织自定义的消息 TLP 类型，详见 [事务类型与消息 TLP](../pcie/wiki/transaction-layer/transaction-types.md)。
- **3.3Vaux 辅助供电**：为什么盘在系统关机或主 12V 断开时 BMC 还能读到 MCTP，详见 [EDSFF 架构与连接器](../pcie-form-factors/edsff-e1-e3-architecture.md)。
- **SPDM 安全认证**：同样跑在 MCTP（`Message Type 05h`）之上，用于 PCIe 设备身份认证。
