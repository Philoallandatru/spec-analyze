# 命令与功能锁定（Command and Feature Lockdown）

**命令与功能锁定（Command and Feature Lockdown）** 是 NVM 子系统级的可选安全特性（由 Identify Controller `OACS.CFLS = 1` 标识），允许管理员在运行时按**操作码空间（Scope）**和**访问接口（Interface）**动态禁止或允许特定命令的执行，用于安全加固与多租户权限隔离。[PDF pp. 284-286, 516](../_source/pages/page-284.md)

> **类比理解**：把 Lockdown 想成办公楼的门禁黑名单系统——保安处（Lockdown 命令）可以把某人列入一楼正门（Admin Queue 带内路径）或三楼偏门（Management Endpoint 带外路径）的禁止名单；大厅张贴栏（Log Page `14h`）随时可查"哪些人支持被禁（Lockable）"以及"谁当前被禁（Prohibited）"。

---

## 写入与查询双路径模型

```text
写入端（Admin Queue）                          查询端（Admin Queue）
+-------------------------+                    +-------------------------+
| Lockdown 命令 (24h)     |                    | Get Log Page (LID=14h)  |
|  SCP: 0/2/3/4 (作用域)  |                    |  CNTTS: 00b 可锁定名单   |
|  OFI: 操作码 / FID      |                    |         01b Admin 禁止   |
|  IFC: 00b Admin /       |                    |         10b ME 禁止      |
|       01b Admin+ME /    |                    |  SCP:   0/2/3/4          |
|       10b ME            |                    |  UUID Index (仅 SCP=2h) |
|  PRHBT: 1=禁止, 0=允许  |                    +------------+------------+
|  CDW14: UUID Index      |                                 |
+------------+------------+                                 v
             |                                 返回 512B 结构：
             v                                  CFILA(回显) + LNGTH +
   [子系统范围同步强制执行]                     CFIL(升序操作码/FID列表)
   所有 Controller & 所有 ME
             |
             +-- Admin Queue 被禁 -> 返回 "Command Prohibited by Command and Feature Lockdown"
             `-- ME 路径被禁     -> 返回 NVMe-MI "Access Denied"
```

### 选择器参数（Figures 265-266, 337-338）

| 维度 | 取值与含义 |
|---|---|
| **Scope (`SCP`)** | `0h` Admin 命令 opcode；`2h` Set Features Feature ID（可配 `CDW14` UUID Index）；`3h` Management Interface (MI) opcode；`4h` PCIe 命令 opcode（`1h` 与 `5h..Fh` 保留） |
| **Contents (`CNTTS`)** | 仅用于 Log Page `14h`：`00b` 查询可锁定（Lockable）名单；`01b` 查询 Admin Queue 路径已禁止名单；`10b` 查询 Management Endpoint (ME) 路径已禁止名单 |
| **Interface (`IFC`)** | 仅用于 Lockdown 命令：`00b` 仅 Admin SQ；`01b` Admin SQ + ME；`10b` 仅 ME |
| **Action (`PRHBT`)** | `1` = Prohibit（禁止）；`0` = Allow（允许） |

---

## 核心规则与错误处理

1. **子系统级生效与路径独立**：一次 Prohibit 会在子系统内**所有控制器和所有管理端点**上同步强制执行；但同一操作在 Admin Queue（带内）和 ME（带外）上可独立配置不同的禁止状态。[PDF pp. 370-371, 516](../_source/pages/page-370.md)
2. **持久性与幂等性**：禁止状态会穿越控制器复位持续保持，直到**整个子系统电源循环（Power Cycle）**或执行显式 `Allow`；重复执行相同的 Prohibit 或 Allow 均为幂等操作，正常返回成功。[PDF p. 516](../_source/pages/page-516.md)
3. **非法组合与报错**：
   - 目标不支持锁定 → `Prohibition of Command Execution Not Supported (28h)`。
   - `Scope = 4h`（PCIe）与 Admin 路径组合（`IFC = 00b/01b`）→ `Invalid Field in Command`（PCIe 命令只能在 ME 路径锁定）。
   - 子系统无 Management Endpoint 但选择了 ME 路径 → `Invalid Field in Command`。[PDF pp. 285, 371-372](../_source/pages/page-285.md)

> **常见排错**：若禁用了 `Format NVM` 但现场仍被格式化，需检查 `IFC` 是否仅禁了 Admin Queue（`00b`）而未同时禁止带外 ME 路径（`01b`）。

---

## 与 Effects Catalogs 及易混概念对比

| 易混概念 | 实际区别 |
|---|---|
| **Lockdown vs Effects Catalogs** | `Effects Catalogs` 描述**静态能力**（是否支持、有什么副作用）；`Lockdown` 描述**动态权限**（当前是否被禁止）。一个命令可以"被支持"但"被禁止"。[PDF pp. 281-286](../_source/pages/page-281.md) |
| **Lockable (`00b`) vs Prohibited (`01b`/`10b`)** | `Lockable` 是"能否被锁定"的能力清单；`Prohibited` 是"此刻已被禁止"的生效清单 |
| **In-band vs Out-of-band** | `In-band` 指主机通过 Admin SQ 提交；`Out-of-band` 指通过管理端点（如 PCIe VDM / 2-Wire）提交 |
| **单条 Allow vs 电源循环** | `Allow` 仅解除指定条目；子系统电源循环会清空所有禁止规则 |

---

## 关联与引申概念

- **OACS（Optional Admin Command Support）**：Identify Controller 中的可选管理命令支持位图，`OACS.CFLS`（bit 25）= 1 表示同时支持 Lockdown 命令与 Log Page `14h`。
- **Management Endpoint（ME）与 NVMe-MI**：带外管理端点与管理接口协议，独立于主机 Admin Queue 运行。
- **UUID Index**：当控制器和 Set Features 均支持厂商 UUID 时，用于精确区分不同厂商定义的 Feature ID。
- **延伸阅读**：
  - [command-effects-and-support.md](command-effects-and-support.md) — 静态命令/特性支持与效果日志
  - [admin-command-model.md](admin-command-model.md) — 可被锁定的 Admin 命令空间
  - [format-nvm-lifecycle.md](format-nvm-lifecycle.md) — 生产环境中常被锁定的高危操作
  - [command-abort-semantics.md](command-abort-semantics.md) — Abort 命令本身也可受 Lockdown 管控

---

## 规范依据

- [Log Page 14h 目的、参数选择器与返回布局（Figures 265-266），PDF pp. 284-286](../_source/pages/page-284.md)
- [Effects Catalogs 与 Lockdown 关系，PDF pp. 281-286](../_source/pages/page-281.md)
- [Lockdown 命令定义、CDW10/CDW14 与状态码（Figures 337-339），PDF pp. 370-372](../_source/pages/page-370.md)
- [子系统级强制执行、持久性与接口语义（Section 8.1.5），PDF p. 516](../_source/pages/page-516.md)
