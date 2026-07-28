# 控制器迁移（Controller Migration）

## 一句话说明

控制器迁移（Controller Migration）通过 Suspend（挂起）→ 分段读取源状态 → Set Controller State（分段写入目标）→ Resume（恢复）四步，把一个控制器的运行时状态（队列指针、属性、厂商数据）整体搬到另一个控制器上，期间所有用户数据修改由 UDMQ 与 Migration Change Tracking 协同记录。

## 生活化类比

把控制器迁移想成**餐厅换厨师**：

- **A 厨师（源控制器）** = 正在做菜，灶台有火（SQ/CQ 持续运转）
- **B 厨师（目标控制器）** = 备好空灶台，等接活
- **挂起（Suspend）** = A 厨师把手头最后几道菜做完，不再接新单
- **状态镜像（Get/Set Controller State）** = 把 A 厨师的"酱料配比、灶火大小、已点单进度"全部抄到 B 的脑子里
- **恢复（Resume）** = B 厨师接棒，按 A 留下的进度继续出菜
- **用户数据迁移队列（UDMQ）** = 一个小本子，专门记录换厨师期间客人临时加单/退单
- **Migration Change Tracking** = 开启/关闭小本子的开关

关键点：**整个过程要"无缝"**——客人完全感觉不到厨师换了。

## 工作流程

```text
[源控制器：运行中]
   |
   | 1. Suspend Notification（可选预告，不改状态）
   | 2. Suspend (STYPE=1h 真实挂起)
   v
[源控制器：挂起中]
   |  - 停止获取新命令
   |  - 排空已获取命令（除 AER）
   v
[源控制器：已挂起]
   |
   | 3. Get Controller State (Migration Receive, SEL=0h)
   |     控制器返回当前标准/厂商状态镜像
   |
   | 4. Set Controller State (Migration Send) 分段传输:
   |     序列号: 01 (第一段) -> 00 (中间段) -> 10 (最后一段)
   |     或:    11 (单次完整传输)
   |
   | 5. 验证 + 提交 + 重建 SQ/CQ
   v
6. Resume (Migration Send) -> 目标控制器恢复命令获取与处理
```

简化说明：若目标控制器在已挂起状态发生 Controller Level Reset，状态自动解除（不再挂起）。

## 初学者案例

**场景：集群中一台控制器要换到新硬件**

1. 运维决定把 `CNTLID=5` 控制器从旧节点迁到新节点。
2. 主机先 `Migration Send: Suspend Notification` 通知源控制器（不挂起）。
3. 主机在目标控制器侧 `Migration Send: Set Controller State` 分段写入：SEQIND=`01` 起头、`00` 中间、`10` 收尾。
4. 主机在源侧 `Migration Send: Suspend (STYPE=1h)`；源停止 fetch、排空 SQ（除 AER）。
5. 主机在源侧 `Migration Receive: Get Controller State` 读取状态镜像（分多段）。
6. 主机通过 [Migration Change Tracking](migration-change-tracking.md) 启 UDMQ 记录挂起期间用户数据修改。
7. 主机把状态镜像完整传完后，目标控制器验证并提交，重建 SQ/CQ（指针、相位、中断向量等）。
8. 主机 `Migration Send: Resume` 恢复目标控制器命令处理。
9. 主机后续按 UDMQ + Identify 拉差量，写回命名空间完成最终一致。

> 速查：分段序列中再来一个 `01` 会丢弃所有未完成的前序传输，从头开始。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 提交前提 | 控制器 Ready + I/O 队列已建立；迁移命令走 Admin 队列 |
| 真实挂起 | `STYPE=1h` 停止 fetch、排空已获取命令（AER 除外）、停止新传输层事务 |
| 排空排除 AER | AER 是"长寿命"命令，挂起期间不排空 |
| 镜像一致性 | Get Controller State 期间控制器必须保持挂起 + 无重置 + 主机不修改属性 |
| 序列号 SEQIND | `01` 首段；`00` 中间段；`10` 最后段；`11` 单次完整 |
| 序列错误 | 非 `01` 起头、最终未提交就 Resume → `Command Sequence Error` |
| 重复挂起 | 重复发相同 Suspend 不报错 |
| 目标有效性 | Set Controller State 目标必须是：已挂起、已启用、离线从控制器 |
| 重建前置 | 提供标准状态时，目标控制器上**不得**已存在 I/O SQ/CQ |
| Resume 失败 | 目标在运行 / 状态未提交 → `Controller Not Suspended` 或 `Command Sequence Error` |
| 镜像字段 | Offset 双字对齐；NUMDL 从零计；Format Index 需对应已公布的格式；Index=0 表示省略该组件 |
| 头部 48 字节 | Controller State 头部含版本、挂起属性、标准/厂商大小 |
| Reset 解除挂起 | 控制器级 Reset 自动清除 Suspend 状态 |
| `CC.EN=0→1` 在挂起期间 | 行为为实现特定（implementation-specific） |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Suspend Notification vs Suspend | Notification 只通知、不挂起；Suspend 才真正停 fetch |
| Get Controller State 一致性 vs 实时性 | 一致性要求"持续挂起 + 无重置 + 属性不变"，非"瞬时采样" |
| 标准镜像 vs 厂商镜像 | 标准镜像含 NVMe 队列状态；厂商镜像由厂商 UUID 索引定义 |
| Resume vs Reset | Resume 恢复已挂起控制器；Reset 是控制器级复位，会自动解挂起 |
| 序列号 11 vs 01+...+10 | `11` 是单命令完整传输；`01..10` 是多段传输 |
| 镜像成功提交前 vs 后 | 提交前目标仍视为"未挂起成功状态"；提交后才可 Resume |

## 进阶细节

- **状态镜像层次**（Figures 358-361）：
  - 48 字节头部：版本、挂起属性、标准/厂商数据大小。
  - NVMe Controller State（可选）：头部 + SQ 状态（每条 24 字节，按 QID 升序） + CQ 状态（每条 24 字节，按 QID 升序）。
  - Vendor Specific Data（可选）：由厂商 UUID 索引指向。
- **SQ 状态字段**：队列创建参数、活跃的 SQ Head/Tail 指针。
- **CQ 状态字段**：队列创建参数、活跃的 CQ Head/Tail 指针、Slot 0 相位、中断使能、中断向量、物理连续性。
- **传输命令字段**：
  - Offset：64 位、双字对齐。
  - 数据量：以双字数计数。
  - Format Index：标准版本索引 / 厂商 UUID 索引；两者不能同时为 0；Index=0 时对应大小为 0。
- **UDMQ 协作**（可选 `DUDMQ` 标志位）：
  - 真实挂起成功时，若 `DUDMQ=1` 则删除关联的 UDMQ。
  - 否则队列在排空后可能收到由厂商定时的"挂起标记"条目。
- **挂起期间仍可工作**：属性访问、CMB、PMR 保持功能；主机应避免修改以防状态不一致。
- **错误码**：
  - `Invalid Controller Identifier`（Set 目标不满足条件）。
  - `Controller Not Suspended`（Resume 目标在运行）。
  - `Command Sequence Error`（Resume 时状态未提交）。
  - `Invalid Field in Command`（Offset 越界/未对齐/索引未公布/索引与大小不匹配）。
  - `Invalid Controller Data Queue`、`Not Enough Resources`（按适用情况返回）。
- **与相关概念关系**：
  - [Identify Command Model](identify-command-model.md) 发布状态版本目录与厂商 UUID 目录。
  - [Controller Data Queues](controller-data-queues.md) 定义 UDMQ 基础设施。
  - [Migration Change Tracking](migration-change-tracking.md) 启停 UDMQ 日志；拒绝缺队列/重复/目标已挂起。
  - [Queue Pair](queue-pair.md) 提供队列身份、几何、指针、相位、中断属性。
  - [Controller Enable, Shutdown, and Reset](controller-enable-shutdown-reset.md) 定义可解挂起的控制器级 Reset。

## 规范依据

- [Migration Receive 封装与一致性（SEL=0h Get Controller State），PDF 第 372 页](../_source/pages/page-372.md)
- [Migration Send 操作目录与 Suspend 入口，PDF 第 375 页](../_source/pages/page-375.md)
- [挂起行为、Resume 与 Set 前置条件，PDF 第 377 页](../_source/pages/page-377.md)
- [传输序列与命令字段定义，PDF 第 378 页](../_source/pages/page-378.md)
- [Controller State 与队列状态布局，PDF 第 380 页](../_source/pages/page-380.md)
- [提交行为与命令状态码，PDF 第 383 页](../_source/pages/page-383.md)

## 相关阅读

- [migration-change-tracking.md](migration-change-tracking.md) - 配套的变更追踪指令集
- [controller-data-queues.md](controller-data-queues.md) - UDMQ 是变更数据通道
- [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) - Reset 是解挂起的前置
- [identify-command-model.md](identify-command-model.md) - 状态版本与厂商 UUID 目录来源
