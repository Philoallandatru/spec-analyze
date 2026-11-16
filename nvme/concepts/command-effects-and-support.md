# 命令效果与支持（Command Effects and Support）

**Commands Supported and Effects Log（`LID = 05h`）** 是一个固定 4,096 字节的日志结构，为每个 Admin 操作码（`0..255`）和当前所选 I/O 命令集的每个 I/O 操作码（`0..255`）各提供一个 32 位描述符，用于告知主机**"控制器支持哪些命令，以及执行它们可能对系统产生什么副作用"**。[PDF p. 236](../_source/pages/page-236.md)

> **类比理解**：把它想成餐厅菜单上的"副作用说明"——每道菜旁边写着"是否售罄（`CSUPP`）"、"影响包间还是整栋楼（`Scope`）"、"是否要暂停上菜（`CSE` 串行化建议）"。服务员（主机）点大菜（如 Format NVM）前先扫一眼说明，避免业务进行到一半突然被清场。

---

## 日志结构与主机协调流程

```text
Get Log Page (LID=05h, 4096 字节)
   ├─ 字节    0..1023 : ACS0..ACS255   (Admin Opcode 0..255 的 32 位描述符)
   ├─ 字节 1024..2047 : IOCS0..IOCS255 (I/O Opcode 0..255 的 32 位描述符)
   └─ 字节 2048..4095 : 保留
            |
            v
主机协调流程（针对涉及能力/清单变更的命令）：
  1. 解析目标命令描述符（保守对待所有置位的标志）
  2. 暂停对受影响命名空间/资源的 I/O，排空在途请求
  3. 提交命令并等待 CQE 完成
  4. 重新执行 Identify / 重新枚举命名空间，刷新本地视图后恢复业务
```

- **I/O 命令集选择**：日志中的 `IOCS` 部分按当前激活的 I/O 命令集解释——通常由控制器配置寄存器 `CC.CSS` 选定；当 `CC.CSS = 110b` 时，由 `CDW14` 中的命令集标识符（CSI）动态指定。[PDF p. 236](../_source/pages/page-236.md)
- **多控制器协调**：当同一个命名空间连接到多个控制器时，各关联主机需要跨控制器协调命令提交，以满足描述符中的串行化建议。[PDF pp. 236-238](../_source/pages/page-236.md)

---

## 32 位描述符字段详解（Figure 210）

| 字段组 | 规范含义 |
|---|---|
| **`CSUPP`（bit 0）** | 命令支持位。为 `0` 表示不支持（此时其余所有位必须为 `0`）；为 `1` 表示支持（其余位全 `0` 表示"支持且无特殊副作用"） |
| **`Scope`（影响范围）** | 命令**可能**影响的资源边界：Namespace、Controller、NVM Set、Endurance Group、Domain、NVM Subsystem（**可同时设置多个位**） |
| **`USS`** | UUID Selection Support，指示该命令是否支持按 UUID 选择特定对象 |
| **`CSE / CSER`** | 命令提交与执行建议：建议在命名空间范围或全局范围内串行化执行；若支持非零的放宽策略（`CSER`），则覆盖默认 `CSE` |
| **Capability / Data Effects** | 指示可能产生的变更：控制器能力变更、命名空间清单变更、单个命名空间能力变更、逻辑块用户数据变更 |

> **保守原则**：描述符反映的是命令**可能**产生的最大影响范围（包括可选行为），主机软件必须保守地按所有已置位的标志做协调。[PDF pp. 237-238](../_source/pages/page-237.md)

---

## 特性与 NVMe-MI 效果日志对比

除命令效果日志外，NVMe 还提供 **Feature Identifiers Supported and Effects Log** 与 **NVMe-MI Opcode Effects Log**，按接口实例（Admin Queue / PCIe VDM Endpoint / 2-Wire Endpoint）独立报告支持与效果（Figures 261-264）：[PDF pp. 281-284](../_source/pages/page-281.md)

| 日志类型 | 支持位 | Scope 范围规则 | 额外特性 |
|---|---|---|---|
| **命令效果日志 (`LID=05h`)** | `CSUPP` | **可同时设置多个范围位** | 区分 Admin 与 I/O opcode，含 `CSE/CSER` 串行化建议 |
| **特性效果日志 (FID)** | `FSUPP` | 非零时**恰好只能有一位为 1**（含 NS / Controller / NVM Set / EG / Domain / Subsystem / CDQ） | 接口实例特定；报告 Get/Set Features 是否接受 UUID（`UIDX`）选择 |
| **NVMe-MI 操作码日志** | `CSUPP` | 非零时**恰好只能有一位为 1**（不含 CDQ） | 索引为 NVMe-MI Opcode `00h..FFh` |

---

## 易混概念对比

| 易混概念 | 实际区别 |
|---|---|
| **`CSUPP = 0` vs 其余字段 = 0** | `CSUPP = 0` 表示命令**不支持**；`CSUPP = 1` 且其余字段全 `0` 表示命令**支持但无特殊副作用** |
| **`CSE` vs `CSER`** | `CSE` 是默认串行化建议；`CSER` 是放宽策略，非零时取代 `CSE` |
| **`Scope` vs `Capability Effects`** | `Scope` 回答"影响哪类资源边界"；`Capability Effects` 回答"资源发生了什么变化（能力/清单/数据）" |
| **Effects Catalogs vs Lockdown** | 前者描述"是否支持与可能副作用"（静态能力）；后者描述"当前是否被禁止"（动态权限） |

---

## 关联与引申概念

- **`CC.CSS` 与 `CSI`**：`CC.CSS` 选择控制器使用的 I/O 命令集，`CSI`（Command Set Identifier）用于多命令集并存时动态指定具体命令集。
- **UUID Index（`UIDX`）**：在支持 UUID 选择的控制器上，用于筛选厂商特定的 Feature 或命令对象。
- **命令排序与仲裁**：硬件/控制器层面的队列调度机制，与效果日志提供的软件层串行化建议互补，详见 [command-ordering-and-arbitration.md](command-ordering-and-arbitration.md)。
- **命名空间标识符（NSID）**：清单变更效果标志常提示主机需要重新枚举 NSID，详见 [namespace-identifiers.md](namespace-identifiers.md)。
- **延伸阅读**：
  - [admin-command-model.md](admin-command-model.md) — Admin opcode 矩阵与命令集分类
  - [command-and-feature-lockdown.md](command-and-feature-lockdown.md) — 运行时命令与特性禁止机制
  - [format-nvm-lifecycle.md](format-nvm-lifecycle.md) — Format NVM 的效果协调实战
  - [command-abort-semantics.md](command-abort-semantics.md) — Abort 的效果描述符与屏障

---

## 规范依据

- [Commands Supported and Effects Log 布局、命令集选择与主机协调（Figure 209），PDF p. 236](../_source/pages/page-236.md)
- [描述符字段定义 Scope 与 CSE（Figure 210），PDF p. 237](../_source/pages/page-237.md)
- [放宽策略与能力/数据变更标志，PDF p. 238](../_source/pages/page-238.md)
- [Feature Identifiers Supported and Effects（Figures 261-262），PDF pp. 281-283](../_source/pages/page-281.md)
- [NVMe-MI 操作码效果日志（Figures 263-264），PDF pp. 283-284](../_source/pages/page-283.md)
- [命令排序与仲裁背景，PDF pp. 120-123](../_source/pages/page-120.md)
- [命名空间标识符与清单枚举，PDF pp. 235-238](../_source/pages/page-235.md)
