# PCIe 三层协议模型

**一句话总结**：PCIe 协议栈自上而下分为**事务层（Transaction Layer）**、**数据链路层（Data Link Layer）**和**物理层（Physical Layer）**，分别负责“读写什么业务数据”、“如何保证相邻两跳不丢不错”以及“如何把比特变成电信号送上铜线”。

> **直观类比**：PCIe 发送一次数据就像寄一封重要快递：
> - **事务层**是寄件人：决定写什么内容、贴上收件人地址和优先级标签（打包成 **TLP**），并提前确认对方仓库还有空位（**流控制**）。
> - **数据链路层**是快递网点：给每个包裹贴上追踪单号（**Sequence Number**）和防篡改封条（**LCRC**），没收到签收单（**Ack**）就从备份仓库（**Retry Buffer**）重新寄一次。
> - **物理层**是货车与高速路：把包裹编码、打乱防连零（**加扰**），拆成串行电信号在车道（**Lane**）上高速发车。

---

## 三层协议栈全景

```
┌─────────────────────────────────────────────────────────┐
│             Software Layer (操作系统 / 驱动程序)          │
└────────────────────────────┬────────────────────────────┘
                             │ 读写请求 / DMA / 中断
┌────────────────────────────▼────────────────────────────┐
│  Transaction Layer (事务层 - TL)                        │
│  • 构建/解析 TLP（Memory / IO / Config / Message / Cpl） │
│  • 基于信用 (Credit) 的流控制、排序规则、虚拟通道 (VC)   │
└────────────────────────────┬────────────────────────────┘
                             │ TLP (+ 可选 ECRC)
┌────────────────────────────▼────────────────────────────┐
│  Data Link Layer (数据链路层 - DLL)                     │
│  • 逐跳可靠传输：12-bit 序列号 + 32-bit LCRC 校验        │
│  • Ack/Nak 硬件重传机制、Retry Buffer、DLLP 控制包      │
└────────────────────────────┬────────────────────────────┘
                             │ TLP/DLLP + Framing 标记
┌────────────────────────────▼────────────────────────────┐
│  Physical Layer (物理层 - PHY)                          │
│  • 编码 (8b/10b, 128b/130b, PAM4 Flit) 与加扰 (Scramble) │
│  • 串并转换 (SerDes)、链路训练 (LTSSM)、差分信号传输     │
└────────────────────────────┬────────────────────────────┘
                             ▼
                      物理链路 (Lane)
```

---

## 1. 事务层（Transaction Layer）

事务层是 PCIe 的最高层，直接面向软件与设备逻辑，核心职责是将读写操作封装为 **TLP（Transaction Layer Packet）**。

### 核心事务类型
- **Memory Read / Write（内存读写）**：最常用的事务（如 CPU 通过 MMIO 访问设备 BAR，或设备通过 DMA 读写系统内存），支持 32 位或 64 位地址。
- **Configuration Read / Write（配置读写）**：用于系统启动枚举、读取 Vendor/Device ID 及配置 BAR 窗口。
- **I/O Read / Write**：仅为兼容老式 x86 设备保留。
- **Message（消息）**：用于电源管理、错误上报（AER）及虚拟中断。
- **Completion（完成包，Cpl / CplD）**：对需要响应的请求返回执行状态或读取到的数据。

### 流控制（Credit-Based Flow Control）
PCIe 禁止“盲发导致接收方缓冲区溢出后再丢包”。接收方在链路初始化时通过 `InitFC` DLLP 告诉发送方自己拥有多少 **Header 信用（HDR）** 和 **Data 信用（DATA）**；发送方每发一个 TLP 就扣减对应信用，待接收方处理完腾出空间后，再通过 `UpdateFC` DLLP 返还信用。流控按三类事务独立核算：
- **Posted（P）**：无需响应的事务（如 Memory Write、Message）。
- **Non-Posted（NP）**：需要等待 Completion 响应的事务（如 Memory Read、Config R/W）。
- **Completion（Cpl）**：对 NP 请求的响应包。

### 排序规则（Ordering Rules）与虚拟通道（VC）
为保证多线程/多设备并发下的数据一致性（例如驱动先写数据缓冲区，再写 Doorbell 门铃寄存器通知设备，后写的门铃绝不能先于数据到达），PCIe 规定了严格保序原则：
- **Posted Write 不能超越先前的 Posted Write**（同向写必须保序）。
- **Posted Write 必须先于后续的 Read Request 到达**（防止读到旧数据）。
- **Read Request 可以超越 Posted Write**（不同方向/非阻塞优化）；若 TLP 头部的 **Relaxed Ordering（宽松排序）** 置位，硬件可进一步放宽排序以提升吞吐。

此外，每个 TLP 携带 3 位 **TC（Traffic Class，TC0–TC7）** 标签，映射到不同的 **VC（Virtual Channel，虚拟通道）**。所有设备强制支持 `VC0`，高级设备可启用 `VC1–VC7` 为实时音视频或高优先级流量提供独立缓冲区与 QoS 保障。

---

## 2. 数据链路层（Data Link Layer）

事务层只管生成端到端的 TLP，而**数据链路层负责相邻两个端口之间的逐跳（Hop-by-Hop）零误码可靠传输**。

### Ack/Nak 硬件重传与 Retry Buffer
1. 发送方数据链路层为每个来自事务层的 TLP 分配一个 12 位 **Sequence Number（0–4095 循环）**，并计算覆盖整个 TLP 的 32 位 **LCRC（Link CRC）**，同时把该 TLP 副本暂存在 **Retry Buffer（重试缓冲区）** 中。
2. 接收方收到 TLP 后校验 LCRC 与序列号：
   - **校验通过**：向上送交事务层，并回发 **Ack DLLP**；发送方收到 Ack 后，将对应 TLP 从 Retry Buffer 中安全删除。
   - **校验失败**：丢弃损坏的 TLP，并回发 **Nak DLLP**；发送方收到 Nak（或 Replay Timer 超时）后，立即从 Retry Buffer 重传未确认的 TLP。

```
发送方 (DLL)                                 接收方 (DLL)
   │── TLP #100 (Seq=100, LCRC) ──────────────▶│ 校验 LCRC 正确
   │◀─ Ack DLLP (AckSeq=100) ─────────────────│
   │  (从 Retry Buffer 删除 #100)              │
   │── TLP #101 (Seq=101, LCRC) ──────────────▶│ 校验 LCRC 出错！
   │◀─ Nak DLLP (NakSeq=101) ─────────────────│
   │── 重传 TLP #101 ─────────────────────────▶│ 校验 LCRC 正确
   │◀─ Ack DLLP (AckSeq=101) ─────────────────│
```

### LCRC vs ECRC
- **LCRC（Link CRC）**：数据链路层强制添加，**逐跳校验**。经过 Switch 时，每进入一个端口就校验并剥离，离开下一个端口时重新生成。
- **ECRC（End-to-End CRC）**：事务层可选添加，**端到端校验**。从源设备生成直到目标设备才验证，用于捕获中间 Switch 内部转发或缓冲时可能出现的硬件故障。

---

## 3. 物理层（Physical Layer）

物理层分为**逻辑子层（Logical Block）**和**电气子层（Electrical Block）**，负责把数据变成物理世界的高速差分电信号。

### 逻辑子层：成帧、加扰与编码
- **成帧（Framing）**：在 TLP/DLLP 前后加上起始/结束标记（如 Gen1/2 的 `STP`/`SDP`/`END` 控制字符，或 Gen3+ 的 Token）。
- **加扰（Scrambling）**：用伪随机序列与数据异或，打散长串连续的 `0` 或 `1`，既降低电磁辐射（EMI），又保证接收端有足够的电平跳变来恢复时钟。
- **编码（Encoding）**：
  - **8b/10b（Gen1/Gen2）**：每 8 位数据编为 10 位符号，确保直流平衡，开销 20%（效率 80%）。
  - **128b/130b（Gen3/4/5）**：每 128 位净荷前加 2 位同步头（`10b` 表示 Data Block 数据块，`01b` 表示 Ordered Set Block 有序集块；`00b` 和 `11b` 为非法保留值），开销降至约 1.54%（效率 98.5%）。
  - **242B/256B Flit + PAM4（Gen6）**：采用固定 256 字节 Flit 封装（含 8B CRC 与 6B 前向纠错 FEC），配合 4 电平 PAM4 调制实现 64 GT/s。

### 链路训练与状态机（LTSSM）
上电后，物理层通过 **LTSSM（Link Training and Status State Machine）** 自动完成链路初始化，无需软件干预：

```
上电 ─▶ Detect (探测对端接收器) ─▶ Polling (比特/符号锁定，交换 TS1/TS2)
          ─▶ Configuration (协商 Lane 宽度与编号) ─▶ L0 (正常全速收发)
                                                        ├─▶ L0s / L1 / L2 (低功耗休眠)
                                                        └─▶ Recovery (动态调速或错误恢复)
```

### 电气子层：差分全双工 Lane
每条 **Lane** 包含一对发送差分线（`TX+/TX-`）和一对接收差分线（`RX+/RX-`），支持全双工同时收发，差分阻抗标准为 100Ω（早期卡规范或特定主板走线常控制在 85Ω–100Ω）。物理层还能在训练阶段自动纠正 PCB 布线导致的**极性反转（Polarity Inversion，+/- 接反）**和 **Lane 序反转（Lane Reversal，如 x16 倒序连接）**。

---

## 跨层发送与接收路径小结

- **发送（TX）**：应用/驱动发起写操作 → **事务层**检查信用并生成 TLP（+可选 ECRC）→ **链路层**加序列号与 LCRC 并存入 Retry Buffer → **物理层**成帧、加扰、编码并串行化发往 Lane。
- **接收（RX）**：Lane 接收差分信号 → **物理层** CDR 时钟恢复、解串、解码、解扰 → **链路层**校验 LCRC（成功回 Ack，失败回 Nak）→ **事务层**解析 TLP、更新流控信用并路由到设备寄存器或内存。

---

## 关联与引申概念

- **[数据包类型](packet-types.md)**：查看 TLP Header（3DW/4DW）、DLLP（Ack/Nak/InitFC）与 Ordered Set（TS1/TS2/SKP）的详细字段位图。
- **[速率与编码](speed-encoding.md)** & **[链路宽度](link-width.md)**：深入了解物理层如何协商速率与 Lane 数量，以及如何计算实际可用带宽。
- **[时钟架构](clock-architecture.md)**：理解物理层 CDR 时钟恢复、弹性缓冲区与 `SKP` 有序集是如何协同工作消除时钟频率差的。
