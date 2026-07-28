# 存储资源层次结构（Storage Resource Hierarchy）

## 一句话说明

NVMe 用一套从"子系统→域→耐久度组→底层介质单元→命名空间"的层级，把物理介质切成不同维度的"格子"，每个格子负责一个关注点：共享状态、寿命管理、数据放置、可寻址卷。

## 生活化类比

把一台 NVMe SSD 想成一座**大型自动化仓库**：

- **仓库大楼** = NVM 子系统（一个所有权、一次购电）
- **楼层** = 域（同一层共享电源、容量信息）
- **温区** = 耐久度组（同温区一起做温度/寿命管理）
- **货架** = 介质单元（实际的闪存芯片）
- **货位编号** = 命名空间（对外可订可取的标准位）

仓库管理员有两种摆货方式：
- **NVM 集方式**：每张订单（命名空间）固定分配到一排货架，搬动少、规则简单。
- **回收组方式**：订单可以分散到仓库里的多个回收篮（回收单元），工人按"放置句柄"动态决定这次塞哪个篮子——这就是灵活数据放置（FDP）。

## 工作流程

```text
NVM 子系统 (NVM Subsystem)
  └─ 域 (Domain)
       └─ 耐久度组 (Endurance Group)        ← 寿命 / 磨损管理边界
            ├─ 介质单元 (Media Unit)        ← 物理闪存芯片
            └─ 二选一组织方式：
                 ├─ 方式 A：NVM 集 (NVM Set)
                 │     └─ 命名空间 (Namespace)   每个 NS 恰好属于一个 NVM Set
                 └─ 方式 B：回收组 (Reclaim Group)
                       └─ 回收单元 (Reclaim Unit)
                             └─ 命名空间 (Namespace)   跨多个 RU 分布
```

简化说明：图中每条 "└─" 都表示"恰好属于一个"；同一耐久度组不会同时混用两种方式。

## 初学者案例

**场景：采购方问"为啥这款企业级 SSD 比消费级贵？"**

1. 消费级 SSD 内部：1 子系统 → 1 域 → 1 耐久度组 → 1 NVM 集 → 1 命名空间，资源整块使用，无需对外报告多 EG / 多 NVM Set。
2. 企业级 SSD 内部：1 子系统 → 多域 → 多耐久度组 → 多个 NVM 集或回收组 → 多个命名空间，可按租户/业务切分容量、隔离故障。
3. 故障速查：若主机 `nvme list-ctrl` 看不到多个控制器，多半是设备把资源都收编到一个域/EG 里——并非故障，而是产品定位。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 包含关系方向 | Domain ⊂ Subsystem；EG ⊂ Domain；NVM Set ⊂ EG；Namespace ⊂ NVM Set 或 EG（视方式而定） |
| 互斥的组织方式 | 一个耐久度组内要么是 NVM Set 方式，要么是 Reclaim Group 方式，二选一 |
| 命名空间归属 | NVM Set 方式下每个 NS 恰好属于一个 NVM Set；Reclaim Group 方式下每个 NS 恰好属于一个 EG、占用若干 RU |
| 可选实体 | 多个 EG、多个 NVM Set、Reclaim Group 机制均为可选，不支持时不需要报告 |
| 介质单元归属 | 每个 Media Unit 恰好属于一个耐久度组 |
| 共享状态边界 | 同一 Domain 内共享电源状态、容量信息等管理属性 |
| 命名空间生命周期 | 创建/删除命名空间是最常见的动态配置操作 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Domain vs Endurance Group | Domain 划的是"共享状态/电源"边界；EG 划的是"磨损/寿命管理"边界 |
| NVM Set vs Reclaim Group | NVM Set 是"分块管理容量"的逻辑池；Reclaim Group 是"按写入分散数据"的回收单位 |
| Media Unit vs NVM Set | Media Unit 是物理芯片；NVM Set 是逻辑容量池 |
| Namespace vs NVM Set | Namespace 是主机可寻址的逻辑卷；NVM Set 是它的物理/逻辑归属 |
| EG 方式 A vs EG 方式 B | 方式 A 把 NS 钉死在 NVM Set 上；方式 B 让 NS 跨多个 RU 分布，支撑 FDP |

## 进阶细节

- **规范 2.3.1 实体清单**（PDF 第 46 页）：NVM Subsystem、Domain、Endurance Group、Reclaim Group、Reclaim Unit、NVM Set、Namespace、Media Unit。
- **包含关系**（规范 2.3.1）：
  - 每个 Domain ⊂ 1 个 NVM Subsystem；
  - 每个 EG ⊂ 1 个 Domain；
  - EG 内可包含一个或多个 NVM Set，或一个或多个 Reclaim Group（二选一）；
  - NVM Set 方式：每个 NVM Set ⊂ 1 个 EG，每个 NS ⊂ 1 个 NVM Set；
  - Reclaim Group 方式：每个 Reclaim Group ⊂ 1 个 EG，每个 Reclaim Unit ⊂ 1 个 Reclaim Group，每个 NS ⊂ 1 个 EG 内的若干 RU；
  - 每个 Media Unit ⊂ 1 个 EG。
- **可选特性的报告义务**（规范 2.3.1）：不支持多 NVM Set 的子**系统不需要**报告 NVM Set；不支持多 EG 的子系统不需要报告 EG——"不存在即可不报"。
- **Figure 11-15 示例图**：规范用 Figure 11（最简）→ Figure 15（多域多 EG 多 Reclaim Group 的复杂形态）展示层级膨胀，命名约定 "Abc" 中 A=Domain、b=EG、c=Reclaim Group。
- **设计原则**：把"共享状态"（Domain）、"寿命管理"（EG）、"可寻址"（Namespace）拆成不同实体，而不是把多种语义混在一个标签里。

## 规范依据

- [存储实体与包含规则，PDF 第 46 页](../_source/pages/page-046.md)
- [复杂层次结构示例 Figure 15，PDF 第 51 页](../_source/pages/page-051.md)
- [简单层次结构示例 Figures 11-12，PDF 第 47 页](../_source/pages/page-047.md)

## 相关阅读

- [NVM 集与耐久度组](nvm-sets-and-endurance-groups.md) - EG/Set 子层详解
- [NVM 容量模型](nvm-capacity-model.md) - 容量按层级分配
- [控制器虚拟化资源](controller-virtualization-resources.md) - 多控制器资源虚拟化
- [域与分区](domains-and-divisions.md) - 域与分区的细节
- [NVM 子系统](nvm-subsystem.md) - 顶层子系统定义
