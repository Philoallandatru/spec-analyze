# 命名空间预留生命周期（Namespace Reservation Lifecycle）

**命名空间预留（Namespace Reservation）** 是 NVMe 用于协调多主机共享同一命名空间时并发访问的控制机制。主机通过 64 位 **Reservation Key（预留密钥）** 向命名空间注册身份，并竞争单一的 **Reservation Holder（预留持有者）** 角色，从而按策略限制读/写权限。[PDF pp. 489-495](../_source/pages/page-489.md)

> **类比**：把共享命名空间想成**一间共享会议室**——`Register` 是员工在前台登记工卡（64 位 Key）；`Acquire` 是刷卡预订成为当前会议主持人（Holder）；预订类型决定“别人能不能进来看（读）或发言（写）”；`Release` 是结束主持但保留门禁登记；`Clear` 是清空所有登记和预订；`Preempt` 则是强行接管主持权。

## 状态流转与核心操作

```text
[未注册 (unregistered)]
     │ Register (注册 Key；IEKEY=1 可跳过当前 Key 校验)
     ▼
[注册者 (registrant)] ── Acquire(type) ──→ [预留持有者 (reservation holder)]
     │                       │                     │
     │ Replace / Unregister  │ Preempt /           │ Release (保留注册者)
     ▼                       │ Preempt+Abort       ▼
[未注册]                     ▼                [注册者]
                      目标 Key 被注销              │
                                             Clear ──→ 清空预留与所有注册
```

| 操作命令 | 核心行为与 Key/IEKEY 规则 |
|---|---|
| **Reservation Register** | 注册、注销或替换提交主机的 64 位 Key；`IEKEY=1` 允许跳过对当前 Key 的校验。同一条命令可同时保持、清除或设置 `PTPL`（Persist Through Power Loss）；若 Reservation Persistence Feature 可保存，则同步更新当前值与已保存值。[PDF pp. 490-492](../_source/pages/page-490.md) |
| **Reservation Acquire** | 按指定类型获取预留，校验当前 Key（**`IEKEY=1` 无效**）。可选 `Preempt` 指定要注销的竞争 Key；`Preempt and Abort` 还会中止该命名空间上未完成的命令。[PDF pp. 489-490](../_source/pages/page-489.md) |
| **Reservation Release** | 释放当前预留但**保留注册关系**，要求提供的预留类型与当前一致；校验当前 Key（`IEKEY=1` 无效）。[PDF pp. 492-493](../_source/pages/page-492.md) |
| **Reservation Clear** | 一次性移除当前预留及**所有注册状态**；校验当前 Key（`IEKEY=1` 无效）。[PDF pp. 492-493](../_source/pages/page-492.md) |

## 6 种预留类型与访问权限

预留类型由“锁写（Write Exclusive）vs 锁读写（Exclusive Access）”和“作用范围（Holder-only / Registrants-only / All Registrants）”组合而成（`All Registrants` 模式下所有已注册主机均视同持有者，无单一独占者）：[PDF pp. 489-490](../_source/pages/page-489.md)

| 预留类型 | 持有者（Holder） | 其他注册者（Registrants） | 未注册主机 |
|---|---|---|---|
| **Write Exclusive** | R/W | 只读（写返回 Conflict） | 只读（写返回 Conflict） |
| **Exclusive Access** | R/W | 无访问权限 | 无访问权限 |
| **Write Exclusive – Registrants Only** | R/W | R/W | 只读（写返回 Conflict） |
| **Exclusive Access – Registrants Only** | R/W | R/W | 无访问权限 |
| **Write Exclusive – All Registrants** | R/W | R/W | 只读（写返回 Conflict） |
| **Exclusive Access – All Registrants** | R/W | R/W | 无访问权限 |

## 掉电持久化（PTPL）与分散命名空间握手

- **PTPL（Persist Through Power Loss）**：[PDF pp. 491, 494-495](../_source/pages/page-491.md)
  - `PTPL=0`：掉电后预留被释放，所有注册者被清空。
  - `PTPL=1`：掉电后预留与注册者关系均持久保留。
- **分散命名空间握手（`DISNSRS`）**：当主机启用了 Host Dispersed Namespace Support 时，每条预留命令必须将 `DISNSRS` 设为 `1`，显式声明自己能够解读跨参与方控制器哨兵值 `FFFDh`；若在分散命名空间上将 `DISNSRS` 清零，命令会被拒绝并返回 `Namespace Is Dispersed`。[PDF pp. 490-494](../_source/pages/page-490.md)

## Reservation Report 与代次计数器

**Reservation Report** 返回一个由前缀长度控制的当前预留快照：[PDF pp. 493-496](../_source/pages/page-493.md)

- **头部信息**：包含可回绕的 **Generation Counter（代次计数器）**（在注册变更、Clear、Preempt/Preempt-and-Abort 成功后递增）、当前预留类型、注册者数量以及 PTPL 状态。
- **注册者记录格式**：必须与主机所选的 Host Identifier 宽度匹配：
  - **基本格式（24 字节）**：承载 64 位 Host Identifier。
  - **扩展格式（64 字节）**：承载 128 位 Host Identifier。
- **CNTLID 哨兵值**：在注册者记录中，`CNTLID=FFFFh` 表示该主机在本子系统内没有控制器关联；`FFFDh` 表示该控制器位于分散命名空间的另一个参与子系统中。[PDF pp. 494-495](../_source/pages/page-494.md)

## 关联与引申概念

- **PTPL（Persist Through Power Loss）**：控制预留和注册状态是否在设备断电重启后继续保留的持久化开关。
- **Generation Counter（代次计数器）**：随预留/注册关系每次成功变更而递增的计数器，主机可通过比对代次检测并发修改。
- **相关文档**：
  - [reservation-notification-log.md](reservation-notification-log.md) — 预留被抢占/释放后的异步通知日志
  - [identity-name-and-list-formats.md](identity-name-and-list-formats.md) — Host Identifier 宽度（64-bit / 128-bit）与报告格式选择
  - [domains-and-divisions.md](domains-and-divisions.md) — 分散命名空间与跨参与方控制器标识
  - [persistent-event-log.md](persistent-event-log.md) — 跨断电事件记录

## 规范依据

- [Acquire 动作、Key 校验、分散命名空间支持与预留类型，PDF pp. 489-490](../_source/pages/page-489.md)
- [Register 动作与 PTPL 副作用，PDF pp. 490-492](../_source/pages/page-490.md)
- [Release / Clear 动作与报告格式选择，PDF pp. 492-494](../_source/pages/page-492.md)
- [Reservation Status 头部与注册者记录，PDF pp. 494-495](../_source/pages/page-494.md)
- [基本与扩展注册者记录的完整字段，PDF pp. 495-496](../_source/pages/page-495.md)
