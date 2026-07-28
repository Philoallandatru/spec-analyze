# 命名空间标识符（Namespace Identifiers）

## 一句话说明

NSID（命名空间标识符）是主机在命令中引用命名空间所用的数字句柄，**只在子系统当前会话内有效**；跨会话或跨路径追踪同一物理命名空间要用 UUID/NGUID/EUI64 这类持久身份。

## 生活化类比

把 NSID 想成**酒店房间号**：

- 房间号（NSID）每天退房后可能重排——你这次住 1208，下次可能住 1605。
- 房间本身的"门卡 ID"（UUID）不会变——它和具体哪间房无关。
- 入住（挂载）才看得到房间号；预订成功（已分配）但未入住，房间号对你不可见。

所以"按房间号找人"在本店有效，"按门卡 ID 找人"跨店也有效。

## 工作流程

```text
  子系统视角（NSID 1..NN）：
   ┌──────────────────────────────────────┐
   │ NSID 1 ── 已分配 ──→ 命名空间 X     │
   │ NSID 2 ── 已分配 ──→ 命名空间 Y     │
   │ NSID 3 ── 未分配                     │
   │ NSID 4..NN  ── 未分配/已分配         │
   └──────────────────────────────────────┘

  控制器视角（针对已分配的 NSID）：
   ┌──────────────────────────────────────┐
   │ 命名空间 X:                          │
   │   控制器 A 上 active  ✅              │
   │   控制器 B 上 inactive ❌             │
   │ 命名空间 Y:                          │
   │   控制器 A 上 active  ✅              │
   │   控制器 B 上 active  ✅  (共享)      │
   └──────────────────────────────────────┘
```

简化说明：分配状态（子系统级）与挂载状态（控制器级）是**两个独立维度**；一个 NSID 可能"子系统已分配"但对某些控制器是 inactive。

## 初学者案例

**场景：为什么我向某个 NSID 发读写命令返回 `Invalid Field in Command`？**

1. 命令里填的 NSID 在子系统里**确实存在**（已分配），但**没有挂到当前控制器**。
2. 主机用 `nvme list-ns /dev/nvme0`（`CNS=02h`）查到当前控制器 active 的 NSID 列表，没有包含目标 NSID。
3. 用 `nvme list-ns -a`（`CNS=10h`）查子系统级列表，确认命名空间已分配。
4. 解决方法：用 `nvme attach-ns` 把命名空间挂到当前控制器，或换一条挂载过的路径访问。

> 错误码速记：**`Invalid Field in Command`** = NSID 对当前控制器 inactive；**`Invalid Namespace or Format`** = 数值本身无效（0、超出 NN、保留范围）。

**另一个常见错误**：把 NSID `0h` 填进命令——**永远是无效值**，不是广播值；广播值是 `FFFFFFFFh`。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| NSID 是临时句柄 | 不是命名空间的永久身份，掉电/重启后可能变 |
| 作用域 | NSID 在整个 NVM 子系统内有效 |
| 活跃性按控制器评估 | 同一 NSID 在不同控制器上 active/inactive 可不同 |
| 数值空间 | 0h = 无效；1h..NN = 有效；NN+1..FFFFFFFEh = 无效；FFFFFFFFh = 广播 |
| 广播值语义 | 命令里 NSID=`FFFFFFFFh` 表示"向该命令允许操作的所有命名空间发送" |
| inactive NSID | 命名空间存在但未挂到当前控制器 → `Invalid Field in Command` |
| invalid NSID | 数值本身在有效范围外 → `Invalid Namespace or Format` |
| active NSID | 正常执行 |
| 分配状态（子系统） | allocated（已分配）/ unallocated（未分配） |
| 附加状态（控制器） | active（已挂载可访问）/ inactive（未挂载到本控制器） |
| 枚举命令 CNS | `CNS=0h` 单个 Namespace 详情；`CNS=2h` 本控制器 active 列表（≤1024）；`CNS=10h` 子系统已分配列表（≤1024） |
| 分页 | 每次最多 1024；用最后一个 NSID 作为下个请求起点 |
| NN vs MNAN | NN 是最大有效 NSID；MNAN 是同时分配最大数；前者可远大于后者 |
| 持久身份标识 | UUID（128-bit，优先级最高）/ NGUID（128-bit）/ EUI64（64-bit） |
| UIDREUSE 字段 | 提示 NGUID/EUI64 是否可被重用 |
| NSID 一致性 | 支持 Namespace Management / ANA Reporting / NVM Sets 任一时，NSID 跨控制器指向同一物理命名空间 |
| 队列无绑定 | 命名空间与 SQ 无固定绑定；任一 I/O SQ 可访问该控制器上所有已挂载命名空间 |
| 变更通知 | Changed Namespace List 日志页（05h）报告 Identify 变化或挂载变化 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| NSID vs UUID | NSID 是临时句柄；UUID 是跨会话稳定的持久身份 |
| 0h vs FFFFFFFFh | 0h = 无效值；FFFFFFFFh = 广播值（两者都不是普通有效 NSID） |
| allocated vs active | allocated 是子系统级（有没有）；active 是控制器级（挂没挂） |
| invalid vs inactive | invalid = 数值本身无效；inactive = 数值有效但未挂到当前控制器 |
| CNS=2h vs CNS=10h | 2h 查本控制器 active 列表；10h 查子系统已分配列表 |
| NN vs MNAN | NN 是 ID 空间上限；MNAN 是同时分配的最大数 |
| UUID vs NGUID | 都是 128-bit，但 UUID 优先级更高、跨会话稳定更强 |
| NSID 一致性 vs 持久 ID | 一致性靠子系统功能支持；持久 ID 是命名空间自带 |
| 队列 vs 命名空间 | 队列是命令通道；命名空间是数据目标；前者无固定绑定后者 |

## 进阶细节

- **NSID 数值空间**（规范 4.2.1.1）：
  - `0h` = 无效
  - `1h..NN` = 有效范围
  - `NN+1..FFFFFFFEh` = 无效
  - `FFFFFFFFh` = 广播值（不是普通有效 NSID）
  - `NN`（Namespace Number）在 Identify Controller 的 NN 字段报告
- **状态判定规则**（规范 4.2.1.1）：
  - 命令中指定 inactive 的 NSID → 返回 `Invalid Field in Command`
  - 命令中指定 invalid 的 NSID（数值范围无效）→ 返回 `Invalid Namespace or Format`
  - 广播值 `FFFFFFFFh` → 命令向该命令允许操作的所有命名空间发送
  - 命名空间 ID 集合的最大可能大小由 `MNN`（Max Namespace Number）报告
- **NSID 是控制器相对的**（规范 4.2.1.1）：NSID 是控制器用来访问命名空间的标识符，也是 SQE 中的字段；同一 NSID 经不同控制器访问，可能命中不同命名空间（私有命名空间场景下）。
- **CNS 枚举机制**（规范 4.2.1.2 / 5.1.13）：
  - `CNS=0h` Identify Namespace：返回指定 NSID 的 Identify Namespace 数据结构
  - `CNS=2h` Attached Namespace List：返回当前控制器 active 的 NSID 列表（≤1024）
  - `CNS=10h` Namespace Identification List：返回子系统已分配的 NSID 列表（≤1024）
- **分页机制**（规范 4.2.1.2 / 5.1.13.2.18）：
  - 单次最多 1024 条
  - 用返回的最后一个 NSID 作为下次请求的起点
  - 容量规划需注意 `NN` 可能远大于 `MNAN`
- **持久身份标识符的优先级**（规范 4.2.5）：
  - UUID（128-bit，RFC 9562）优先级最高
  - NGUID（128-bit）次之
  - EUI64（64-bit）最后
  - `UIDREUSE` 字段报告 NGUID/EUI64 是否可重用
- **NSID 跨控制器一致性**（规范 4.2.1.4）：
  - 若子系统支持 Namespace Management、ANA Reporting、NVM Sets 任一 → 同一 NSID 在所有控制器指向同一物理命名空间
  - 否则：共享命名空间 NSID 子系统内唯一；私有命名空间 NSID 可不唯一
- **命名空间与队列的关系**（规范 4.2.1.4）：
  - 命名空间与 SQ 无固定绑定
  - 主机可自由选择任一 I/O SQ 访问该控制器上所有已挂载命名空间
  - 典型应用：按 CPU 核心分配 SQ、按命名空间优先级分配 SQ
- **Changed Namespace List 日志页**（规范 5.2.12 / Figure 213）：
  - Log Page Identifier `05h`
  - 报告自上次成功读取以来发生变化的命名空间
  - 变化类型：Identify 数据变化、命名空间被挂到当前控制器、从当前控制器分离、被删除
  - 正常格式：NSID 列表
  - 溢出格式：首条 = `FFFFFFFFh`（溢出标记），其余填 0；主机应**重新全量枚举**
  - 主机应定期轮询并在溢出时立刻全量重扫

## 规范依据

- [NSID 数值空间、分配与附加状态，PDF 第 97-98 页](../_source/pages/page-097.md)
- [CNS 枚举、唯一性、跨控制器一致性，PDF 第 98-99 页](../_source/pages/page-098.md)
- [持久身份标识符与队列无绑定，PDF 第 99 页](../_source/pages/page-099.md)
- [Changed Namespace List 日志页（05h），PDF 第 235 页](../_source/pages/page-235.md)

## 相关阅读

- [namespace.md](namespace.md) - 被标识的命名空间对象
- [namespace-access-models.md](namespace-access-models.md) - 不同模型下 NSID 一致性
- [namespace-management-lifecycle.md](namespace-management-lifecycle.md) - 标识随生命周期变化
- [namespace-topology-and-change-logs.md](namespace-topology-and-change-logs.md) - 变更日志通过 NSID 记录
