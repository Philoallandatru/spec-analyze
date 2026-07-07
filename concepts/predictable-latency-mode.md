# 可预测延迟模式（Predictable Latency Mode）

## 概述

可预测延迟模式是 NVMe 规范中的一项服务质量（QoS）特性，用于保证 I/O 操作的延迟可预测性。该模式将每个支持的 NVM Set（NVM 集合）划分为两个交替运行的时间窗口：

- **确定性窗口（DTWIN, Deterministic Window）**：在此窗口内，延迟表现可预测，主机可以执行有延迟保证需求的 I/O 操作
- **非确定性窗口（NDWIN, Non-Deterministic Window）**：在此窗口内，控制器执行后台维护操作，为下一个确定性窗口做准备

控制器通过日志页面（Log Page）向主机报告当前所处的窗口状态、警告和转换事件、静态边界值以及动态剩余资源估计。[PDF pp. 248-250](../_source/pages/page-248.md)

## 窗口运行模型

### 基本状态转换

可预测延迟模式的核心是两个窗口之间的状态转换：

```text
              读/写/时间预算耗尽
[DTWIN] ----------------------------------------------> [NDWIN]
确定性窗口                                              非确定性窗口
   ^                                                       |
   |     最小准备间隔（后台维护完成）                        |
   `-------------------------------------------------------'
```

**转换逻辑说明：**

- **DTWIN → NDWIN**：当读/写操作次数或运行时间超过预算，或发生确定性偏差时，控制器转入非确定性窗口
- **NDWIN → DTWIN**：完成必要的后台维护后，经过最小准备间隔，控制器才能重新进入确定性窗口

该状态模型描述了日志中报告的窗口状态，详细的转换规则在规范后续章节中定义。[PDF pp. 249-250](../_source/pages/page-249.md)

### 日志报告内容

控制器通过日志页面向主机提供以下信息：

| 报告项 | 说明 |
|--------|------|
| **窗口状态（Window Status）** | 当前所处状态：已禁用（Disabled）、确定性窗口（Deterministic Window）或非确定性窗口（Non-Deterministic Window） |
| **事件（Events）** | 读/写/时间警告事件，以及因偏差或超限导致的自主 DTWIN 到 NDWIN 转换事件 |
| **静态值（Static Values）** | DTWIN 的典型读/写预算、最大 DTWIN 时间、NDWIN 的最小时间上下限 |
| **可靠估计值（Reliable Estimates）** | 在当前活动负载和运行条件下，DTWIN 中剩余可用的读操作次数、写操作次数和时间（毫秒） |

**参数特性：**

- **静态值**：基于终身最坏情况的参数，在设备生命周期内保持不变
- **可靠估计值**：随主机活动和运行条件动态变化，单调递减或演变
- **计数规则**：
  - 读操作按 4 KiB 随机读计算
  - 写操作使用该 NVM Set 的最优写入大小（Optimal Write Size）作为单位

[PDF pp. 249-250](../_source/pages/page-249.md)

## 事件管理

### 聚合日志（Aggregate Log）

聚合日志提供了一个集中的事件通知机制：

```text
事件发生 -> 对应 NVM Set ID 加入有序聚合日志
                    |
                    v
           异步事件通知发送给主机
                    |
                    v
     主机读取聚合日志，获取待处理事件的 NVM Set 列表
                    |
                    v
以 RAE=0 读取每个 NVM Set 的日志 -> 清除该 Set 的事件位
                    |
                    v
         对应条目从聚合日志中移除
```

**工作机制：**

- 聚合日志是一个**按升序排列**的非零 NVM Set 标识符列表，包含所有具有待处理已启用事件的 NVM Set
- 当新的 NVM Set 加入列表时，生成一个异步事件通知
- 读取超过列表长度的部分返回零值
- 主机使用 `RAE=0`（Retain Asynchronous Event = 0）参数成功读取对应 NVM Set 的日志页面后，该 Set 的事件位被清除，同时从聚合日志中移除相应条目

[PDF pp. 249-250](../_source/pages/page-249.md)

## Feature 配置命令

### Predictable Latency Mode Config（Feature ID: 13h）

**功能：**启用一个 NVM Set 的可预测延迟模式，并配置事件阈值

**行为特性：**

- 启用操作仅在必需的后台准备完成后才会成功返回
- 成功启用后，NVM Set 初始进入非确定性窗口（NDWIN）
- 事件配置结构支持独立启用两类事件：
  - **偏差/超限事件**：当发生确定性偏差或超过预算值时触发
  - **警告事件（读/写/时间）**：当对应的可靠估计值跌至配置阈值以下时触发

[PDF pp. 405-406](../_source/pages/page-405.md)

### Predictable Latency Mode Window（Feature ID: 14h）

**功能：**为指定 NVM Set 内的每个命名空间（Namespace）选择目标窗口

**操作规则：**

- 仅在模式已启用时有效，模式禁用时该命令无效
- 可以请求进入确定性窗口（DTWIN）或非确定性窗口（NDWIN）
- 命令完成标志着窗口转换的边界
- **特殊行为**：请求进入 DTWIN 时，如果最小 NDWIN 驻留时间未满，命令完成可能被延迟直到该时间条件满足

[PDF pp. 406-407](../_source/pages/page-406.md)

## 相关主题

### 关联功能

- **[NVM Sets and Endurance Groups](nvm-sets-and-endurance-groups.md)**  
  提供 NVM Set 的资源分组、标识符定义，这些标识符被每个 NVM Set 的日志页面选择器使用  
  [PDF pp. 99-103, 248](../_source/pages/page-099.md)

- **[Asynchronous Event Reporting](asynchronous-event-reporting.md)**  
  异步事件报告机制，为新加入聚合日志的 NVM Set 事件提供通知与确认行为  
  [PDF pp. 195-202, 249-250](../_source/pages/page-195.md)

## 规范索引

### 按主题分类的规范页面引用

| 主题 | 规范页面 |
|------|---------|
| 每个 NVM Set 的选择器与 Endurance Group 边界 | [PDF p. 248](../_source/pages/page-248.md) |
| 窗口/事件字段与静态/可靠估计值 | [PDF pp. 249-250](../_source/pages/page-249.md) |
| 有序聚合列表与每个 Set 的事件清除 | [PDF p. 250](../_source/pages/page-250.md) |
| 模式启用、事件阈值与窗口选择 | [PDF pp. 405-407](../_source/pages/page-407.md) |
