# DLLP 格式详解

**DLLP（Data Link Layer Packet，数据链路层包）** 是 PCIe 相邻两个端口的数据链路层之间交换的"内部控制信令"，专门负责可靠传输确认（Ack/Nak）、流控制信用交换（InitFC/UpdateFC）和链路电源管理（PM），不承载上层业务数据。

> **直观类比**：如果把 **TLP** 比作高速公路上运货的"集装箱卡车"，那么 **DLLP** 就是收费站和调度员之间传递的"对讲机指令"（比如"上一批车到了"、"前方车位空出 4 个"、"准备夜间节能"）——短小固定、只在相邻站点之间生效、丢了就再喊一声。

---

## 1. DLLP 在协议栈中的位置与基本结构

```
事务层 (TL) ──▶ 生成 TLP ──────────────┐
                                       ▼
数据链路层 (DLL) ──▶ 添加 Seq# + LCRC ─▶ 交由物理层加 STP 帧定界发送
                 └──▶ 生成 DLLP ───────▶ 交由物理层加 SDP 帧定界发送
```

### 通用 6 字节核心结构

在非 Flit 模式（PCIe 1.0 ~ 5.0）下，所有 DLLP 在数据链路层均为**固定 6 字节**（物理层再于首尾各加 1 字节 `SDP`/`END` 帧定界符，在线路上占 8 字节）：

```
 Byte 0             Byte 1 ~ Byte 3                 Byte 4 ~ Byte 5
┌──────────────────┬───────────────────────────────┬──────────────────┐
│  DLLP Type       │  Type-Specific Payload        │  CRC-16          │
│  (8 bits)        │  (24 bits)                    │  (16 bits)       │
└──────────────────┴───────────────────────────────┴──────────────────┘
```

### DLLP 与 TLP 核心对比

| 特性 | DLLP | TLP |
|------|------|-----|
| **生成与终结层** | 数据链路层（点对点相邻端口） | 事务层（端到端，可穿越 Switch） |
| **核心长度** | 固定 6 字节（含 2B CRC-16） | 可变（12 B Header 起，最大 4 KB+ 负载） |
| **序列号** | 无 | 有（12-bit Seq#） |
| **校验方式** | 16-bit CRC（前 4 字节计算） | 32-bit LCRC（可选另加 ECRC） |
| **可靠重传** | 不确认、不重传（CRC 错直接丢弃） | 通过 Ack/Nak + Replay Buffer 重传 |
| **流控制** | 不消耗 Flow Control Credit | 消耗对应 VC 的 Header/Data Credit |

---

## 2. DLLP 类型总览

| Type 编码 (Byte 0) | 名称 | 所属类别 | 核心作用 |
|--------------------|------|----------|----------|
| `0x00` | `Ack` | 可靠传输 | 累积确认已正确接收的 TLP 序列号 |
| `0x10` | `Nak` | 可靠传输 | 通知对端校验/顺序出错，触发重传 |
| `0x20` | `PM_Enter_L1` | 电源管理 | 请求链路进入 L1 低功耗状态 |
| `0x21` | `PM_Enter_L23` | 电源管理 | 下游向上游请求进入 L2/L3 深度休眠 |
| `0x24` | `PM_Active_State_Request_L1` | 电源管理 | ASPM 主动状态请求进入 L1 |
| `0x30` | `PM_Request_Ack` | 电源管理 | 确认对端的电源管理状态请求 |
| `0x40 + VC` (`0x40~0x47`) | `InitFC1-P` | 流控制初始化 | 通告 Posted 初始接收信用（阶段 1） |
| `0x50 + VC` (`0x50~0x57`) | `InitFC1-NP` | 流控制初始化 | 通告 Non-Posted 初始接收信用（阶段 1） |
| `0x60 + VC` (`0x60~0x67`) | `InitFC1-Cpl` | 流控制初始化 | 通告 Completion 初始接收信用（阶段 1） |
| `0xC0 + VC` (`0xC0~0xC7`) | `InitFC2-P` | 流控制初始化 | 确认已收到对端 P 信用（阶段 2） |
| `0xD0 + VC` (`0xD0~0xD7`) | `InitFC2-NP` | 流控制初始化 | 确认已收到对端 NP 信用（阶段 2） |
| `0xE0 + VC` (`0xE0~0xE7`) | `InitFC2-Cpl` | 流控制初始化 | 确认已收到对端 Cpl 信用（阶段 2） |
| `0x80 + VC` (`0x80~0x87`) | `UpdateFC-P` | 运行时流控 | 返还已释放的 Posted 缓冲区信用 |
| `0x90 + VC` (`0x90~0x97`) | `UpdateFC-NP` | 运行时流控 | 返还已释放的 Non-Posted 缓冲区信用 |
| `0xA0 + VC` (`0xA0~0xA7`) | `UpdateFC-Cpl` | 运行时流控 | 返还已释放的 Completion 缓冲区信用 |

> **注**：流控制 DLLP 的 Byte 0 低 3 位 `[2:0]` 为 **VC ID**（`0~7`，对应 VC0~VC7）。

---

## 3. 各大类 DLLP 帧格式

### 3.1 Ack / Nak DLLP

```
 Byte 0            Byte 1       Byte 2              Byte 3       Byte 4-5
┌─────────────────┬────────────┬───────────────────┬────────────┬────────────┐
│ 0x00 (Ack)      │ Reserved   │ Rsvd(4b) │ AckNak_Seq_Num[11:0]│  CRC-16    │
│ 0x10 (Nak)      │ (0x00)     │          │ (12 bits)           │  (16 bits) │
└─────────────────┴────────────┴───────────────────┴────────────┴────────────┘
```

- `AckNak_Seq_Num`：接收方**最后成功接收**的连续 TLP 序列号。收到 `Ack` 释放重传缓冲区；收到 `Nak` 则从 `AckNak_Seq_Num + 1` 开始重传。

### 3.2 流控制 DLLP（InitFC1 / InitFC2 / UpdateFC）

三者共享完全一致的 6 字节布局，通过 Byte 0 区分阶段与流量类别（Posted / Non-Posted / Completion）：

```
 Byte 0             Byte 1            Byte 2               Byte 3         Byte 4-5
┌──────────────────┬─────────────────┬────────────────────┬──────────────┬────────────┐
│ Type[7:4]|R|VCID │ HdrScale|HdrFC  │ HdrFC[3:0]|DataFC  │ DataFC[7:0]  │  CRC-16    │
│ (InitFC1/2/Upd)  │ (2b)    |(6b)   │ (4b)      |[11:8]  │ (8 bits)     │  (16 bits) │
└──────────────────┴─────────────────┴────────────────────┴──────────────┴────────────┘
```

*(注：在 Scaled Flow Control 启用前，`HdrScale` 与 `DataScale` 域段为保留/0，标准 `HdrFC` 占 8 位、`DataFC` 占 12 位以 4 DW = 16 字节为单位。)*

- **InitFC1 → InitFC2**：链路训练完成后，双方先循环发送 `InitFC1` 广播自己的缓冲区容量；收到对端的 `InitFC1` 后转入发送 `InitFC2` 确认，完成初始化后方可进入 `L0` 收发 TLP。
- **UpdateFC**：运行期间接收方每处理并腾出缓冲区空间，通过 `UpdateFC` 向对端返还信用（采用累计已分配信用值 `Alloc` 模运算机制，防止单个 `UpdateFC` 丢失导致信用永久泄漏）。

### 3.3 电源管理 DLLP（PM）

- Byte 0 为对应的 PM Type（如 `0x20` 请求进 L1），Byte 1~3 全 0，Byte 4~5 为 CRC-16。
- 通过 `PM_Enter_L1` / `PM_Request_Ack` 等握手，在链路空闲时安全关闭高速收发器以节省功耗（ASPM）。

---

## 4. 为什么 DLLP 不需要重传与确认？

如果 DLLP 也需要确认，就会陷入"确认的确认的确认……"的无限递归。PCIe 通过**自愈机制**避免对 DLLP 进行确认：
1. **Ack/Nak 丢失**：发送方的 `Replay Timer` 超时后会自动重传，接收方收到重复 TLP 会再次回发 `Ack`。
2. **UpdateFC 丢失**：`UpdateFC` 携带的是**累计信用值**而非增量，下一次周期性或事件触发的 `UpdateFC` 会自然覆盖并补齐丢失的更新。
3. **CRC-16 校验失败**：接收端静默丢弃损坏的 DLLP，并在 AER 中记录一次可纠正错误 `BadDLLP`。

---

## 5. 实战观测与调试

在 Linux 下，可通过 `lspci` 确认数据链路层是否已完成 `InitFC` 握手并处于活跃状态（`DLLLA+`）：

```bash
lspci -vvv -s 01:00.0 | grep "LnkSta:"
# 输出示例：LnkSta: Speed 16GT/s (ok), Width x4 (ok), TrErr- Train- SlotClk+ DLActive+
# DLActive+ (Data Link Layer Link Active) 表示 InitFC 已成功完成
```

---

## 6. 关联与引申概念

- **ACK/NAK 协议**：详细阐述序列号比较、累积确认与超时重传状态机，见 [ACK/NAK 协议详解](ack-nak.md)。
- **流控制初始化（FC Init）**：深入拆解 `InitFC1` / `InitFC2` 的三阶段握手与 `VC0~VC7` 信用池，见 [流控制初始化](fc-init.md) 与 [事务层流控制](../transaction-layer/flow-control.md)。
- **ASPM（Active State Power Management）**：链路层在操作系统无干预下自主进入 `L0s`/`L1` 低功耗状态的机制，依赖 `PM_*` DLLP 完成握手。
- **Flit Mode（PCIe 6.0）**：在 PCIe 6.0 PAM4 信号下，链路取消了传统的 `STP`/`SDP` 变长包定界，改用固定 256 字节的 Flit（Flow Control Unit），DLLP 也被统一封装进 Flit 负载中。
