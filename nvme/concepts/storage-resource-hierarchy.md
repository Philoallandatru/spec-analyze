# 存储资源层次结构（Storage Resource Hierarchy）

NVMe 通过一套从顶层 **NVM 子系统** 逐级向下细分到 **域（Domain）、耐久度组（Endurance Group）、介质单元（Media Unit）** 以及 **命名空间（Namespace）** 的层次结构，把"共享状态"、"寿命管理"、"数据放置"和"逻辑卷寻址"四个关注点清晰解耦。[PDF pp. 46-51](../_source/pages/page-046.md)

可以把一台 NVMe SSD 想成**大型自动化仓库**：
- **NVM 子系统** = 整座仓库大楼
- **域（Domain）** = 大楼楼层（同一层共享供电和容量状态）
- **耐久度组（Endurance Group, EG）** = 温控区（同区介质一起统计磨损与寿命）
- **介质单元（Media Unit）** = 底层实际的物理闪存芯片货架
- **命名空间（Namespace）** = 对外可寻址的逻辑货位（主机可见的卷）
- 仓库有两种摆货方式：传统的 **NVM Set**（固定划分容量池）和面向 FDP 的 **Reclaim Group / Reclaim Unit**（按写入句柄分散放置，降低垃圾回收写放大）。

---

## 层次结构与互斥组织方式

```text
  NVM 子系统 (NVM Subsystem)
  └── 域 (Domain)  ─────────────────────────── 共享电源状态、容量信息等管理属性
      └── 耐久度组 (Endurance Group) ───────── 统一的介质磨损与寿命管理边界
          ├── 介质单元 (Media Unit) ────────── 一个或多个底层物理存储介质
          └── 组织方式（同一 EG 内二选一，互斥）：
              ├── 方式 A：NVM 集合 (NVM Set)
              │   └── 命名空间 (Namespace) ─── 每个 NS 恰好属于一个 NVM Set
              └── 方式 B：回收组 (Reclaim Group)  ← 用于 FDP 灵活数据放置
                  └── 回收单元 (Reclaim Unit, RU)
                      └── 命名空间 (Namespace) ── 属于同一 EG，可跨越多个 RG 中的 RU
```

### 核心实体边界与规则

| 实体 | 职责边界与包含规则 |
|------|-------------------|
| **NVM Subsystem** | 顶层存储设备实体，包含一个或多个 Domain |
| **Domain（域）** | 共享状态（电源状态、容量信息）的最小不可分割单元；每个 Domain 恰好属于一个子系统 [PDF p. 30](../_source/pages/page-030.md) |
| **Endurance Group（耐久度组）** | 寿命与磨损管理的最小整体；属于一个 Domain，由一个或多个 Media Unit 构成 |
| **Media Unit（介质单元）** | 底层物理存储介质单元（如闪存 Die/芯片），恰好属于一个 EG [PDF pp. 32, 34](../_source/pages/page-032.md) |
| **NVM Set vs Reclaim Group** | 同一 EG 内**互斥二选一**。NVM Set 是传统逻辑容量池；Reclaim Group 包含多个 Reclaim Unit（RU），支撑灵活数据放置 |
| **可选报告原则** | 多个 EG、多个 NVM Set 和 Reclaim Group 均为**可选特性**；消费级单盘不支持时无需在管理接口中报告这些实体 [PDF p. 51](../_source/pages/page-051.md) |

---

## 灵活数据放置（FDP）与间接映射

在采用 Reclaim Group 组织的耐久度组中，主机可以通过 **FDP（Flexible Data Placement）** 机制精确控制每次写入的物理放置位置，从而隔离不同生命周期的数据、减少垃圾回收（GC）写放大：[PDF pp. 103-104](../_source/pages/page-103.md)

```text
  主机侧 (Namespace 作用域):   Placement Handle (放置句柄)
                                     |  (Placement ID = Reclaim Group ID + Placement Handle)
                                     v
  控制器侧 (EG 作用域):       Reclaim Unit Handle (RUH，回收单元句柄)
                              ├── 回收组 0 ──> 当前可写的 Reclaim Unit
                              ├── 回收组 1 ──> 当前可写的 Reclaim Unit
                              └── 回收组 P-1 ─> 当前可写的 Reclaim Unit
                                                   |  (写满 / 退役)
                                                   v
                                            控制器自动重定向到空闲 RU
```

- **一对一引用约束**：每个 RUH 在每个回收组中同时只引用恰好一个可写的 RU；同一时刻一个 RU 最多被一个 RUH 引用。
- **动态重定向**：当被引用的 RU 写满退役后，控制器自动将句柄重定向到空闲 RU；原 RU 经擦除回收后可重新分配给相同或不同的句柄。

---

## 关联与引申概念

- **Exported NVM Resources（导出的 NVM 资源）**：用于远程访问的导出子系统、导出命名空间与导出端口，与本地存储资源边界正交。
- **FDP（Flexible Data Placement）**：NVMe 2.0+ 引入的写放置优化特性，通过 Placement Handle 让主机把冷热数据写入不同的 Reclaim Unit，显著降低 SSD 写放大。
- **相关文档**：
  - [NVM 集与耐久度组](nvm-sets-and-endurance-groups.md) — Endurance Group 与 NVM Set 子层详解
  - [NVM 容量模型](nvm-capacity-model.md) — 容量如何在各层级间划分与分配
  - [域与分区](domains-and-divisions.md) — 多域划分与状态共享细节
  - [控制器虚拟化资源](controller-virtualization-resources.md) — 多控制器下的资源分配
  - [NVM 子系统](nvm-subsystem.md) — 顶层子系统定义

## 规范依据

- [存储实体清单与包含规则（Figures 11-15），PDF pp. 46-51](../_source/pages/page-046.md)
- [Domain、Endurance Group 与导出资源定义，PDF p. 30](../_source/pages/page-030.md)
- [Media Unit 与数据放置定义，PDF pp. 32, 34](../_source/pages/page-032.md)
- [FDP 回收组、RUH 与动态重定向规则，PDF pp. 103-104](../_source/pages/page-103.md)
