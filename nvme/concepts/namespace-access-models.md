# 命名空间访问模型（Namespace Access Models）

## 一句话说明

NVMe 命名空间按"被谁看见、谁并发访问"分为三种模型：私有（同子系统中同一时刻最多一个控制器挂载）、共享（同一子系统内多控制器并发挂载）、分散（跨多个子系统的多控制器并发访问）；它们还和"多路径 I/O"共同构成命名空间的访问轴。

## 生活化类比

把三种访问模型想成**三种合租方式**：

- **私有** = 一室一厅整租：整套房只给一个租户（控制器）专用，别的租户想用得等他退房。
- **共享** = 大开间合租：同一屋檐（同一 NVM 子系统）下住多个租户（多控制器），各自有钥匙（NSID），但都看见同一套房。
- **分散** = 同品牌连锁公寓的同一套房：名字一样、用途一样，但实际房间分布在不同城市的不同楼里（多个参与的子系统），靠调度让租户感觉是"同一套房"。

而"多路径 I/O"是另一个维度——同一个租户通过两扇门进入同一套房（同一主机多条路径到同一命名空间）。

## 工作流程

```text
  私有命名空间：
     Controller A ──NSID 1──→ 命名空间 P
     (此 NSID 此刻不挂到任何其他控制器)

  共享命名空间：
     Controller A ──NSID 2──┐
                            ├──→ 命名空间 S (同一子系统内)
     Controller B ──NSID 2──┘

  分散命名空间：
     子系统 1: Controller A1 ──NSID 5──┐
                                        ├──→ 命名空间 D
     子系统 2: Controller A2 ──NSID 5──┘

  多路径 I/O (独立维度):
     主机 ── path 1 ──→ Controller A ──→ 命名空间 S
     主机 ── path 2 ──→ Controller B ──→ 命名空间 S
```

简化说明：私有/共享/分散是**命名空间本身的属性**；多路径 I/O 是**主机到命名空间的访问属性**——两个维度可以同时存在（如"一个共享命名空间，主机还走多路径"）。

## 初学者案例

**场景：想给 ESXi 双控存储配置命名空间，应该选哪个模型？**

1. ESXi 主机上有两张 HBA 卡，分别走两个控制器到同一台双控 NVMe 阵列。
2. 想让两个控制器都能访问到该命名空间（用于故障切换、负载均衡）→ 选**共享命名空间**。
3. 创建命名空间时指定 NVM Set；用 Namespace Attachment 把它挂到两个控制器；ESXi 主机用 `nvme list` 在两条路径上都能看到。
4. 接着配置 ANA Reporting，控制器报告 Optimized/Non-Optimized 状态，ESXi 据此选最优路径。

**反例**：如果只把命名空间挂到控制器 A（私有模式），控制器 B 故障时这条路径就完全不可用——单点故障。

> 选型口诀：**单机本地盘用私有；双控/集群用共享；跨数据中心用分散**。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 私有命名空间 | 同一时刻最多一个控制器挂载 |
| 共享命名空间 | 同一子系统内 2+ 控制器并发挂载 |
| 分散命名空间 | 跨 2+ 参与的子系统并发访问 |
| 指定命名空间 | 由命令中 NSID 字段选中的命名空间 |
| NSID 控制器相对 | 同一 NSID 经不同控制器可能指向不同命名空间（私有场景） |
| 多路径 vs 共享 | 多路径 I/O 是"一主机多路径到一命名空间"；共享是"多主机经不同控制器到一命名空间" |
| 多路径前提 | 多路径 I/O 要求子系统内至少 2 个控制器 |
| 共享前提 | 共享命名空间要求子系统内至少 2 个控制器 |
| 一致性要求 | 对每个能访问同一共享命名空间的控制器，NSID 相同 + Identify Namespace 内容相同 |
| 持久身份 | 跨路径识别同一命名空间要用全局唯一 ID（如 UUID），不是 NSID |
| 并发原子性 | 各控制器并发操作共享命名空间时，原子性以"接收命令的控制器"为单位，**不跨控制器保证** |
| 跨控制器排序 | 主机软件或应用负责跨控制器命令排序 |
| 端口隔离 | 独立端口的复位只影响该端口上的控制器，不影响其他控制器、共享命名空间或别的控制器正在进行的操作 |
| SR-IOV 视图 | 每个 VF 是一个独立控制器；命名空间可跨 VF 共享——共享模型的另一种实现 |
| ANA 触发 | 子系统对不同控制器呈现不同访问特性时可报告 ANA Reporting |
| 主机协调 | 多主机并发访问需主机间协调；协调流程**不在基础规范范围** |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 私有 vs 共享 | 私有同一时刻最多 1 控制器挂载；共享 2+ 控制器并发挂载 |
| 共享 vs 分散 | 共享是同一子系统内多控制器；分散是跨多个子系统 |
| 多路径 I/O vs 共享 | 多路径是 1 主机多路径；共享是多主机经不同控制器 |
| 多路径 I/O vs 多控制器 | 多路径要求"同一主机 + 至少 2 控制器"；多控制器不一定是多路径 |
| NSID 一致 vs 身份一致 | NSID 相同还不够；Identify Namespace 内容必须相同 |
| NSID vs 全局 ID | NSID 是控制器相对的；全局 ID（UUID）跨路径/跨会话稳定 |
| 共享 vs 多主机 | 共享命名空间可以被多主机访问，但多主机访问还需主机间协调 |
| SR-IOV vs 共享 | SR-IOV 是"用 VF 实现的共享"；共享不一定要 SR-IOV |
| ANA vs 共享 | 共享是"能不能访问"；ANA 是"访问特性是不是最优" |
| 端口隔离 vs 控制器隔离 | 端口隔离是物理层；控制器隔离是逻辑层（同一端口上也有多控制器） |

## 进阶细节

- **三类命名空间的边界**（规范 1.5.x / 2.3.3 / 4.2.1）：
  - Private namespace：任何时刻最多一个控制器挂载
  - Shared namespace：同一 NVM 子系统内两个或更多控制器并发挂载
  - Dispersed namespace：跨两个或更多参与子系统的控制器并发访问
- **Specified namespace**：命令中 NSID 字段选中的那个命名空间。
- **NSID 是控制器相对的**（规范 4.2.1）：NSID 是控制器用来提供命名空间访问的标识符，也是 SQE 中的字段；同一 NSID 值经不同控制器访问可能命中不同命名空间。
- **共享命名空间的一致性**（规范 4.2.2 / Figure 20-21）：
  - 每个能访问同一共享命名空间的控制器上 NSID **必须相同**
  - Identify Namespace 返回内容对每个控制器**完全一致**
  - 全局唯一 ID（不是 NSID）让软件识别同一命名空间的多条访问路径
- **并发操作**（规范 4.2.2）：控制器可并发操作共享命名空间；原子性是"接收命令的控制器"边界；**不跨控制器保证**；主机/应用负责跨控制器排序。
- **多路径 I/O 与端口隔离**（规范 4.2.3）：
  - 独立端口的复位**只影响该端口上的控制器**
  - 不影响其他控制器、不影响共享命名空间、不影响其他控制器正在进行的操作
  - SR-IOV 是同一访问模型的另一种实现：每个 VF 一个控制器，跨 VF 共享同一命名空间
- **Asymmetric Namespace Access Reporting (ANA)**（规范 8.1.14 / 4.2.3 / 5.2.12）：
  - 当子系统对不同控制器呈现不同访问特性时，可报告 ANA
  - 状态：Optimized / Non-Optimized / Inaccessible / Persistent Loss / Change
  - **Optimized**（必报状态）：所有支持的操作都以优化路径执行
  - **Non-Optimized**：所有支持的操作可执行，但可能更慢或低效
  - **Inaccessible**：用户数据不可用；多数命令中止；可恢复
  - **Persistent Loss**：用户数据**持久**不可用；该控制器/组的关系**永远不再转移**（终态）
  - **Change**：状态转移进行中；多数命令中止，返回 Asymmetric Access Transition
- **ANA 状态机**（规范 4.2.3 / Figure 23 / 5.2.12）：
  - Optimized ↔ Non-Optimized：可双向
  - 任何可达状态都可通过 Change 中转
  - Inaccessible → Optimized/Non-Optimized：恢复
  - Persistent Loss：**终态**，不可再转移
  - 规范**不要求**所有箭头的直接转移；上图是参考性图示
- **ANA 日志布局**（规范 5.2.12 / Figure 211）：
  - 头部：全局 change count（CLR 后从 0 开始，绕回 0）
  - ANA Group 描述符（按 AGID 升序）：
    - 组级 change count（1-based，绕回 1；为 0 表示不报告）
    - 该控制器对组的 ANA 状态
    - 挂载的 NSID 列表（升序，RGO=1 时省略）
  - 仅包含"有命名空间挂到 Get Log Page 处理控制器"的组
  - `RGO=1` 返回组描述符但不返回 NSID 列表
- **分段读取的快照一致性**（规范 5.2.12）：描述符变长时，主机用索引偏移（0=头部，1=第一个描述符，依此类推）；分段读时主机**重读头部**，仅当全局 change count 仍匹配才接受快照。
- **change count 语义**：
  - 全局 change count：CLR 后从 0 开始，绕回 0
  - 每组 change count：可选；1 开始，绕回 1；为 0 表示**不报告**
  - 不报告时主机必须直接比较描述符内容
- **Inaccessible / Change 的重试策略**（规范 5.2.12）：主机可重试另一条可访问路径，或在原路径上至少重试 `ANATT` 秒。
- **ANA Change Notice**（规范 5.2.12）：启用时控制器在以下情况后报告——ANA Group ID 变化、转移失败、进入 Optimized/Non-Optimized/Inaccessible/Persistent Loss；**但**状态进入是命名空间挂载导致的不报告。
- **Inaccessible/Persistent Loss 的副作用**：可能把 Identify 中的容量字段清零（准确值不可用）；主机应换用 Optimized/Non-Optimized 路径。
- **ANA 下的 Admin 命令**（规范 5.2.12）：ANA **不全面禁用** Admin 命令；不依赖命名空间、不专属 I/O 命令集的 Admin 操作通常仍可用；但 Get/Set Features、Get Log Page、Identify 有明确例外（详见 5.2.12）。
- **参与的 NVM 子系统**（规范 1.5.x）：包含提供分散命名空间访问的控制器集合；这不把分散命名空间变成"每个子系统的独立命名空间"。

## 规范依据

- [Dispersed namespace 定义，PDF 第 30 页](../_source/pages/page-030.md)
- [Namespace 与 NSID 关系，PDF 第 33 页](../_source/pages/page-033.md)
- [Private / Shared / Specified 命名空间，PDF 第 34-35 页](../_source/pages/page-034.md)
- [共享命名空间的 NSID 一致性与并发，PDF 第 55-56 页](../_source/pages/page-055.md)
- [ANA 状态机、ANATT 重试、Change Notice，PDF 第 497-503 页](../_source/pages/page-500.md)

## 相关阅读

- [namespace.md](namespace.md) - 三种访问模型针对命名空间
- [namespace-management-lifecycle.md](namespace-management-lifecycle.md) - Attach 决定可见性
- [namespace-topology-and-change-logs.md](namespace-topology-and-change-logs.md) - 拓扑变更配套访问模型
- [dispersed-namespace-lifecycle.md](dispersed-namespace-lifecycle.md) - 分散命名空间生命周期细节
