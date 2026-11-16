# 分散命名空间生命周期（Dispersed Namespace Lifecycle）

分散命名空间（Dispersed Namespace）允许主机（Host）通过**两个或多个 NVM 子系统**中的控制器同时访问同一个逻辑命名空间。主机依靠全局唯一的标识符（NGUID 或 UUID）和非对称命名空间访问（ANA）状态，将分布在不同子系统上的本地命名空间实例识别为同一块盘，从而实现无中断在线迁移、跨子系统复制和多路径高可用。[PDF 第 531-534 页](../_source/pages/page-531.md)

## 心智模型：同一本书的多家连锁店

- **各参与子系统的本地命名空间** = 不同分店里的同一本书，各自有自己的货架号（本地 `NSID` 可以不同）。
- **NGUID / UUID** = 书的 ISBN 编号，在所有参与子系统中**必须完全一致**，主机凭它认出“这是同一卷”。
- **ANA 状态** = 每家店的库存与路线状态（优化、非优化、不可访问），指导主机多路径软件把 I/O 路由到最优子系统。
- **Base 规范边界**：规范只保证身份统一与路径协调，**不规定**子系统之间如何同步/复制数据（由厂商或上层工具自行实现）。

```text
                  稳定的全局身份（相同 NGUID 或 UUID）
                                  |
   +------------------+           v           +------------------+
   | NVM 子系统 A     | <===================> | NVM 子系统 B     |
   | NSID = x (本地)  |       源 / 副本       | NSID = y (本地)  |
   | ANA 状态         |                       | ANA 状态         |
   +------------------+                       +------------------+
            ^                                          ^
            +--------------- 主机多路径 ---------------+
                       （视为 1 个逻辑命名空间）
```

## 身份标识与生命周期规则

### 1. 标识符规则

| 标识符 | 跨子系统可用性 | 规则与说明 |
|---|---|---|
| **NGUID** | ✅ 最高优先级 | 若存在，必须优先用于跨子系统关联 |
| **UUID** | ✅ 次优 | 当 NGUID 不存在时使用 |
| **EUI64** | ❌ **禁止** | **绝不能**作为分散命名空间的跨子系统标识符 |
| **NSID** | ❌ 仅本地有效 | 同一子系统内各控制器一致，但跨子系统可以不同 |

### 2. 生命周期操作约束

- **创建与挂载**：通过 Namespace Management 创建支持跨子系统访问的命名空间，并通过 Namespace Attachment 独立挂/卸到各控制器。[PDF 第 534 页](../_source/pages/page-534.md)
- **转换规则**：**共享命名空间（Shared）可以转为分散命名空间**；**私有命名空间（Private）禁止直接转换**。任何转换都会触发 `Namespace Attribute Changed` 异步事件。
- **单参与方降级**：可解除部分参与方使分散命名空间退化为单子系统访问，保留原有 NSID 与全局 ID 以便日后重新扩展。
- **删除操作**：由接收删除命令的参与子系统执行，**不强制**传播到其他参与方，删除前需评估对全局视图的影响。

## 能力发现与主机声明（`HDISNS`）

主机与控制器必须双向确认对分散命名空间的理解：[PDF 第 535-537 页](../_source/pages/page-535.md)

1. **主机发现**：通过 `NMIC.DISNS` 确认控制器支持分散命名空间，并读取 `Dispersed Namespace Participating NVM Subsystems` 日志页获取所有参与子系统列表。
2. **主机声明（`HDISNS`）**：主机通过 Host Behavior Support 特性将 `HDISNS` 设为 `1h`，承诺使用全局唯一的 Host Identifier 并将同 NGUID/UUID 的命名空间合并为单一逻辑卷。
3. **未声明（`HDISNS=0`）的惩罚**：参与方可限制仅单子系统访问、屏蔽该分散命名空间、或在主机连入第二个参与子系统时直接拒绝 `Connect`。若访问被禁止，发往该命名空间的 I/O 命令、部分管理命令、包含该命名空间的广播 `Set Features` 和宽作用域 `Format NVM` 均会失败，返回 `Host Dispersed Namespace Support Not Enabled`。

## ANA 多路径与 Reservation 协调

- **ANA 多路径**：所有暴露同一分散命名空间的参与方应支持 ANA（Asymmetric Namespace Access），以便主机在在线迁移或链路故障时动态切换 I/O 路径。[PDF 第 537 页](../_source/pages/page-537.md)
- **Reservation 远端哨兵（`FFFDh`）**：跨子系统执行持久预留（Reservation）时，本地控制器在 `Reservation Report` 中显示真实 `CNTLID`，而同一主机在远端参与方上的所有控制器会被合并为一条记录，使用哨兵值 **`CNTLID = FFFDh`**。主机必须在 Acquire/Register/Release/Report 命令中设置 **`DISNSRS = 1`**（声明支持分散预留哨兵）；若主机已开启 `HDISNS=1` 但 `DISNSRS=0`，预留命令将失败并返回 `Namespace Is Dispersed`。[PDF 第 537-538 页](../_source/pages/page-537.md)

## 关联与引申概念

- **基础命名空间生命周期**：创建、删除、挂载与容量管理的基础规则，见 [命名空间管理生命周期](namespace-management-lifecycle.md)。
- **命名空间标识符**：NGUID、UUID、EUI64 与 NSID 的格式与分配约定，见 [命名空间标识符](namespace-identifiers.md)。
- **ANA 与拓扑变更日志**：非对称命名空间访问状态与属性变更事件，见 [命名空间拓扑与变更日志](namespace-topology-and-change-logs.md)。
- **控制器虚拟化资源**：多控制器与导出资源的资源视图，见 [控制器虚拟化资源](controller-virtualization-resources.md)。

## 规范依据

- [在线迁移与复制场景，PDF 第 531-534 页](../_source/pages/page-531.md)
- [管理、转换与删除语义，PDF 第 534 页](../_source/pages/page-534.md)
- [全局标识符与发现边界，PDF 第 534-535 页](../_source/pages/page-534.md)
- [主机访问禁止与命令约束，PDF 第 536-537 页](../_source/pages/page-536.md)
- [ANA 与 Reservation 协调，PDF 第 537-538 页](../_source/pages/page-537.md)
