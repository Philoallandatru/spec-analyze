# 命名空间管理生命周期（Namespace Management Lifecycle）

NVMe 命名空间管理（Namespace Management）将生命周期拆分为两个独立层次：**子系统级的创建与删除（Create / Delete）**负责容量分配与回收，**控制器级的挂接与解挂（Attach / Detach）**负责控制哪些控制器可以访问该命名空间。[PDF pp. 383-387](../_source/pages/page-383.md)

> **类比**：把命名空间想成**连锁酒店的可出租房型**——`Create` 是总部在系统里“上线新房型”（圈定资源但还没分配给具体门店）；`Attach / Detach` 是把房型“上架/下架”到某家分店（控制器），且上架状态断电重启后依然记住；`Delete` 则是总部直接下线该房型，所有分店自动下架并回收资源。

## 生命周期状态流转

```text
[未分配的 NSID]
       │
       │ Namespace Management: Create (NSID=0，由控制器分配返回)
       ▼
[已分配但未挂接 (allocated, inactive)]
       │                       ▲
       │ Attach (持久化)       │ Detach (持久化)
       ▼                       │
[在选定 I/O 控制器上激活 (active)]
       │
       │ Namespace Management: Delete (指定 NSID 或 FFFFFFFFh)
       ▼
[隐式从所有控制器解挂 + 资源回收]
```

- **核心原则**：`Create` **不会**自动执行 `Attach`；`Delete` **会**自动从所有控制器解挂。管理操作是非阻塞的，执行期间其他 I/O 命令可继续运行。[PDF pp. 383-386](../_source/pages/page-383.md)
- **能力约束**：支持 Namespace Management 的控制器必然支持 Namespace Attachment；在**导出的 NVM 子系统（Exported NVM Subsystem）**中禁止支持 Namespace Management。[PDF p. 385](../_source/pages/page-385.md)

## 挂接机制（Attach / Detach）

- **持久化特性**：Attach 和 Detach 使用 4 KiB 的 **Controller List（控制器列表）** 作为命令数据，其结果**持久化保存**，能穿越所有控制器级和子系统级 Reset 事件，且不受虚拟化管理操作（如将从控制器设为 Offline）影响。[PDF pp. 383-385](../_source/pages/page-383.md)
- **目标要求**：目标必须是 I/O 控制器，且已支持并启用该命名空间所用的 I/O 命令集。[PDF p. 384](../_source/pages/page-384.md)
- **遇错即停**：处理 Controller List 时遇到第一个失败条目立即停止，并在 Error Information Log 中记录失败条目的字节偏移（列表中排在前面的条目可能已成功生效）。[PDF pp. 384-385](../_source/pages/page-384.md)

| Attach/Detach 限制条件 | 返回错误状态码 |
|---|---|
| Domain 级总挂接数超过 `MAXDNA` | `Namespace Attachment Limit Exceeded` |
| 目标控制器挂接数超过 `MAXCNA` | `Namespace Attachment Limit Exceeded` |
| I/O 命令集不受支持或未启用 | `I/O Command Set Not Supported / Not Enabled` |
| 私有命名空间已挂接到其他控制器 | `Namespace Is Private` |
| 列表包含管理控制器或格式错误 | `Controller List Invalid` |
| 挂接会产生不允许的 ANA 条件 | `ANA Attach Failed` |

## 创建与删除（Create / Delete）

- **Create 操作**：请求中的 `NSID` 必须设为 `0`，由控制器选择并返回新分配的 NSID；命令携带 Command Set Identifier、512 字节命令集特定参数区及可选 Vendor Data。[PDF pp. 385-387](../_source/pages/page-385.md)
  - 常见失败原因：不支持的格式、容量不足（Error Information Log 会报告还差多少未分配字节）、NSID 资源耗尽（达 `MNAN` 上限）、不支持精简配置（Thin Provisioning）、无效的 ANA Group 或不支持的 I/O 命令集。[PDF pp. 386-387](../_source/pages/page-386.md)
- **Delete 操作**：指定具体 NSID 删除单个命名空间，或使用 `FFFFFFFFh` 删除所有命名空间（即使当前无命名空间也会成功）。[PDF p. 385](../_source/pages/page-385.md)
  - **副作用与最佳实践**：Delete 会隐式解挂所有控制器，若命名空间仍处于挂接状态还会触发 attached-namespace 变更事件（且每次成功 Delete 必生成 allocated-namespace 变更报告）。因此主机应在 Delete 前先手动执行 Detach，避免多余事件风暴。[PDF p. 385](../_source/pages/page-385.md)

## 关联与引申概念

- **Controller List（控制器列表）**：Attach/Detach 命令传入的 4 KiB 数据结构，按顺序列出目标控制器的 CNTLID。
- **Thin Provisioning（精简配置）**：创建命名空间时不立即占用全部物理存储块，而是按实际写入按需分配空间的特性。
- **相关文档**：
  - [namespace-identifiers.md](namespace-identifiers.md) — 区分分配状态、挂接状态与持久身份
  - [nvm-capacity-model.md](nvm-capacity-model.md) — Create 消耗的未分配容量模型
  - [namespace-access-models.md](namespace-access-models.md) — Attach 如何影响私有/共享访问与 ANA 规则
  - [namespace-topology-and-change-logs.md](namespace-topology-and-change-logs.md) — Attach/Detach/Delete 产生的变更通知日志

## 规范依据

- [Namespace Attachment 持久化特性与命令封装，PDF p. 383](../_source/pages/page-383.md)
- [Attachment 限制、命令集检查与首错行为，PDF p. 384](../_source/pages/page-384.md)
- [Attachment 状态码与 Namespace Management 生命周期，PDF p. 385](../_source/pages/page-385.md)
- [Create 数据结构与失败信息，PDF p. 386](../_source/pages/page-386.md)
- [创建后返回的 NSID 完成值，PDF p. 387](../_source/pages/page-387.md)
