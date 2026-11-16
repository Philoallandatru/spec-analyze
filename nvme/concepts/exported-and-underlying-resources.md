# 导出资源与底层资源（Exported and Underlying NVM Resources）

NVMe 将实际提供存储能力的物理或虚拟资源（**底层资源**）与专门面向远程主机展示的逻辑视图（**导出资源**）严格分离：底层是数据事实，导出是远程访问的“门面”。两者拥有独立的生命周期，清空或删除导出配置不会影响底层存储数据。[PDF 第 30-31 页](../_source/pages/page-030.md)

## 心智模型：国内仓 vs 海外门店

- **底层资源（Underlying Resources）**：好比“国内总仓”，包含实际存数据的**底层命名空间**和接入传输的**底层端口**。[PDF 第 35-36 页](../_source/pages/page-035.md)
- **导出资源（Exported Resources）**：好比“海外门店”，包含**导出 NVM 子系统**、**导出命名空间**（门店货架编号）和**导出端口**（收件窗口），并可选维护一份**允许主机列表（Allowed Host List）**。
- **管理边界**：所有导出管理命令（如 `Manage Exported NVM Subsystem`、`Manage Exported Namespace`）**必须在底层子系统上执行**，不能在导出子系统上执行。

```text
底层 NVM 子系统（Underlying NVM Subsystem）
 ├── 底层命名空间 ──关联 (Associate)──> 导出命名空间 ──┐
 └── 底层端口     ──端口列表 (Ports)──> 导出端口     ──┤
                                                       v
                                          导出 NVM 子系统
                                           ├── 控制器 [0..*]
                                           ├── 导出命名空间 [0..*]
                                           ├── 导出端口 [0..*]
                                           └── 允许主机列表 [可选]
```

## 资源发现目录（Identify）

基于消息的控制器通过两个带版本计数器（Generation Counter）的 Identify 目录暴露可用的底层资源：[PDF 第 368-370 页](../_source/pages/page-368.md)

| Identify `CNS` | 目录名称 | 条目核心字段 |
|---|---|---|
| **`1Dh`** | 底层命名空间列表 | 底层 Subsystem NQN + NSID + 子系统内唯一的 Controller ID |
| **`1Eh`** | 底层端口列表 | Underlying Port ID + 传输地址、子类型、类型、地址族等 |

> **版本计数器规则**：在底层 NVM 子系统重置时清零，每次目录变更递增 1（全 1 后回绕到 0），主机可据此检测并发修改。

## 导出生命周期与依赖顺序

### 1. 导出子系统的创建与清除

- **创建（`Create Exported NVM Subsystem`）**：必须指定初始访问模式——**不受限（Unrestricted）**（允许任意主机连接）或**受限（Restricted）**（仅允许列表内主机）。新建后允许主机列表为空、无导出命名空间，命令返回新分配的 `SUBNQN`。[PDF 第 448-449 页](../_source/pages/page-448.md)
- **清除（`Clear Exported NVM Resource Configuration`）**：删除全部导出配置而不影响底层资源，但**前置条件是必须先断开所有导出子系统上的主机连接**，否则报 `Command Sequence Error`。[PDF 第 448 页](../_source/pages/page-448.md)
- **删除检查**：若仍存在活跃控制器连接、已关联导出命名空间或已创建导出端口，删除导出子系统会以 `Command Sequence Error` 失败。[PDF 第 462-463 页](../_source/pages/page-462.md)

### 2. 导出命名空间的关联与分离

导出命名空间的创建区分**存储关联（Association）**与**控制器附加（Attachment）**：[PDF 第 460-462 页](../_source/pages/page-460.md)

1. **关联（Associate）**：通过目标导出子系统 NQN 与底层三元组（Underlying Subsystem NQN + Controller ID + Active NSID）建立映射。前提是底层命名空间已分配且已附加到该控制器。关联后**不会**自动附加到导出控制器，需单独执行 Namespace Attachment；两处访问同一份用户数据。
2. **解除关联（Disassociate）**：必须遵循“先分离再删除”原则——先将导出命名空间从所有控制器 **Detach**，再解除关联并删除导出命名空间 ID（ENSID）。关联与解除均会触发 `Namespace Attribute Changed` 异步事件。

### 3. 访问控制与导出端口管理

- **访问模式与权限管理**：切换到 `Restricted` 模式或执行 `Revoke Access` 会**立即断开**不在允许列表中的已连接主机；`Grant Access` 绑定 `128-bit Host Identifier + HOSTNQN` 与 `SUBNQN + Underlying Port ID`，失败条目可在 Error Information Log 中按字节偏移定位。[PDF 第 463-466 页](../_source/pages/page-463.md)
- **导出端口（Exported Port）**：`Create Exported Port` 将导出子系统绑定到底层端口和传输服务标识符（端口 ID 可由主机指定或控制器生成）。`Delete Exported Port` 应在端口空闲时执行；**删除使用中的端口会终止所有关联且对未完成命令和 Fabric 资源产生未定义行为**。[PDF 第 467-469 页](../_source/pages/page-467.md)

## 易混概念：Allowed Host List vs Fabric Zoning

| 机制 | 作用层级 | 职责说明 |
|---|---|---|
| **Allowed Host List** | 子系统层（应用逻辑控制） | 导出 NVM 子系统的可选状态，控制哪些主机可连接该导出子系统 |
| **Fabric Zoning** | 网络/传输层（基础设施控制） | 由 CDC 集中维护，控制主机在 Discovery 阶段能看到哪些子系统端口及网络包过滤 |

两者互为独立，可叠加使用以提供多层安全防护。[PDF 第 31 页](../_source/pages/page-031.md)

## 关联与引申概念

- **Admin 命令模型**：导出资源管理命令（Create/Clear/Associate/Grant 等）的操作码框架，见 [Admin 命令模型](admin-command-model.md)。
- **控制器数据队列**：主机内存分配与 PRP/SGL 描述符用法，见 [控制器数据队列](controller-data-queues.md)。
- **域与分区（Domains & Divisions）**：底层与导出资源的故障域和共享状态边界，见 [域与分区](domains-and-divisions.md)。
- **Keep Alive**：Fabrics 传输下维持主机与导出控制器连接存活的机制，见 [Keep Alive](keep-alive.md)。

## 规范依据

- [导出资源与底层资源定义，PDF 第 30-31、35-36 页](../_source/pages/page-030.md)
- [底层命名空间列表与端口列表 Identify 目录，PDF 第 368-370 页](../_source/pages/page-368.md)
- [清除与创建导出 NVM 子系统命令，PDF 第 448-449 页](../_source/pages/page-448.md)
- [导出命名空间的关联、解除与删除控制，PDF 第 460-463 页](../_source/pages/page-460.md)
- [访问模式与允许主机列表的修改，PDF 第 463-466 页](../_source/pages/page-463.md)
- [导出端口的创建与删除，PDF 第 467-469 页](../_source/pages/page-467.md)
