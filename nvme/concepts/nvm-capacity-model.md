# NVM 容量模型（NVM Capacity Model）

## 一句话说明

NVMe 把"逻辑容量分配"和"物理介质组织"分成两件事：容量自上而下按子系统/域/EG/NVM Set/命名空间 5 个层级报告与分配（每层支持都是可选的），而物理数据流走的是"通道（Channel）↔ 介质单元（Media Unit）"另一条线。

## 生活化类比

把企业级 SSD 想成一座**多电梯的立体仓库**：

- **仓库账本**（逻辑层）：总账 → 楼层 → 温区 → 货架排 → 箱子（命名空间），一层层可分配、可盘点。
- **搬运通道**（物理层）：电梯（Channel） ↔ 货架（Media Unit），每件货走哪条电梯是另一套账。

账本可以"先切大块，再细分"，不必用尽；搬运通道的数量与走向决定单次最大并发带宽。两套账**可以**但不**必须**互相对应——比如 1 个 EG 跨 4 个电梯就拿到最大带宽；按电梯各分 1 个 EG 则变成 4 路隔离。

## 工作流程

```text
逻辑容量分配（自上而下）            物理数据通路
─────────────────────────           ──────────────────
NVM 子系统容量                      Channel(s)  ←→  Media Unit(s)
 └─ 域容量                           1..*            1..*
     └─ 耐久度组容量                 （每个 Media Unit
         └─ NVM Set 容量             恰好属于 1 个 EG；
             ├─ 格式化命名空间        支持 NVM Set 时
             └─ 未分配容量            还恰好属于 1 个 NVM Set）
```

简化说明：左半边是"分配账"，右半边是"运输账"——两套账不一定是 1:1 镜像。

## 初学者案例

**场景：买盘时标称 2TB，可用只有 1.8TB，差异从哪来？**

1. 系统下 `nvme id-ctrl /dev/nvme0` 看 `TNVMCAP`（总容量）= 2,000 GB；这是"账本总额"。
2. 实际可分配容量看 `UNVMCAP`（未分配容量）+ 各 EG / NVM Set 的可用空间——一般 1.8 TB。
3. 差额来自 OP（Over-Provisioning，厂商预留的备用块），OP 不算"未分配"也不分配给任何命名空间。
4. 运维再用 `nvme capacity-mgmt` 在不同时段调整 EG / NVM Set 的"已分配/未分配"账目。
5. 故障速查：若 `UNVMCAP` 突然掉很多但 `TNVMCAP` 没变，多半有命名空间被扩缩容或 EG 在调整。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 5 级容量报告 | 子系统、域、EG、NVM Set、命名空间都可选上报；"支持哪一级报哪一级" |
| 分配不必用尽 | 任何一层的容量都可保留为"未分配"，不必全部分配到下一层 |
| 命名空间归属 | 含格式化的命名空间整体在一个 NVM Set 内，**不跨集** |
| Media Unit 归属 | 每个 Media Unit 恰好属于 1 个 EG；支持 NVM Set 时还恰好属于 1 个 NVM Set |
| 容量标识符清零 | 不支持 NVM Set 时，所有 `NVMSETID` 字段清 0；不支持 EG 时所有 `ENDGID` 字段清 0 |
| 容量与通道是不同关系 | 容量按层级"向下分配"；通道与 Media Unit 之间是"对多的双向连接" |
| 分配入口 | Capacity Management（创建 EG / NVM Set + 分配 Media Unit）；Namespace Management（创建格式化命名空间） |
| 零值语义 | 容量/寿命估算字段 `0` = "未上报"；`FF…F`（全 1）= "饱和"（值 ≥ 该值） |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 容量分配 vs 通道连接 | 分配是层级包含；通道是数据通路，二者独立 |
| 命名空间容量 vs NVM Set 容量 | NS 容量是 NVM Set 容量的"切片"，未分配 NVM Set 容量仍在 NVM Set 账上 |
| TNVMCAP vs UNVMCAP | TNVMCAP 是子系统总容量（账本总额）；UNVMCAP 是当前未分配容量（可用余额） |
| Media Unit vs Namespace | Media Unit 是物理芯片；Namespace 是逻辑卷 |
| Channel vs Domain | Channel 是"数据搬运通道"；Domain 是"共享状态边界"，二者不一一对应 |

## 进阶细节

- **规范 3.8.1 概览**：容量报告涵盖子系统、域、EG、NVM Set、命名空间、Media Unit；"支持哪些层、是否全支持"由控制器决定（规范 3.8.1 / PDF 第 141 页）。
- **分配规则**（规范 3.8.2 / PDF 第 142 页）：
  - NVM Set 容量 → 1 个或多个命名空间；每个 NS 整体在一个 NVM Set 内；不必全部分配。
  - EG 容量 → 1 个或多个 NVM Set（支持 NVM Set 时）；不必全部分配。
  - 域容量 → 1 个或多个 EG（支持 EG 时）；不必全部分配。
  - 每个 Media Unit 恰好属于 1 个 EG；支持 NVM Set 时还恰好属于 1 个 NVM Set。
  - Channel ↔ Media Unit 是 1..* 双向连接。
- **零值约束**（规范 3.8.2）：不支持 NVM Set 时所有 `NVMSETID` 清 0；不支持 EG 时所有 `ENDGID` 清 0。
- **容量创建命令**：
  - Capacity Management（规范 8.1.4）：把域容量分配到 EG；EG 容量分配到 NVM Set；把 Media Unit 分配到 EG / NVM Set。
  - Namespace Management Create（规范 8.1.15）：分配命名空间容量。
- **Figure 86 简单最大带宽**：1 域 1 EG 1 NVM Set × 4 通道 × 4 MU = 16 MU 跨通道并发，单次读/写可同时访问所有 MU。
- **Figure 87 垂直组织**：每个通道的 4 个 MU 各自组成 1 个 EG / 1 个 NVM Set，4 路隔离，每路带宽受单通道限制（规范 3.8.2.2 / PDF 第 144 页）。
- **Figure 88 水平双 NAND**：4 通道各出 1 个 SLC MU + 3 个 QLC MU，分别组 SLC 集合与 QLC 集合；SLC 容量调整因子示例 ≈ 400，QLC ≈ 100（仅是该图示例值，非通用 SLC/QLC 比率）。
- **Figure 89 容量信息选择矩阵**（规范 3.8.2 / PDF 第 146 页）：

| 创建/删除对象 | 系统支持形态 | 查/改的容量信息 |
|------|------|------|
| EG | 无 Domain | 子系统（`TNVMCAP` / `UNVMCAP` / `MEGCAP`） |
| EG | 有 Domain | Domain Attributes Entry |
| NVM Set | 同时支持 NVM Set 与 EG | Endurance Group Information log |
| 格式化 NS | 既无 EG 也无 Domain | NVM 子系统 |
| 格式化 NS | 有 Domain 无 EG | Domain Attributes Entry |
| 格式化 NS | 有 EG 无 NVM Set | Endurance Group Information log |
| 格式化 NS | 支持 NVM Set | NVM Set Attributes Entry |

- **Media Unit Status log**（规范 5.2.1.13 / PDF 第 276 页）：按域选择；描述符按 MUID 升序，含 Domain/EG/NVM Set 归属、容量调整因子、剩余备用、已用百分比、按通道 ID 升序去重排列；多 Domain 子系统中标识符作用域为 Domain 本地、单 Domain 时为子系统全局；`0` 域选择 = 控制器所在域；不可访问的域导致该日志不可用。
- **未选容量配置时的回退**：未选配置时描述符中 EG / NVM Set / 容量调整因子 / 通道计数字段全部清 0；spare 与 percentage-used 是 Media Unit 本地测量，与 EG 值的对应关系"未规定"（规范 5.2.1.13 / PDF 第 277-278 页）。
- **Supported Capacity Configuration List**（规范 5.2.1.14 / PDF 第 278 页）：按域选择；按 CCID 升序去重排列；每条为变长配置 → 变长 EG 配置描述符 → 唯一升序的 NVM Set ID 列表 + 通道配置描述符；EG 描述符含容量/备用/调整因子/寿命估算（"向上取整十亿字节"，0 = 未上报）；通道描述符列出所连 MU；Base 2.1 规定当前 Media Unit Configuration Descriptor 的扩展长度字段为 0（规范 5.2.1.14 / PDF 第 279-281 页）。

## 规范依据

- [容量报告层级与可选性，PDF 第 141 页](../_source/pages/page-141.md)
- [分配、零值标识符、Media Unit 与 Channel，PDF 第 142 页](../_source/pages/page-142.md)
- [Figure 86 简单最大带宽组织，PDF 第 143 页](../_source/pages/page-143.md)
- [Figure 87-88 垂直与双 NAND 组织，PDF 第 144 页](../_source/pages/page-144.md)
- [Figure 89 容量信息选择矩阵，PDF 第 146 页](../_source/pages/page-146.md)
- [Media Unit Status log 描述符，PDF 第 276 页](../_source/pages/page-276.md)
- [Supported Capacity Configuration 描述符层级，PDF 第 278 页](../_source/pages/page-278.md)

## 相关阅读

- [存储资源层次结构](storage-resource-hierarchy.md) - 容量对应资源层级
- [NVM 集与耐久度组](nvm-sets-and-endurance-groups.md) - EG/Set 是容量容器
- [容量管理操作](capacity-management-operations.md) - 容量管理操作入口
- [命名空间管理生命周期](namespace-management-lifecycle.md) - 命名空间创建与容量
- [NVM 格式化生命周期](format-nvm-lifecycle.md) - 格式化影响可用容量
