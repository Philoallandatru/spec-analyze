# 命令排序与仲裁（Command Ordering and Arbitration）

NVMe 默认把提交队列（SQ）中的每条命令视为**相互独立**：SQ 内的提交顺序**不强制决定**处理顺序、介质执行顺序或完成顺序。控制器可自由重排与并行执行以最大化吞吐，只有**融合操作（Fused Operation）**和**特定 I/O 命令集定义的原子操作**才强制保证原子性。[PDF pp. 120-121](../_source/pages/page-120.md)

> **类比理解**：把控制器想成外卖调度中心——调度员拿到 30 张订单后，不会死板地按接单顺序派送，而是谁顺路就先送谁（默认无序）；"取餐+送餐"这种必须一气呵成的组合单才绑定执行（融合操作）；而多个平台（SQ）抢骑手时，要么轮流派（Round Robin），要么 VIP 订单优先再按权重分（加权轮询 + 紧急）。

---

## 处理流程与仲裁机制

```text
  主机          SQ1  SQ2  SQ3 ...  Admin SQ
   | 写 SQE  →  [█][ ][█]    [█][█]
   | 敲尾门铃 → ──────────────→ 控制器
                                |
                                ▼
              选择候选命令（Candidate Commands）
                                |
                                ▼
   ┌─ 仲裁机制（CAP.AMC）─────────────────────────────────┐
   │  1. Round Robin（必选）：所有 SQ（含 Admin）等优先级轮转 │
   │  2. WRR + Urgent（可选）：                              │
   │       Admin SQ（严格最高）                              │
   │         > Urgent SQ（严格次高）                         │
   │           > High / Medium / Low（加权轮询 WRR）         │
   │     单次最多启动 = min(剩余权重积分, Arbitration Burst) │
   └─────────────────────────────────────────────────────────┘
                                |
                                ▼
              并行执行（完成顺序 ≠ 提交顺序）
                                |
                                ▼
   CQ ←───── 翻转 Phase Tag 的完成条目（全局可见点）
```

- **候选命令与可见性**：已转入控制器且就绪的命令称为**候选命令**；当一条命令的 CQE 发布到完成队列后，它产生的所有状态与数据修改对后续提交的命令全局可见。[PDF p. 121](../_source/pages/page-121.md)
- **Round Robin（必选）**：所有控制器都必须支持，Admin SQ 与各 I/O SQ 享有同等调度优先级。[PDF p. 122](../_source/pages/page-122.md)
- **WRR + Urgent（可选）**：由 `CAP.AMC` 位图宣告支持；创建 SQ 时通过 `QPRIO` 指定优先级（`00b` Urgent、`01b` High、`10b` Medium、`11b` Low）。权重与突发上限（Arbitration Burst）可通过 `Set Features (FID=01h)` 配置。**注意饥饿风险**：Urgent SQ 与 WRR 之间没有公平协议，持续的 Urgent 流量可能饿死 High/Medium/Low 队列。[PDF pp. 122-123](../_source/pages/page-122.md)
- **依赖排序由主机负责**：控制器不检查跨命令的 LBA 重叠；若主机先后对同一 LBA 发出两条写命令，必须在主机软件层加屏障等待第一条完成，或使用原子操作，否则可能因乱序完成导致读回旧数据。

| 规则 | 要点 |
|------|------|
| 默认命令独立 | SQ 内 / 跨 SQ 命令默认无序，控制器不检查 LBA 重叠或依赖 |
| 主机负责排序 | 有依赖的命令由主机软件在控制器之上强制 |
| 原子操作例外 | 仅"融合操作"和"命令集定义的原子操作"被强制原子 |
| 融合对位置 | 必须占同一 SQ 相邻条目；允许环绕到 0 |
| 融合对提交 | 内存模型：一次尾门铃提交；Fabrics：两个 capsule 之间无其他 capsule |
| 融合对完成 | 每条命令各产生一条 CQ 条目 |
| FUSE 字段 | 命令双字 0 第 8-9 位；`00b`=普通；`01b`=融合第一条；`10b`=融合第二条 |
| 仲裁必支持 | Round Robin 仲裁（所有控制器必须实现） |
| 仲裁可选 | WRR + Urgent Priority Class（控制器可选；不支持时 SQ 优先级被忽略） |
| Admin 优先级 | Admin SQ 在 WRR 模式中属"最高严格优先级" |
| Urgent 风险 | 紧急 SQ 与 WRR 之间无公平协议，可能饿死 WRR 三级 |
| 仲裁突发 | Identify Controller 暴露 `Recommended Arbitration Burst`（特性 `01h`）可配 |
| 完成后可见 | 一条命令的 CQ 条目发布后，对该命令的修改对后续提交命令全局可见 |
| 中止融合对 | 主机需为每条命令各发一条 Abort 命令 |

## 融合操作（Fused Operation）

融合操作将两条命令绑定为一个不可分割的原子执行单元，具体支持的命令组合由 I/O 命令集规范定义，控制器通过 Identify Controller `FUSES` 字段声明支持。[PDF pp. 119-121](../_source/pages/page-119.md)

| 维度 | 规范要求 |
|---|---|
| **`FUSE` 字段（CDW0[9:8]）** | `00b` 普通命令；`01b` 融合第一条；`10b` 融合第二条；`11b` 保留 |
| **队列位置与提交** | 必须占用同一 SQ 中**相邻条目**（允许队尾环绕到槽位 0）；Memory-based 需通过**一次**尾门铃更新提交，Fabrics 需连续交付且中间无其他 Capsule |
| **完成与中止** | 两条命令**各自产生一个 CQE**；若需中止，必须分别为每条命令提交一个 `Abort` |
| **错误处理** | 不支持融合 → `Invalid Field in Command`；缺少配对命令或格式错 → `Command Aborted due to Missing Fused Command`；第一条失败 → 自动中止第二条 |

---

## 未完成命令的终止边界

从命令提交开始，直到满足以下任一协议证明之前，该命令始终处于**未完成（Outstanding）**状态：[PDF pp. 123-124](../_source/pages/page-123.md)

1. 收到该命令的 **CQE**（正常完成或报错完成）；
2. 通过 **Abort** 命令成功中止；
3. 通过 **Cancel** 命令成功取消；
4. 观察到控制器进入未就绪或关机完成状态（仅非 Fabrics）；
5. 删除承载该命令的 SQ（Memory-based）；
6. Fabrics **Disconnect** 成功断开对应 I/O 队列，或在通信丢失后与同一控制器重建通信。

---

## 易混概念对比

| 易混概念 | 实际区别 |
|---|---|
| **提交顺序 vs 完成顺序** | 提交按 SQ 槽位顺序；执行与完成可由控制器自由重排 |
| **SQ 优先级 vs 命令优先级** | 仲裁优先级绑定在 **SQ** 上（`QPRIO`）；命令 `CDW0` 中没有优先级字段 |
| **Urgent 类 vs High 级** | `Urgent` 属于**严格次高优先级类**；`High` 只是 WRR 类里的最高权重档，两者不在同一层级 |
| **Arbitration Burst vs Queue Size** | 突发（AB）是单次调度最多连续启动的命令数；Queue Size 是队列总槽位数 |

---

## 关联与引申概念

- **Fused Operation（融合操作）**：通过 `CDW0.FUSE` 将两条相邻命令合成原子对执行（如 Compare and Write）。
- **WRR（Weighted Round Robin）**：加权轮询仲裁，按 High/Medium/Low 权重比例调度 I/O SQ。
- **Phase Tag（阶段标签）**：CQE 中的翻转标志位，控制器通过翻转它来标识新写入的完成条目。
- **Doorbell（门铃寄存器）**：主机写 SQ Tail Doorbell 一次性暴露一条或多条新提交的命令。
- **延伸阅读**：
  - [common-command-format.md](common-command-format.md) — SQE 中 `FUSE` 位的定义
  - [completion-queue-entry-and-status.md](completion-queue-entry-and-status.md) — CQE 的 `CID` 与 Phase Tag 机制
  - [common-controller-features.md](common-controller-features.md) — `FID 01h` 仲裁权重与突发参数配置
  - [feature-values-and-scope.md](feature-values-and-scope.md) — Set Features 的作用域与生效边界

---

## 规范依据

- [融合操作提交规则，PDF pp. 119-121](../_source/pages/page-119.md)
- [命令排序、候选命令与完成可见性，PDF pp. 120-121](../_source/pages/page-120.md)
- [Round Robin 与 WRR + Urgent 仲裁（Figures 80-81），PDF pp. 122-123](../_source/pages/page-122.md)
- [未完成命令的终止条件，PDF pp. 123-124](../_source/pages/page-123.md)
