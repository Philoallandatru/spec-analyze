# 通用 I/O 控制命令（Common I/O Control Commands）

通用 I/O 控制命令是横跨所有 I/O 命令集（I/O Command Set）的 I/O 队列控制命令，用于在**控制器就绪（`CSTS.RDY=1`）且 I/O 队列已建立**后，执行命令取消、缓存刷盘以及与管理面交换资源状态。[PDF pp. 482-486](../_source/pages/page-482.md)

可以把它们想象成餐厅厨房里的四种“控制指令”：
- **Cancel**：紧急叫停某桌（或全部）还没上菜的订单
- **Flush**：让已经出锅的菜立刻装盘落桌（刷入非易失介质）
- **I/O Management Receive**：厨房向前台汇报当前锅台/资源状态
- **I/O Management Send**：前台通知厨房调整资源分配

## 架构与命令总览

```text
I/O 提交队列（I/O Submission Queue）
   |-- Cancel --------------------> 已抓取命令 -> 立即/延迟中止（按 CID 或多命令模式）
   |-- Flush ---------------------> 已完成写入 -> 持久化屏障（易失缓存 -> 非易失介质）
   |-- I/O Management Receive ----> 控制器管理状态 -> 主机缓冲（如 RUH Status）
   `-- I/O Management Send -------> 控制器资源调整（如 RUH Update）
```

[PDF pp. 482-486](../_source/pages/page-482.md)

## 核心命令与规则

### 1. Cancel：取消已提交的 I/O 命令

- **同队列约束**：Cancel 只能作用于**提交它自身的那条 I/O SQ**，命令中编码的 `SQID` 必须与当前 SQ 匹配。[PDF pp. 482-484](../_source/pages/page-482.md)
- **两种选中模式**：
  - **单命令模式**：指定具体 `CID` + 单个 `NSID`（或 `FFFFFFFFh` 表示全部 NSID）。
  - **多命令模式**：`CID=FFFFh`，选中该 SQ 上除 Cancel 自身之外的所有匹配命令（可指定单个 `NSID` 或 `FFFFFFFFh`）。
- **抓取与中止时机**：处理 Cancel 时，在其之前已被控制器抓取（fetched）的匹配命令会被施加取消；处理过程中刚被抓取的命令**可以**一并取消，尚未抓取的不必受影响。[PDF pp. 483-484](../_source/pages/page-483.md)
- **结果确认**：目标命令可能在 Cancel CQE 返回前立即中止，也可能延迟中止（目标命令自身的 CQE 随后报告 `Command Abort Requested`）。

| Cancel CQE DW0 字段 | 含义 |
|---|---|
| `CMDA` | 被本 Cancel 立即中止的命令数 |
| `CEDA` | 匹配但只能延迟中止的命令数，饱和到 `FFFFh` |

> **注意**：Cancel CQE 本身不保证延迟中止最终一定发生。当 `CMDA=0` 或使用多命令模式时，主机仍需逐条检查目标命令的 CQE 才能确认最终结果。[PDF pp. 484-485](../_source/pages/page-483.md)

**典型用法**：业务切换需丢弃某 SQ 上所有待发写请求时，主机在同一 SQ 提交 `CID=FFFFh`、`NSID=FFFFFFFFh` 的 Cancel，先读 `CMDA`/`CEDA` 了解规模，再通过目标 CQE 确认延迟中止完成。

### 2. Flush：持久化屏障

- **核心语义**：将“在 Flush 提交之前已完成”的写入数据和元数据，从**易失写缓存（Volatile Write Cache）**刷写到**非易失介质**；控制器可顺带刷写其他命名空间的数据。[PDF p. 485](../_source/pages/page-485.md)
- **命名空间与能力控制**：
  - 广播 `NSID=FFFFFFFFh` 的行为由 Identify Controller 的 `FB` 字段决定。
  - FDP 命名空间可通过 `VWCNP` 单独报告“无易失缓存”。
- **无缓存与 Sanitize 场景**：若未启用易失写缓存，Flush 无实际动作但正常返回成功；在 Sanitize 期间 Flush 同样正常成功。[PDF p. 485](../_source/pages/page-485.md)

### 3. I/O Management Receive：读取管理状态

- **通用行为**：指定一个 Operation 和 0-based 的 dword 长度，允许只读前缀，但越过定义结构末尾的读取返回 0。[PDF pp. 486-487](../_source/pages/page-486.md)
- **Base 规范操作 `Reclaim Unit Handle Status`**：
  - 对每个可访问的 Placement Handle / Reclaim Group 关系返回一条由 I/O 命令集定义的描述符。
  - 要求指定具体命名空间，且其所属耐久度组（EG）必须已启用 FDP。
  - 状态按描述符逐条采样，可能无法反映未完成命令的最新影响。[PDF pp. 486-487](../_source/pages/page-486.md)

### 4. I/O Management Send：下发管理调整

- **Base 规范操作 `Reclaim Unit Handle Update`**：接收 0-based 的 Placement Identifier 计数，将每个引用的 Handle 重新指向一个**空的** Reclaim Unit（含用户数据的 Unit 必须替换，已为空的也可替换）。[PDF pp. 487-489](../_source/pages/page-487.md)
- **边界与并发规则**：
  - 计数同时受 `MAXPIDS` 和 `NRG * NRUH` 双重限制；每个 Identifier 必须指向有效的 Reclaim Group 和 Placement Handle。
  - **失败非原子**：出错时前面已处理的 Identifier 可能已更新也可能未更新。
  - **并发写**：同一 Identifier 上的并发写可能落到处理前引用的 Unit，也可能落到处理完成时引用的 Unit。[PDF pp. 488-489](../_source/pages/page-487.md)

## 常见误区辨析

| 容易混淆 | 实际区别 |
|---|---|
| `CMDA` vs 总取消结果 | `CMDA` 仅统计立即中止数；延迟中止是否生效必须看目标命令各自的 CQE |
| Flush 命令 vs 控制器刷缓存 | Flush 是 I/O 队列中的同步屏障命令；控制器自发刷缓存是后台特性行为 |
| I/O MR vs I/O MS | Receive（MR）是控制器 → 主机读状态；Send（MS）是主机 → 控制器改配置 |
| RUH Status vs RUH Update | Status 读取 Handle → Reclaim Group 映射关系；Update 将 Handle 重定向到空 Unit |

## 关联与引申概念

- **Abort 与命令中止语义（[Command Abort Semantics](command-abort-semantics.md)）**：Admin 队列的 Abort 命令与 I/O 队列的 Cancel 复用同一套“立即/延迟效果屏障”，前者跨队列按 SQID+CID 取消，后者仅限当前 I/O SQ。[PDF pp. 194-195, 483-485](../_source/pages/page-194.md)
- **易失写缓存（[Common Controller Features](common-controller-features.md)）**：通过 Set Features 开关控制器的 Volatile Write Cache，决定了 Flush 是否真正触发介质落盘。[PDF pp. 398-400, 485](../_source/pages/page-398.md)
- **FDP（[Flexible Data Placement Configurations](flexible-data-placement-configurations.md)）**：柔性数据放置机制，通过 Endurance Group（耐久度组）、Reclaim Group 和 Placement Handle 减少写放大，是 I/O Management Receive/Send 操作的底层模型。[PDF pp. 294-301, 486-487](../_source/pages/page-294.md)
- **控制器数据队列（[Controller Data Queues](controller-data-queues.md)）**：主机与控制器之间用于管理事件与遥测数据交互的队列机制（如 UDMQ），可与 I/O 管理命令配合使用。
- **Admin 与 I/O 命令模型（[Admin Command Model](admin-command-model.md)）**：区分提交在 Admin SQ 的管理命令与提交在 I/O SQ 的通用控制命令。

## 规范依据

- [Common I/O 操作码矩阵与 Cancel 范围（Figure 555），PDF 第 482 页](../_source/pages/page-482.md)
- [Cancel 抓取命令与中止时序规则，PDF 第 483 页](../_source/pages/page-483.md)
- [Flush 持久化与命名空间能力规则，PDF 第 485 页](../_source/pages/page-485.md)
- [I/O Management Receive 与 Reclaim Unit Handle Status，PDF 第 486 页](../_source/pages/page-486.md)
- [I/O Management Send 与 Reclaim Unit Handle Update，PDF 第 487 页](../_source/pages/page-487.md)
