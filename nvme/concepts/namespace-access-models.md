# 命名空间访问模型（Namespace Access Models）

NVMe 命名空间按“被哪些控制器挂载、被多少主机并发访问”划分为**私有（Private）**、**共享（Shared）**和**分散（Dispersed）**三种访问模型，并与**多路径 I/O（Multi-Path I/O）**共同构成访问维度。[PDF pp. 30, 33-35](../_source/pages/page-030.md)

> **类比**：把三种模型想成三种租房方式——**私有**是“一室一厅整租”（同一时刻只给一个控制器用）；**共享**是“同一栋楼内合租”（同一 NVM 子系统下多个控制器并发挂载）；**分散**是“跨城市连锁公寓的同一逻辑房间”（跨越 2 个及以上参与子系统并发访问）。而**多路径 I/O**则是“同一个租户通过两扇不同的门进同一套房”。

## 核心访问模型与维度

```text
私有命名空间 (Private)   ──→ 同一时刻最多挂接到 1 个控制器
共享命名空间 (Shared)    ──→ 同一 NVM 子系统内，2 个及以上控制器并发挂接
分散命名空间 (Dispersed) ──→ 跨越 2 个及以上参与子系统，由其控制器并发访问

独立维度对比：
  多路径 I/O (Multi-Path I/O) : 1 台主机 ──(2+ 条独立路径)──→ 1 个命名空间
  命名空间共享 (Sharing)      : 2+ 台主机 ──(不同控制器)────→ 1 个共享命名空间
```

| 术语 | 作用边界与核心契约 |
|---|---|
| **私有命名空间（Private）** | 同一时刻最多被 1 个控制器挂载；同一 NSID 在不同控制器上可能指向各自独立的私有命名空间。[PDF pp. 34-35](../_source/pages/page-034.md) |
| **共享命名空间（Shared）** | 在同一 NVM 子系统内被 2 个及以上控制器并发挂载；支持一主机多路径，也支持多主机共享。[PDF pp. 34-35, 54](../_source/pages/page-034.md) |
| **分散命名空间（Dispersed）** | 通过 2 个及以上**参与的 NVM 子系统（Participating NVM Subsystems）**内的控制器并发访问；并非“每个子系统各一份独立拷贝”。[PDF pp. 30, 33](../_source/pages/page-030.md) |
| **指定命名空间（Specified）** | 由提交队列条目（SQE）中 `NSID` 字段显式选中的目标命名空间。[PDF p. 33](../_source/pages/page-033.md) |

## 关键规则与并发行为

1. **NSID 一致性与全局身份**：
   - `NSID` 是控制器相对的（Controller-relative）命令句柄。
   - 对于能访问同一**共享命名空间**的所有控制器，其 `NSID` **必须相同**，且 `Identify Namespace` 返回内容完全一致。[PDF p. 55](../_source/pages/page-055.md)
   - 主机软件应使用全局唯一标识（如 UUID / NGUID / EUI64），而非仅靠 NSID，来识别通往同一命名空间的多条路径。[PDF p. 55](../_source/pages/page-055.md)

```text
Controller A ── NSID 2 ──┐
                         ├──→ 同一共享命名空间（全局标识 G）
Controller B ── NSID 2 ──┘
  (不同路径必须报告相同的 NSID 与 Identify Namespace 数据)
```

2. **并发原子性与跨主机协调**：
   - 多路径 I/O 与命名空间共享均要求子系统内至少有 2 个控制器。[PDF p. 54](../_source/pages/page-054.md)
   - 多个控制器可并发操作同一共享命名空间，但**原子性仅以“接收命令的单个控制器”为边界**，不跨控制器保证；跨控制器命令的顺序及多主机并发协调由主机软件或应用负责（不在基础规范范围内）。[PDF pp. 54-56](../_source/pages/page-054.md)
3. **端口隔离与 SR-IOV**：
   - 独立端口（Independent Port）的复位只影响该端口上的控制器，不影响其他控制器、共享命名空间及其他控制器正在进行的 I/O。[PDF pp. 56-57](../_source/pages/page-056.md)
   - **SR-IOV（单根 I/O 虚拟化）**是同一模型的另一种实现：每个 PF（物理功能）或 VF（虚拟功能）即为一个独立控制器，可跨它们共享同一命名空间。[PDF pp. 56-57](../_source/pages/page-056.md)

## 非对称命名空间访问（ANA）与发现日志

当子系统对不同控制器呈现出不同的访问性能或可达性时，可通过 **ANA（Asymmetric Namespace Access Reporting）** 向上层报告“控制器与 ANA 组”之间的相对关系（而非命名空间的全局属性）。[PDF pp. 57, 497-500](../_source/pages/page-057.md)

- **组约束**：支持 ANA 的控制器上，每个挂载的命名空间都属于一个有效的 ANA 组；同组命名空间共享状态转移，且必须位于同一个 Domain 内。分散命名空间在各参与子系统内分属各自的 ANA 组（AGID 不必跨子系统相同）。[PDF pp. 497-500](../_source/pages/page-497.md)

```text
                 内部/路径重配置
 [Optimized] <----------------------> [Non-Optimized]
      |                                      |
      +-------------> [Change] <-------------+
                         |
                         v
                  [Inaccessible] ---- 恢复 ----> 可访问状态
                         |
                         `----> [Persistent Loss] (终态)
```

| ANA 状态 | 访问契约与主机应对 |
|---|---|
| **Optimized（最优）** | 所有支持的操作均以最优路径执行（强制可报告状态）。[PDF pp. 500-502](../_source/pages/page-500.md) |
| **Non-Optimized（非最优）** | 所有支持的操作可执行，但延迟更高或效率更低。[PDF pp. 500-502](../_source/pages/page-500.md) |
| **Inaccessible（不可访问）** | 用户数据不可用，多数命令中止；可能恢复，Identify 容量字段可能清零。主机可切换其他路径或按 `ANATT` 秒数在原路径重试。[PDF pp. 500-502](../_source/pages/page-500.md) |
| **Persistent Loss（持久丢失）** | 用户数据持久不可用；该控制器/组关系不再转移（终态），Identify 容量字段可能清零。[PDF pp. 500-502](../_source/pages/page-500.md) |
| **Change（状态转移中）** | 状态正在转移；多数命令中止并返回 `Asymmetric Access Transition`；主机可切换路径或按 `ANATT` 重试。[PDF pp. 500-502](../_source/pages/page-500.md) |

### ANA 日志布局与通知规则

```text
ANA Log Header: 全局 change count（CLR 后从 0 开始，绕回 0）
  └── ANA Group Descriptors（按 AGID 升序）
        ├── 组级 change count（可选：1 开始并绕回 1；0 表示不报告）
        ├── 该控制器对该组的 ANA 状态
        └── 挂载的 NSID 列表（升序；RGO=1 时省略）
```

- **过滤与分段读取**：ANA 日志仅包含“有命名空间挂载到当前控制器”的组。当描述符长度可变且支持索引偏移时，主机应按索引偏移读取（索引 0 为头部，1 为首个描述符），并在分段读完后重读头部校验全局 `change count` 一致性。[PDF pp. 250-252](../_source/pages/page-250.md)
- **ANA Change Notice**：启用后，控制器在 AGID 变化、转移失败或进入上述状态后发送通知（但由命名空间挂载直接引起的状态进入除外）。[PDF p. 502](../_source/pages/page-502.md)
- **Admin 命令行为**：ANA 不会一刀切禁用所有 Admin 命令。不依赖命名空间与特定 I/O 命令集的 Admin 操作通常仍可用，但 `Get/Set Features`、`Get Log Page`、`Identify` 存在例外（如容量清零、特定日志或广播 Set Features 失败）。[PDF pp. 502-503](../_source/pages/page-502.md)

## 实践选型与常见误区

- **典型选型**：单机本地盘选用**私有**；双控高可用阵列（如 ESXi 多路径连接）选用**共享 + ANA Reporting**；跨子系统多活架构选用**分散**。
- **常见误区**：
  - 误以为“多控制器 = 多路径”：多路径专指**同一主机**通过多条路径访问同一命名空间；多主机经不同控制器访问属于**共享**。
  - 仅凭 `NSID` 识别多路径设备：跨路径识别同一物理命名空间必须依赖 UUID/NGUID/EUI64 等全局标识。

## 关联与引申概念

- **ANA（Asymmetric Namespace Access，非对称命名空间访问）**：描述不同控制器路径访问同一组命名空间时的性能与可达性差异，是多路径驱动选路和故障切换的核心依据。
- **SR-IOV（Single Root I/O Virtualization，单根 I/O 虚拟化）**：PCIe 虚拟化机制，允许单个物理设备虚拟出多个 VF，每个 VF 在 NVMe 中表现为一个独立控制器并可共享命名空间。
- **相关文档**：
  - [namespace.md](namespace.md) — 命名空间基础定义
  - [namespace-management-lifecycle.md](namespace-management-lifecycle.md) — 挂接（Attach）如何决定控制器可见性
  - [namespace-topology-and-change-logs.md](namespace-topology-and-change-logs.md) — 拓扑变更与通知日志
  - [dispersed-namespace-lifecycle.md](dispersed-namespace-lifecycle.md) — 分散命名空间生命周期细节

## 规范依据

- [Dispersed namespace 定义，PDF p. 30](../_source/pages/page-030.md)
- [Namespace 与 NSID 关系，PDF p. 33](../_source/pages/page-033.md)
- [Private / Shared / Specified 命名空间，PDF pp. 34-35](../_source/pages/page-034.md)
- [多路径 I/O、并发、端口隔离与 SR-IOV，PDF pp. 54-57](../_source/pages/page-054.md)
- [ANA 发现日志与 change count，PDF pp. 250-252](../_source/pages/page-250.md)
- [ANA 状态机、ANATT 重试与通知，PDF pp. 497-503](../_source/pages/page-497.md)
