# 异步事件上报（Asynchronous Event Reporting）

## 概述

异步事件请求（Asynchronous Event Request, AER）是 NVMe 规范中的一种特殊管理命令，它允许控制器（Controller）主动向主机（Host）报告各类事件。与传统的命令-响应模式不同，AER 是一个"长寿命"命令——主机提交后，控制器会持有这个请求，直到有事件需要报告时才通过完成队列条目（Completion Queue Entry, CQE）返回结果。

**支持的事件类型包括：**
- 状态事件（Status）
- 健康状态事件（Health）
- 通知事件（Notice）
- I/O 相关事件（I/O-specific）
- 即时事件（Immediate）
- 一次性事件（One-shot）
- 厂商自定义事件（Vendor）

**规范参考：** [PDF pp. 195-202](../_source/pages/page-195.md)

---

## 工作原理

### 交互流程图

下面的流程图展示了主机和控制器如何通过 AER 机制协同工作：

```text
主机（Host）                控制器（Controller）            日志页（Log Page）
    |
    |---- 提交一个或多个 AER ---->|
    |                              |
    |                              |<---- 事件发生
    |                              |
    |<---- CQE: AET + AEI + LID --|
    |                              |
    |                              |---- 屏蔽相同事件类型
    |                              |
    |---- Get Log Page (RAE=0) -------------------------------->|
    |                                                            |
    |<---- 返回事件详情 ----------------------------------------|
    |                              |
    |                              |<---- 事件类型解除屏蔽/清除
    |                              |
    |---- 提交新的 AER 补充 ------>|
    |                              |
```

### 流程说明

1. **提交阶段**：主机预先向控制器提交一个或多个 AER 命令（在控制器支持的上限范围内）
2. **等待阶段**：控制器持有这些请求，不立即返回
3. **事件触发**：当控制器检测到需要报告的事件时，通过 CQE 返回事件信息（AET + AEI + LID）
4. **自动屏蔽**：控制器自动屏蔽同类型事件，防止重复报告
5. **主机处理**：主机通过 Get Log Page 命令读取详细日志（RAE=0 表示读取后确认）
6. **解除屏蔽**：日志读取完成后，该事件类型的屏蔽被解除
7. **补充请求**：主机提交新的 AER 以维持未完成请求的数量

**特殊情况：**
- **即时事件（Immediate）**：仅在事件发生时已有未完成 AER 的情况下才报告
- **一次性事件（One-shot）**：报告后自动清除，不需要日志读取步骤

**规范参考：** [PDF pp. 195-197](../_source/pages/page-195.md)

---

## 请求状态管理

### 未完成请求（Outstanding）与待处理事件（Pending）

| 场景 | 行为说明 | 规范参考 |
|------|----------|----------|
| **无超时机制** | AER 命令没有超时限制，可以无限期等待事件发生 | [PDF pp. 195-196](../_source/pages/page-195.md) |
| **多个未完成请求** | 主机可同时提交多个 AER（在控制器通告的上限内），减少事件上报延迟 | [PDF pp. 195-196](../_source/pages/page-195.md) |
| **重置时中止** | 控制器重置（Reset）会中止所有未完成的 AER，且不返回 CQE | [PDF p. 196](../_source/pages/page-196.md) |
| **事件保留机制** | 除即时事件外，当启用的事件发生但没有未完成 AER 时，事件会被保留到下次 AER 提交时报告 | [PDF p. 197](../_source/pages/page-197.md) |
| **事件合并与排队** | 相同的事件响应可以合并为一次报告；不同的事件响应应排队等待报告 | [PDF p. 197](../_source/pages/page-197.md) |
| **事件类型屏蔽** | 普通事件一旦被报告，该事件类型就会被屏蔽，直到清除操作成功完成 | [PDF p. 196](../_source/pages/page-196.md) |
| **介质未就绪阻塞** | 如果清除事件所需的日志暂时无法读取（media-not-ready），则阻止该事件的报告 | [PDF p. 196](../_source/pages/page-196.md) |

### 关键理解点

**屏蔽机制的作用**：防止同一事件在主机处理之前反复报告。只有当主机读取相关日志（执行清除操作）后，控制器才会再次报告同类型的新事件。

---

## 完成队列条目（CQE）格式

### 字段布局

当 AER 命令完成时，控制器返回的 CQE 包含以下信息：

```text
AER CQE DW0（双字 0）
31        24 23        16 15         8 7       3 2       0
+-----------+------------+-------------+---------+---------+
| 保留      | LID        | AEI         | 保留    | AET     |
|           | 日志标识   | 事件信息    |         | 事件类型|
+-----------+------------+-------------+---------+---------+

AER CQE DW1（双字 1）
事件特定的 32 位参数（Event-specific Parameter）
```

### 字段说明

| 字段 | 位范围 | 说明 |
|------|--------|------|
| **AET** | 2:0 | 异步事件类型（Asynchronous Event Type）<br>标识事件所属的大类（族） |
| **AEI** | 15:8 | 异步事件信息（Asynchronous Event Information）<br>标识该族内的具体事件 |
| **LID** | 23:16 | 日志标识符（Log Identifier）<br>指向包含事件详细信息和清除操作的日志页 |
| **DW1** | 全部 | 事件特定参数<br>携带与该事件相关的附加数据（32 位） |

**规范参考：** [PDF pp. 197-198](../_source/pages/page-197.md)（Figures 147-148）

---

## 事件分类与清除机制

### 按清除模型分类

| 事件族 | 清除/保留模型 | 处理方式 |
|--------|--------------|----------|
| **Error（错误）** | 日志支持型 | 使用 Get Log Page 命令读取指定的日志页，并设置 RAE=0（Retain Asynchronous Event = 0）以清除事件 |
| **SMART/Health（健康）** | 日志支持型 | 同上 |
| **Notice（通知）** | 日志支持型 | 同上 |
| **I/O-specific（I/O 特定）** | 日志支持型 | 同上 |
| **Vendor（厂商自定义）** | 日志支持型 | 同上 |
| **Immediate（即时）** | 发生时上报型 | 仅在事件发生时存在未完成 AER 的情况下才报告 |
| **One-Shot（一次性）** | 自动清除型 | 报告一次后，控制器在完成时自动清除，无需主机操作 |

### 具体事件类型（AEI 值）

规范的 Figures 149-155 中定义了详细的 AEI 值。主要事件族涵盖：

**Notice 事件族：**
- 命名空间属性变更
- 固件激活（Firmware Activation）
- 遥测日志（Telemetry）
- 非对称命名空间访问（ANA）状态变更
- 耐久性/可达性/发现日志变更

**I/O-specific 事件族：**
- 预留（Reservation）状态
- 清理（Sanitize）状态

**Immediate 事件族：**
- 子系统关机（Subsystem Shutdown）

**One-Shot 事件族：**
- 控制器数据队列尾部/满载通知

**规范参考：** [PDF pp. 198-202](../_source/pages/page-198.md)

### 持久条件与瞬态条件的处理

对于**持久条件（Persistent Condition）**或**瞬态阈值事件（Transient Threshold Event）**：

1. **问题**：如果底层触发条件仍然存在，清除事件后可能立即再次触发通知
2. **解决方案**：主机在清除事件前应：
   - 调整相关阈值参数（如温度阈值），或
   - 临时屏蔽该事件类型
3. **目的**：避免事件通知风暴，给主机时间处理根本原因

**规范参考：** [PDF p. 197](../_source/pages/page-197.md)

---

## 异步事件配置功能

### 功能概述

**异步事件配置功能（Asynchronous Event Configuration Feature）** 是一个事件族级别的启用屏蔽（Enable Mask）。通过 Set Features 命令配置此功能，主机可以控制哪些类型的事件会被报告。

### 可配置的事件类型

主机可以独立启用或禁用以下事件类型的通知：

| 事件类别 | 具体事件 |
|---------|---------|
| **日志变更** | Discovery Log、Host Log、AVE Log、Pull-Model Log 的内容变更 |
| **命名空间变更** | 命名空间的分配（Allocated）或连接（Attached）状态变更 |
| **网络状态** | 可达性（Reachability）变化 |
| **温度恢复** | 温度滞后恢复（Temperature-hysteresis Recovery） |
| **关机通知** | 正常关机（Normal Shutdown） |
| **耐久性组** | Endurance Group 事件 |
| **延迟聚合** | Predictable Latency Aggregates 事件 |
| **ANA** | 非对称命名空间访问（Asymmetric Namespace Access）状态变更 |
| **遥测** | Telemetry Log 更新 |
| **固件激活** | Firmware Activation 完成 |
| **SMART 警告** | SMART/Health 警告 |

### 重要行为特征

| 场景 | 行为 |
|------|------|
| **启用已成立的持久条件** | 如果启用的事件对应的条件已经成立，控制器会立即发送该事件 |
| **启用不支持的资源特定事件** | 对于控制器不支持的资源特定事件族，启用操作是非法的 |
| **配置的作用范围** | 此功能仅控制事件通知的交付，不影响底层条件是否发生或日志是否更新 |
| **恢复事件的特殊要求** | 某些恢复事件要求在事件发生时刻存在未完成的 AER，否则不会被保留 |
| **保留事件行为** | 已启用但无未完成 AER 时发生的事件，其保留行为遵循各事件族的规则 |

**规范参考：** [PDF pp. 399-400](../_source/pages/page-399.md)

---

## 关键关系与约束

### 命令与功能的关联

| 组件 | 作用 | 规范参考 |
|------|------|----------|
| **AER 命令** | 接收事件通知，CQE 指示应读取哪个日志页以获取详细信息和执行清除操作 | [PDF pp. 196-198](../_source/pages/page-196.md) |
| **Get Log Page 命令** | 读取日志详情并清除事件（当 RAE=0 时），将通知机制与日志检索耦合 | [PDF pp. 196-198](../_source/pages/page-196.md) |
| **Asynchronous Event Configuration** | 选择性启用或禁用可配置的事件条件，控制哪些事件会生成通知 | [PDF p. 196](../_source/pages/page-196.md) |

### 错误处理

**超出 AER 数量限制**：如果主机提交的 AER 数量超过控制器支持的上限，控制器会返回命令特定状态码 `05h`（Asynchronous Event Request Limit Exceeded）。

**规范参考：** [PDF p. 197](../_source/pages/page-197.md)

---

## 规范引用索引

以下是本文档内容对应的 NVMe 规范原文位置：

### 核心机制

- [AER 生命周期、屏蔽机制、事件族定义，PDF pp. 195-197](../_source/pages/page-195.md)
- [Figures 146-148：状态码与 CQE 编码格式，PDF pp. 197-198](../_source/pages/page-197.md)

### 事件类型详细定义

- [Figures 149-150：Error 事件与 SMART/Health 事件，PDF p. 198](../_source/pages/page-198.md)
- [Figure 151：Notice 事件，PDF pp. 199-201](../_source/pages/page-199.md)
- [Figures 152-155：I/O-specific、Immediate 与 One-Shot 事件，PDF pp. 201-202](../_source/pages/page-201.md)
