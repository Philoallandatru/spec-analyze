# 命名空间管理生命周期（Namespace Management Lifecycle）

命名空间管理（Namespace Management）机制将命名空间的生命周期划分为两个独立的管理层次：

1. **子系统级别的资源分配**：通过 Create 和 Delete 操作在整个 NVMe 子系统（Subsystem）范围内管理命名空间的分配和回收
2. **控制器级别的挂接管理**：通过 Attach 和 Detach 操作控制单个控制器（Controller）对命名空间的访问权限

这种设计的核心特点是：
- Create 操作仅分配命名空间资源，并不会自动将其挂接到任何控制器
- Attach/Detach 操作的效果会持久化保存,能够穿越控制器重置（Reset）
- Delete 操作会自动移除所有控制器的挂接关系，并回收分配的资源

该能力主要面向设备制造商或系统管理员使用。[PDF pp. 383-387](../_source/pages/page-383.md)

## 概念模型

为了更好地理解命名空间管理的工作机制，我们可以从生命周期视角来看待：

```text
[未分配的 NSID]
       |
       | Namespace Management: Create（创建）
       v
[已分配但未挂接的命名空间]
       |          ^
  Attach|          |Detach
  （挂接）|          |（解挂）
       v          |
[在选定的控制器上激活]
       |
       | Namespace Management: Delete（删除）
       v
[已删除 + 从所有控制器解挂]
```

这个生命周期示意图清晰地展示了**分配**与**挂接**是两个独立的维度：
- Create 操作不会自动执行 Attach
- Delete 操作会移除命名空间，因此会自动从所有控制器解挂

[PDF pp. 383-386](../_source/pages/page-383.md)

## 挂接机制的约定

### 持久化特性

Attach 和 Detach 操作使用 4 KiB 的控制器列表（Controller List）作为命令数据，并具有以下持久化特性：
- **穿越所有 Reset 事件**：无论是控制器重置还是子系统重置，挂接状态都不会丢失
- **不受虚拟化管理操作影响**：即使将 Secondary 控制器设为 Offline，挂接状态也会保持

[PDF pp. 383-385](../_source/pages/page-383.md)

### 挂接目标要求

挂接目标必须是一个 I/O 控制器，并且该控制器必须：
1. 支持该命名空间的 I/O 命令集（I/O Command Set）
2. 已启用该 I/O 命令集（通过配置或命令集配置文件）

### 挂接操作的限制与错误

下表列出了执行 Attach/Detach 操作时可能遇到的限制条件及对应的错误状态：

| 限制条件 | 返回的错误状态 |
|----------|----------------|
| 域（Domain）级别的总挂接数超过 `MAXDNA` | Namespace Attachment Limit Exceeded |
| 目标控制器的挂接数超过 `MAXCNA` | Namespace Attachment Limit Exceeded |
| 命令集不受支持或配置文件未启用 | I/O Command Set Not Supported / Not Enabled |
| 私有命名空间已挂接到其他位置 | Namespace Is Private |
| 目标包含管理控制器或列表格式错误 | Controller List Invalid |
| 挂接会产生不允许的 ANA 条件 | ANA Attach Failed |

[PDF pp. 384-385](../_source/pages/page-384.md)

### 错误处理机制

当处理控制器列表时：
- 遇到第一个失败的条目时，处理立即停止
- 错误信息日志（Error Information Log）会记录失败条目的字节偏移量
- 这意味着列表中较早的条目可能已经被成功处理

[PDF pp. 384-385](../_source/pages/page-384.md)

## 创建与删除机制的约定

### 能力要求

- 如果控制器支持 Namespace Management，则必然也支持 Namespace Attachment
- 在导出的 NVM 子系统（Exported NVM Subsystem）中，控制器**禁止**支持 Namespace Management
- I/O 命令在管理操作进行期间会继续执行（非阻塞式设计）

[PDF p. 385](../_source/pages/page-385.md)

### Create 操作的参数

Create 操作使用以下参数：

| 参数 | 说明 |
|------|------|
| `NSID` | 必须设置为 `0`（由控制器选择并返回新的 NSID） |
| Command Set Identifier | 选择要使用的 I/O 命令集 |
| 512 字节参数区域 | 命令集特定的参数 |
| Vendor Data（可选） | 厂商自定义数据 |

命令完成时，控制器会选择并返回新分配的 NSID。

[PDF pp. 385-387](../_source/pages/page-385.md)

### Delete 操作的行为

Delete 操作的参数设置：

| NSID 值 | 操作行为 |
|---------|----------|
| 具体的 NSID | 删除指定的一个命名空间 |
| `FFFFFFFFh` | 删除所有命名空间（即使不存在命名空间也会成功） |

### Delete 操作的副作用

执行 Delete 时会产生以下副作用：
1. **隐式解挂**：命名空间会自动从所有控制器解挂
2. **事件通知**：
   - 如果命名空间仍然挂接着，会生成 attached-namespace 变更事件
   - 每次成功的删除都会生成 allocated-namespace 变更报告

**最佳实践**：主机应当先手动执行 Detach，再执行 Delete，以避免不必要的事件通知。

[PDF p. 385](../_source/pages/page-385.md)

### Create 操作的失败原因

Create 操作可能因以下原因失败：

| 失败原因 | 说明 |
|----------|------|
| 不支持的格式 | 请求的格式参数超出控制器能力 |
| 容量不足 | 可用容量无法满足请求 |
| NSID 资源耗尽 | 已分配的命名空间数量达到上限 |
| 不支持精简配置 | 请求了精简配置但控制器不支持 |
| 无效的 ANA 组 | ANA Group 标识符无效 |
| 不支持的 I/O 命令集 | 请求的命令集不受支持 |

**容量不足时的额外信息**：错误信息日志会报告还需要多少未分配字节才能满足请求。

[PDF pp. 386-387](../_source/pages/page-386.md)

## 相关概念

### 核心关联

- **[Namespace Identifiers](namespace-identifiers.md)**：区分了分配（allocation）、挂接（attachment）和持久命名空间标识（durable namespace identity）三个概念。[PDF pp. 97-99](../_source/pages/page-097.md)

- **[NVM Capacity Model](nvm-capacity-model.md)**：提供了命名空间创建所消耗的未分配容量。[PDF pp. 140-143](../_source/pages/page-140.md)

- **[Namespace Topology and Change Logs](namespace-topology-and-change-logs.md)**：向主机暴露 attach/detach/delete 等变更信息。[PDF pp. 288-293](../_source/pages/page-288.md)

## 规范引用

以下是本文档引用的 NVMe 规范页面：

- [Namespace Attachment 持久化特性与命令封装，PDF p. 383](../_source/pages/page-383.md)
- [Attachment 限制、命令集检查与首错误行为，PDF p. 384](../_source/pages/page-384.md)
- [Attachment 状态码与 Namespace Management 生命周期，PDF p. 385](../_source/pages/page-385.md)
- [Create 数据结构与失败信息，PDF p. 386](../_source/pages/page-386.md)
- [创建后返回的 NSID 完成值，PDF p. 387](../_source/pages/page-387.md)
