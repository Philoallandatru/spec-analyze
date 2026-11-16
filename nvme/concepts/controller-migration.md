# 控制器迁移（Controller Migration）

控制器迁移（Controller Migration）允许在业务不感知的情况下，将一个正在运行的 NVMe 控制器的完整运行时状态（队列指针、配置属性与厂商数据）转移到另一个目标控制器上。[PDF pp. 372-383](../_source/pages/page-372.md)

可以把它想成**餐厅换厨师**：先让 A 厨师（源控制器）把手头已接的单做完并暂停接新单（Suspend），把灶台火候、调料配比和未完成订单进度抄给 B 厨师（Get/Set Controller State），同时用小本子记下切换期间客人临时加减的菜（UDMQ），最后让 B 厨师无缝接棒（Resume）。

## 核心状态机与四步流程

```text
[源控制器：运行中 Running]
    |  1. Migration Send: Suspend Notification（可选预告，不改变状态）
    |  2. Migration Send: Suspend (STYPE=1h 真实挂起)
    v
[源控制器：挂起中 Suspending] -> 停止获取新命令，排空已获取命令（AER 除外）
    |
    v
[源控制器：已挂起 Suspended]（发生 Controller Level Reset 会自动解除挂起）
    |  3. Migration Receive: Get Controller State (SEL=0h) -> 读取一致状态镜像
    |  4. Migration Send: Set Controller State -> 分段写入目标控制器：
    |     SEQIND: 01b(首段) -> 00b(中间段, 可重复) -> 10b(末段, 可 0 dword)
    |         或: 11b(单次完整传输)
    |  5. 目标控制器验证并提交镜像 -> 重建 SQ/CQ 状态（指针、相位、中断向量）
    v
    6. Migration Send: Resume -> 目标控制器恢复命令获取与处理
```

[PDF pp. 376-378, 383](../_source/pages/page-376.md)

## 迁移命令操作对照

| 命令与操作 | 作用 | 关键前提与约束 |
|---|---|---|
| **Migration Receive: Get Controller State** | 从源控制器分段读取标准与/或厂商状态镜像 | 源须持续处于挂起状态、无重置且属性未被修改；CQE DW0 报告是否全程保持挂起 [PDF pp. 372-374](../_source/pages/page-372.md) |
| **Migration Send: Suspend Notification** | 通知即将发生真实挂起 | 仅作预告，不改变控制器状态 |
| **Migration Send: Suspend** | `STYPE=1h` 真实挂起：停 fetch、排空已获取命令（AER 除外）、停新传输事务 | 重复发送相同类型的挂起不报错 [PDF pp. 375-376](../_source/pages/page-375.md) |
| **Migration Send: Set Controller State** | 分段传输并提交状态镜像 | 目标必须是：已挂起、已启用或离线的从控制器（否则报 `Invalid Controller Identifier`） [PDF p. 377](../_source/pages/page-377.md) |
| **Migration Send: Resume** | 重启命令获取与处理 | 目标必须已挂起（否则 `Controller Not Suspended`）且状态已提交（否则 `Command Sequence Error`） |

## 挂起行为与一致性保证

- **真实挂起对 UDMQ 的影响**：若设置 `DUDMQ=1`，成功挂起时会删除关联的 UDMQ；若未设置，队列在排空后可能收到厂商定时的“挂起标记”条目。[PDF pp. 376-377](../_source/pages/page-376.md)
- **挂起期间仍可工作的功能**：属性访问、CMB、PMR 保持正常功能，但主机应避免修改它们以防状态不一致；挂起期间将 `CC.EN` 从 `0→1` 的就绪行为由实现决定。
- **分段传输规则**（`SEQIND`）：
  - 多段传输必须以 `01b` 开始，每段之间必须等待上一段完成；若途中新发 `01b`，会丢弃前序未完成传输；最后一段 `10b` 可携带 0 dword 触发验证与提交。[PDF pp. 378-380](../_source/pages/page-378.md)
  - Offset 须 64 位 dword 对齐且在范围内；Format Index 中标准版本索引与厂商 UUID 索引**不能同时为 0**（索引为 0 时对应大小必须为 0）。

## 状态镜像结构与队列重建

```text
Controller State 镜像（Figures 358-361）
├── 48 字节头部：版本信息、挂起属性、标准/厂商数据大小
├── NVMe Controller State（可选）
│   ├── 头部：SQ 数量、CQ 数量
│   ├── SQ 状态（每条 24 字节，按 QID 升序）：创建参数 + 活跃 Head/Tail 指针
│   └── CQ 状态（每条 24 字节，按 QID 升序）：创建参数 + 活跃 Head/Tail + Slot 0 Phase + 中断属性
└── Vendor Specific Data（可选，由厂商 UUID 索引定义）
```

[PDF pp. 380-382](../_source/pages/page-380.md)

- **重建前置条件**：提供标准状态时，目标控制器上**不得已存在** I/O SQ 或 CQ。
- **失败恢复**：若任一段传输失败，主机应从头重新传输整份镜像以覆盖部分状态。[PDF p. 383](../_source/pages/page-383.md)

## 常见错误码速查

| 错误码 | 触发原因 |
|---|---|
| `Invalid Controller Identifier` | Set Controller State 目标不满足“已挂起/已启用/离线从控制器”条件 |
| `Controller Not Suspended` | 对正在运行的控制器执行 Resume |
| `Command Sequence Error` | 传输未从 `01b` 开始，或状态未验证提交就执行 Resume |
| `Invalid Field in Command` | Offset 越界/未对齐、使用未公布的 Format Index 或索引与大小不匹配 |

## 关联与引申概念

- **控制器数据队列（[Controller Data Queues](controller-data-queues.md)）**：定义 UDMQ（用户数据迁移队列），用于记录迁移期间目标控制器被修改的用户数据。[PDF pp. 207-209](../_source/pages/page-207.md)
- **迁移变更追踪（[Migration Change Tracking](migration-change-tracking.md)）**：启停主机内存或命名空间的用户数据变更日志，配合挂起与空最终报告关闭脏页窗口。[PDF pp. 434-438](../_source/pages/page-434.md)
- **Identify 命令模型（[Identify Command Model](identify-command-model.md)）**：发布迁移命令所需的“支持的控制器状态格式”版本目录与厂商 UUID 目录。[PDF pp. 365-366](../_source/pages/page-365.md)
- **队列对（[Queue Pair](queue-pair.md)）**：定义提交期间重建 SQ/CQ 所需的几何结构、指针、Phase Tag 与中断属性。[PDF pp. 381-383](../_source/pages/page-381.md)
- **控制器启用与重置（[Controller Enable, Shutdown, and Reset](controller-enable-shutdown-reset.md)）**：Controller Level Reset 会自动解除挂起状态并影响 `CC.EN`。

## 规范依据

- [Migration Receive 封装与快照一致性，PDF 第 372-374 页](../_source/pages/page-372.md)
- [Migration Send 操作目录与 Suspend 行为，PDF 第 375-377 页](../_source/pages/page-375.md)
- [分段传输序列与命令字段，PDF 第 378-379 页](../_source/pages/page-378.md)
- [Controller State 与队列状态布局，PDF 第 380-382 页](../_source/pages/page-380.md)
- [提交行为与命令状态，PDF 第 383 页](../_source/pages/page-383.md)
