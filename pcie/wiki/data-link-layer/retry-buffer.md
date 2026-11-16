# 重试缓冲区 (Retry Buffer / Replay Buffer)

**重试缓冲区（Retry Buffer，规范中称 Replay Buffer）** 是 PCIe 数据链路层在**发送端**维护的一块高速环形缓存。每一个发出去的 TLP 都必须在这里留存副本，直到收到对端发回的 `Ack` DLLP 确认安全送达后才被释放；一旦收到 `Nak` 或等待超时，发送端就从这里把未确认的 TLP 原封不动地重新发出。

> **直观类比**：就像发微信或发快递时的"已发未签收暂存箱"——包裹发出去了，但底单（TLP 副本）不能扔；只有对方签收（收到 `Ack`）才能销毁底单；如果对方说第 101 号包裹破损（收到 `Nak`）或者一直没回音（`Replay Timer` 超时），就立刻按底单顺序重新寄一遍。

---

## 1. 核心职责与工作流

```
事务层 (TL) ──▶ 传入 TLP
                   │
                   ▼
          分配 12-bit Seq#，计算 LCRC
                   │
          ┌────────┴────────┐
          ▼                 ▼
   存入 Replay Buffer    交由物理层发送
   (环形队列留存副本)       │
          ▲                 ▼
          │            等待对端反馈：
          ├─ 收到 Ack(N) ──▶ 释放 Seq# ≤ N 的所有副本，重置 Replay Timer
          ├─ 收到 Nak(N) ──▶ 暂停新 TLP，从 Seq# = N+1 起按序重传副本
          └─ Timer 超时  ──▶ 主动重传所有未确认副本，Replay Number + 1
```

- **保持严格顺序**：重传期间必须暂停发送来自事务层的新 TLP，优先按原序列号顺序重放 Replay Buffer 中的数据，避免乱序。
- **缓冲区满时的反压（Backpressure）**：如果对端长期不回 `Ack` 导致 Replay Buffer 写满，数据链路层会暂停向事务层拉取新的 TLP，直到收到 `Ack` 腾出空间。

---

## 2. 容量设计与带宽延迟积（BDP）

PCIe 规范并未规定固定的字节数，但要求 Replay Buffer **至少能容纳链路往返延迟（RTT）期间连续发送的所有 TLP**，否则发送端会频繁因缓冲区满而停顿（Buffer Stall），导致链路带宽跑不满：

$$\text{建议最小容量} \ge \text{链路有效带宽} \times \text{往返时间 (RTT)}$$

| 设备/场景 | 典型配置 | 说明 |
|-----------|----------|------|
| 低速或短距端点 | 4 KB ~ 8 KB | 满足 `4 × Max_Payload_Size` 等基础需求 |
| 高速 NVMe SSD / NIC (Gen 4/5) | 16 KB ~ 64 KB+ | 适应高带宽与多级 Switch 带来的更大 BDP |

---

## 3. 两种重传触发与异常恢复

### 3.1 Nak 触发（显式错误）
当接收端检测到 **LCRC 校验失败** 或 **TLP 序列号不连续** 时，回发 `Nak(AckNak_Seq_Num = 最后正确序列号)`。发送端收到后立即定位到对应位置，重传后续所有未确认的 TLP。

### 3.2 Replay Timer 超时（隐式防死锁）
由于 `Ack`/`Nak` DLLP 本身没有确认机制，若 `Ack` 在传输途中因 CRC-16 错误被丢弃，发送端就会一直等不到确认。此时依靠 **Replay Timer**：
- **启动/重置**：发送未确认 TLP 时启动；收到推进窗口的 `Ack` 时重置。
- **超时动作**：若在规定时间内（由链路速率、宽度及内部延迟决定，通常在数百纳秒到微秒级）未收到 `Ack`，发送端自动触发一次全量重放（Replay），并将 `REPLAY_NUM` 计数器加 1。
- **连续失败升级（Link Retraining）**：若连续重传 4 次仍未成功（`REPLAY_NUM` 翻转），说明物理链路可能已失锁或信号严重恶化，数据链路层会上报 AER 可纠正错误 `Replay Num Rollover`，并请求物理层 **LTSSM** 进入 `Recovery` 状态重新训练链路。

---

## 4. 调试与性能指标

在 Linux 下，硬件层面的重传事件会通过 **AER（Advanced Error Reporting）** 寄存器暴露给操作系统：

```bash
# 查看 AER 可纠正错误状态（关注 BadTLP / Timeout / Rollover）
lspci -vvv -s 01:00.0 | grep -A 3 "CESta:"
```

| 现象 | 可能根因 | 排查方向 |
|------|----------|----------|
| `BadTLP` 伴随 `Rollover` 升高 | 物理信号完整性差（眼图闭合、EMI、线缆过长） | 检查 PCIe 插槽/线缆，测试降低链路速率（如 Gen4 降 Gen3） |
| `Timeout` 升高但无 `BadTLP` | 对端设备响应异常或链路时钟/低功耗唤醒异常 | 检查对端固件、电源管理（ASPM）设置 |
| 吞吐量达不到理论上限 | Replay Buffer 或流控制 Credit 过小（长距 Retimer 场景） | 检查链路延迟与 Max_Payload_Size (MPS) 配置 |

---

## 5. 关联与引申概念

- **ACK/NAK 协议**：Replay Buffer 的核心调度协议，定义了序列号比较与累积确认语义，详见 [ACK/NAK 协议详解](ack-nak.md)。
- **LCRC（Link CRC）**：接收端触发 `Nak` 重传的最主要原因，详见 [LCRC 详解](lcrc.md)。
- **流控制（Flow Control）**：与 Replay Buffer 配合工作的另一套缓冲机制——**流控制防止接收端溢出，Replay Buffer 负责发送端重传备份**，二者缺一不可。详见 [流控制初始化](fc-init.md)。
- **LTSSM Recovery**：物理层链路训练状态机中的恢复状态，当 Replay 连续失败达到阈值时被唤醒以重新校准信号均衡与时钟同步。
