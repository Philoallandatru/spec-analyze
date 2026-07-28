# 命令中止语义（Command Abort Semantics）

## 一句话说明

**Abort（中止）命令** 用 SQID + CID 唯一标识一个已提交命令并请求取消；但 **Abort CQE 返回成功并不代表目标命令真的被中止了**——目标命令的 CQE 状态才是事实的唯一凭证。

## 生活化类比

把 Abort 想成在**复印店里按下"取消"键**：

- **按取消的人** = 主机，发出 Abort
- **复印机** = 控制器
- **正在复印的文件** = 目标命令
- **复印机屏幕** = Abort CQE 告诉你"我收到取消指令了"
- **文件是否真停了** = 目标命令的 CQE 才知道

复印机可能："立刻停"（immediate abort）→ 屏幕立刻显示"已取消"；"慢慢停"（deferred abort）→ 屏幕显示"已收到"，但文件还要再打几页才出结果；"已经印完"（no abort）→ 屏幕说"已收到"，但文件其实早就交付出去了。

## 工作流程

```text
主机                         控制器                          目标命令
 |                            |                                |
 |--- Abort(SQID,CID) ------->|                                |
 |                            |--- 目标命令能立即中止? --------|
 |                            |        yes: 立即停             |
 |                            |        no : 延后或不动          |
 |<- Abort CQE (含 IANP 位) --|                                |
 |                            |                                |
 |<----- 目标 CQE: Command Abort Requested (若被中止) --------|
 |                            |                                |
 关键观察点:
   IANP = 0  立即中止已发生 (在 Abort CQE 之前)
   IANP = 1  未发生立即中止 (deferred / not found / 不满足)
   目标 CQE 状态  才是"目标命令到底有没有被中止"的最终答案
```

简化说明：上图把"两个独立观察"分开——IANP 报告"Abort CQE 之前是否已发生立即中止"；目标命令的 CQE 报告"目标命令最终是否被中止"。两者可能不一致，必须都看。

## 初学者案例

**场景：主机提交了一条大块 Write，然后后悔了，怎么办？**

1. 主机提交 Write 到 I/O SQ，CID=5。
2. 主机觉得数据写错，立刻构造 Abort 命令：CDW10 = `{CID=5, SQID=I/O SQ0}`。
3. 控制器取出 Abort 命令，先在 Admin CQ 上回 Abort CQE。
4. 出现两种情况：
   - **情况 A（immediate abort）**：IANP=0，目标命令的 CQE 已经先于 Abort CQE 发出，状态为 `Command Abort Requested`。数据未真正写入介质。
   - **情况 B（deferred abort）**：IANP=1，Abort CQE 先出，目标命令仍在执行；过了一会儿目标命令的 CQE 出来，状态可能仍是 `Command Abort Requested`，但也可能已经 `Successful`（因为它没被中止就完成了）。
5. 主机只看 Abort CQE 会误判"已经中止"，必须结合目标命令的 CQE 状态才能下结论。

> 故障速查："Abort 返回成功了但数据还是被改了"——90% 是 **IANP=1 走了 deferred 路径**，目标命令已完成。预防：用 Identify 的 `ACL` 限制并发 Abort；大量取消用 Cancel 命令或删/重建 I/O SQ。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 目标识别 | 通过 `CDW10 = {CID[31:16], SQID[15:0]}` 唯一定位一个已提交命令 |
| 成功 ≠ 中止 | Abort CQE 成功只表示控制器处理了请求；目标命令可能已执行完成、根本不存在或被延后处理 |
| IANP 含义 | CQE DW0 bit0 = 0 表示立即中止已发生；= 1 表示未发生立即中止 |
| 立即中止副作用屏障 | 若执行立即中止，目标命令在 Abort CQE 之后不得再访问主机内存、不得修改任何 NVM 介质 / NVM Set / Endurance Group / 命名空间 / 控制器 / Domain / 子系统状态 |
| 延后中止语义 | IANP=1 时控制器可选择延后中止或根本中止；最终通过目标 CQE 的 `Command Abort Requested` 告知主机 |
| 不可立即中止的典型场景 | 数据传输已发起但未完成、且目标 CQE 尚未发出——控制器必须禁止立即中止 |
| 并发上限 | Identify Controller `ACL` 字段限制并发 Abort 数；超出可能返回 `Abort Command Limit Exceeded` (`3h`) |
| 大量取消策略 | 优先用 Cancel 命令（针对 I/O SQ / 单命名空间），或删除并重建 I/O SQ，避免超过 ACL |
| 命令字段 | Abort 仅使用 CDW10；其余命令专属字段保留 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| IANP=0 vs 目标 CQE `Command Abort Requested` | 前者说"在 Abort CQE 之前已发生中止"；后者说"目标命令被报告为已中止"，两者都满足才表示"干净的立即中止" |
| IANP=1 vs 中止失败 | IANP=1 仅代表"未发生立即中止"；后续仍可能 deferred 中止（目标 CQE = `Command Abort Requested`），或已完成（目标 CQE = `Successful`） |
| Abort vs Cancel | Abort 按 (SQID, CID) 定位单条命令；Cancel 按 I/O SQ（或按 SQ + NSID）一次性取消整批 |
| Abort vs Delete I/O SQ | Delete 销毁整个队列、自动终止其中所有命令；Abort 只针对单条 |
| Abort 与 Queue Level Reset | Abort 是单条语义；QLR 会复位队列中所有未完成命令，二者影响面不同 |
| 中止 vs 失败 | 中止不计入错误统计；超时 / 校验错误等是失败，要用 `Error Information` 日志查询 |

## 进阶细节

- **Abort CDW10 布局（Figure 143）**：

  ```text
  31                  16 15                    0
  +---------------------+-----------------------+
  |  target CID         |  target SQID          |
  +---------------------+-----------------------+
  ```

- **Abort CQE DW0 布局（Figure 144）**：

  ```text
  31                                     1  0
  +--------------------------------------+--+
  |  Reserved                            |IA|
  +--------------------------------------+--+
                                    NP=0  立即中止
                                    NP=1  未发生立即中止
  ```

- **立即中止副作用屏障（5.1.1.1）**：除发出"目标命令自身的 CQE"之外，立即中止后不得再有：主机内存访问、NVM 介质修改、NVM Set / Endurance Group / 命名空间 / 控制器 / Domain / NVM 子系统状态修改。
- **命令专属状态码（Figure 145）**：仅 `Abort Command Limit Exceeded (3h)` 一条；其余状态由通用状态码定义。
- **Identify Controller 字段**：`ACL`（Abort Command Limit）标识控制器允许的并发 Abort 数。主机应保证未完成 Abort 数量不超过此值，否则控制器可能直接以 `3h` 完成超额 Abort。
- **Abort CQE 与目标 CQE 的相对顺序**：
  - 立即中止 + 目标 CQE 先于 Abort CQE 发出：强制顺序
  - 立即中止 + 无后续副作用：Abrot CQE 与目标 CQE 顺序任意
  - 延后中止：Abort CQE 先；目标 CQE 在 `Command Abort Requested` 状态

## 规范依据

- [Abort 命令定义与 Figure 143 CDW10，PDF 第 194 页](../_source/pages/page-194.md)
- [立即中止副作用屏障与 Figure 144/145 CQE/状态码，PDF 第 195 页](../_source/pages/page-195.md)

## 相关阅读

- [common-io-control-commands.md](common-io-control-commands.md) - Cancel 与 Abort 的差异
- [admin-command-model.md](admin-command-model.md) - Abort 是 Admin opcode 之一
- [command-effects-and-support.md](command-effects-and-support.md) - Abort 的效果描述符
- [communication-loss-and-command-retry.md](communication-loss-and-command-retry.md) - 中止与重试决策
