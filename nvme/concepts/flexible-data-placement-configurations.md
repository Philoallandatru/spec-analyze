# 灵活数据放置配置（Flexible Data Placement Configurations）

**FDP（Flexible Data Placement，灵活数据放置）**旨在减少 SSD 内部垃圾回收（GC）导致的写放大：主机在写命令中通过放置标识（Placement Identifier）显式提示数据的生命周期分组，控制器则把不同分组写入不同的介质回收单元（Reclaim Unit）。**FDP Configurations 日志（LID `20h`）**是以**耐久度组（Endurance Group, EG）**为作用域的静态目录，列出该 EG 当前允许启用的放置配置。[PDF pp. 294-295](../_source/pages/page-294.md)

可以把耐久度组想成**一个快递分拣中心**：`FDP Configurations` 日志是墙上挂的"分拣方案表"（规定有几个分拣区、每个区有几个篮子、篮子之间如何隔离）；主机通过 **Set Features FID `1Dh`** 选中并启用其中一套方案后，才能在写数据时指定篮子，并通过三份运行时日志实时观察篮子使用量、写放大统计和重定向事件。

## 配置目录与隔离模型

```text
Endurance Group (由 ENDGID 选择)
  `-- FDP Configurations 日志 (LID 20h, 静态目录)
        |-- FDPCV=1 ? (当前是否可用)
        |-- NRG (回收组数) + RGIF (RGID 占 Placement ID 高几位)
        |-- NRUH (回收单元句柄数) + 按 RUH ID 升序排列的句柄类型
        |-- NNS (命名空间上限) + 标称 RU 容量 + ERUTL (重定向时间上限)
        |
        v  Set Features FDP (FID 1Dh) 选定有效 index 并启用
  +--------------------+---------------------+---------------------+
  v                    v                     v                     |
RU Handle Usage     FDP Statistics        FDP Events               | 运行时观测
(LID 21h)           (LID 22h)             (LID 23h)                | (EG 作用域)
```

| 配置字段 / 约束 | 含义与不变量 |
|---|---|
| `NUMFDPC` | 配置描述符的 0 基数量；描述符变长，末尾补 0 至 8 字节对齐 |
| `FDPCV` | `FDPCV=1` 表示该配置当前可用；`FDPCV=0` 时主机应忽略该条目 |
| `NRG` / `NRUH` | 回收组（Reclaim Group）数与回收单元句柄（Reclaim Unit Handle, RUH）数，均非 0 |
| `RGIF` | 当 `NRG > 1` 时非 0，指定 Placement Identifier 高几位作为 RGID（其余低位为 Placement Handle `PHNDL`）；`NRG=1` 且 `RGIF=0` 时整个 16 位 Placement ID 均为 Placement Handle |
| `MAXPIDS` | 0 基上限，且必须严格小于 `NRG * NRUH` |
| `NNS` | 命名空间上限；非 0 时受 `MNAN` 约束，否则受 `NN` 约束 |
| `FDPVWC=1` | 表示该配置带有易失写缓存（VWC），会强制 Identify Controller 的 `VWCP=1`（即使控制器整体未为非 FDP 主机提供 VWC） |
| `ERUTL` | 句柄在控制器重定向（retarget）前可引用同一回收单元的估计最长时间；0 表示不上报 |

句柄类型决定了数据在控制器内部 GC 搬移后的隔离强度：[PDF pp. 295-296](../_source/pages/page-295.md)
- **Initially Isolated（初始隔离，`01h`）**：最初会把经由同组内不同句柄写入的数据隔离开，但在厂商内部数据迁移（如 GC）后，同一类型句柄写入的数据可能被合并到一起。
- **Persistently Isolated（持续隔离，`02h`）**：即使经过 GC 等内部迁移，数据仍保证隔离在同组内同一 Reclaim Unit Handle 对应的回收单元上。

## 运行时启用与三份观测日志

- **Flexible Data Placement（FID `1Dh`）**：mandatory-save（必保存）特性，为指定 EG 选择有效配置索引并启停 FDP。切换配置可能改变 Identify 报告的 User Data Format 字段，主机应重新发现受影响的 Identify 视图。若 FDP 未启用就下发带放置指令（DPD）的写命令或读取运行时日志，会返回 `FDP Disabled`。[PDF pp. 296-298, 412](../_source/pages/page-412.md)
- **FDP Events（FID `1Eh`）**：针对经由命名空间 Placement Handle 触达的 RUH 启用有序事件通知。FDP 禁用时无效，拒绝广播 NSID，且作用域为同 EG 内跨命名空间共享的句柄。[PDF pp. 413-414](../_source/pages/page-413.md)

| 运行时日志（EG 作用域） | LID | 稳定契约 |
|---|---|---|
| **RU Handle Usage** | `21h` | 每个已配置句柄对应一条描述符（未使用 / 主机显式占用 / 控制器选择）；"控制器选择"条目至多 1 条 |
| **FDP Statistics** | `22h` | 128 位饱和计数：主机写入字节、介质写入字节、介质擦除字节（用于计算写放大 WAF）；切换 FDP 特性会清零，**固件更新不清零** |
| **FDP Events** | `23h` | 固定 4 KiB 事件历史，按发生先后（非严格时间戳数值，因 CLR 后 Timestamp 可能变化）从旧到新排列；写满丢弃最旧条目；FDP 从禁用切到启用时计数清零 |

事件日志支持按**主机事件**（提前重定向、时限到期、重置引发重定向、非法放置）与**控制器事件**（介质再分配、隐式重定向）筛选，每条事件独立携带 Placement Identifier、NSID 与 RG/RUH 位置的有效性位。[PDF pp. 298-301](../_source/pages/page-298.md)

## 规范依据

- [FDP 目录头部、配置上限与句柄类型（Figures 279-283），PDF pp. 294-296](../_source/pages/page-294.md)
- [RU Handle Usage、FDP Statistics 与 FDP Events 日志，PDF pp. 296-301](../_source/pages/page-296.md)
- [Flexible Data Placement Feature（FID 1Dh），PDF p. 412](../_source/pages/page-412.md)
- [FDP Events Feature（FID 1Eh），PDF pp. 413-414](../_source/pages/page-413.md)

## 关联与引申概念

- **写放大（Write Amplification, WAF）**：介质实际写入量与主机写入量之比，FDP Statistics（LID `22h`）直接提供计算 WAF 所需的分子与分母。
- **NVM 集与耐久度组**：FDP 配置与统计均以 Endurance Group 为作用域。详见 [NVM 集与耐久度组](nvm-sets-and-endurance-groups.md) 与 [存储资源层次结构](storage-resource-hierarchy.md)。
- **特性值与作用域**：FDP 的启停与事件掩码均通过 Set/Get Features 配置。详见 [特性值与作用域](feature-values-and-scope.md)。
- **日志页读取**：LID `20h–23h` 均通过 Admin 命令 Get Log Page 读取。详见 [日志页读取](log-page-retrieval.md)。
- **指令交换（Directives）**：写命令中的 Data Placement Directive（DPD）字段用于携带 Placement Identifier。详见 [指令交换](directive-exchange.md)。
