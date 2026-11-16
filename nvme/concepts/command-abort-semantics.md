# 命令中止语义（Command Abort Semantics）

**Abort 命令**通过 `SQID + CID` 定位一条先前已提交的命令并请求取消。核心原则是：**Abort 命令本身处理成功 ≠ 目标命令真的被中止了**——只有目标命令自己的 CQE 状态才是最终凭证。[PDF pp. 194-195](../_source/pages/page-194.md)

> **类比理解**：把 Abort 想成在复印店按下"取消"键——机器屏幕立刻提示"收到取消请求"（Abort CQE），但文件是否真停下来，得看取纸口出来的回执（目标 CQE）：可能当场停下、可能延后几页停下，也可能文件早已印完交付。

---

## 工作流程与两个独立观察点

```text
主机                         控制器                          目标命令
 |                            |                                |
 |--- Abort(SQID, CID) ------>|                                |
 |                            |--- 目标命令能否立即中止? ------|
 |                            |      能：停掉后续所有副作用    |
 |                            |      不能：延后处理或继续完成  |
 |<- Abort CQE (含 IANP 位) --|                                |
 |                            |                                |
 |<----- 目标 CQE: Command Abort Requested（若确实被中止）------|
```

这里有两个独立的观察点：**`IANP`（Abort CQE DW0 bit 0）** 回答"在 Abort CQE 之前是否已发生立即中止"；**目标命令的 CQE** 回答"目标命令最终是否被中止"。[PDF pp. 194-195](../_source/pages/page-194.md)

### 立即中止 vs 延迟中止

| 模式 | `IANP` | CQE 顺序与效果语义 |
|---|---|---|
| **立即中止（目标 CQE 先到）** | `0` | 目标 CQE 在 Abort CQE 之前发出，状态为 `Command Abort Requested` |
| **立即中止（无后续副作用）** | `0` | Abort CQE 与目标 CQE 到达顺序不限，但必须满足**立即中止效果屏障** |
| **延迟中止（Deferred Abort）** | `1` | Abort CQE 先出；目标命令可能在之后被中止（返回 `Command Abort Requested`） |
| **未中止（已完成/未找到）** | `1` | 目标命令正常完成（如 `Successful`），或未找到/早已完成 |

- **立即中止效果屏障（Effects Barrier）**：`IANP = 0` 只有在 Abort CQE 发出后，目标命令**不再访问主机内存**、且**不再修改** NVM 介质、NVM Set、Endurance Group、命名空间、控制器、Domain 或子系统状态时才合法。因此，一旦数据传输已发起但未完成且目标 CQE 尚未发出，控制器**禁止**执行立即中止。[PDF pp. 194-195](../_source/pages/page-195.md)

---

## 字段布局与并发限制

```text
Abort CDW10 (Figure 143):
31                 16 15                  0
+-------------------+---------------------+
| 目标 CID          | 目标 SQID           |
+-------------------+---------------------+

Abort CQE DW0 (Figure 144):
bit 0 = IANP (0 = 已发生立即中止；1 = 未发生立即中止)
```

- **并发上限（`ACL`）**：Identify Controller 的 `ACL`（Abort Command Limit）字段限制同时未完成的 Abort 命令数量；超出上限可能返回状态码 `3h`（`Abort Command Limit Exceeded`）。[PDF pp. 194-195](../_source/pages/page-194.md)
- **批量取消策略**：需要批量取消大量命令时，主机应优先使用 `Cancel` 命令（按 I/O SQ 或 SQ + NSID 取消），或直接删除并重建 I/O 提交队列（本质上对应 Queue Level Reset），避免用一连串 Abort 撞上 `ACL` 上限。[PDF p. 194](../_source/pages/page-194.md)

---

## 易混概念对比

| 易混概念 | 实际区别 |
|---|---|
| **`IANP = 1` vs 中止失败** | `IANP = 1` 仅表示"未发生立即中止"，目标命令仍可能走延迟中止路径被取消，或正常完成 |
| **Abort vs Cancel** | `Abort` 按 `(SQID, CID)` 取消单条命令；`Cancel` 按 I/O SQ（或 SQ + NSID）批量取消 |
| **Abort vs Delete I/O SQ** | `Delete I/O SQ` 触发队列级复位（QLR），销毁队列并终止其中所有未完成命令；`Abort` 仅针对单条命令 |
| **命令中止 vs 命令失败** | 被中止的命令不计入错误统计；超时或介质校验错误属于失败，会记入 `Error Information` 日志 |

---

## 关联与引申概念

- **SQID / CID**：提交队列 ID 与命令 ID 的二元组，唯一标识一条在途命令，也是 CQE 关联请求方的依据。[PDF pp. 160-161](../_source/pages/page-160.md)
- **ACL（Abort Command Limit）**：控制器通过 Identify Controller 上报的最大并发 Abort 上限。
- **Queue Level Reset（QLR）**：通过删除 I/O 队列一次性放弃该队列所有未完成命令的机制。[PDF p. 141](../_source/pages/page-141.md)
- **延伸阅读**：
  - [common-io-control-commands.md](common-io-control-commands.md) — `Cancel` 与 `Abort` 的差异及使用场景
  - [admin-command-model.md](admin-command-model.md) — `Abort` 所属的 Admin 命令模型
  - [command-effects-and-support.md](command-effects-and-support.md) — `Abort` 的效果描述符
  - [communication-loss-and-command-retry.md](communication-loss-and-command-retry.md) — 通信丢失后的中止与重试决策

---

## 规范依据

- [Abort 命令定义与 Figure 143 CDW10 布局，PDF p. 194](../_source/pages/page-194.md)
- [立即中止效果屏障与 Figures 144-145（CQE 与状态码），PDF p. 195](../_source/pages/page-195.md)
- [SQID / CID 匹配语义，PDF pp. 160-161](../_source/pages/page-160.md)
- [队列级复位（QLR）语义，PDF p. 141](../_source/pages/page-141.md)
