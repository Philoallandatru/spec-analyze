# Keep Alive（保活机制）

**Keep Alive** 是 NVMe 的控制器级"看门狗（Watchdog）"机制：每个支持该功能的控制器独立维护一个倒计时器，用于检测主机与控制器之间是否发生通信失联；一旦超时，控制器会主动停止处理命令并进入致命状态（`CSTS.CFS=1`），避免主机无限期等待永远不会返回的完成响应。[PDF pp. 146-147](../_source/pages/page-146.md)

可以把 Keep Alive 想象成**物业保安的巡逻打卡**：控制器是岗亭里的倒计时牌，只要在规定时间内收到主机的消息（心跳命令或正常流量），保安就重置倒计时；如果倒计时归零仍无任何动静，岗亭就拉响警报宣告失联并切断连接。

## 核心术语与配置激活条件

| 术语 / 寄存器 | 含义与规则 |
|---|---|
| **`KAS`（Keep Alive Support）** | 控制器在 Identify Controller 中通告的支持能力与粒度（以 **100 ms** 为单位的整数倍）；`KAS=0` 表示不支持 Keep Alive |
| **`KATO`（Keep Alive Timeout）** | 主机通过 **Fabrics Connect** 或 **Set Features（FID `0Fh`）** 配置的超时值（毫秒）。PCIe 默认不强制（默认 0）；RDMA/TCP 默认 **120,000 ms（120 秒）** |
| **`KATT`（Keep Alive Timeout Time）** | 控制器当前实际生效的超时值。非零 `KATO` 低于下限时会被提升到适用最小值并向上取整到 `KAS` 粒度；超过上限则拒绝并返回 `Keep Alive Timeout Invalid`；可用 Get Features 查询 |

Keep Alive 定时器仅在**同时满足**以下 5 个条件时才会激活并初始化为 `KATT`（定时器激活期间，主机应为 Keep Alive 命令在 Admin Submission Queue 中预留槽位）：[PDF p. 147](../_source/pages/page-147.md)

`CC.EN=1`（控制器已启用） ∧ `CSTS.RDY=1`（已就绪） ∧ `CC.SHN=00b`（未在关闭） ∧ `CSTS.SHST=00b`（正常未关闭） ∧ `生效的 KATO ≠ 0`

## 两种工作模式与最坏延迟

由 `TBKAS`（Traffic Based Keep Alive Support）位决定控制器工作在哪种模式：[PDF pp. 147-150](../_source/pages/page-147.md)

| 对比维度 | Command Based（基于命令，`TBKAS=0`） | Traffic Based（基于流量，`TBKAS=1`） |
|---|---|---|
| **重启定时器的证据** | 成功完成 **Keep Alive Admin 命令（OPC `18h`）**，或成功完成设置非零 `KATO` 的 Set Features | 在一个 `KATT` 监测间隔内控制器获取到（fetched）**任意 Admin 或 I/O 命令** |
| **主机发送建议** | 按 `KATT/2` 周期发送 Keep Alive 命令；若提交后 `KATT` 内未收到完成则判定主机端超时 | 按 `KATT/4` 检查；有正常流量完成时可抑制发送 Keep Alive |
| **最坏超时检测延迟** | 约 `KATT` | 最长 **`2 × KATT`**（见下图） |

```text
Traffic Based 模式最坏情况 (Figure 90):
检查点 1       最后一条命令       检查点 2                 检查点 3
   |────────── KATT ──────────|────────── KATT ──────────|
                       ↑ 被获取       ✓ 检测到流量            ✗ 无流量
                                       继续新间隔              超时触发
```
注：即使控制器支持 Traffic Based，主机侧仍可保守采用 Command Based 行为；但只有当控制器也支持时，主机才可依赖 Traffic Based 模式。[PDF pp. 149-150](../_source/pages/page-149.md)

## 超时处理与恢复

当控制器检测到 Keep Alive 超时后，必须在 **`CQT`（Completion Queue Timeout / Command Quiesce Time）** 时间内完成以下清理步骤：[PDF p. 151](../_source/pages/page-151.md)

1. **记录错误**：在 Error Information Log（LID `01h`）中写入一条 `Keep Alive Timeout Expired` 记录。
2. **停止处理并置致命状态**：停止命令处理，并将 `CSTS.CFS`（Controller Fatal Status）置 `1`。
3. **消息传输额外动作（Fabrics）**：终止相关 Association 的传输连接并断开关联，主机后续可通过新的 Admin Queue Connect 重建关联。

若主机在仍有未完成命令时检测到超时，应遵循通信丢失恢复（communication-loss recovery）流程；在通过其他控制器重提依赖命令前，需确保先前命令不会与后续命令产生并发冲突。[PDF pp. 148, 150-151](../_source/pages/page-148.md)

## 规范依据

- [Keep Alive 能力、传输策略、KAS 粒度与激活条件，PDF pp. 146-147](../_source/pages/page-146.md)
- [Command Based 与 Traffic Based 行为及 Figure 90 时序，PDF pp. 147-150](../_source/pages/page-147.md)
- [超时清理流程（Error Log / CFS / Fabrics 断连），PDF p. 151](../_source/pages/page-151.md)
- [Keep Alive Timer Feature（FID 0Fh），PDF p. 403](../_source/pages/page-403.md)

## 关联与引申概念

- **控制器启用、关闭与重置**：`CC.EN`、`CSTS.RDY`、`CC.SHN` 与 `CSTS.CFS` 共同决定定时器启停与致命故障后的恢复路径。详见 [控制器启用、关闭与重置](controller-enable-shutdown-reset.md)。
- **通信丢失与命令重试**：Keep Alive 超时后主机侧的标准重连、静默等待（`CQT`）与命令重试规则。详见 [通信丢失与命令重试](communication-loss-and-command-retry.md)。
- **控制器初始化**：Fabrics Connect 阶段对 `KATO` 的初始协商。详见 [控制器初始化](controller-initialization.md)。
- **异步事件上报（AEN）**：关联终止或错误日志更新的相关通知机制。详见 [异步事件上报](asynchronous-event-reporting.md)。
