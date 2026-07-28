# 域与分区（Domains and Divisions）

## 一句话说明

域（Domain）是子系统内"共享状态"（电源、容量信息等）的**最小不可分割单元**；在多域子系统中，域之间的边界同时是通信边界、故障边界与管理边界；而"分区（Division）"就是这些边界上发生的中断或动作。

## 生活化类比

把多域 NVMe 子系统想成一座**双路供电的数据中心**：

- **市电 A 路 + 服务器集群 A** = 一个 Domain（共享同一路市电、同一组开关）
- **市电 B 路 + 服务器集群 B** = 另一个 Domain
- **两路之间的联络线** = 域间通信链路

正常时两路互相备份、共同对外服务。一旦其中一路掉电或联络线中断（**分区 division**），每边只能看到自己的本地状态，全局视图（如 `TNVMCAP`、SMART、Sanitize、Format）就不一致了——数据中心进入"分区运行模式"。

## 工作流程

```text
                 一个 NVM 子系统
   +---------------------+   +---------------------+
   | Domain 1            |   | Domain 2            |
   | 控制器 0..*         |<->| 控制器 0..*         |
   | 耐久度组 0..*       |   | 耐久度组 0..*       |
   +---------------------+   +---------------------+
              通信 / 故障 / 管理边界
                          ×  分区 (division)
```

简化说明：一个域可"只装控制器"、"只装存储"、"两者皆有"或"两者皆无"；命名空间可跨域共享。

## 初学者案例

**场景：多域 NVMe-oF 集群中一台存储节点掉线，业务报"部分命名空间不可达"。**

1. 主机通过 ANA（Asymmetric Namespace Access Reporting）读到受影响命名空间进入 `ANA Inaccessible` 状态。
2. 管理员查子系统的两个 Domain：一个仍在线，另一个心跳丢失——这就是发生了"分区"。
3. 节点恢复后，控制器尝试同步各命名空间的 Persistent Reservation 状态：
   - 同步成功 → 命名空间切回 `ANA Accessible`，业务恢复；
   - 同步失败 → 切到 `ANA Persistent Loss`，或控制器直接置 `CSTS.CFS=1` 停止处理命令。
4. 故障速查：多域场景下"看到控制器在线但 I/O 报 inaccessible"→ 大概率是 division 导致 reservation 状态未同步，先排查链路与对端 Domain 状态。
5. 主机在多域环境下应使用 `CTRATT.MDS` 位与 Domain Identifier 区分"哪个控制器属哪个故障域"，便于故障定位。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| Domain 的本质 | 共享状态（电源、容量信息等）的最小不可分割单元 |
| 单域最常见 | "单域子系统"是 NVMe 子系统最常见的简化实现 |
| 多域强制项 | 多域子系统**必须**支持 Asymmetric Namespace Access Reporting（ANA） |
| 域标识符作用域 | Domain Identifier 在 NVM 子系统内唯一 |
| 单域零值 | 单域子系统中所有 Domain Identifier 字段为 0 |
| 多域非零 | 多域时所有控制器需设置 `CTRATT.MDS`，控制器 / 耐久度组的 Domain Identifier 均非 0 |
| 边界性质 | 域之间同时是通信、故障、管理边界；多域要"合作"才能对外表现为同一子系统 |
| 分区的影响 | 全局状态可能"局部化"；命名空间访问、子系统级操作（`TNVMCAP`、Sanitize、Format、SMART）都可能被波及 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Domain vs Endurance Group | Domain 划"共享状态"边界；EG 划"寿命管理"边界 |
| Domain vs NVM Subsystem | 子系统是"整台设备"边界；Domain 是子系统内"小边界" |
| Division vs Controller Reset | Division 是域间通信中断；Controller Reset 是单控制器重启 |
| ANA Inaccessible vs ANA Persistent Loss | 前者是临时不可达（reservation 同步期间）；后者是永久丢失 |
| `CSTS.CFS=1` vs 控制器停止响应 | CFS=1 是"控制器已自我宣告致命状态"；停止响应可能是命令没下到控制器 |

## 进阶细节

- **规范 3.2.5.1 概览**（PDF 第 104 页）：子系统可由 1 个或多个 Domain 组成；多域**必须**支持 ANA（规范 8.1.1）；最常见实现是单域；每个域独立，域间是"通信/故障/管理"边界，多域需要相互合作。
- **Division 定义**（规范 3.2.5.1 / PDF 第 104 页）：是"子系统内事件或动作"（如某域失效、运维动作、重新配置）影响域间通信；存在 division 时全局状态可能局部化（控制器只能看到它能通信的域）；divison 可能影响命名空间访问（规范 8.1.1）以及子系统范围操作（`TNVMCAP`、Sanitize、Format、SMART）。
- **Figure 71-72 拓扑示例**（规范 3.2.5.1 / PDF 第 105-106 页）：
  - Domain 可只装控制器 / 只装存储 / 两者皆有 / 两者皆无；
  - 命名空间可跨域共享；
  - 通信边界用虚线表示（Figure 72 的 Key）。
- **Domain Identifier 使用**（规范 3.2.5.3 / PDF 第 106-107 页）：标识符表明"哪部分资源属于同一域"；主机据此判断哪些 EG 在同一域、哪些跨域；典型用途包括运维分组、故障域感知放置、报告聚合。
- **多域表示**（规范 3.2.5.1 / PDF 第 105 页）：所有控制器置 `CTRATT.MDS=1`；控制器与 EG 的 Domain Identifier 都非 0；单域时相关字段全 0。
- **Persistent Reservation 与分区**（规范 3.2.5.2 / PDF 第 106 页）：
  - 多域 + Persistent Reservation（规范 8.1.22）场景，分区恢复（恢复运行 / 恢复通信）要求把"已不再分区"的所有域内的 reservation 状态**同步**。
  - 若某 NS 的 reservation 状态未同步 → 该 NS 所在 ANA Group 进入 `ANA Inaccessible`（规范 8.1.1.6）并保持，直到同步完成。
  - 若 reservation 状态**永远无法同步**，则：
    - 进入 `ANA Persistent Loss`（规范 8.1.1.7）按其规则处理命令；或
    - 控制器停止处理命令并置 `CSTS.CFS=1`（规范 9.5）。
- **主机处理建议**（规范 3.2.5.3 Informative）：
  - 读 `CTRATT.MDS` 判定多域支持；
  - 用 Domain Identifier 把控制器 / EG 归到同一故障域；
  - 接收 ANA 变化通知时，将"未同步 reservation 的命名空间"标记为不可达；
  - 同一域内的 EG 出现 capacity / SMART 偏差时，优先怀疑域内故障而非跨域网络。
- **典型恢复状态机**（基于 Figure 71 解释性重构）：

```text
[Divided]
   |
   | 通信恢复
   v
[同步 Persistent Reservation 状态]
   | 成功                          | 无法同步
   v                              +--> [ANA Persistent Loss]
[Accessible]                      `--> [控制器停止处理命令，置 CSTS.CFS=1]
   ^
   |
[ANA Inaccessible while 未同步]
```

- **CSTS 关键位**（规范 3.1.5 / 9.5）：`CSTS.CFS=1` 表明控制器致命状态；主机在多域下应监控 CFS 与 ANA 变化以区分"控制器死亡"与"分区未同步"。

## 规范依据

- [Domain 与 Division 语义，PDF 第 104 页](../_source/pages/page-104.md)
- [组成、标识符与拓扑示例，PDF 第 105 页](../_source/pages/page-105.md)
- [Reservation 同步与标识符使用，PDF 第 106 页](../_source/pages/page-106.md)

## 相关阅读

- [communication-loss-and-command-retry.md](communication-loss-and-command-retry.md) - 分区恢复与通信丢失协同处理
- [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) - 关机/重置在 Domain 范围内生效
- [command-and-feature-lockdown.md](command-and-feature-lockdown.md) - 子系统级锁定强制在域内执行
- [exported-and-underlying-resources.md](exported-and-underlying-resources.md) - 资源分层与远程视图的域归属
