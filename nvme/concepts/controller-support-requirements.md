# 控制器支持要求（Controller Support Requirements）

NVMe 在**命令（Commands）、Log 页（Log Pages）、Feature（特性）**三个相互独立的维度上，按控制器角色（I/O、Administrative、Discovery）规定了 `M`（必选 Mandatory）、`O`（可选 Optional）或 `P`（禁止 Prohibited）；**矩阵下方的注释（Notes）往往会在特定条件下把 `O` 变为 `M` 或 `P`**，因此绝不能只看行名推断支持情况。[PDF pp. 65-70](../_source/pages/page-065.md)

可以把它想成**三本独立的“营业资格清单”**：分别规定某类控制器能接哪些工单（命令）、能披露哪些报表（Log 页）、能调哪些开关（Feature），而清单底部的“特别条款”就是根据附加能力动态生效的补充契约。

## 三维支持矩阵导航

```text
                          控制器角色（Figures 28-32）
                 +-------------+---------------+-------------+
能力维度         | I/O         | Administrative| Discovery   |
+----------------+-------------+---------------+-------------+
| Admin 命令     | 大而全      | 管理类        | 发现类      |
| Fabrics 命令   | 仅 Fabrics 传输适用；PCIe 全部禁止          |
| I/O 命令       | 数据路径    | 禁止          | 禁止        |
| Log 页         | I/O / 健康  | 管理          | 发现        |
| Feature        | I/O / 控制  | 管理          | 发现        |
+----------------+-------------+---------------+-------------+
每格判定 = M / O / P / 条件；I/O 命令集专属命令另见各 I/O 命令集规范
```

[PDF pp. 65-70](../_source/pages/page-065.md)

| 维度 | 运行时发现通道 | 关键契约 |
|---|---|---|
| **命令** | `Commands Supported and Effects` 日志页（LID `07h`） | I/O 命令集专属要求不并入 Base 矩阵，另见对应命令集规范 |
| **Log 页** | `Supported Log Pages` 日志页（LID `00h`） | 作用域和前置能力可能把 `O` 提升为 `M` 或改为 `P` |
| **Feature** | `Feature Identifiers Supported and Effects` 日志页（LID `1Ah`） | 支持任意 Feature 均要求 `Get Features` 与 `Set Features` 配套可用 |

## 传输、角色与条件性边界

- **传输边界**：
  - **NVMe over PCIe**：Fabrics 命令全部**禁止（P）**；I/O 控制器必选 `Create/Delete I/O SQ/CQ`。[PDF pp. 65-67](../_source/pages/page-065.md)
  - **NVMe over Fabrics**：`Property Get/Set` 与 `Connect` 对所有控制器类型**必选（M）**；`Disconnect` 对 I/O 控制器可选、对 Admin/Discovery 禁止。
- **角色边界（以 Discovery 控制器为例）**：Discovery 控制器仅处理发现类命令与日志（如 Discovery Log Page），**禁止暴露命名空间与普通 I/O 数据路径**；若向其发送普通 Read 命令，将返回 `Invalid Command Opcode`。[PDF pp. 65-68](../_source/pages/page-065.md)
- **条件性覆盖（Notes 1-10）**：标为 `O` 的条目常因前置能力变为 `M`——例如：支持 Reservations 时保留类命令变必选；`FDPS=1` 时 FDP 命令/日志变必选；持久 Discovery 连接要求支持 Keep Alive 与 AER。[PDF pp. 66-70](../_source/pages/page-066.md)

## 概念辨析

| 容易混淆 | 实际区别 |
|---|---|
| 矩阵里的 `O` vs 自由可选 | `O` 仅表示 Base 规范层面非强制，附注条件常使其在实际产品中变为 `M` |
| 命令支持 vs Feature 支持 | 两者拥有各自独立的发现日志页（LID `07h` vs `1Ah`），互不自动推导 |
| 跨控制器类型通用性 | 同一 Feature 在 Admin 控制器上的判定与在 I/O 控制器上可能完全不同 |

## 关联与引申概念

- **控制器属性（[Controller Properties](controller-properties.md)）**：能力位与属性寄存器（如 `CAP`）在控制平面的定义。
- **命令效果与支持（[Command Effects and Support](command-effects-and-support.md)）**：通过 LID `07h` 在运行时动态发现控制器实际支持的命令及其副作用标志。
- **日志页读取（[Log Page Retrieval](log-page-retrieval.md)）**：`Get Log Page` 命令如何读取 LID `00h`/`07h`/`1Ah` 等发现与状态日志。
- **通用控制器特性（[Common Controller Features](common-controller-features.md)）**：各 Feature ID 的具体配置语义。
- **NVMe-MI（Management Interface）**：带外管理接口的支持要求独立定义在 NVMe-MI 规范中，不计入本 Base 矩阵。

## 规范依据

- [Admin 命令支持矩阵（Figure 28），PDF 第 65-66 页](../_source/pages/page-065.md)
- [Fabrics 与通用 I/O 命令矩阵，PDF 第 66-67 页](../_source/pages/page-066.md)
- [Log 页支持矩阵，PDF 第 67-69 页](../_source/pages/page-067.md)
- [Feature 支持矩阵，PDF 第 69-70 页](../_source/pages/page-069.md)
