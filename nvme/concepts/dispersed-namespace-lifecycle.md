# 分散命名空间生命周期（Dispersed Namespace Lifecycle）

## 一句话说明

分散命名空间（Dispersed Namespace）是把"同一个逻辑命名空间"横跨**两个或更多 NVM 子系统**来暴露，主机透过全局标识符（NGUID/UUID）以及 ANA（Asymmetric Namespace Access）状态把它们看成同一卷，从而支持在线迁移、跨子系统复制和多路径并发。

## 生活化类比

把它想成**同一本书在两家连锁书店都能买到**：

- **每家书店的"书"** = 各参与子系统中各自的"本地命名空间"（NSID 不同）
- **ISBN 编号** = NGUID 或 UUID；任何一家书店都标着同一个 ISBN
- **读者（主机）** 看到的是同一本书，可以从任意一家买
- **两家书店偶尔会调货**（数据复制 / 在线迁移），但 ISBN 始终不变
- **某家书店临时断货**（控制器失效）= 对应路径 `ANA=inaccessible`；读者改去另一家买

NVMe Base 规范不定义"如何调货"，它只规定"两家书店必须使用同一个 ISBN，并且让读者知道每家店的库存状态"。

## 工作流程

```text
                  稳定的全局身份 (NGUID / UUID)
                          |
                          v
  +------------------+         +------------------+
  | NVM Subsystem A  |         | NVM Subsystem B  |
  |------------------|         |------------------|
  | NSID = x (本地)  |  <==>   | NSID = y (本地)  |
  | ANA state        |         | ANA state        |
  +------------------+         +------------------+
        ^                             ^
        | 主机可通过任一子系统的控制器访问         |
        +------------------+----------+
                           |
                      [主机视角: 1 个逻辑 NS]
```

**生命周期阶段**：

| 阶段 | 关键操作 | 注意点 |
|------|---------|--------|
| 创建 | Namespace Management 命令 | 必须带全局标识符（NGUID/UUID）|
| 挂载 | 各自 Namespace Attachment | 可独立挂/卸到不同控制器 |
| 共享 → 分散 | 通过控制面切换 | 上报"Namespace Attribute Changed" |
| 私有 → 分散 | 不允许 | 规范明文禁止 |
| 降级 | 解除部分参与方 | NSID 与全局 ID 不变，可再扩展 |
| 删除 | 由处理方决定是否传播 | 其他参与方不受强制同步 |

## 初学者案例

**场景：把生产库从旧阵列不中断地搬到新阵列。**

1. 旧阵列在子系统 A，NSID=10，NGUID=`0xA1B2C3...`。
2. 在新阵列子系统 B 上创建一个空命名空间，NSID=42，**复用相同的 NGUID**。
3. 主机通过 Host Behavior Support 设 `HDISNS=1`，声明理解跨子系统。
4. 后台复制工具把数据从子系统 A 拷到子系统 B。
5. 主机把 B 的路径加入多路径，主机视角下依然是同一个 NGUID，ANA 状态：旧路径"非优化"，新路径"优化"。
6. 复制完成后，把 A 的路径降级 → 解除，最后删除 A 上的本地 NS。
7. 整个过程应用不停机、不重新挂载。

> 关键收获：分散命名空间不要求子系统之间有"内置复制协议"——它只确保双方对"这是同一卷"达成共识，复制由外部工具完成。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 全局标识符必须一致 | NGUID/UUID 在所有参与子系统中保持相同值 |
| NSID 仅本地有效 | 同一分散命名空间在不同子系统可有不同 NSID |
| 必须用 NGUID 或 UUID | EUI64 不得用于跨子系统关联 |
| 控制器必须支持命令集 | 每个提供访问的控制器都要支持该 NS 关联的 I/O 命令集 |
| 主机必须显式声明 | 不声明 `HDISNS=1` 时，参与方可禁止跨子系统访问 |
| 支持 ANA | 参与方应支持 ANA 特性以反映路径状态 |
| 私有 → 分散禁止 | 私有命名空间不可转换为分散命名空间 |
| 删除不强制传播 | 发起删除的子系统决定是否同步到其他参与方 |
| Reservation 需 `DISNSRS=1` | 在分散命名空间上做预留必须声明支持远端哨兵值 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 分散 NS vs 共享 NS | 共享 NS 在同一子系统内多控制器可见；分散 NS 跨多个子系统 |
| NGUID vs NSID | NSID 仅在子系统内有效；NGUID 是全局唯一 |
| HDISNS vs DISNSRS | `HDISNS` 声明主机支持分散 NS；`DISNSRS` 声明主机支持分散 NS 上的 Reservation 哨兵值 |
| CNTLID 本地 vs 远端 | 本地 CNTLID 是真实 ID；远端用 `FFFDh` 表示"在另一参与方" |
| 分散 vs 复制 | 规范只规定身份统一，不规定数据如何复制 |
| DISNSRS=0 在分散 NS 上失败 | 已在分散 NS 上启用 `HDISNS=1` 的主机若 Reservation 命令不设 `DISNSRS=1`，命令失败 |

## 进阶细节

- **能力发现路径**（规范 7.6.1）：
  1. `NMIC.DISNS` 位 → 控制器是否支持
  2. Dispersed Namespace Participating NVM Subsystems Log Page → 列出所有参与方
  3. Host Behavior Support 特性设 `HDISNS=1` → 主机声明理解
- **未声明支持时的策略**（规范 7.6.2）：参与方可任选：
  - 仅在主机通过单个子系统访问时允许
  - 禁止该分散 NS 的访问
  - Connect 时拒绝（连带屏蔽子系统内所有 NS）
  - 接受 Connect 但只屏蔽该分散 NS
- **失败命令类型**（规范 7.6.3）：当分散访问被禁止时，下列命令返回 "Host Dispersed Namespace Support Not Enabled"：
  - I/O 命令指向该分散 NS
  - 规范列出的特定管理命令
  - 作用域包含该分散 NS 的广播 Set Features
  - 宽作用域 Format NVM
- **Reservation 远端哨兵**（规范 7.6.4）：在 Reservation Report 中：
  - 本地 CNTLID 仍为真实 ID
  - 同一主机在远端参与方上的所有控制器合并为一条记录，CNTLID 填 `FFFDh`
- **ANA 协调**（规范 7.6.5）：使用全局标识符（NGUID/UUID）来跨参与方关联同一 NS 的不同路径，ANA 状态由各参与方独立维护。
- **生命周期事件**（规范 7.6.6）：分散/共享切换都会触发 "Namespace Attribute Changed" 异步事件。
- **数据一致性**：NVMe 规范不规定跨参与方的数据一致性模型；多数实现采用最终一致或主备复制。
- **驱动开发要点**：在多路径层识别"同一全局 ID 的多个本地 NS"是关键；ANA 状态变化应驱动 I/O 重路由。

## 规范依据

- [分散 NS 概念与典型场景，PDF 第 531 页](../_source/pages/page-531.md)
- [全局身份与本地 NSID 的区分，PDF 第 532 页](../_source/pages/page-532.md)
- [管理、转换与删除语义，PDF 第 534 页](../_source/pages/page-534.md)
- [能力发现与主机声明 HDISNS，PDF 第 535 页](../_source/pages/page-535.md)
- [访问禁止与命令约束，PDF 第 536 页](../_source/pages/page-536.md)
- [ANA 协调与 Reservation 远端哨兵，PDF 第 537 页](../_source/pages/page-537.md)

## 相关阅读

- [namespace-management-lifecycle.md](namespace-management-lifecycle.md) - 基础命名空间生命周期
- [namespace-identifiers.md](namespace-identifiers.md) - NGUID/UUID 的分配规则
- [namespace-topology-and-change-logs.md](namespace-topology-and-change-logs.md) - ANA 状态变更日志
- [controller-virtualization-resources.md](controller-virtualization-resources.md) - 导出资源的多控制器视角
