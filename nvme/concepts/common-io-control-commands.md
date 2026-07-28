# 通用 I/O 控制命令（Common I/O Control Commands）

## 一句话说明

通用 I/O 控制命令是定义在所有 I/O 命令集（I/O Command Set）之上的 I/O 队列命令族，包含 Cancel（取消）、Flush（刷写）、I/O Management Receive / Send（管理收/发）四类——它们在控制器就绪且 I/O 队列已建立后才能提交。

## 生活化类比

把通用 I/O 控制命令想成**餐厅厨房里的四种"叫停"按钮**：

- **Cancel** = 服务员紧急叫停某桌某道菜（"13 号桌的牛排别做了"）
- **Flush** = 经理喊"停，把已出锅的菜立刻装盘送走"（让已完成的写入立刻落盘）
- **I/O Management Receive** = 厨房给前台回执"我手头还能不能再接一单"
- **I/O Management Send** = 前台告诉厨房"把这几道菜换到别的锅台去做"

它们都走 I/O 队列，但分工不同：Cancel 管"撤回未做的菜"，Flush 管"确保已做的菜送到客人手里"，Receive/Send 管"调整厨房资源分配"。

## 工作流程

```text
I/O 提交队列（I/O Submission Queue）
   |
   |-- Cancel  --------------------> 已抓取命令 -> 立即/延迟中止
   |                                  （按 CID 或多命令模式 + NSID 匹配）
   |
   |-- Flush  ----------------------> 已完成写入 -> 持久化屏障
   |                                  （将易失缓存刷到非易失介质）
   |
   |-- I/O Management Receive ------> 控制器管理状态 -> 主机缓冲
   |   （如 Reclaim Unit Handle Status：每个 Placement Handle 一条描述符）
   |
   `-- I/O Management Send ---------> 控制器资源调整
       （如 Reclaim Unit Handle Update：将 Handle 重定向到空的 Reclaim Unit）
```

## 初学者案例

**场景：业务方临时取消大批待发写入**

1. 主机在 I/O SQ 上提交了 1000 个 Write 等待执行。
2. 业务切换，决定丢掉这些 Write。
3. 主机在同一 SQ 上提交 Cancel：`CID=FFFFh`（多命令模式）、`NSID=FFFFFFFFh`（全部 NSID）。
4. 控制器立刻处理 Cancel：把"匹配且已抓取"的命令在 Cancel CQE 之前中止。
5. Cancel CQE DW0 的两个字段返回结果：
   - `CMDA`：被本 Cancel 立即中止的命令数。
   - `CEDA`：匹配但只做延迟中止的命令数（饱和到 `FFFFh`）。
6. 主机仍需逐条检查目标命令的 CQE，以确认"延迟中止"是否最终生效。
7. 业务切换完成，主机重新发新的 Write。

> 速查：Cancel 只能作用于**同一 SQ** 上的命令；`SQID` 字段必须匹配。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 提交前提 | 控制器必须 Ready（`CSTS.RDY=1`）+ I/O SQ/CQ 已创建 |
| Cancel 同 SQ 限制 | Cancel 仅作用于提交它的同一 I/O SQ |
| 单命令模式 | `CID` 指定单一命令 + 一个 NSID 或全部 NSID |
| 多命令模式 | `CID=FFFFh` 选中该 SQ 上**除 Cancel 自身**外所有匹配命令 |
| NSID 全部 | 多命令模式可指定一个 NSID 或全部 NSID（`FFFFFFFFh`） |
| Cancel CQE 字段 | `CMDA`（立即中止数）+ `CEDA`（可延迟中止数，饱和 `FFFFh`） |
| 延迟中止无顺序保证 | 目标命令最终 CQE 报 `Command Abort Requested` |
| Flush 持久化屏障 | 把"在 Flush 提交之前"完成的写入刷到非易失介质 |
| Flush 可选扩展 | 控制器可顺带刷其他 NS 数据；`FB` 控制广播 NSID 行为；FDP NS 可通过 `VWCNP` 自报"无易失缓存" |
| 无缓存时 Flush 无效果 | 正常成功完成；Sanitize 期间也可正常完成 |
| I/O MR Receive | 一个具体 NS + 该 EG 启用 FDP；越界读取返回 0；按描述符采样状态，可能忽略未完成命令效果 |
| I/O MR Send | 数量受 `MAXPIDS` 与 `NRG*NRUH` 双重限制；**非原子**，已处理项可能已更新 |
| 写并发 | 并发写使用 Send 中某 Identifier 时，可能落到处理前或处理完成时引用的 Reclaim Unit |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Cancel CQE 立即数 vs 总结果 | `CMDA` 仅含"被本 Cancel 立即中止"的；延迟中止须看目标命令 CQE |
| 多命令模式 vs 单命令 | 多命令模式用 `CID=FFFFh` 选中该 SQ 上"其它所有"命令 |
| Flush vs 控制器刷缓存 | Flush 是 I/O 命令；"控制器刷缓存"是 Common Controller Features 的内部行为 |
| I/O MR vs I/O MS | MR（Receive）= 控制器 → 主机；MS（Send）= 主机 → 控制器 |
| Reclaim Unit Handle Status vs Update | Status 读 Handle→Reclaim Group 关系；Update 改 Handle 指向的 Reclaim Unit |
| FDP vs 普通 NS | I/O MR/Send 的 Reclaim Unit 操作要求 EG 启用 FDP；普通 NS 下该 Base 操作可能未定义 |

## 进阶细节

- **Cancel 命令关键字段**：
  - `CID`：单命令模式时指定目标命令；多命令模式时设为 `FFFFh`。
  - `NSID`：单/多模式下都可用 `FFFFFFFFh` 表示"全部 NSID"。
  - `SQID`：必须等于本 Cancel 提交所在的 SQ。
- **Cancel 取消时机**：
  - 处理 Cancel 时已抓取（fetched）的命令**可能**被立即中止或在 Cancel CQE 之前中止。
  - 处理过程中刚抓取到的命令**可被包含**；尚未抓取的不必被影响。
- **Flush 与命名空间能力**：
  - 通过 Identify Controller `FB` 决定广播 NSID（`FFFFFFFFh`）语义。
  - FDP NS 可独立通过 `VWCNP` 报告"无易失缓存"——这种情况下 Flush 仍可成功但无效果。
- **I/O Management Receive（Base 定义）**：
  - `Reclaim Unit Handle Status` 操作：每个"可访问的 Placement Handle/Reclaim Group 关系"返回一条由 I/O 命令集定义的描述符。
  - 状态按描述符采样，可能不反映未完成命令的效果。
  - 读取越过结构末尾部分返回 0。
- **I/O Management Send（Base 定义）**：
  - `Reclaim Unit Handle Update` 操作：把指定 Placement Identifier 引用的 Handle 重新指向一个**空的** Reclaim Unit。
  - 含用户数据的 Unit **必须被替换**；已为空的 Unit 也可被替换。
  - 失败不原子：前面的 Identifier 可能已更新、可能未更新。
  - 同一 Identifier 上的并发写可能落到处理前引用的 Unit 或处理完成时引用的 Unit。
- **命令级协调建议**：
  - Cancel 复用 [Command Abort Semantics](command-abort-semantics.md) 的立即/延迟效果屏障。
  - Flush 与 [Common Controller Features](common-controller-features.md) 控制的易失写缓存配合。
  - Reclaim Unit Handle 模型来自 [Flexible Data Placement Configurations](flexible-data-placement-configurations.md)。

## 规范依据

- [Common I/O 操作码矩阵与 Cancel 范围（Figure 555），PDF 第 482 页](../_source/pages/page-482.md)
- [Cancel 抓取命令与中止时序规则，PDF 第 483 页](../_source/pages/page-483.md)
- [Flush 持久化与命名空间能力规则，PDF 第 485 页](../_source/pages/page-485.md)
- [I/O Management Receive 与 Reclaim Unit Handle Status，PDF 第 486 页](../_source/pages/page-486.md)
- [I/O Management Send 与 Reclaim Unit Handle Update，PDF 第 487 页](../_source/pages/page-487.md)

## 相关阅读

- [command-abort-semantics.md](command-abort-semantics.md) - Cancel 与 Abort 的差异
- [controller-data-queues.md](controller-data-queues.md) - I/O MR Send 与 UDMQ 协作
- [admin-command-model.md](admin-command-model.md) - Admin 与 I/O 命令的分类
- [capacity-management-operations.md](capacity-management-operations.md) - FDP 与 EG 资源前提
