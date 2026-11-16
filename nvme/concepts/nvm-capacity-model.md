# NVM 容量模型（NVM Capacity Model）

**NVM 容量模型**将“逻辑容量如何逐层分配”与“物理介质单元（Media Unit）如何经由通道（Channel）组织”彻底解耦。NVMe 支持按 **NVM 子系统 → Domain（域）→ 耐久度组（Endurance Group）→ NVM Set → 格式化命名空间** 五个层级上报与分配容量，每一级均可选支持。[PDF pp. 141-142](../_source/pages/page-141.md)

> **类比**：把企业级 SSD 想成一座**多电梯立体仓库**——左半边是**仓库账本**（总账 → 楼层 → 温区 → 货架排 → 箱子，层层切分但不必切光）；右半边是**搬运通道**（电梯 Channel ↔ 货架 Media Unit）。两套账相互独立：你可以让一个温区跨所有电梯以榨干带宽，也可以按电梯隔离成多个独立温区。

## 逻辑分配与物理组织

```text
逻辑容量分配（自上而下，每层可选）          物理数据通路（1..* 双向连接）
──────────────────────────────────          ────────────────────────────
NVM Subsystem Capacity (TNVMCAP / UNVMCAP)  Channel(s) <── data path ──> Media Unit(s)
  └── Domain Capacity                         (每个 Media Unit 恰好属于 1 个 EG；
      └── Endurance Group Capacity             支持 NVM Set 时，还恰好属于 1 个 NVM Set)
          └── NVM Set Capacity
              ├── 格式化命名空间 (整体位于 1 个 NVM Set 内)
              └── 未分配容量 (Unallocated)
```

- **核心分配契约**：[PDF p. 142](../_source/pages/page-142.md)
  - 每一层都不必用尽全部容量，剩余部分保留为未分配容量（注意：`TNVMCAP` 与 `UNVMCAP` 之间的差额可能包含厂商预留的 **OP（Over-Provisioning）**，OP 不参与命名空间分配）。
  - **Capacity Management** 命令负责创建/分配 Endurance Group、NVM Set 及 Media Unit；**Namespace Management** 命令负责从底层分配格式化命名空间。
  - 若不支持 NVM Set，所有 `NVMSETID` 字段清零；若不支持 Endurance Group，所有 `ENDGID` 字段清零（支持 NVM Set 隐含必须支持 Endurance Group）。[PDF pp. 142, 145-146](../_source/pages/page-142.md)

## 介质组织权衡与容量上报矩阵

不同的 Media Unit 与 Channel 映射方式直接决定带宽与隔离性：[PDF pp. 143-145](../_source/pages/page-143.md)

| 组织方式（规范原图） | Media Unit 分组策略 | 设计权衡 |
|---|---|---|
| **简单型（Figure 86）** | 1 个 Endurance Group 与 1 个 NVM Set 跨全部 4 个 Channel（如 16 个 MU） | 最大化单操作聚合带宽 |
| **垂直型（Figure 87）** | 每个 Channel 的 4 个 MU 各自组成独立的 Endurance Group 与 NVM Set | 4 组完全隔离，但单组带宽受限于单 Channel |
| **水平双 NAND 型（Figure 88）** | 4 个 Channel 各出 1 个 SLC MU 与 3 个 QLC MU，分别组建 SLC 集与 QLC 集 | 同时获得高耐久/低延迟 SLC 池与大容量 QLC 池的全通道带宽（图中 SLC/QLC 容量调整因子示例为 ~400 与 ~100） |

### 容量账目查询与更新层级（Figure 89）

创建或删除对象后，主机与控制器依据系统支持的层级在对应位置核对可用/已分配容量：[PDF pp. 145-146](../_source/pages/page-145.md)

| 创建/删除的对象 | 系统支持的层级形态 | 查询与更新容量账目的位置 |
|---|---|---|
| **Endurance Group** | 不支持 Domain | NVM 子系统（`TNVMCAP`、`UNVMCAP`、`MEGCAP`） |
| **Endurance Group** | 支持 Domain | Domain Attributes Entry |
| **NVM Set** | 同时支持 NVM Set 与 Endurance Group | Endurance Group Information 日志页 |
| **格式化命名空间** | 既不支持 EG 也不支持 Domain | NVM 子系统 |
| **格式化命名空间** | 支持 Domain 但不支持 EG | Domain Attributes Entry |
| **格式化命名空间** | 支持 EG 但不支持 NVM Set | Endurance Group Information 日志页 |
| **格式化命名空间** | 支持 NVM Set | NVM Set Attributes Entry |

## 运行时清单与容量配置日志

- **Media Unit Status 日志**：按 Domain 选择（`0` 表示控制器所在 Domain），按 `MUID` 升序报告可访问的 Media Unit 及其所属 Domain/EG/NVM Set、容量调整因子、剩余备用块（Spare）、已用百分比（Percentage Used）及去重升序的 Channel ID 列表。若未选中容量配置，相关归属与通道计数字段清零。[PDF pp. 276-278](../_source/pages/page-276.md)
- **Supported Capacity Configuration List**：按 Domain 选择并按 `CCID` 升序排列，以嵌套变长描述符展示设备支持的各种容量配置方案（`Capacity Configuration → Endurance Group → NVM Set IDs + Channel → Media Unit`）。其中寿命估计值（Endurance Estimate）以十亿字节（GB）向上取整报告：`0` 表示未上报，全 `1` 饱和值表示“实际值不低于该值”；NVMe 2.1 中 Media Unit Configuration Descriptor 为 8 字节且扩展长度字段为 `0`。[PDF pp. 278-281](../_source/pages/page-278.md)

## 关联与引申概念

- **OP（Over-Provisioning，预留空间）**：SSD 固件内部保留用于垃圾回收（GC）、磨损均衡和坏块替换的物理闪存空间，不计入主机可分配容量。
- **Capacity Adjustment Factor（容量调整因子）**：用于将不同类型介质（如 SLC vs QLC）的物理磨损/容量特性折算到统一逻辑容量账目中的系数。
- **相关文档**：
  - [storage-resource-hierarchy.md](storage-resource-hierarchy.md) — 存储资源总体层次结构
  - [nvm-sets-and-endurance-groups.md](nvm-sets-and-endurance-groups.md) — Endurance Group 与 NVM Set 的隔离与寿命管理
  - [capacity-management-operations.md](capacity-management-operations.md) — Capacity Management 命令操作流程
  - [namespace-management-lifecycle.md](namespace-management-lifecycle.md) — 命名空间创建与容量消耗

## 规范依据

- [容量上报层级与可选性，PDF p. 141](../_source/pages/page-141.md)
- [分配契约、零值标识符、Media Unit 与 Channel，PDF p. 142](../_source/pages/page-142.md)
- [Figure 86–88 介质组织示例，PDF pp. 143-145](../_source/pages/page-143.md)
- [Figure 89 容量信息选择矩阵，PDF p. 146](../_source/pages/page-146.md)
- [Media Unit Status 与 Supported Capacity Configuration 日志，PDF pp. 276-281](../_source/pages/page-276.md)
