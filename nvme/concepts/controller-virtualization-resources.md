# 控制器虚拟化资源（Controller Virtualization Resources）

## 一句话说明

控制器虚拟化资源是主控制器（Primary Controller）拥有的可调配 VQ（虚拟队列）/ VI（虚拟中断）资源池，主控制器可在自身和最多 127 个从控制器（Secondary Controller）之间灵活分配这些资源。

## 生活化类比

把虚拟化资源想成**公寓楼的总配电箱**：

- **主控制器 = 物业管理处**，拥有整栋楼的电力总容量。
- **VQ / VI 灵活资源 = 可调电表**，可以动态从总容量里划给各个住户。
- **VQ / VI 私有资源 = 物业自用电表**，永远归物业，租户碰不到。
- **从控制器 = 各住户**，可以按需申请电表（Online 前必须先申请）。

管理处的电表箱上有一个清单（`CNS=15h` Identify），显示每户当前分配了多少电、是否在住。管理处想给某户调电，必须先让该户搬出（Offline）、调完电表再搬回（Online）。

## 工作流程

```text
  +--------------------- 主控制器资源池 ----------------------+
  |                                                          |
  |  VQ 灵活资源总量 (VQFRT)                                 |
  |  ├── 已分配给从控制器 (VQRFA)                            |
  |  ├── 已分配给主控制器自身 (VQRFAP)                       |
  |  └── 未分配池                                            |
  |                                                          |
  |  VQ 私有资源总量 (VQPRT)        ← 只归主控制器           |
  |                                                          |
  |  VI 灵活资源总量 (VIFRT)        ← 结构同上                |
  |  VI 私有资源总量 (VIPRT)        ← 只归主控制器           |
  +----------------------------------------------------------+
```

简化说明：VQ 单位 = 一对 SQ + CQ；VI 单位 = 一个中断向量（NVMe 2.1 仅支持 MSI-X）。各计数通过 `CNS=14h` 的 Primary Controller Capabilities 结构查询。

## 初学者案例

**场景：SR-IOV 部署时，VF 数量设多了但从控制器报 "Invalid Resource Identifier"**

1. 主机配置 SR-IOV，把 NumVFs 设成 32；系统创建了 32 个 VF（每个 VF 对应一个从控制器）。
2. 主机用 `CNS=15h` 列出从控制器，逐一在线（Online）。前 8 个成功，第 9 个报 `Invalid Resource Identifier`。
3. 排错步骤：
   - 用 `CNS=14h` 读主控制器能力结构，查 `VQFRSM`（每个从控制器最大 VQ 灵活资源数）和 `VQRFA`（已分配总数）。
   - 若 `VQRFA + 请求数 > VQFRT`：灵活资源池耗尽，把部分从控制器 Offline 后回收再重试。
   - 若 `请求数 > VQFRSM`：超过单从控制器上限，要么换小一点的队列配置，要么换主控制器。
4. 另一类常见错误：尝试对**私有资源**发起分配——私有资源永远归主控制器，分配请求必被拒。

> 排错口诀：**先看总量（VQFRT/VIFRT）够不够，再看单上限（VQFRSM/VIFRSM）够不够，最后确认请求的是灵活不是私有**。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 资源支持继承 | 主控制器支持的资源类型（VQ/VI），其所有从控制器必须同样支持 |
| VQ 单位 | 一个提交队列 + 一个完成队列 = 一个 VQ 资源（VQ ID = Queue ID） |
| VI 单位 | 一个中断向量 = 一个 VI 资源（VI ID = 中断向量号） |
| VI 唯一支持 | NVMe 2.1 中 VI 资源**仅**支持 MSI-X |
| 互不依赖 | VQ 与 VI 支持相互独立；不支持的类型自动视为私有 |
| 灵活资源可调 | 灵活资源可在主控制器、各从控制器和未分配池之间动态移动 |
| 私有资源不可动 | 私有资源专属主控制器，任何从控制器请求都失败 |
| 默认值实现决定 | 主控制器自身初始的灵活资源分配默认由实现决定 |
| 灵活资源变更时机 | 虚拟化管理命令设置的新值，在**适用的控制器级复位后**生效（除 Controller Reset） |
| 上限与粒度 | 每个从控制器有 VQFRSM/VIFRSM 上限；分配粒度 VQGRAN/VIGRAN 决定最佳申请步长 |
| 从控制器上限 | 一个主控制器最多关联 127 个从控制器（`CNS=15h` Identify 返回） |
| 查询范围 | `CNS=15h` 从 CNTID 起返回，包含 Offline 状态和被禁用的 SR-IOV VF |
| VF 编号字段 | 非 SR-IOV VF 类型的从控制器把 VF 编号报告为 0 |
| 资源分配门控 | 资源分配请求必须满足：①目标存在；②非私有；③当前可用 |
| 状态约束 | 资源分配要求从控制器 Offline；Online 要求配置完成且主控制器已使能 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| VQ 资源 vs VQ 私有 | 灵活资源可分配，私有资源永远归主控制器 |
| VQ 灵活资源 vs I/O 队列 | 灵活资源是容量配额；I/O 队列是实际创建出的 SQ/CQ |
| 主控制器 vs 从控制器 | 主控制器拥有资源池；从控制器是消耗资源的租户 |
| VQ vs VI | VQ = 队列资源（SQ+CQ 对）；VI = 中断资源（向量） |
| 灵活分配 vs 私有分配 | 前者可在线下后调整；后者不可调 |
| 资源 ID vs 队列 ID | VQ 资源 ID 等同于 Queue ID；VI 资源 ID 等同于中断向量号 |
| 资源支持位 vs 资源数量 | CRT.VQRS=1 是"支持"；VQFRT 是"支持多少"——两件事 |
| NRM vs NR | NR 是请求数；NRM 是实际修改数（可能 <、=、> 请求数） |
| 虚拟化管理命令的"对主"vs"对从" | 对主调整自身分配；对从做 Offline/分配/Online |

## 进阶细节

- **主控制器能力结构**（规范 5.1.13.3.1，`CNS=14h`，Figure 330）：
  - Bytes 01:00 — CNTLID（控制器 ID）
  - Bytes 03:02 — PORTID（端口 ID）
  - Byte 04 — CRT（Controller Resource Types）：
    - bit 0 — VQRS（VQ Resources Support）
    - bit 1 — VIRS（VI Resources Support）
    - bits 7:2 — 保留
  - Bytes 35:32 — VQFRT（VQ 灵活资源总量，主+从共享）
  - Bytes 39:36 — VQRFA（VQ 灵活资源已分配给从控制器数）
  - Bytes 41:40 — VQRFAP（VQ 灵活资源已分配给主控制器自身数；Controller Reset 之外的其他 CLR 后可变）
  - Bytes 43:42 — VQPRT（VQ 私有资源总量）
  - Bytes 45:44 — VQFRSM（每从控制器 VQ 灵活资源上限）
  - Bytes 47:46 — VQGRAN（VQ 灵活资源首选粒度）
  - Bytes 67:64 — VIFRT（VI 灵活资源总量）
  - Bytes 71:68 — VIRFA（VI 灵活资源已分配给从控制器数）
  - Bytes 73:72 — VIRFAP（VI 灵活资源已分配给主控制器自身数）
  - Bytes 75:74 — VIPRT（VI 私有资源总量）
  - Bytes 77:76 — VIFRSM（每从控制器 VI 灵活资源上限）
  - Bytes 79:78 — VIGRAN（VI 灵活资源首选粒度）
- **从控制器列表**（规范 5.1.13.3.2，`CNS=15h`，Figure 331）：
  - 最多返回 127 个条目
  - 从 CDW10.CNTID 起列出，**包含** Offline 状态与被禁用的 SR-IOV VF
  - 每个条目含：Controller ID、关联主控制器 ID、在线/离线状态、SR-IOV VF 编号、当前灵活 VQ/VI 分配数
- **从控制器状态机**（规范 8.2.6 / 5.2.6）：
  - 起始：在线（Online）
  - Offline 操作 → 进入 Offline 状态（所有灵活 VQ/VI 资源移除）
  - Offline 期间可分配/调整 VQ/VI 灵活资源
  - Online 操作（要求配置完成 + 主控制器已使能）→ 回到在线
  - 资源分配请求**要求**目标处于 Offline 状态
- **虚拟化管理命令**（规范 5.2.6）：
  - 仅具备 Virtualization Enhancements capability 的主控制器可发
  - 三个用途：调整主控制器自身灵活资源分配、为从控制器分配灵活资源、设置从控制器 Online/Offline
  - 关键参数：Resource Type（VQ/VI）、CNTLID、NR（请求数量）
  - 拒绝条件（任一）：①资源范围不存在；②属于私有资源；③当前不可用（如 NR > 剩余灵活资源）
  - 拒绝状态码：`Invalid Resource Identifier`
- **完成结果**：CQE Dword 0 报告 NRM（Number of Resources Modified）——实际被修改的资源数；NRM 与 NR 的关系由实现决定，可 < / = / > NR。
- **状态码分类**（命令失败时）：
  - 非法控制器标识符
  - 非法控制器状态
  - 非法请求资源数量
  - 不可用或非法资源标识符
- **能力继承约束**（Figure 330 CRT 字段说明）：主控制器支持的资源类型，所有关联从控制器**必须**支持；若不满足则不一致（规范措辞 shall）。

## 规范依据

- [Primary Controller Capabilities 结构（Figure 330），PDF 第 366 页](../_source/pages/page-366.md)
- [VQ/VI 资源池字段定义与从控制器列表（Figure 331），PDF 第 367 页](../_source/pages/page-367.md)
- [虚拟化管理命令与门控（5.2.6），PDF 第 446 页](../_source/pages/page-446.md)
- [虚拟化管理命令的完成与状态码分类，PDF 第 447-448 页](../_source/pages/page-447.md)

## 相关阅读

- [controller.md](controller.md) - VQ/VI 资源由主控制器调配
- [queue-pair.md](queue-pair.md) - VQ 是虚拟化的队列对
- [controller-data-queues.md](controller-data-queues.md) - 数据队列相关虚拟化资源
- [common-controller-features.md](common-controller-features.md) - 控制器通用特性范畴
