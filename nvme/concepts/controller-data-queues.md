# 控制器数据队列（Controller Data Queues）

## 一句话说明

控制器数据队列（Controller Data Queue, CDQ）是主机内存中由控制器写入特定数据的队列；通过 CDQ Admin 命令创建/删除，Base 规范定义的唯一队列类型是"用户数据迁移队列"（User Data Migration Queue, UDMQ），用于记录目标控制器在迁移期间被修改的用户数据。

## 生活化类比

把控制器数据队列想成**图书馆的"借阅变更日志本"**：

- **日志本** = CDQ（一个主机内存中的循环队列）
- **图书馆员** = 控制器（往日志本里记录变更）
- **A 阅览室在装修** = 某个控制器正在被迁移
- **B 阅览室的人借书/还书** = 用户数据的修改
- **日志本里"谁借了什么"** = 队列条目（格式由 I/O 命令集定义）
- **撕掉/合上日志本** = Delete CDQ 或控制器重置

馆长（主机）需要知道"A 阅览室装修期间，有哪些书被动过"，才能在装修完后整理；这本日志本由 B 阅览室管理员（控制器）实时记录。

## 工作流程

```text
主机分配主机内存（page-aligned）
   |
   +-- 连续: PRP1 指向单一缓冲区
   `-- 分散: PRP1 指向 PRP List -> 多个页
   |
Create CDQ(SEL=0h, QT=0h[UDMQ], CDQSIZE, PC, ...)
   v
控制器 ---- 持续按 QT 写入 ----> CDQ 条目
   |
   v
Delete CDQ(CDQID) 或 控制器重置
   v
主机可释放/重新映射该段主机内存
```

简化说明：PRP List 在 CDQ 生命周期内必须**保持位置不变 + 值不变**，否则行为未定义。

## 初学者案例

**场景：在控制器迁移开始前建一个 UDMQ**

1. 主机确定目标控制器 `CNTLID=2` 即将被迁移。
2. 主机先 `nvme get-controller-id` 等确认控制器存在；Identify Controller 中读到 `MCUDMQ >= 1`。
3. 主机分配 1 MiB 主机内存（页对齐），构造 PRP1（连续缓冲）。
4. 主机提交 `Controller Data Queue` Admin 命令：
   - CDW10：`SEL=0h`（Create），`QT=0h`（UDMQ），`PC=1b`（连续 PRP）。
   - CDW11：`CDQSIZE = 1 MiB / 4 = 262144`（双字数）。
   - CDW12：包含 `CNTLID=2` 等。
5. 控制器创建 UDMQ，CQE DW0 bit 15:0 返回新 `CDQID`（在处理控制器范围内唯一）。
6. 主机后续发起 Migration Change Tracking，控制器把 CNTLID=2 上的用户数据修改写到 UDMQ。
7. 迁移完成后主机提交 `Controller Data Queue` Admin 命令 `SEL=1h, CDQID=新ID` 删除队列并释放内存。

> 速查：每个处理控制器下，同一目标控制器最多 1 个 UDMQ；超额返回 `Not Enough Resources`。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 队列类型（QT） | Base 定义 `QT=0h` = User Data Migration Queue |
| SEL 操作码 | `0h` Create；`1h` Delete；`C0h..FFh` Vendor Specific |
| 内存位置 | CDQ 内存必须位于**主机内存**（host memory），不可用 CMB 或 PMR |
| 偏移为 0 | PRP1 起始偏移为零 |
| CDQSIZE 单位 | 双字数（dword），且为队列类型条目大小的整数倍 |
| PRP List 不可变 | 在 Delete 或控制器重置前，PRP List 的位置和值都不可改；改则行为未定义 |
| 数量上限 | 受 Identify Controller 中 `MCMR`（单命令内存范围数）+ `NMCMR`（子系统级）+ `MCUDMQ` + `MNSUDMQ` 限制 |
| UDMQ 唯一性 | 同一目标控制器在同一处理控制器下最多 1 个 UDMQ |
| CDQID 范围 | 仅在"处理控制器"范围内唯一 |
| 错误码 | 越界数量上限 → `Invalid Field in Command`；队列数耗尽 → `Not Enough Resources`；删除不存在 ID → `Invalid Controller Data Queue` |
| Feature 21h | Controller Data Queue Feature：推进 head，可选单次尾槽触发；Set 成功清除待定尾事件；触发后 enable bit 自清 |
| AER One-Shot | 一次性 AER 可携带 CDQID，宣告"尾指针到达"或"满载" |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| CDQ vs I/O CQ | CDQ 存**数据**（如迁移用户数据变更）；I/O CQ 存**完成条目** |
| UDMQ vs 控制器状态 | UDMQ 记录"用户数据"修改；控制器迁移状态镜像另由 Set Controller State 走 |
| PRP1 连续 vs PRP List | 单页/连续缓冲用 `PC=1`；跨页用 `PC=0` + PRP List |
| CDQID 全局 vs 本地 | CDQID 仅在**处理控制器**范围内唯一；跨控制器可能重复 |
| Controller Data Queue Feature vs 命令 | Feature 21h 推进 head/配置触发器；CDQ Admin 命令创建/删除 |
| 一次性 AER 触发 vs 手动轮询 | One-shot AER 报"尾指针到"或"满"；平时靠 Feature 21h 主动推进 head |

## 进阶细节

- **Create 关键字段**（Figures 161-165）：
  - `SEL`（CDW10 bit 7:0）：操作选择（0h=Create，1h=Delete）。
  - `QT`（CDW10）：队列类型，Base 仅 0h = UDMQ。
  - `PC`（CDW11）：`1`=PRP1 指向连续缓冲；`0`=PRP1 指向 PRP List。
  - `CDQSIZE`（CDW11）：队列大小（双字数）。
  - `PRP1`（CDW12）：连续缓冲基地址或 PRP List 地址（页对齐、偏移为 0）。
  - `MOS`（CDW10 bit 31:16）：操作特定字段；某些操作不使用则为保留。
- **限制来源**（Identify Controller, Figure 312）：
  - `MCMR`：单条 CDQ 命令允许的最大内存范围数。
  - `NMCMR`：整个 NVM 子系统的最大 CDQ 内存范围数。
  - `MCUDMQ`：单控制器最大 UDMQ 数。
  - `MNSUDMQ`：整个 NVM 子系统最大 UDMQ 数。
- **Delete 行为**：
  - 删除一个 CDQID 后，主机可释放或重用对应主机内存。
  - 控制器重置会隐式删除所有 CDQ。
- **Controller Data Queue Feature（21h）行为**：
  - 推进 head 指针时：空队列 → 标识当前 head；非空 → 标识控制器已发布的最新条目。
  - 触发器需指定一个有效已发布条目槽位。
  - 触发器触发后 enable bit 自清；Set 成功会清除一个待定尾事件。
  - 因旧触发槽位可能在新配置完成前被发布，主机在 Set CQE 后应再次检查队列并按需重新配触发。
- **与迁移的协作**：
  - UDMQ 是 [Controller Migration](controller-migration.md) 期间追踪用户数据修改的关键设施。
  - [Migration Change Tracking](migration-change-tracking.md) 控制 UDMQ 日志的启停。
  - 缺队列/对同一目标重复开启/目标已挂起等异常都会被拒绝。

## 规范依据

- [CDQ 命令 CDW10 与 SEL 定义（Figure 161），PDF 第 207 页](../_source/pages/page-207.md)
- [Create 布局与限制（Figures 162-165），PDF 第 208 页](../_source/pages/page-208.md)
- [UDMQ 唯一性约束与删除行为（Figures 166-167），PDF 第 209 页](../_source/pages/page-209.md)
- [完成 CDQID 与错误状态（Figures 168-169），PDF 第 210 页](../_source/pages/page-210.md)
- [Controller Data Queue Feature 21h 行为，PDF 第 415 页](../_source/pages/page-415.md)

## 相关阅读

- [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) - 控制器重置隐式删除 CDQ
- [asynchronous-event-reporting.md](asynchronous-event-reporting.md) - One-Shot 尾指针 AER 通知
- [admin-command-model.md](admin-command-model.md) - CDQ Admin 命令的 opcode
- [common-io-control-commands.md](common-io-control-commands.md) - I/O MR 与 CDQ 协作
