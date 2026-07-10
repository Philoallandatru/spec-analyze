# 控制器迁移（Controller Migration）

## 概述

控制器迁移（Controller Migration）是 NVMe 规范中一项高级功能，它允许将一个正在运行的控制器状态完整地转移到另一个控制器上。这个过程通过以下四个关键步骤来保证状态的一致性和完整性：

1. **挂起（Suspension）**：暂停控制器的命令处理
2. **分段状态传输（Segmented State Transfer）**：逐段读取和传输控制器状态
3. **队列重建（Queue Reconstruction）**：在目标控制器上重建队列结构
4. **恢复（Resumption）**：恢复控制器的正常运行

整个迁移过程由两组命令协调完成：

- **Migration Receive**：从源控制器读取状态镜像
- **Migration Send**：向目标控制器发送 Suspend（挂起）、Resume（恢复）和 Set Controller State（设置控制器状态）操作

[PDF pp. 372-383](../_source/pages/page-372.md)

---

## 核心状态机模型

为了更好地理解控制器迁移过程，我们先看一个描述性的状态转换模型：

```text
[运行中 running]
    |
    | 1. Suspend Notification（可选的准备阶段，不改变状态）
    | 2. Suspend 命令
    v
[挂起中 suspending]
    | - 停止获取新命令（stop fetch）
    | - 排空已获取的命令（drain fetched commands，AER 除外）
    v
[已挂起 suspended]
    |
    | 3. Get Controller State --> [获得稳定的状态镜像]
    |
    | 4. Set Controller State 分段传输：
    |    - 序列号：01（第一段）-> 00（中间段）-> 10（最后一段）
    |    - 或：11（单次完整传输）
    |
    | 5. 验证 + 提交 + 重建 SQ/CQ 状态
    |
    v
    6. Resume 命令 --> [恢复获取和处理命令]
```

**重要说明**：
- 在已挂起状态下，如果发生控制器级重置（Controller Level Reset），也会自动解除挂起状态
- 这个状态机由 Suspend、Resume、分段传输和提交规则共同定义

[PDF pp. 376-378](../_source/pages/page-376.md) | [PDF p. 383](../_source/pages/page-383.md)

---

## 迁移命令详解

### 命令功能对照表

| 命令操作 | 功能说明 | 关键要求与结果 |
|---------|---------|---------------|
| **Migration Receive: Get Controller State** | 从源控制器读取标准状态和/或厂商特定状态的某一段数据 | 控制器必须处于挂起状态，加上重置/属性稳定性，以确保镜像内部一致 |
| **Migration Send: Suspend Notification** | 通知即将发生真实挂起 | 本身不会挂起目标控制器 |
| **Migration Send: Suspend** | 停止获取新命令，排空已获取的命令（AER 除外） | 重复执行相同类型的挂起不会报错 |
| **Migration Send: Set Controller State** | 传输并提交状态镜像 | 目标必须是：已挂起、已启用或离线的从控制器 |
| **Migration Send: Resume** | 重新启动命令获取和处理 | 目标必须已挂起，且接收的状态必须已验证和提交 |

这些命令的详细定义和操作选择器（selector）说明见规范 PDF pp. 375-377；状态传输的前置条件和恢复序列见 pp. 377-383。

[PDF pp. 375-383](../_source/pages/page-375.md)

---

## 状态读取的一致性保证

### Migration Receive 快照机制

Migration Receive 命令当前定义 `SEL=0h` 用于 Get Controller State（获取控制器状态）操作。该操作通过以下参数控制读取行为：

**参数要求**：

- **偏移量（Offset）**：64 位字节偏移，用于选择返回的数据切片
  - 必须双字（dword）对齐
  - 必须在可用数据范围内
- **数据长度（NUMDL）**：从零开始计数的双字数量
- **格式索引（Format Index）**：
  - 非零的标准版本索引和厂商 UUID 索引必须对应 "Supported Controller State Formats"（支持的控制器状态格式）中公布的条目
  - 使用索引 0 表示省略该组件，其返回大小为 0

### 一致性保证机制

只要满足以下条件，返回的状态镜像就能保持内部一致性：

1. **持续挂起**：目标控制器在整个 Get Controller State 操作期间保持挂起状态
2. **无重置**：期间没有发生控制器级重置
3. **属性稳定**：主机在读取期间不修改目标控制器的属性

**特别说明**：完成队列的双字 0（Completion Dword 0）会报告控制器在整个操作期间是否持续保持挂起状态，而不仅仅是采样最终状态。这确保了数据的完整性。

[PDF pp. 372-374](../_source/pages/page-372.md) | [PDF pp. 373-374](../_source/pages/page-373.md)

---

## 控制器挂起机制详解

### 真实挂起模式（Real Suspension，`STYPE=1h`）

当执行真实挂起时，控制器会按以下顺序操作：

1. **停止获取命令**：不再从提交队列获取新命令
2. **完成已获取的命令**：处理完所有已获取的命令（未完成的异步事件请求 AER 除外）
3. **停止传输事务**：停止发起新的传输层事务

**用户数据迁移队列处理**：

- 如果设置了可选的 `DUDMQ` 标志位，成功挂起时会删除关联的用户数据迁移队列（User Data Migration Queue）
- 如果未设置该标志，队列可能会在排空完成后稍后收到一个由厂商定时的挂起标记

### 挂起期间的行为规范

**正常工作的功能**：

- **属性访问（Property Access）**：按常规方式继续工作
- **CMB（Controller Memory Buffer，控制器内存缓冲区）**：保持正常功能
- **PMR（Persistent Memory Region，持久化内存区域）**：保持正常功能

**特殊行为**：

- 将 `CC.EN`（Controller Configuration Enable，控制器配置启用）从 0 设为 1 时的就绪行为是实现特定的（implementation-specific）

**主机端建议**：

- 尽管挂起期间这些功能仍然可用，主机应避免修改属性、CMB 或 PMR，以防止状态不一致

[PDF p. 376](../_source/pages/page-376.md) | [PDF p. 377](../_source/pages/page-377.md)

---

## 分段状态传输机制

### 传输模式

控制器状态可以通过两种方式传输：

```text
【单次完整传输】
    使用序列索引 11：一次性传输完整镜像
    ↓
    验证并提交

【多段传输】
    01（第一段）-> 00（中间段，可重复）-> 10（最后一段）
     ↑          每段之间必须等待完成              ↓
     |                                      验证并提交
     └── 新的 01 会丢弃任何未完成的前序传输
```

### 序列索引（SEQIND）规则详解

| 序列索引 | 含义 | 说明 |
|---------|------|------|
| `01b` | 第一段 | 开始新的传输序列，丢弃任何之前未完成的序列 |
| `00b` | 中间段 | 每段之间必须等待上一段完成 |
| `10b` | 最后一段 | 可以携带 0 个双字，触发验证和提交 |
| `11b` | 完整单次传输 | 在一条命令中携带完整状态 |

**命令序列错误（Command Sequence Error）的情况**：

- 传输序列从非 `01b` 开始
- 在最终状态未验证和提交之前执行 Resume 操作

### 数据格式要求

**Set Controller State 命令参数**：

- **偏移量**：64 位，双字对齐
- **数据量**：以双字为单位计数
- **格式选择**：
  - 标准版本索引（standard-version index）
  - 厂商 UUID 索引（vendor-UUID index）
  - 两个索引不能同时为 0
  - 索引为 0 时，对应的大小必须为 0

[PDF pp. 378-379](../_source/pages/page-378.md) | [PDF p. 377](../_source/pages/page-377.md) | [PDF pp. 379-380](../_source/pages/page-379.md) | [PDF pp. 382-383](../_source/pages/page-382.md)

---

## 状态镜像结构与队列重建

### 控制器状态镜像的层次结构

```text
Controller State（控制器状态）
├── 48 字节头部
│   ├── 版本信息
│   ├── 挂起属性
│   └── 标准/厂商数据大小
│
├── NVMe Controller State（NVMe 控制器状态，可选）
│   ├── 头部
│   │   ├── SQ（提交队列）数量
│   │   └── CQ（完成队列）数量
│   │
│   ├── 提交队列状态（每个 24 字节）
│   │   └── 按 QID（队列 ID）升序排列
│   │
│   └── 完成队列状态（每个 24 字节）
│       └── 按 QID 升序排列
│
└── Vendor Specific Data（厂商特定数据，可选）
```

这个结构汇总自规范 Figures 358-361。

### 队列状态的详细信息

**标准队列镜像包含的内容**：

- **提交队列（SQ）状态**：
  - 队列创建参数
  - 活跃的 SQ Head 指针
  - 活跃的 SQ Tail 指针

- **完成队列（CQ）状态**：
  - 队列创建参数
  - 活跃的 CQ Head 指针
  - 活跃的 CQ Tail 指针
  - Slot 0 的相位（Phase）
  - 中断使能（Interrupt Enable）
  - 中断向量（Interrupt Vector）
  - 物理连续性属性（Physical Contiguity）

### 重建过程与约束

**前置条件**：

- 如果提供了标准状态，目标控制器上不得已存在 I/O 提交队列（SQ）或完成队列（CQ）

**提交流程**：

1. 对最后一段数据的成功处理会触发验证和提交
2. 创建镜像中指定的所有队列
3. 恢复队列的记录状态（指针、相位等）

**失败处理**：

- 如果任何一段传输失败，主机应重新传输整份镜像，以便覆盖部分状态

[PDF pp. 380-382](../_source/pages/page-380.md) | [PDF p. 380](../_source/pages/page-380.md) | [PDF p. 383](../_source/pages/page-383.md)

---

## 相关概念

控制器迁移涉及多个 NVMe 概念的协同工作：

| 相关概念 | 关系说明 | 规范引用 |
|---------|---------|---------|
| **[Identify Command Model](identify-command-model.md)** | 发布状态版本目录和厂商 UUID 目录，供两个迁移命令使用 | [PDF pp. 365-366](../_source/pages/page-365.md) |
| **[Controller Data Queues](controller-data-queues.md)** | 定义用户数据迁移队列，用于跟踪控制器状态镜像之外的用户数据修改 | [PDF pp. 207-209](../_source/pages/page-207.md) |
| **[Migration Change Tracking](migration-change-tracking.md)** | 在状态迁移期间记录主机内存或命名空间的用户数据修改；挂起配合空的最终报告可关闭主机内存变更窗口 | [PDF pp. 434-438](../_source/pages/page-434.md) |
| **[Queue Pair](queue-pair.md)** | 提供提交期间重建队列所需的身份、几何结构、指针、相位和中断属性 | [PDF pp. 381-383](../_source/pages/page-381.md) |
| **[Controller Enable, Shutdown, and Reset](controller-enable-shutdown-reset.md)** | 定义可解除挂起的控制器级重置，以及迁移引用的 `CC.EN` 行为 | [PDF pp. 376-377](../_source/pages/page-376.md) |

---

## 错误情况与边界条件

### Set Controller State 的目标要求

**有效目标**：

- 已挂起的控制器
- 已启用的控制器
- 离线的从控制器（Offline Secondary Controller）

**错误情况**：如果目标不满足以上任一条件，命令会失败并返回 Invalid Controller Identifier（无效控制器标识符）。

[PDF p. 377](../_source/pages/page-377.md)

### Resume 命令的失败条件

| 失败原因 | 返回错误 |
|---------|---------|
| 目标控制器正在运行（未挂起） | Controller Not Suspended（控制器未挂起） |
| 接收的状态尚未验证和提交 | Command Sequence Error（命令序列错误） |

[PDF p. 377](../_source/pages/page-377.md)

### 参数验证错误

以下情况都会导致命令无效：

- 镜像偏移量超出状态大小
- 偏移量未按双字对齐
- 使用了未公布的格式索引
- 索引与大小不匹配

[PDF pp. 379-383](../_source/pages/page-379.md)

### 其他可能的错误

Migration Send 命令在适用情况下还可能返回：

- **Invalid Controller Data Queue**：无效的控制器数据队列
- **Not Enough Resources**：资源不足

[PDF p. 383](../_source/pages/page-383.md)

---

## 规范引用

本文档基于 NVMe 规范以下章节整理：

- [Migration Receive 封装与快照一致性，PDF pp. 372-374](../_source/pages/page-372.md)
- [Migration Send 操作目录与 Suspend，PDF pp. 375-376](../_source/pages/page-375.md)
- [挂起行为、Resume 和 Set 前置条件，PDF p. 377](../_source/pages/page-377.md)
- [传输序列与命令字段，PDF pp. 378-379](../_source/pages/page-378.md)
- [Controller State 和队列状态布局，PDF pp. 380-382](../_source/pages/page-380.md)
- [提交行为与命令状态，PDF p. 383](../_source/pages/page-383.md)
