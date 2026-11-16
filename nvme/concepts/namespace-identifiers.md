# 命名空间标识符（Namespace Identifiers, NSID）

**NSID（Namespace Identifier）** 是主机在 NVMe 命令（SQE）中用来指定目标命名空间的数字句柄。它**仅在当前子系统会话内有效**，并非命名空间的永久身份；跨重启或跨路径追踪同一物理命名空间需依赖 UUID / NGUID / EUI64 等持久标识符。[PDF pp. 97-99](../_source/pages/page-097.md)

> **类比**：把 NSID 想成**酒店房间号**——每次入住（挂载）可能分到不同房号，但你手里的**身份证/门卡永久编号（UUID）**不会变；预订了房间（子系统已分配）却没办入住（未挂接到当前控制器），房间号对你依然不可用。

## NSID 的三个独立维度

```text
子系统视角（Allocation State）         控制器视角（Attachment State）
┌──────────────────────────────────┐  ┌─────────────────────────────┐
│ NSID 1: 已分配 (allocated)  ─────┼─→│ 控制器 A: active  (已挂接)   │
│                                  └─→│ 控制器 B: inactive (未挂接) │
│ NSID 2: 已分配 (allocated)  ─────┬─→│ 控制器 A: active            │
│                                  └─→│ 控制器 B: active  (共享)    │
│ NSID 3: 未分配 (unallocated) ────┴─→│ 所有控制器: inactive        │
└──────────────────────────────────┘  └─────────────────────────────┘
```

1. **数值空间（Numeric Space）**：
   - `0h`：**无效值（Invalid）**，绝非广播值。
   - `1h ~ NN`：**有效范围（Valid Range）**，`NN` 为 Identify Controller 报告的最大有效 NSID 值（注意 `NN` 可远大于同时分配的最大命名空间数 `MNAN`，因此 NSID 空间可以是稀疏的）。[PDF pp. 97-99](../_source/pages/page-097.md)
   - `NN+1 ~ FFFFFFFEh`：**无效值（Invalid）**。
   - `FFFFFFFFh`：**广播值（Broadcast）**，向该命令允许操作的所有命名空间发送命令（它不是普通有效 NSID）。[PDF p. 97](../_source/pages/page-097.md)
2. **分配状态（子系统级）**：`allocated`（已指向真实存在的命名空间）vs `unallocated`（空闲编号）。[PDF pp. 97-98](../_source/pages/page-097.md)
3. **挂接状态（控制器级）**：`active`（已挂接到当前控制器，可访问）vs `inactive`（未挂接到当前控制器，不可访问；未分配的 NSID 对所有控制器必为 `inactive`）。[PDF pp. 97-98](../_source/pages/page-097.md)

## 命令错误处理与发现机制

向控制器发送带 NSID 的命令时，常见响应与排查如下：[PDF p. 97](../_source/pages/page-097.md)

| 命令中的 NSID 状态 | 控制器行为 / 返回错误 | 常见原因与排查 |
|---|---|---|
| **active** | 正常执行 | 已分配且已挂接到当前控制器 |
| **inactive** | 拒绝：`Invalid Field in Command` | 命名空间在子系统存在（`nvme list-ns -a` 可见），但未 `attach-ns` 到本控制器 |
| **invalid**（数值越界或为 `0h`） | 拒绝：`Invalid Namespace or Format` | 填入了 `0h` 或超出 `NN` 的值 |
| **FFFFFFFFh** | 广播至命令允许的所有命名空间 | 仅用于支持广播语义的管理/特定命令 |

### Identify 枚举与分页

主机通过 `Identify` 命令的不同 `CNS`（Controller or Namespace Structure）值发现命名空间，每次列表查询最多返回 1024 个 NSID；若未读完，则以返回的最后一个 NSID 作为下次请求的起点进行分页：[PDF pp. 98-99](../_source/pages/page-098.md)

| 发现目标 | CNS 值 | 返回内容 |
|---|---|---|
| 单个活跃命名空间详情 | `CNS=0h` | 指定 NSID 的 Identify Namespace 数据结构 |
| 当前控制器的活跃 NSID 列表 | `CNS=2h` | 当前控制器 `active` 的 NSID 列表（≤ 1024 条） |
| 子系统已分配的 NSID 列表 | `CNS=10h` | 子系统级 `allocated` 的 NSID 列表（≤ 1024 条） |

## 持久身份、一致性与队列关系

- **持久身份优先级**：由于 NSID 在重启或重新分配后可能变化，跨路径/跨会话识别同一命名空间应按优先级选用：**UUID**（128-bit，RFC 9562，最高） > **NGUID**（128-bit） > **EUI64**（64-bit）；`UIDREUSE` 字段用于提示 NGUID/EUI64 是否允许重用。[PDF p. 99](../_source/pages/page-099.md)
- **多控制器 NSID 一致性**：若子系统支持 Namespace Management、ANA Reporting 或 NVM Sets 之一，同一 NSID 在所有控制器上必指向同一物理命名空间；否则仅保证共享命名空间的 NSID 在子系统内一致，私有命名空间跨控制器可不唯一。[PDF p. 98](../_source/pages/page-098.md)
- **与队列无固定绑定**：命名空间与提交队列（SQ）之间没有绑定关系，任一 I/O SQ 都可访问该控制器上所有已挂接的命名空间（主机可按 CPU 核心或业务优先级自由调度）。[PDF p. 99](../_source/pages/page-099.md)

## 变更通知与溢出处理

**Changed Attached Namespace List（日志页 `05h`）** 记录自上次成功读取以来发生 Identify 变化、挂接、解挂或被删除的命名空间：[PDF p. 235](../_source/pages/page-235.md)

```text
正常情况 (变更数 ≤ 1024):  [ NSID 1 | NSID 2 | ... | NSID N ]
溢出情况 (变更数 > 1024):  [ FFFFFFFFh (溢出标记) | 0 | ... | 0 ]  ──→ 主机必须全量重新枚举
```

## 关联与引申概念

- **SQE（Submission Queue Entry，提交队列条目）**：主机向控制器下发命令的 64 字节结构，其中 DW1 即用于填入目标 `NSID`。
- **CNS（Controller or Namespace Structure）**：`Identify` 命令中的选择字段，决定返回控制器信息、单个命名空间详情还是 NSID 列表。
- **相关文档**：
  - [namespace.md](namespace.md) — 命名空间基础模型
  - [namespace-access-models.md](namespace-access-models.md) — 共享/私有模型下的 NSID 一致性
  - [namespace-management-lifecycle.md](namespace-management-lifecycle.md) — 分配与挂接的生命周期操作
  - [namespace-topology-and-change-logs.md](namespace-topology-and-change-logs.md) — 拓扑变更日志的完整机制

## 规范依据

- [NSID 数值空间、分配与附加状态，PDF pp. 97-98](../_source/pages/page-097.md)
- [CNS 枚举、唯一性与跨控制器一致性，PDF pp. 98-99](../_source/pages/page-098.md)
- [持久身份标识符与队列独立性，PDF p. 99](../_source/pages/page-099.md)
- [Changed Attached Namespace List 与溢出行为，PDF p. 235](../_source/pages/page-235.md)
