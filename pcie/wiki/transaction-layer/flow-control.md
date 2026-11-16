# 流控制机制 (Flow Control)

**流控制（Flow Control, FC）** 是 PCIe 事务层基于**信用（Credit）**的防溢出机制：接收方预先通告自己还有多少缓冲区空间（Header 与 Data 信用），发送方在发包前先扣除对应信用，信用不够就暂停发送，直到接收方处理完 TLP 并通过 `UpdateFC` DLLP 返还信用。

> **直观类比**：就像停车场入口的"剩余车位显示屏"——停车场（接收端缓冲区）共有 32 个车位，入口闸机（发送端）每放进一辆车就减 1 个车位；车位显示为 0 时绝不放行，避免场内堵死；等场内有车开走腾出空位，管理员就用对讲机（`UpdateFC` DLLP）通知入口加回可用车位数。

---

## 1. 为什么流控制必须按类别与 VC 隔离？

### 1.1 点对点（逐跳）作用域

PCIe 流控制**不是端到端**的，而是严格发生在**单条物理链路相邻两个端口之间**。穿越 Switch 时，每一段链路各自独立管理自己的信用池。

```
发送端 (Device A)                              接收端 (Device B)
┌─────────────────┐                            ┌─────────────────┐
│ 扣减本地 Credit  │ ────── 发送 TLP ─────────▶ │ 放入对应 VC 缓存 │
│ (不足则等待)     │                            │ 处理完后释放空间 │
│ 更新本地上限     │ ◀──── UpdateFC DLLP ────── │ 累加返还 Credit  │
└─────────────────┘                            └─────────────────┘
```

*(注：数据链路层因 LCRC 错误触发的重传直接从 Replay Buffer 读取，**不重复消耗**事务层信用。)*

### 1.2 三类流量 × 两种信用 = 六个独立信用池

为了避免"等待响应的读请求"堵死"无需响应的写请求"或"读响应包"（防止协议死锁），每个启用的 **虚拟通道（VC0~VC7）** 都必须独立维护 **3 类流量 × 2 种维度 = 6 个信用池**：

| 流量类别 | 缩写 | 包含的 TLP | 信用维度 |
|----------|------|-----------|----------|
| **Posted** | `P` | Memory Write, Message（发后即忘，无响应） | `PH` (Header) + `PD` (Data) |
| **Non-Posted** | `NP` | Memory Read, I/O R/W, Config R/W, AtomicOp（需等待响应） | `NPH` (Header) + `NPD` (Data) |
| **Completion** | `Cpl` | Completion (Cpl), Completion with Data (CplD) | `CplH` (Header) + `CplD` (Data) |

---

## 2. 信用单位与消耗规则

- **Header Credit（`HdrFC`）**：`1 单位 = 1 个 TLP Header`（无论 3 DW、4 DW 还是带 ECRC）。
- **Data Credit（`DataFC`）**：`1 单位 = 4 DW = 16 字节`（负载不足 16 字节时向上取整，例如 4 字节数据也消耗 1 个 Data Credit）。

| 常见 TLP 类型 | 消耗的信用 | 示例 |
|--------------|------------|------|
| **Memory Read (MRd)** | `1 NPH` | 读 256 字节：请求仅耗 `1 NPH`；对端回包耗 `1 CplH + 16 CplD` |
| **Memory Write (MWr)** | `1 PH + ⌈Len / 4DW⌉ PD` | 写 128 字节：消耗 `1 PH + 8 PD` |
| **Config / I/O Write** | `1 NPH + 1 NPD` | 写 4 字节配置寄存器：消耗 `1 NPH + 1 NPD` |
| **Completion w/ Data (CplD)** | `1 CplH + ⌈Len / 4DW⌉ CplD` | 返回 64 字节读数据：消耗 `1 CplH + 4 CplD` |
| **Message (无数据)** | `1 PH` | 如 `INTx` 中断消息 |

---

## 3. 生命周期：初始化与运行时更新

1. **初始化阶段（InitFC1 → InitFC2）**：
   - 链路训练完成后，双方通过 `InitFC1` DLLP 广播各自接收缓冲区的初始信用上限（规范要求至少能容纳 1 个最大 TLP，例如 `PD ≥ Max_Payload_Size / 16`、`CplD ≥ Max_Read_Request_Size / 16`）。
   - 收到对端 `InitFC1` 后回复 `InitFC2` 确认，随即激活链路开始收发 TLP。
2. **运行时更新（UpdateFC）**：
   - 发送端维护已消耗累计值 `CREDITS_CONSUMED` 与对端通告上限 `CREDIT_LIMIT`，仅当 `CREDIT_LIMIT - CREDITS_CONSUMED ≥ 所需信用`（模运算）时才允许发送。
   - 接收端每处理完 TLP 并腾出缓冲区，就将释放量累加到 `CREDITS_ALLOCATED`，通过 `UpdateFC` DLLP 通知发送端推进 `CREDIT_LIMIT`。
   - **防丢包设计**：`UpdateFC` 携带的是**累计信用值**而非增量，即使某个 `UpdateFC` 在物理层损坏被丢弃，后续的 `UpdateFC` 也会直接覆盖修正，不会导致信用永久泄漏。

---

## 4. 无限信用（Infinite Credit）与性能调优

- **无限信用（`0x00` 编码）**：如果接收端通告某类信用为 `0`，在 PCIe 规范中表示**无限信用**（常见于某些始终有足够内部处理能力的 Completion 或 Posted 接收端），发送端无需检查该类信用即可直接发送。
- **Credit Stall（信用阻塞）**：若对端通告的初始信用小于 `链路带宽 × RTT`（往返延迟），发送方会在等待 `UpdateFC` 返回期间耗尽信用而停顿，导致带宽跑不满。增大接收缓冲或调优 `UpdateFC` 触发阈值可消除此瓶颈。

---

## 5. 关联与引申概念

- **流控制初始化（FC Init）**：深入解析 `InitFC1` / `InitFC2` 两阶段握手状态机与 6 字节 DLLP 格式，详见 [流控制初始化](../data-link-layer/fc-init.md)。
- **虚拟通道（Virtual Channel, VC）**：PCIe 最多支持 8 个独立 VC（`VC0~VC7`），每个 VC 拥有独立流控制信用池以实现 QoS 隔离，详见 [虚拟通道](virtual-channels.md)。
- **Scaled Flow Control（缩放流控制）**：PCIe 4.0+ 引入的 `HdrScale`/`DataScale` 倍乘机制，用于支持超高速、长距离链路的大容量信用通告。
- **事务类型（Posted / Non-Posted / Completion）**：流控制分类的根本依据，详见 [PCIe 事务类型详解](transaction-types.md)。
