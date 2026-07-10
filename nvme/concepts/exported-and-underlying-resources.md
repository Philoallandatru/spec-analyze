# 导出资源与底层资源（Exported and Underlying NVM Resources）

## 概念概述

在 NVMe 规范中，**导出（Exporting）** 是一个重要概念，它将实际提供存储能力的物理或虚拟资源（底层资源）与为远程访问而展示的逻辑视图（导出资源）明确区分开来。

简单来说：
- **底层资源（Underlying Resources）**：实际的物理存储资源，包括命名空间、子系统和端口
- **导出资源（Exported Resources）**：基于底层资源创建的逻辑视图，专门用于向远程主机提供访问

一个导出的 NVM 子系统（Exported NVM Subsystem）可以包含以下组件：
- 导出命名空间（Exported Namespaces）
- 控制器（Controllers）
- 导出端口（Exported Ports）
- 允许主机列表（Allowed Host List，可选）

[PDF pp. 30-31](../_source/pages/page-030.md)

## 架构模型

下图展示了底层资源与导出资源之间的映射关系：

```text
底层 NVM 子系统（Underlying NVM Subsystem）
|
|-- 底层命名空间（Underlying Namespaces）
|   |
|   └─ 关联（associate）──> 导出命名空间（Exported Namespaces）
|
`-- 底层端口（Underlying Ports）
    |
    └─ 端口列表（Ports List）──> 导出端口（Exported Ports）
                                      |
                                      v
                          导出 NVM 子系统（Exported NVM Subsystem）
                          |
                          |-- 控制器（Controllers）[0..*]
                          |-- 导出命名空间（Exported Namespaces）[0..*]
                          |-- 导出端口（Exported Ports）[0..*]
                          `-- 允许主机列表（Allowed Host List）[可选]
```

**说明**：这是便于理解的逻辑视图，用于展示资源的所有权和映射关系。规范的后续章节会给出精确的管理数据结构定义。

[PDF pp. 30-31, 35-36](../_source/pages/page-030.md) [PDF p. 35](../_source/pages/page-035.md) [PDF p. 36](../_source/pages/page-036.md)

## 核心概念定义

| 概念 | 说明 |
|---|---|
| **导出 NVM 子系统**<br>（Exported NVM Subsystem） | 向远程主机呈现底层存储资源的逻辑子系统，是远程访问的入口 |
| **导出命名空间**<br>（Exported Namespace） | 属于导出 NVM 子系统的命名空间，提供给远程主机访问 |
| **导出端口**<br>（Exported Port） | NVMe-oF 传输端点，通过导出端口 ID 进行标识 |
| **底层命名空间**<br>（Underlying Namespace） | 实际的存储命名空间，可以被关联到某个导出子系统中供远程访问 |
| **底层端口**<br>（Underlying Port） | 将底层子系统连接到传输层的物理或虚拟端口 |

**重要说明**：导出资源是为远程访问而专门创建的独立逻辑实体，它们**不是**底层物理资源的简单别名或引用。两者之间存在明确的边界和生命周期管理。

[PDF pp. 30-31, 35-36](../_source/pages/page-030.md) [PDF p. 35](../_source/pages/page-035.md) [PDF p. 36](../_source/pages/page-036.md)

## 资源发现：Identify 目录

基于消息传递的控制器（Message-based Controller）会暴露两个带有版本计数的目录，用于发现可用的底层资源：

### 1. 底层命名空间列表（`CNS=1Dh`）

列出所有可通过物理或虚拟功能（Function）访问的底层命名空间。每个条目包含：
- 底层 NVM 子系统的 NQN（NVMe Qualified Name）
- 命名空间 ID（NSID）
- 关联的控制器 ID（Controller ID，在子系统内唯一）

### 2. 底层端口列表（`CNS=1Eh`）

列出所有可用于导出 NVMe-oF 子系统的底层端口。

### 资源目录汇总

| 目录类型 | 条目标识信息 | 连接数据 |
|---|---|---|
| **底层命名空间列表**<br>（`CNS=1Dh`） | Subsystem NQN<br>+ NSID<br>+ Controller ID | 标识所有可供导出使用的存储命名空间 |
| **底层端口列表**<br>（`CNS=1Eh`） | Underlying Port ID | 包含传输地址、子类型、类型、地址族及相关要求 |

### 版本计数机制

两个版本计数器（Generation Counter）的行为规则：
- 在底层 NVM 子系统重置时清零
- 每次目录内容发生变更时递增 1
- 达到全 1 后回绕到 0

**说明**：端口的传输地址和传输特定子类型的具体格式由相应的 NVMe 传输规范定义。

[PDF pp. 368-370](../_source/pages/page-368.md)

## 导出子系统的生命周期管理

### 创建与清除流程

```text
创建导出子系统（Create Exported Subsystem）
选择访问模式：受限（Restricted）或 不受限（Unrestricted）
          |
          v
[新建的导出子系统]
- 允许主机列表：空
- 导出命名空间：无
          |
          +-- 后续操作：
          |   - 关联命名空间（Associate Namespaces）
          |   - 创建导出端口（Create Exported Ports）
          |   - 配置允许主机（Configure Allowed Hosts）
          |
使用阶段
          |
清除前准备：
断开所有主机连接
          |
          v
清除全部导出配置（Clear Exported Configuration）
          |
          v
[导出资源已删除]
          |
          X -- 底层命名空间/子系统不受影响 --
```

### 创建导出子系统（Create Exported NVM Subsystem）

**操作说明**：
- 创建时需要选择初始访问模式：
  - **不受限模式（Unrestricted）**：接受任意主机的连接请求
  - **受限模式（Restricted）**：仅允许列表中的主机连接
- 新创建的导出子系统初始状态：
  - 允许主机列表为空
  - 没有关联任何导出命名空间
- 命令成功后，通过命令数据缓冲区返回新分配的子系统 NQN（`SUBNQN`）
- **限制**：此命令不能在导出 NVM 子系统上执行

[PDF pp. 448-449](../_source/pages/page-448.md)

### 清除导出配置（Clear Exported NVM Resource Configuration）

**操作说明**：
- 删除所有导出相关的配置信息
- **不影响**底层命名空间或底层 NVM 子系统的状态
- **前置条件**：必须先将所有主机从所有导出 NVM 子系统上断开连接
  - 如果存在未断开的主机连接，命令将以"命令序列错误"（Command Sequence Error）失败
- **限制**：此命令同样不能在导出 NVM 子系统上执行

**设计意图**：这种设计将逻辑的导出配置与实际的底层资源清晰分离，确保清除导出配置时不会意外影响底层的物理存储资源。

[PDF p. 448](../_source/pages/page-448.md)

## 导出命名空间的关联管理

### 关联与解除关联流程

```text
前提条件：
[有效的、未被使用的导出命名空间 ID (ENSID)]
+
[已激活并已附加到控制器的底层命名空间]
          |
          v
执行关联命名空间操作（Associate Namespace）
          |
          v
[已分配的导出命名空间]
- 与底层命名空间访问相同的用户数据
- 此时尚未附加到任何控制器
          |
          v
命名空间附加操作（Namespace Attachment）
- 这是独立的操作流程
- 将导出命名空间附加到控制器
          |
          v
使用阶段
          |
          v
解除关联前准备：
从所有控制器分离（Detach）
          |
          v
执行解除关联操作（Disassociate Namespace）
          |
          v
[导出命名空间 ID 被删除]
```

**设计要点**：
- 此流程明确区分了两个概念：
  - **存储关联（Association）**：建立导出命名空间与底层存储之间的映射
  - **控制器附加（Controller Attachment）**：将命名空间附加到具体的控制器供访问
- 遵循"先分离再删除"（Detach-before-Delete）的前置条件原则

[PDF pp. 460-462](../_source/pages/page-460.md)

### 关联命名空间（Associate Namespace）

**操作参数**：
- 通过 NQN 标识目标导出子系统
- 通过三元组标识底层命名空间：
  - 底层子系统 NQN（Underlying Subsystem NQN）
  - 控制器 ID（Controller ID）
  - 激活的命名空间 ID（Active NSID）

**前置条件**：
- 底层命名空间必须已经分配（Allocated）
- 底层命名空间必须已附加（Attached）到指定的控制器

**操作效果**：
- 在导出子系统中分配一个新的导出命名空间
- 导出命名空间**不会**自动附加到控制器（需要单独的附加操作）
- 导出命名空间和底层命名空间访问同一份用户数据存储
- 触发"命名空间属性已变更"（Namespace Attribute Changed）异步事件

[PDF pp. 460-461](../_source/pages/page-460.md)

### 解除关联命名空间（Disassociate Namespace）

**前置条件**：
- 导出命名空间必须已从所有控制器上分离（Detached）

**操作效果**：
- 移除导出子系统与底层命名空间之间的关联
- 删除导出命名空间 ID（ENSID）
- 触发"命名空间属性已变更"（Namespace Attribute Changed）异步事件

[PDF pp. 461-462](../_source/pages/page-461.md)

## 导出子系统的删除控制

### 删除操作的前置条件检查

| 存在的依赖关系 | 删除操作结果 |
|---|---|
| 存在活跃的控制器连接 | 命令失败，返回"命令序列错误"（Command Sequence Error） |
| 存在已关联的导出命名空间 | 命令失败，返回"命令序列错误"（Command Sequence Error） |
| 存在已创建的导出端口 | 命令失败，返回"命令序列错误"（Command Sequence Error） |
| 无任何依赖 | 导出 NVM 子系统被成功删除 |

**命令执行限制**：
- `Manage Exported Namespace` 命令不能在导出 NVM 子系统上执行
- `Manage Exported NVM Subsystem` 命令同样不能在导出 NVM 子系统上执行

**设计意图**：这些限制确保了管理操作必须在底层子系统上执行，避免通过导出视图进行可能导致混乱的管理操作。

[PDF pp. 460, 462-463](../_source/pages/page-460.md)

## 导出子系统的访问控制

### 访问模式状态转换

```text
[不受限模式（Unrestricted）]
允许所有主机连接
          |
          | 执行：更改访问模式（Change Access Mode）→ 受限模式
          | 副作用：断开所有不在允许列表中的主机
          v
[受限模式（Restricted）]
仅允许"允许主机列表"中的主机连接
          |
          +-- 授予访问权限（Grant Access）
          |   添加新的 主机/子系统/底层端口 组合
          |
          `-- 撤销访问权限（Revoke Access）
              移除指定组合并立即断开受影响的主机连接
```

**访问控制行为特点**：
- 切换到受限模式会**立即断开**不在允许列表中的已连接主机
- 撤销访问权限会**立即断开**受影响的主机连接
- 这种设计确保访问控制策略能够实时生效

[PDF pp. 463-466](../_source/pages/page-463.md)

### 访问控制操作详解

#### 更改访问模式（Change Access Mode）

- **切换到受限模式**：断开任何不在该子系统允许主机列表中的已连接主机
- **切换到不受限模式**：允许所有主机连接

#### 授予访问权限（Grant Access）

**操作对象**：
- 针对非空的主机列表和导出子系统列表进行操作
- 将每个目标子系统与一个底层端口 ID（Underlying Port ID）关联

**主机条目结构**：
- 128 位主机标识符（Host Identifier）
- 主机 NQN（`HOSTNQN`）

**子系统条目结构**：
- 子系统 NQN（`SUBNQN`）
- 用于授予访问的底层端口 ID

**错误处理**：
- 失败的列表条目可以通过错误信息日志（Error Information Log）中的字节偏移量（Byte Offset）定位

[PDF pp. 464-466](../_source/pages/page-464.md)

#### 撤销访问权限（Revoke Access）

**操作对象**：与授予操作类似，针对主机和子系统列表

**操作效果**：
- 在选定的导出端口上断开受影响的已连接主机
- 移除相应的访问权限条目

[PDF pp. 465-466](../_source/pages/page-465.md)

## 导出端口的生命周期管理

### 创建导出端口（Create Exported Port）

**操作说明**：
- 将一个导出子系统绑定到：
  - 一个底层端口（Underlying Port）
  - 一个传输服务标识符（Transport Service Identifier）

**端口 ID 分配方式**：
- **主机指定**：主机可以提供一个唯一的导出端口 ID
- **控制器生成**：主机可以请求控制器自动生成端口 ID
- 命令成功完成时，返回所选的端口 ID

[PDF pp. 467-468](../_source/pages/page-467.md)

### 删除导出端口（Delete Exported Port）

**操作参数**：
- 子系统 NQN（`SUBNQN`）
- 关联的导出端口 ID（Exported Port ID）

**使用建议**：
- 应该在端口未被使用时执行删除操作

**删除使用中端口的后果**：
- 会终止通过该端口的所有主机/控制器关联
- **未定义行为**：未完成的命令（Outstanding Commands）和底层 Fabric 资源的行为是未定义的
- 因此强烈建议先断开所有连接再删除端口

[PDF pp. 468-469](../_source/pages/page-468.md)

## 相关概念说明

### Fabric Zoning 与 Allowed Host List 的区别

**Fabric Zoning（网络分区）**定义主机与 NVM 子系统之间的访问控制配置。它与**允许主机列表（Allowed Host List）**是不同的概念——后者是导出 NVM 子系统的可选状态。

| 概念 | 说明 | 作用域 |
|---|---|---|
| **Fabric Zoning** | 在网络架构层面定义主机与 NVM 子系统之间的访问控制配置 | 传输层/网络层 |
| **Allowed Host List** | 导出 NVM 子系统的可选状态，用于应用层的访问控制 | 子系统层 |

**关键区别**：
- Fabric Zoning 是基础设施级别的访问控制
- Allowed Host List 是子系统级别的逻辑访问控制
- 两者可以配合使用，提供多层次的安全保护

[PDF p. 31](../_source/pages/page-031.md)

## 规范引用索引

以下是本文档涉及的所有规范章节引用，便于深入查阅：

### 基础概念定义
- [导出资源定义，PDF pp. 30-31](../_source/pages/page-030.md)
- [底层命名空间定义，PDF p. 35](../_source/pages/page-035.md)
- [底层子系统与端口定义，PDF p. 36](../_source/pages/page-036.md)

### 资源发现与目录
- [底层命名空间列表头与条目格式，PDF pp. 368-369](../_source/pages/page-368.md)
- [端口列表与 Fabrics 传输条目格式，PDF pp. 369-370](../_source/pages/page-369.md)

### 子系统管理命令
- [清除与创建导出 NVM 子系统命令，PDF pp. 448-449](../_source/pages/page-448.md)
- [导出子系统管理与删除控制，PDF pp. 462-463](../_source/pages/page-462.md)

### 命名空间管理
- [导出命名空间的关联与解除关联，PDF pp. 460-462](../_source/pages/page-460.md)

### 访问控制
- [访问模式与允许主机列表的修改，PDF pp. 463-466](../_source/pages/page-463.md)

### 端口管理
- [导出端口的创建与删除，PDF pp. 467-469](../_source/pages/page-467.md)
