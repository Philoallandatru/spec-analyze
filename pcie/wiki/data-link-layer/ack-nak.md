# ACK/NAK 协议详解

**ACK/NAK 协议**是 PCIe 数据链路层（Data Link Layer, DLL）在**相邻两个端口之间**实现可靠传输的核心机制：发送方给每个 TLP 贴上 12 位序列号并保留副本，接收方校验无误后回复 `Ack`，发现错误则回复 `Nak` 触发硬件自动重传，整个过程对上层软件完全透明。

> **直观类比**：就像快递站之间的"扫码交接"——发件站给每个包裹编上流水号（`Seq#`）并留底（Replay Buffer）；收件站核对封条（LCRC）没破损且编号连续，就回传一张"已收到 102 号及之前所有包裹"的回执（`Ack`）；如果发现 101 号破损，就回传"只收到 100 号，请从 101 号起补发"（`Nak`）。

---

## 1. 在协议栈中的位置

```
┌─────────────────────────────────────────┐
│  Transaction Layer (事务层)              │  生成 TLP
└───────────────────┬─────────────────────┘
                    │ TLP
┌───────────────────▼─────────────────────┐
│  Data Link Layer (数据链路层)            │
│  • 发送侧：分配 12-bit Seq#，计算 LCRC，  │
│    存入 Replay Buffer，接收 Ack/Nak      │
│  • 接收侧：校验 LCRC 与 Seq# 连续性，     │
│    回发 Ack / Nak DLLP                  │
└───────────────────┬─────────────────────┘
                    │ Seq# + TLP + LCRC (或 DLLP)
┌───────────────────▼─────────────────────┐
│  Physical Layer (物理层)                 │  添加 STP/SDP 帧定界并编码传输
└─────────────────────────────────────────┘
```

---

## 2. 序列号与 Ack/Nak DLLP 格式

### 2.1 TLP 序列号（Sequence Number）

数据链路层为每个待发送的 TLP 分配一个 **12 位序列号**（范围 `0 ~ 4095`，模 4096 循环使用）：

```
物理层帧定界          数据链路层        事务层          数据链路层
┌──────────┬──────────────────┬──────────────┬──────────────┬──────────┐
│   STP    │  Seq# (12 bits)  │  TLP Header  │ Data Payload │   LCRC   │
│ (帧起始)  │  + Reserved (4b) │  (3 ~ 4 DW)  │ (0 ~ 1024 DW)│ (32-bit) │
└──────────┴──────────────────┴──────────────┴──────────────┴──────────┘
```

- **作用域**：每条 PCIe 链路独立维护自己的发送/接收序列号，穿越 Switch 时每跳重新编号。
- **回绕比较**：采用模 4096 算术比较大小（例如 `4095` 的下一个是 `0`）。

### 2.2 Ack / Nak DLLP 结构（6 字节核心 + 2 字节物理层定界）

DLLP（Data Link Layer Packet）是数据链路层内部的控制包，本身**不分配序列号、也不需要被对方 Ack**。其 6 字节核心结构如下：

```
 Byte 0             Byte 1          Byte 2               Byte 3         Byte 4-5
┌──────────────────┬───────────────┬────────────────────┬──────────────┬──────────────┐
│ Type             │ Reserved (0)  │ Rsvd(4b) │ AckNak_Seq_Num[11:0]   │   CRC-16     │
│ 0x00=Ack         │               │          │ (12 bits)              │  (16 bits)   │
│ 0x10=Nak         │               │          │                        │              │
└──────────────────┴───────────────┴────────────────────┴──────────────┴──────────────┘
```

| DLLP 类型 | Type 编码 | `AckNak_Seq_Num` 语义 | 发送方收到后的动作 |
|-----------|-----------|----------------------|------------------|
| **Ack** | `0x00` | 接收方**已成功接收**的最后一个连续 TLP 序列号 `N`（累积确认） | 从 Replay Buffer 释放所有 `Seq# ≤ N` 的 TLP，重置 Replay Timer |
| **Nak** | `0x10` | 接收方在出错前**最后正确收到**的 TLP 序列号 `N` | 从 Replay Buffer 中自 `N + 1` 起重传所有未确认的 TLP |

> **注意**：`Ack` 是**累积确认**——接收方连续收到 `100, 101, 102` 后，只需回一个 `Ack(102)` 即可确认全部 3 个包；同理，`Nak(100)` 表示 `100` 正常、`101` 及之后需要重传。

---

## 3. 正常确认、错误重传与超时恢复

### 3.1 正常传输（累积确认）

```
发送方 (TX)                                      接收方 (RX)
  │──── TLP (Seq#=100) ───────────────────────────▶│  LCRC ✓, Seq 连续
  │──── TLP (Seq#=101) ───────────────────────────▶│  LCRC ✓, Seq 连续
  │                                                │  (可合并延迟发送 Ack)
  │◀─── Ack DLLP (AckNak_Seq_Num = 101) ───────────│
  │  释放 Replay Buffer 中 Seq# ≤ 101 的 TLP       │
```

### 3.2 LCRC 失败或跳号（Nak 触发重传）

当接收方遇到以下情况时，丢弃错误 TLP 并发送 `Nak`：
1. **LCRC 校验失败**（链路传输出现比特翻转）。
2. **序列号跳号**（例如期望 `101`，却收到 `102`）。

```
发送方 (TX)                                      接收方 (RX)
  │──── TLP (Seq#=100) ───────────────────────────▶│  LCRC ✓ → 更新期望为 101
  │──── TLP (Seq#=101) ───────────────────────────▶│  LCRC ✗ → 丢弃！
  │──── TLP (Seq#=102) ───────────────────────────▶│  非期望 101 → 丢弃！
  │◀─── Nak DLLP (AckNak_Seq_Num = 100) ───────────│  报告最后正确包为 100
  │                                                │
  │  从 Replay Buffer 重传 Seq# ≥ 101 的 TLP：      │
  │──── TLP (Seq#=101) [重传] ─────────────────────▶│  LCRC ✓
  │──── TLP (Seq#=102) [重传] ─────────────────────▶│  LCRC ✓
  │◀─── Ack DLLP (AckNak_Seq_Num = 102) ───────────│
  │  释放 100~102                                  │
```

### 3.3 Ack 丢失与 Replay Timer 超时

由于 DLLP 本身没有重传机制，如果 `Ack` 或 `Nak` 在半路损坏被丢弃，发送方依靠 **Replay Timer（重传定时器）** 防止死锁：

1. 发送方每当发送未确认 TLP 时启动 `Replay Timer`。
2. 收到有效的 `Ack`（推进了确认窗口）时重置 Timer。
3. 若超时仍未收到 `Ack`：发送方主动重传 Replay Buffer 中所有未确认 TLP，并将 **Replay Number** 计数器加 1。
4. 若连续超时/重传达到阈值（规范定义 `REPLAY_NUM` 翻转，每 4 次重传触发一次计数翻转并通知物理层进入 **LTSSM Recovery** 重新训练链路）。

---

## 4. 调试与故障排查

在 Linux 中可通过 `lspci` 或 `setpci` 查看 **AER（Advanced Error Reporting，高级错误报告）** 寄存器定位链路层重传问题：

```bash
# 查看设备的 AER 错误状态（Uncorrectable / Correctable）
lspci -vvv -s 01:00.0 | grep -A 6 "Advanced Error Reporting"
```

| AER 状态位 | 寄存器类型 | 含义与常见原因 |
|------------|-----------|--------------|
| **DLP** (Data Link Protocol Error) | Uncorrectable | 严重的数据链路协议错误（如流控制/序列号异常） |
| **BadTLP** | Correctable | 收到 LCRC 错误或格式异常的 TLP（已触发 Nak 重传） |
| **BadDLLP** | Correctable | 收到 CRC-16 错误的 DLLP（已被静默丢弃） |
| **Rollover** (Replay Num Rollover) | Correctable | 重传次数连续达到 4 次，链路质量可能恶化 |
| **Timeout** (Replay Timer Timeout) | Correctable | Replay Timer 超时，常见于对端未及时回发 Ack |

**排查路径**：
- `BadTLP` / `BadDLLP` 高 → 物理信号完整性问题（线缆、插槽接触、EMI），可尝试检查金手指或降速测试。
- `Timeout` 高 → 对端设备挂起或链路时钟/唤醒异常。
- `Rollover` 伴随链路降速 → 触发了 LTSSM Recovery 重新训练。

---

## 5. 关联与引申概念

- **LCRC（Link CRC）**：数据链路层为每个 TLP 附加的 32 位校验码，用于逐跳检测传输比特错误；与事务层可选的端到端校验 **ECRC** 不同。详见 [LCRC 详解](lcrc.md) 与 [ECRC 详解](../transaction-layer/ecrc.md)。
- **Replay Buffer（重传缓冲区）**：发送方暂存已发未确认 TLP 的环形缓冲区，是 `Nak` 与超时重传得以实现的前提。详见 [重试缓冲区](retry-buffer.md)。
- **DLLP（Data Link Layer Packet）**：除 Ack/Nak 外，还承载流控制（InitFC/UpdateFC）与电源管理（PM）信令。详见 [DLLP 格式详解](dllp-format.md)。
- **LTSSM（Link Training and Status State Machine）**：物理层链路训练状态机；当 `REPLAY_NUM` 连续超限时，数据链路层会请求物理层进入 `Recovery` 状态重新训练链路。
- **AER（Advanced Error Reporting）**：PCIe 扩展配置空间中的标准化错误日志与上报机制，用于向操作系统暴露 `BadTLP`、`Rollover` 等硬件事件。
