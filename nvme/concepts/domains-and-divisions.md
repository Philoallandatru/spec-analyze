# 域（Domain）与分区（Division）

**域（Domain）** 是 NVM 子系统中“共享状态”（如电源状态、容量信息等）的最小不可分割单元。在多域子系统中，域与域之间的边界同时是**通信边界、故障边界和管理边界**；而 **分区（Division）** 是指破坏域间通信的事件或动作（如节点掉线、链路中断或运维隔离），会导致子系统局部化甚至命名空间访问受限。[PDF 第 104-105 页](../_source/pages/page-104.md)

## 心智模型：双路供电的数据中心

把多域子系统想成一座双路供电的数据中心：每个机房集群是一个独立的 Domain，平时通过联络线协同对外呈现为同一台存储设备；一旦联络线断开（发生 **Division**），两边只能看到各自的本地状态，全局操作（如 `TNVMCAP` 容量统计、Sanitize、Format、SMART）都会受到波及。

```text
                 一个 NVM 子系统
+---------------------+   +---------------------+
| Domain 1            |   | Domain 2            |
| 控制器: 0..*        |<->| 控制器: 0..*        |
| 耐久度组: 0..*      |   | 耐久度组: 0..*      |
+---------------------+   +---------------------+
           通信 / 故障 / 管理边界
                       ×  分区 (Division)
```

> 一个域可以只装控制器、只装存储（耐久度组 Endurance Group）、两者都有或两者都没有；命名空间可以跨域共享。[PDF 第 105-106 页](../_source/pages/page-105.md)

## 多域与单域契约

| 场景 | 必须满足的规范条件 |
|---|---|
| **单域子系统** | 最常见的简化实现；所有 Domain Identifier 字段全部为 `0` |
| **多域子系统** | 所有控制器必须置位 `CTRATT.MDS = 1`；控制器和支持的耐久度组的 Domain Identifier 均**非零**且在子系统内唯一 |
| **多域强制特性** | 多域子系统**必须**支持非对称命名空间访问报告（ANA），以便在分区时向主机通告路径可达性 |

- **主机侧用途**：主机软件通过 `CTRATT.MDS` 判断多域支持，并利用 Domain Identifier 识别哪些控制器或耐久度组共享同一个故障域，用于故障定位与高可用部署。[PDF 第 106-107 页](../_source/pages/page-106.md)

## 分区后的持久预留（Persistent Reservation）恢复

当多域子系统从分区（Division）中恢复通信时，必须同步各域之间的持久预留状态；否则主机可能读到不一致的锁状态：[PDF 第 106 页](../_source/pages/page-106.md)

```text
[已分区 Divided]
       |  域间通信恢复
       v
[同步 Persistent Reservation 状态]
       |                                  |
       | 同步成功                         | 无法同步
       v                                  +--> [ANA Persistent Loss 永久丢失]
[ANA Accessible 可访问]                   `--> [控制器停止处理命令，置 CSTS.CFS=1]
       ^
       | (同步完成前保持)
[ANA Inaccessible 不可访问]
```

- **排错提示**：多域环境下遇到“控制器在线但 I/O 报 `ANA Inaccessible`”，通常是发生了分区或正在等待持久预留同步；若控制器置 `CSTS.CFS = 1`，则表明控制器已自我宣告致命状态（Controller Fatal Status）。

## 关联与引申概念

- **耐久度组（Endurance Group）**：管理存储介质磨损寿命的边界，可归属于某个 Domain，Domain 则划定共享状态与故障边界。
- **通信丢失与命令重试**：分区恢复与传输通信丢失时的重试策略，见 [通信丢失与命令重试](communication-loss-and-command-retry.md)。
- **控制器使能、关机与复位**：关机或复位动作在所属 Domain 范围内生效，见 [控制器使能、关机与复位](controller-enable-shutdown-reset.md)。
- **命令与特性锁定**：子系统级安全锁定需要在域内强制执行，见 [命令与特性锁定](command-and-feature-lockdown.md)。
- **导出资源与底层资源**：远程导出视图与底层物理资源的域归属关系，见 [导出资源与底层资源](exported-and-underlying-resources.md)。

## 规范依据

- [Domain 与 Division 语义，PDF 第 104 页](../_source/pages/page-104.md)
- [组成、标识符与拓扑示例，PDF 第 105 页](../_source/pages/page-105.md)
- [Persistent Reservation 同步与标识符使用，PDF 第 106-107 页](../_source/pages/page-106.md)
