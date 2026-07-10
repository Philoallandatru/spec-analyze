# 控制器初始化（Controller Initialization）

控制器初始化是一个依赖于传输类型的启动过程（transport-specific bootstrap process），主要完成以下工作：

- 建立管理通道（Admin Path）
- 选择控制器支持的配置参数
- 使能控制器（Enable Controller）
- 发现其命令集（Command Set）和命名空间（Namespace）配置
- 创建 I/O 队列（I/O Queue）
- 可选地启用异步事件通知（Asynchronous Events）

> **规范参考**：[PDF pp. 124-127](../_source/pages/page-124.md)

## 概念模型

不同传输类型的控制器初始化流程存在差异，但核心步骤相似。下图对比了基于内存的传输（Memory-based，如 PCIe）和基于消息的传输（Message-based，如 NVMe-oF Fabrics）：

```text
基于内存的传输 (PCIe)                基于消息的传输 (Fabrics)
等待 RDY=0                          建立传输层连接
配置 AQA / ASQ / ACQ                 执行 Connect → 创建 Admin 队列 + 关联
选择 CSS / AMS / MPS                 如果 AUTHREQ != 0 则执行身份认证
设置 EN=1; 等待 RDY=1                Property Get/Set CAP, CC; EN=1; 等待 RDY=1
          \                         /
           +---> 执行 Identify 识别控制器和 I/O 命令集
                 枚举活动的命名空间
                 协商/创建 I/O 队列
                 启用事件监听 + 提交 AER
```

> **说明**：此流程综合了规范中的两个标准序列以及 Figure 82 中的队列创建时序图。[PDF pp. 124-126](../_source/pages/page-124.md)

## 基于能力寄存器的通用初始化序列

### 配置命令集选择

主机（Host）需要根据控制器能力寄存器 `CAP.CSS`（Controller Capabilities - Command Sets Supported）来决定命令集配置 `CC.CSS`（Controller Configuration - Command Set Selection）：

| CSS 值 | 含义 | 说明 |
|--------|------|------|
| `111b` | 无 I/O 命令集模式（No I/O Command Set） | 仅支持管理命令 |
| `110b` | I/O 命令集组合（I/O Command Set Combination） | 支持多种命令集的组合 |
| `000b` | NVM 命令集（NVM Command Set） | 标准块存储命令集，需满足能力条件 |

### 初始化前的准备工作

在设置 `CC.EN=1`（使能控制器）之前，主机还需要完成：

1. **选择仲裁机制**（Arbitration Mechanism）：从控制器支持的仲裁方式中选择一种
2. **选择内存页大小**（Memory Page Size）：与系统内存管理匹配
3. **等待就绪信号**：设置 `CC.EN=1` 后，轮询等待 `CSTS.RDY=1`（Controller Status - Ready）

> **规范参考**：[PDF pp. 124-126](../_source/pages/page-124.md)

### 控制器就绪后的发现流程

控制器进入就绪状态后，主机按以下顺序执行发现操作：

1. **识别控制器**：执行 `Identify Controller` 命令，获取控制器基本信息
2. **命令集发现**：
   - 如果控制器支持命令集组合（Combinations），主机识别可用组合并选择合适的配置文件（Profile）
   - 对每个已使能的 I/O 命令集，执行命令集特定的识别操作
3. **枚举命名空间**：遍历活动的命名空间标识符（Active NSID）
4. **读取标识结构**：获取命令集特定和命令集独立的 Identify 数据结构
5. **创建 I/O 队列**：
   - **基于内存的传输**：按照"先完成队列（CQ）后提交队列（SQ）"的顺序创建
   - **Fabrics 传输**：使用 Connect 命令创建队列对（Queue Pair）
6. **配置异步事件**：在控制器就绪后的任意时刻配置异步事件通知

> **规范参考**：[PDF pp. 125-127](../_source/pages/page-125.md)

## 不同传输类型的启动细节

下表对比了基于内存和基于消息两种传输类型在初始化各阶段的差异：

| 初始化阶段 | 基于内存的控制器（PCIe） | 基于消息的控制器（Fabrics） |
|-----------|------------------------|---------------------------|
| **建立管理路径** | 在禁用状态下通过寄存器编程：<br>• `AQA`（Admin Queue Attributes）<br>• `ASQ`（Admin Submission Queue）<br>• `ACQ`（Admin Completion Queue） | 建立传输层连接后：<br>• 执行 Connect 命令<br>• 创建 Admin 队列对和关联（Association） |
| **属性访问方式** | 通过绑定定义的寄存器访问（Binding-defined register access） | 使用 Fabrics Property Get 和 Property Set 命令 |
| **身份认证** | 不属于此初始化序列 | 当 `AUTHREQ != 0` 时，必须在正常初始化之前完成认证 |
| **创建 I/O 队列** | 1. 通过 Set Features 设置队列数量（Number of Queues）<br>2. 配置中断（Interrupt）设置<br>3. 先创建完成队列（Create CQ）<br>4. 后创建提交队列（Create SQ） | 1. 查询队列数量限制（Number of Queues 和 `CAP.MQES`）<br>2. 对每个 I/O 队列对执行 Connect |

> **补充说明**：Figure 82 展示了成功的 Connect 命令会在任何必需的带内身份认证（in-band authentication）交换之前创建 Admin 或 I/O 队列。如果主机在建立关联后 2 分钟内未设置 `CC.EN=1`，控制器可能会移除该关联。

> **规范参考**：[PDF pp. 125-127](../_source/pages/page-125.md)

## Discovery 控制器的初始化流程

Discovery 控制器（Discovery Controller）是 NVMe-oF 架构中的特殊控制器，专门用于发现可用的存储目标。其初始化流程根据是否请求变更通知而有所不同：

```text
收到 Connect 请求
  |
  |-- 如果请求变更通知 + 非零 KATO
  |      ├─> 控制器可调整 KATO（Keep Alive Timeout）
  |      ├─> 预分配 AER/AEN 资源（异步事件）
  |      ├─> Connect 成功
  |      └─> 主机提交 AER（Asynchronous Event Request）
  |
  └-- 否则
         ├─> 使用固定的 KATO
         |   （或对不支持的通知请求返回 Connect 错误）
         └─> 主机完成连接，不提交 AER
```

### 初始化步骤详解

上述流程简化自规范中的 Figure 83，保留了关键的决策分支：

- **变更通知支持** + **非零 Keep Alive Timeout** → 控制器预留异步事件资源
- Connect 成功后的操作序列：
  1. 如需身份认证，执行认证流程
  2. 读取控制器能力（Capabilities）
  3. 写入控制器配置 `CC`（包括设置 `EN=1`）
  4. 等待就绪信号 `RDY=1`
  5. 识别相应的 Controller 数据结构
  6. 读取 Discovery Log Page（发现日志页）

> **规范参考**：[PDF p. 128](../_source/pages/page-128.md)

## 关键术语对照表

| 中文术语 | 英文术语 | 缩写 | 说明 |
|---------|---------|-----|------|
| 控制器 | Controller | - | NVMe 设备的管理核心 |
| 主机 | Host | - | 发起 I/O 操作的系统 |
| 管理队列 | Admin Queue | AQ | 用于管理命令的队列 |
| 提交队列 | Submission Queue | SQ | 主机提交命令的队列 |
| 完成队列 | Completion Queue | CQ | 控制器返回完成状态的队列 |
| 命名空间 | Namespace | NS | NVMe 的逻辑存储单元 |
| 异步事件请求 | Asynchronous Event Request | AER | 主机提交的异步事件监听命令 |
| 异步事件通知 | Asynchronous Event Notification | AEN | 控制器发送的异步事件 |
| 能力寄存器 | Controller Capabilities | CAP | 报告控制器支持的功能 |
| 配置寄存器 | Controller Configuration | CC | 控制器的配置参数 |
| 状态寄存器 | Controller Status | CSTS | 控制器的当前状态 |
| 队列对 | Queue Pair | QP | 一对提交队列和完成队列 |
| 关联 | Association | - | Fabrics 中主机与控制器的连接关系 |

## 规范参考

以下页面提供了初始化流程的详细规范说明：

- [基于内存的初始化起点，PDF p. 124](../_source/pages/page-124.md)
- [基于内存的完成流程与 Fabrics 启动，PDF p. 125](../_source/pages/page-125.md)
- [Figure 82 时序图与 Fabrics 初始化起点，PDF p. 126](../_source/pages/page-126.md)
- [Fabrics 初始化延续与超时处理，PDF p. 127](../_source/pages/page-127.md)
- [Figure 83 时序图与 Discovery 初始化序列，PDF p. 128](../_source/pages/page-128.md)
