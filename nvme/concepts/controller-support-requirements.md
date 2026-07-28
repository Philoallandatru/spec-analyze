# 控制器支持要求（Controller Support Requirements）

## 一句话说明

NVMe 把"支持什么"切成**命令、Log 页、Feature** 三个维度，每个维度对 I/O 控制器、Admin 控制器、Discovery 控制器分别给出 `M/O/P` 判定，附带的注释往往会把"Optional"变成"Mandatory"或"Prohibited"。

## 生活化类比

把控制器支持要求想成**三本独立的"营业资格清单"**：

- **第一本：能接哪些单（命令清单）**——每种控制器类型对应一份"必接/可拒/禁接"的工单目录。
- **第二本：能披露哪些信息（Log 页清单）**——必须能提供、可以提供、禁止提供三档。
- **第三本：能调哪些配置（Feature 清单）**——同上，附带 Get/Set Features 也要能配套。

三本清单**互相独立**——一个控制器即使没列在 Feature 清单上，命令清单里也不会自动有对应支持；清单下面的**注释**（Notes 1-10）就像边检的特别条款，会把"可选"临时变"必选"（例如支持保留功能就强制要求保留类命令）。

主机不能从行名直接推断支持与否，必须**逐格查表 + 看注释**——这是规范反复强调的反直觉点。

## 工作流程

```text
             控制器类型
         +-----------+-----------+-----------+
能力面   | I/O       | Admin.    | Discovery |
+--------+-----------+-----------+-----------+
|命令    | 大而全    | 管理类    | 发现类    |
|        | I/O 命令集具体规格补充                  |
|        | 由 Commands Supported and Effects 发现 |
+--------+-----------+-----------+-----------+
|Log 页  | I/O/健康 | 管理     | 发现      |
|        | 由 Supported Log Pages 发现            |
+--------+-----------+-----------+-----------+
|Feature | I/O/控制 | 管理     | 发现      |
|        | 由 Feature Identifiers Supported 发现  |
+--------+-----------+-----------+-----------+

每一格 = M（Mandatory）/ O（Optional）/ P（Prohibited）/ 条件
注脚 1-10 是规则的"补丁"，是契约的一部分
```

简化说明：上图只是导航地图，不是 Figures 28-32 的替代品。

## 初学者案例

**场景：Discovery 控制器为什么不能发普通读命令？**

1. 主机通过 `GetLogPage(LID=00h)` 读到 Supported Log Pages 矩阵。
2. 在"Discovery 控制器"那列查到：`Get Log Page` 本身是 M；普通 I/O 命令在 Discovery 控制器上是 **P（Prohibited）**。
3. 主机如果尝试发一条 Read 到 Discovery 控制器：返回 `Invalid Opcode`（操作码无效）。
4. 正确做法：Discovery 控制器只接 Fabrics Connect、Get Log Page（Discovery Log）、Discovery 相关命令，**不暴露命名空间数据路径**。

> 故障速查：管理员发出"看似合理"的命令被拒，先看 Figure 28-32 矩阵的"控制器类型列"再排查——**不是命令本身有问题，是发错了控制器角色**。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 三类能力 | 命令支持、Log 页支持、Feature 支持分别独立定义、各有各的发现日志页 |
| 三种判定 | `M` Mandatory、`O` Optional、`P` Prohibited；不允许只凭行名推断 |
| 三个控制器类型 | I/O、Administrative、Discovery 三列都要看，不能省略 |
| 注释是契约的一部分 | 矩阵下方的"Notes 1-10"会把某些 `O` 提升为 `M`，或反向变 `P` |
| 传输决定命令 | PCIe 上 Fabrics 命令被禁；Fabrics 上 Property Get/Set + Connect 必选 |
| I/O 命令集独立 | I/O Command Set 自己的命令支持要求在 I/O Command Set 规范里，**不并入基础矩阵** |
| Discovery 行为 | Discovery 控制器必须能完成发现任务，**禁止普通 I/O 数据路径** |
| Get/Set Features 配套 | 任何支持的 Feature 都要保证能 Get 和 Set |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 命令支持 vs Feature 支持 | 两者独立的发现机制：Commands Supported and Effects vs Feature Identifiers Supported and Effects |
| 矩阵里的 `O` vs 真正的"可选" | 矩阵 `O` 是"基础规范层面"的可选；附注常常把它变成"实际必选" |
| 控制器支持 vs 子系统支持 | 矩阵按"控制器类型"分；某些条目还受子系统类型（如 Exported NVM Subsystem）约束 |
| 同一 Feature 在不同控制器类型 | Feature 列在三种控制器上**可能不同**；不要假设 Admin 控制器上支持的 Feature 也能在 I/O 控制器上支持 |
| Discovery 控制器 vs I/O 控制器 | Discovery 控制器**不接普通 I/O**，只用发现类命令/日志；命名空间路径对它而言是被禁的 |
| Mandatory for PCIe | "PCIe 必选"的项可能在 Fabrics 上是 `P`，是**传输强制的**而非全平台 |
| Note 编号作用域 | 每条 Note 解释一行或一组，**不要跨行套用** |

## 进阶细节

- **Figures 28-32 的位置**：Admin 命令矩阵 Figure 28 在规范第 65-66 页；Fabrics 与通用 I/O 命令在 66-67 页；Log 页矩阵在 67-69 页；Feature 矩阵在 69-70 页。
- **常见条件化条目举例**：
  - 保留（Reservation）相关命令在控制器/命名空间支持保留时变必选。
  - Flexible Data Placement 相关命令/日志在 `FDPS=1` 时变必选。
  - 持久 Discovery 连接要求 Keep Alive 与 AER 支持。
  - Subsystem Health Snapshot Polling 依赖 Telemetry / Firmware Commit / SMART 关键警告支持。
- **跨控制器类型差异**：
  - I/O 控制器：Create I/O CQ/SQ 必选，Fabrics 命令被禁。
  - Admin 控制器：不支持 I/O 队列，因此 I/O 命令集专属命令在 Admin 控制器上不存在。
  - Discovery 控制器：Disconnection 对 I/O 是可选、对 Admin/Discovery 是禁。
- **不计入基础矩阵的能力**：
  - 子系统角色（Exported NVM Subsystem）见规范 8.3.3。
  - I/O Command Set 专属命令在每个 I/O Command Set 规范里。
  - NVMe-MI 相关的项在 NVM Express Management Interface Specification。
- **支持的运行时发现通道**：
  - 命令：`Commands Supported and Effects log page`（LID 07h）。
  - Log 页：`Supported Log Pages`（LID 00h）。
  - Feature：`Feature Identifiers Supported and Effects`（LID 1Ah）。
- **Transport-dependent overrides**：很多条件（PCIe 必选、Fabrics 禁、Discovery 必选）都是"行内注释"给出的，先看 Note 编号再看单元格判定。

## 规范依据

- [命令支持要求总述与 Figure 28，PDF 第 65–66 页](../_source/pages/page-065.md)
- [Fabrics 与通用 I/O 命令矩阵，PDF 第 66–67 页](../_source/pages/page-066.md)
- [Log 页支持矩阵，PDF 第 67–69 页](../_source/pages/page-067.md)
- [Feature 支持矩阵，PDF 第 69–70 页](../_source/pages/page-069.md)

## 相关阅读

- [controller-properties.md](controller-properties.md) - 能力位定义在属性空间
- [common-controller-features.md](common-controller-features.md) - 通用 Feature 集合背景
- [command-effects-and-support.md](command-effects-and-support.md) - 命令支持的运行时报告
- [log-page-retrieval.md](log-page-retrieval.md) - Log Page 矩阵的读取接口
