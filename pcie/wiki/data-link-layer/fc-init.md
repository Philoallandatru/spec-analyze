# 流控制初始化 (Flow Control Initialization)

**流控制初始化（FC Initialization）** 是 PCIe 链路在物理层训练完成后、正式收发任何 TLP 之前必须完成的"缓冲区容量握手"。双方通过交换 `InitFC1` 和 `InitFC2` DLLP，互相通告各自接收缓冲区的 **Header Credit** 和 **Data Credit** 上限，从根本上杜绝接收端缓冲区溢出。

> **直观类比**：两家仓库之间开通货车专线前，必须先互相通报"我这边有 32 个卸货泊位（Header Credit）和 512 立方米货架（Data Credit）"。只有确认双方都记下了对方的容量（`InitFC1` → `InitFC2`），货车（TLP）才允许发车。

---

## 1. 信用（Credit）与流控制分类

### 1.1 两种信用单位

| 信用类型 | 含义 | 换算单位 |
|----------|------|----------|
| **Header Credit (`HdrFC`)** | 可接收的 TLP 头部数量 | `1 Credit = 1 个 TLP Header`（无论 3 DW 还是 4 DW） |
| **Data Credit (`DataFC`)** | 可接收的 TLP 数据负载容量 | `1 Credit = 4 DW（16 字节）`（向上取整） |

### 1.2 三类流量与独立信用池

为防止需要等待响应的读请求阻塞无需响应的写请求（避免协议死锁），PCIe 为每个启用的虚拟通道（**VC0 ~ VC7**）独立维护 **3 组信用池**：

| 流控制类别 | 缩写 | 包含的 TLP 类型 | 说明 |
|------------|------|----------------|------|
| **Posted** | `P` | Memory Write, Message | 发后即忘，不需要返回 Completion |
| **Non-Posted** | `NP` | Memory Read, I/O Read/Write, Config Read/Write | 必须等待对端返回 Completion |
| **Completion** | `Cpl` | Completion (Cpl), Completion with Data (CplD) | 对 Non-Posted 请求的响应包 |

每个启用的 VC（默认至少启用 `VC0`）都要独立完成上述 `P / NP / Cpl` 三类信用的初始化。

---

## 2. InitFC1 与 InitFC2 的两阶段握手

### 2.1 DLLP 帧格式

`InitFC1` 与 `InitFC2` 使用完全相同的 6 字节核心格式，仅靠 Byte 0 的高 4 位区分阶段与类别：

```
 Byte 0                  Byte 1            Byte 2               Byte 3         Byte 4-5
┌───────────────────────┬─────────────────┬────────────────────┬──────────────┬────────────┐
│ Type[7:4] | R | VC[2:0]│ HdrScale|HdrFC  │ HdrFC[3:0]|DataFC  │ DataFC[7:0]  │  CRC-16    │
│                       │                 │          |[11:8]   │              │  (16 bits) │
└───────────────────────┴─────────────────┴────────────────────┴──────────────┴────────────┘
```

- **InitFC1**（Type 高 4 位：`0100b`=P, `0101b`=NP, `0110b`=Cpl）：主动广播本地初始信用。
- **InitFC2**（Type 高 4 位：`1100b`=P, `1101b`=NP, `1110b`=Cpl）：确认已收到对端的 `InitFC1`（携带相同的信用值）。

### 2.2 状态机与时序流程

物理层 LTSSM 进入正常链路工作前，数据链路层控制状态机（DLCMSM）按如下两阶段推进：

```
物理层链路训练完成 (LinkUp = 1)
         │
         ▼
┌───────────────────────────────────────────────────────────┐
│ 阶段 1: FC_INIT1                                          │
│ • 双方持续循环发送 InitFC1-P, InitFC1-NP, InitFC1-Cpl      │
│ • 携带本地接收缓冲区提供的初始 HdrFC / DataFC               │
│ • 一旦收到对端全部三类合法的 InitFC1 → 记录对端信用上限     │
└────────────────────────┬──────────────────────────────────┘
                         │ (收到对端 P/NP/Cpl 三类 InitFC1)
                         ▼
┌───────────────────────────────────────────────────────────┐
│ 阶段 2: FC_INIT2                                          │
│ • 转为循环发送 InitFC2-P, InitFC2-NP, InitFC2-Cpl          │
│ • 向对端确认"我已收到你的初始信用"                          │
│ • 只要收到对端的任意 InitFC2（或首个合法 TLP/UpdateFC）     │
└────────────────────────┬──────────────────────────────────┘
                         │
                         ▼
         链路激活 (DL_Active = 1, 置位 DLLLA)
         允许事务层开始发送 TLP 与 UpdateFC
```

> **为什么需要 InitFC2？**  
> 如果只发 `InitFC1`，A 收到 B 的 `InitFC1` 后就开始发 TLP，但此时 B 可能还没收到 A 的 `InitFC1`（处于初始化中）。`InitFC2` 相当于"我确认收到你的容量了，你可以放心收我的包"，确保双方同步进入 `DL_Active` 状态。

---

## 3. 运行时信用追踪与 UpdateFC 机制

初始化完成后，发送方与接收方通过两个计数器实现**无丢失的环形信用追踪**（模运算比较）：

1. **发送方维护**：
   - `CREDITS_CONSUMED`（已发送 TLP 消耗的累计信用）
   - `CREDIT_LIMIT`（对端通告的累计已释放上限，初始等于 `InitFC` 值）
   - **发送条件**：仅当 `CREDIT_LIMIT - CREDITS_CONSUMED >= 本次 TLP 所需信用`（模运算）时才允许发送。
2. **接收方维护**：
   - `CREDITS_ALLOCATED`（接收端累计已分配/释放的信用总量）
   - 每当接收端事务层处理完一个 TLP 并腾出缓冲区，就将释放量累加到 `CREDITS_ALLOCATED`，并通过 `UpdateFC` DLLP 将最新累计值发给对端。

**为什么用"累计值"而不是"增量值"？**  
因为 `UpdateFC` DLLP 本身没有 Ack 重传保护。如果某次 `UpdateFC` 在物理层损坏被丢弃，下一次 `UpdateFC` 携带的依然是最新的累计上限 `CREDITS_ALLOCATED`，发送方的 `CREDIT_LIMIT` 会直接追平，不会造成信用永久丢失！

---

## 4. 吞吐量瓶颈与缓冲区权衡

流控制信用大小直接决定了链路能否跑满带宽。根据带宽延迟积（BDP）原则：

$$\text{最小所需 Data Credit (字节)} \ge \text{链路有效带宽} \times \text{往返时间 (RTT)}$$

- **信用过小**：发送方在等待 `UpdateFC` 返回期间耗尽信用，出现 **Credit Stall（信用阻塞）**，即使物理链路空闲也无法发包，吞吐量大幅下降。
- **信用过大**：占用过多片上 SRAM 缓冲资源，增加芯片面积与功耗。
- **UpdateFC 频率**：规范要求在信用消耗或超时达到阈值时及时发送 `UpdateFC`（例如消耗一半信用或超过特定微秒数未更新时触发），避免发送端饿死。

---

## 5. 关联与引申概念

- **事务层流控制（Flow Control）**：从事务层视角详解 `VC` 仲裁、信用计算与阻塞避免规则，见 [流控制机制](../transaction-layer/flow-control.md)。
- **虚拟通道（Virtual Channel, VC）**：PCIe 支持最多 8 个独立 VC（`VC0~VC7`），每个 VC 拥有独立缓冲区与流控制信用池，用于 QoS 隔离，见 [虚拟通道](../transaction-layer/virtual-channels.md)。
- **Scaled Flow Control（缩放流控制）**：PCIe 4.0/5.0 引入的扩展机制，利用 `HdrScale`/`DataScale`（2-bit 倍乘因子 `1x/4x/16x`）突破 8-bit/12-bit 信用字段上限，以适配高带宽长距离链路的大缓冲区需求。
- **DLLLA（Data Link Layer Link Active）**：PCIe 链路状态寄存器（`LnkSta`）中的标志位，仅当 `FC_INIT1` 与 `FC_INIT2` 全部完成后才置位为 `1`。
