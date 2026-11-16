# 通信丢失与命令重试（Communication Loss and Command Retry）

当主机与控制器之间的通信链路中断时，已提交但未收到完成响应的命令（Outstanding Commands）处于“不知是否已执行”的危险状态。NVMe 通过**“先建立停止处理屏障，再按幂等性与影响范围分类重试”**的机制，防止过早或盲目重试导致数据损坏。[PDF pp. 691-694](../_source/pages/page-691.md)

可以把它类比成**快递柜寄件时突然断网**：你不知道柜机到底有没有收到指令，必须先等够时间让原指令彻底失效（停止屏障），再区分“查余额（无限制重试）”“发同一份快递（延迟确认不重投）”和“改收件人（状态依赖，交由人工核实）”。

## 恢复流程与停止处理屏障

```text
[通信丢失发生]
       |
       v
1. 断开传输路径：终止 Association 与所有传输连接（基于内存的传输跳过此步）
       |
       v
2. 等待控制器检测：Command 模式等 2×KATT | Traffic 模式等 3×KATT | 或传输层通知
       |
       v
3. 等待命令停止：CQT ≠ 0 时等 CQT 时长；CQT = 0 时等实现特定时长
       |
       v
[停止屏障建立完成：控制器已不可能再处理原命令]
       |
       +-------------------+-------------------+-------------------+
       |                   |                   |
  无限制重试            延迟重试           状态依赖重试
  (立即执行)        (等所有尝试停止后再上报)  (交由上层软件决策)
```

[PDF pp. 691-694](../_source/pages/page-691.md)

- **时间计算原理**：`2×KATT` / `3×KATT` 假设传输层最多延迟 1 个 KATT（Keep Alive Timeout），控制器检测超时需要 1–2 个 KATT。[PDF p. 692](../_source/pages/page-692.md)
- **CQT 默认差异**：PCIe 默认 `CQT=0`（实现特定停止时长）；Fabrics 强制非零以覆盖异步链路丢失。
- **后台操作例外**：Sanitize、Format NVM、Firmware Update 等长耗时后台操作在通信丢失后**不会自动取消**，主机需通过其他控制器路径继续跟踪进度。[PDF p. 692](../_source/pages/page-692.md)

## 错误严重度与恢复契约

| 错误类型 | 影响范围 | 恢复方法 |
|---|---|---|
| **普通命令错误** | 单个命令失败 | 检查 CQE 中的 `DNR`/`CRD` 位决定是否重试 |
| **严重队列错误** | 整个 SQ 或 CQ 损坏 | 删除并重建队列（删除 SQ 会中止所有未完成命令） |
| **控制器级严重错误** | 管理命令失败或队列删除无响应 | 执行控制器级重置（Controller Level Reset） |
| **致命错误** | `CSTS.CFS=1`，无法报告错误 | 优先控制器重置；失败且支持时才用 NVM 子系统重置（NSSR） |

[PDF p. 690](../_source/pages/page-690.md)

**两条关键不变式**：
1. **数据完整性**：发生介质错误、读取错误或内部错误时，与失败命令相关的所有已传输数据**不得**作为有效数据使用。[PDF pp. 690-691](../_source/pages/page-690.md)
2. **重置优先级与 NSSR 风险**：优先使用 Controller Reset；NSSR 会导致 PCIe 链路断开（Link Down），对固件优先错误处理（FFH）等环境不安全。

## 命令重试三分类

NVMe 定义**幂等（Idempotent）**为：在无干预命令的情况下重复提交，子系统终态与可观测结果保持相同（辅助状态如统计计数器变化不破坏幂等性）。[PDF pp. 693-695](../_source/pages/page-693.md)

| 重试类别 | 适用条件 | 命令效果位要求 | 恢复策略 | 典型示例 |
|---|---|---|---|---|
| **无限制重试** | 幂等，且仅改变辅助状态 | `CCC=0, NIC=0, NCC=0, LBCC=0` | 停止屏障建立后立即重试 | 读不变数据、查询状态 |
| **延迟重试** | 幂等，会改变用户数据/子系统状态，但不影响其他主机 | 不强制全 0 | 必须等原命令及**之前所有重试**都确认停止后，才能向上层暴露结果 | 单主机下确定 LBA 的 Write |
| **状态依赖重试** | 非幂等，**或**可能影响其他主机 | 至少一位非 0 或涉及跨主机 | **禁止驱动自动重试**，交由上层软件（FS/DB）检查状态后决策 | Create NS、Unregister Reservation、Compare and Write、多主机 Write |

**保守降级**：主机实现可将“无限制”降级为“延迟/状态依赖”，或将“延迟”降级为“状态依赖”，以安全性换取恢复效率。[PDF pp. 694-695](../_source/pages/page-694.md)

## 常见误区与反例

### 误区 1：建立停止屏障就能立刻暴露“延迟重试”的成功结果

```text
t1: 主机经控制器 1 提交 Write A → 通信丢失
t2: 主机在控制器 2 建立停止屏障并重试 Write A → 成功（值为 A）
t3: 上层软件误以为 Write A 已尘埃落定，继续提交 Write B → 值变为 B
t4: 原 Write A 在控制器 1 延迟完成 → 值退回 A（Write B 被静默覆盖！）
```

- **结论**：跨控制器路径下，必须等**原路径与所有重试**都不再可能执行后，才能向上层报告结果，以保护 Write-after-Write 顺序依赖。[PDF pp. 702-703](../_source/pages/page-702.md)

### 误区 2：重试成功 = 原命令失败；重试失败 = 原命令成功

| 场景 | 原命令实际结果 | 重试观测结果 | 真实后果 |
|---|---|---|---|
| Create Namespace | 已创建 NS1 | 重试成功，创建 NS2 | 意外多出一个命名空间 |
| Unregister Reservation | 已注销 | 报 `Reservation Conflict` 失败 | 原命令成功，重试失败 |
| Compare and Write | 融合操作已改数据 | 因比较条件变化而失败 | 数据已被修改 |

- **结论**：对于状态依赖重试，原命令结果、重试返回状态、子系统当前状态三者不可互相替代。[PDF p. 704](../_source/pages/page-704.md)

### 误区 3：已建停止屏障，多主机写入就安全

- **反例**：主机 1 建立停止屏障后延迟重试 Write A；在此窗口内主机 2 观察到旧状态并写入 Write B，主机 1 的重试会覆盖主机 2 的 Write B。[PDF p. 705](../_source/pages/page-705.md)
- **结论**：普通读写**不能**用作多主机协调机制；多主机场景必须使用 **Reservations（保留）** 或 **Compare and Write（比较并写入）** 强制串行化。

## 概念辨析

| 容易混淆 | 实际区别 |
|---|---|
| `KATT` vs `CQT` | `KATT` 是 Keep Alive 心跳超时；`CQT` 是超时后等待控制器清理命令队列的宽限时间 |
| 幂等 vs 无副作用 | 幂等允许修改数据但重复执行终态一致；无副作用仅改辅助计数器 |
| Command 模式 vs Traffic 模式 | 决定控制器检测通信丢失所需的 KATT 倍数（2× vs 3×） |

## 关联与引申概念

- **Keep Alive（[Keep Alive](keep-alive.md)）**：主控之间的心跳保活机制，提供 KATT 时钟基准并在超时时触发 `CSTS.CFS=1`。[PDF pp. 691-692](../_source/pages/page-691.md)
- **命令效果位（[Command Effects and Support](command-effects-and-support.md)）**：定义 `CCC`/`NIC`/`NCC`/`LBCC`（控制器/命名空间/逻辑块状态改变标志），是判定“无限制重试”的必要依据。[PDF pp. 693-694](../_source/pages/page-693.md)
- **控制器复位与 NSSR（[Controller Enable, Shutdown, and Reset](controller-enable-shutdown-reset.md)）**：阐述 Controller Reset 与 NVM Subsystem Reset 的优先级和适用边界。
- **命令中止与重试延迟（[Command Abort Semantics](command-abort-semantics.md)、[Completion Queue Entry and Status](completion-queue-entry-and-status.md)）**：CQE 中的 `DNR` 与 `CRD` 位提供单命令级重试建议。
- **Sanitize 生命周期（[Sanitize Operation Lifecycle](sanitize-operation-lifecycle.md)）**：可在通信丢失后继续运行并需跨路径轮询的典型后台操作。[PDF p. 692](../_source/pages/page-692.md)

## 规范依据

- [错误类型与重置优先级，PDF 第 690-691 页](../_source/pages/page-690.md)
- [通信丢失检测与控制器停止处理屏障，PDF 第 691-692 页](../_source/pages/page-691.md)
- [CQT 与 KATT 等待时长，PDF 第 692 页](../_source/pages/page-692.md)
- [幂等性定义与重试分类，PDF 第 693-695 页](../_source/pages/page-693.md)
- [错误重试的 Write-after-Write 与跨控制器风险，PDF 第 702-703 页](../_source/pages/page-702.md)
- [非幂等重试的重复创建、错误完成与跨主机覆盖，PDF 第 704-705 页](../_source/pages/page-704.md)
