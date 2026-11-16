# 控制器数据队列（Controller Data Queues）

控制器数据队列（CDQ, Controller Data Queue）是分配在**主机内存**中、由控制器按特定格式写入数据的队列，通过 CDQ Admin 命令创建与删除；Base 规范中定义的唯一队列类型是**用户数据迁移队列（UDMQ, User Data Migration Queue）**，用于记录目标控制器在迁移期间被修改的用户数据。[PDF pp. 206-210](../_source/pages/page-206.md)

可以把它想成**图书馆的“借阅变更日志本”**：当 A 阅览室（目标控制器）准备搬迁装修时，管理员（处理控制器）把期间被借还修改的图书清单实时写进主机内存中的日志本（UDMQ），搬迁完成后据此补齐增量差异。

## 生命周期与创建契约

```text
主机分配页对齐的主机内存（禁止使用 CMB 或 PMR）
   |-- PC=1（连续）：PRP1 直接指向连续缓冲区
   `-- PC=0（分散）：PRP1 指向 PRP List -> 多个页
        |
        v
Create CDQ (SEL=0h, QT=0h[UDMQ], CDQSIZE, CNTLID) -> 返回处理控制器内唯一的 CDQID
        |
        v
控制器持续按 QT 格式写入条目（PRP List 在生命周期内位置与值不可更改）
        |
        v
Delete CDQ (SEL=1h, CDQID) 或 Controller Reset（隐式删除所有 CDQ）-> 主机释放内存
```

[PDF pp. 207-209](../_source/pages/page-207.md)

| 字段 | 含义与约束 |
|---|---|
| `SEL` | `0h`=Create，`1h`=Delete，`C0h..FFh`=Vendor Specific |
| `QT` | 队列数据格式类型；Base 规范仅定义 `0h`=UDMQ |
| `PC` | `1`=PRP1 指向连续缓冲；`0`=PRP1 指向 PRP List |
| `PRP1` | 页对齐的主机内存缓冲或 PRP List（偏移必须为 `0`）；**不可使用 CMB/PMR** |
| `CDQSIZE` | 队列大小（单位：dword），必须是该队列类型条目大小的整数倍 |

**容量与数量上限**（来自 Identify Controller）：
- `MCMR`：单条 CDQ 命令允许的最大内存范围数；`NMCMR`：整个子系统最大 CDQ 内存范围数（超限报 `Invalid Field in Command`）。
- `MCUDMQ`：单控制器最大 UDMQ 数；`MNSUDMQ`：子系统最大 UDMQ 数（耗尽报 `Not Enough Resources`）。
- **PRP List 不可变**：在 Delete CDQ 或控制器重置之前，PRP List 的位置和内容均不得修改，否则行为未定义。[PDF pp. 207-208](../_source/pages/page-207.md)

## 用户数据迁移队列（UDMQ）与 Feature 21h

- **唯一性约束**：在同一个**处理控制器**下，针对同一个目标控制器（`CNTLID`）最多只能创建 **1 个 UDMQ**，重复创建返回 `Not Enough Resources`；删除不存在的 `CDQID` 返回 `Invalid Controller Data Queue`。[PDF pp. 209-210](../_source/pages/page-209.md)
- **`CDQID` 作用域**：仅在**处理控制器**范围内唯一，不同控制器之间可能重复。
- **UDMQ 条目格式**：由所选 I/O 命令集定义（如 NVM 命令集定义被修改的 LBA 范围）。

**队列推进与通知机制**：
- **Controller Data Queue Feature（`21h`）**：用于推进头指针（Head）并可选配置“尾槽位到点”的一次性触发器。推进头指针时，若队列为空则报告当前头，非空则报告控制器最新发布的条目；触发器必须指向有效已发布槽位，触发后 enable 位自动清零。[PDF pp. 415-416](../_source/pages/page-415.md)
- **防竞态建议**：Set Feature 成功会清除一个挂起的尾事件；由于旧触发槽位可能在新配置生效前已被写入，主机在收到 Set CQE 后应**再次检查队列**并按需重配触发器。
- **一次性 AER**：异步事件可携带 `CDQID` 报告“CDQ 尾指针到点”或“队列满”。[PDF p. 202](../_source/pages/page-202.md)

## 概念辨析

| 容易混淆 | 实际区别 |
|---|---|
| CDQ vs I/O CQ | CDQ 由控制器写入**数据日志**（如 UDMQ）；I/O CQ 存放命令**完成条目（CQE）** |
| UDMQ vs 控制器状态镜像 | UDMQ 追踪**用户数据修改**；控制器内部状态快照由 Suspend/Set Controller State 迁移 |
| CDQ Admin 命令 vs Feature `21h` | Admin 命令负责创建与删除 CDQ；Feature `21h` 负责推进 Head 指针与配置尾指针触发器 |

## 关联与引申概念

- **控制器迁移（[Controller Migration](controller-migration.md)）**：实时迁移期间利用 UDMQ 捕获目标控制器上的增量用户数据写入，确保切换后数据一致。[PDF p. 209](../_source/pages/page-209.md)
- **迁移变更追踪（[Migration Change Tracking](migration-change-tracking.md)）**：通过管理命令启停 UDMQ 日志记录；若缺队列、对同一目标重复开启或目标已挂起则会拒绝。[PDF pp. 436-437](../_source/pages/page-436.md)
- **PRP/SGL 内存模型（[Data Pointer Layouts](data-pointer-layouts.md)）**：CDQ 复用通用 PRP/PRP List 机制描述主机内存页，但额外要求“只能放主机内存且生命周期内不可变”。[PDF pp. 173-174](../_source/pages/page-173.md)
- **异步事件上报（[Asynchronous Event Reporting](asynchronous-event-reporting.md)）**：提供 CDQ 队列满或尾指针到点的一次性事件通知。
- **CMB 与 PMR（[Controller Memory Windows](controller-memory-windows.md)）**：控制器侧内存窗口，注意 CDQ 明确禁止放在 CMB/PMR 中。

## 规范依据

- [CDQ 创建限制与 CDW10/SEL（Figure 161），PDF 第 207 页](../_source/pages/page-207.md)
- [Create 字段布局（Figures 162-165），PDF 第 208 页](../_source/pages/page-208.md)
- [UDMQ 唯一性与删除行为（Figures 166-167），PDF 第 209 页](../_source/pages/page-209.md)
- [完成 CDQID 与错误状态（Figures 168-169），PDF 第 210 页](../_source/pages/page-210.md)
- [Controller Data Queue Feature 21h，PDF 第 415-416 页](../_source/pages/page-415.md)
