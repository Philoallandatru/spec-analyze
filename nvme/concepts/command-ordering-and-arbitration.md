# 命令排序与仲裁

## 一句话说明

NVMe 默认不保证命令在 SQ 内的处理顺序或完成顺序——控制器可自由重排以提升性能；只有"融合操作"和"命令集定义的原子操作"被强制为原子单元。

## 生活化类比

把控制器想成**外卖调度中心**：

- **默认无序** = 调度员看到 30 个订单，不会按订单到达的顺序派骑手；谁顺路、谁顺时、谁骑手有空，谁就先送。
- **融合操作** = "取餐+送餐"必须一气呵成：骑手必须先到 A 店取，再到 B 店送，两单间不能塞别单。
- **轮询仲裁** = 多家外卖平台共用一个调度中心，轮流派单，谁也不偏。
- **加权轮询 + 紧急** = "VIP 订单（紧急）必须先派，剩下按高/中/低优先级加权轮流派"。
- **饥饿问题** = VIP 单子太多，低优先级单子可能几小时派不出去——系统设计时要小心。

## 工作流程

```text
  主机          SQ1  SQ2  SQ3 ...  Admin SQ
   | 写命令条目 → [█][ ][█]    [█][█]
   | 敲尾门铃  → ──────────────→ 控制器
                                |
                                ▼
              候选命令选择（vendor-specific 算法）
                                |
                                ▼
   ┌─ 仲裁 ─────────────────────────────────────────────┐
   │  Round Robin 模式：所有 SQ（含 Admin）等优先级轮转 │
   │  WRR+Urgent 模式：                                   │
   │     Admin（严格最高）                                │
   │       > Urgent SQ（严格次高）                       │
   │         > High / Medium / Low（WRR 加权）           │
   │  每次最多启动 = min(剩余权重积分, Arbitration Burst) │
   └────────────────────────────────────────────────────┘
                                |
                                ▼
              并行执行；完成顺序 ≠ 提交顺序
                                |
                                ▼
   CQ ←───── 阶段标签翻转的完成条目
   中断/轮询通知主机
```

## 初学者案例

**场景：连续写同一 LBA，读却先返回旧数据。**

1. 应用连续发了两条 Write 到同一个 4KB LBA：`Write(A, value1)`、`Write(A, value2)`。
2. 主机期望最终读返回 `value2`。
3. 但控制器内部有并行通道、后台压缩/GC，可能先完成第二条再完成第一条。
4. 如果**没有**强制同步，应用程序层读到的可能是 `value1`——**违反直觉**。
5. 解决：在应用层加 fence/屏障（如 Linux `blk-mq` 的 `REQ_FSEQ`、`REQ_PREFLUSH`），或在 NVMe 层用**融合操作 / Compare and Write 之类原子写**。
6. 排错提示：在 NVMe 抓 trace 时看到**完成顺序乱**是正常现象；不要假设 SQ 顺序 = 执行顺序。

## 必须记住的规则

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

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 提交顺序 vs 执行顺序 | 提交按 SQ 槽位序；执行由控制器自由排 |
| 执行顺序 vs 完成顺序 | 启动次序 ≠ 完成次序；乱序完成是常态 |
| 仲裁优先级 vs 命令优先级 | 仲裁优先级挂在 SQ 上；命令 dword 0 无优先级字段 |
| Round Robin vs WRR | RR 等优先级；WRR 有 3 个加权级别 + 严格优先级类 |
| Urgent class vs High class | Urgent 属"严格次高优先级类"；High 属"WRR 类的最高级别"，二者不在同一档 |
| 融合操作 vs 原子操作 | 融合由 Base 规范定义；原子操作由各 I/O 命令集规范定义 |
| Arbitration Burst vs Queue Size | 突发是单次仲裁最多启动的命令数；队列大小是槽位数 |

## 进阶细节

- **FUSE 编码**（规范 3.4.2）：`00b`=普通；`01b`=Fused Operation First；`10b`=Fused Operation Second。控制器通过 Identify Controller `FUSES` 字段声明支持。
- **未支持的 FUSE 行为**：控制器用状态码 `Invalid Field in Command` 中止非零 FUSE 字段。
- **融合对缺失/格式错**：控制器用状态码 `Command Aborted due to Missing Fused Command` 中止非零 FUSE 命令。
- **仲裁突发**（规范 3.4.4）：Identify Controller 暴露 `Recommended Arbitration Burst`；可通过 Set Features (`01h`) 调整。
- **优先级字段**：SQ 创建时指定 `QPRIO`（`00b`=Urgent、`01b`=High、`10b`=Medium、`11b`=Low）；仅当 `CAP.AMC` 位 `Weighted Round Robin with Urgent` 启用时才有意义。
- **未完成命令的终止证明**（规范 3.4.5）：
  1. Abort 成功
  2. Cancel 成功
  3. 控制器进入未就绪/关机完成（仅非 Fabrics）
  4. 删除承载该命令的 SQ（内存模型）
  5. Fabrics Disconnect 成功
  6. 通信丢失后与同一控制器重建连接
- **候选命令（candidate command）**（规范 3.4.4）：已转入控制器、且控制器认为可处理的命令。
- **命令"在处理中"的判定**：Feature 被读/写、用户数据（LBA 或 KV 对）被读/写。
- **仲裁类别识别**：`CAP.AMC`（Arbitration Mechanism Capabilites）位图，bit0=Round Robin 必 1，bit1=WRR with Urgent 可选 1。
- **Admin 协调**：管理命令可能影响多条 I/O SQ；主机软件需协调（规范 3.3.3.4）。

## 规范依据

- [命令处理与命令排序总述，PDF 第 120 页](../_source/pages/page-120.md)
- [融合操作规则，PDF 第 120-121 页](../_source/pages/page-120.md)
- [原子操作与仲裁机制，PDF 第 121 页](../_source/pages/page-121.md)
- [Round Robin 仲裁 Figure 80，PDF 第 122 页](../_source/pages/page-122.md)
- [加权轮询 + Urgent 仲裁 Figure 81，PDF 第 122-123 页](../_source/pages/page-122.md)
- [未完成命令终止条件，PDF 第 123-124 页](../_source/pages/page-123.md)

## 相关阅读

- [通用命令格式](common-command-format.md) - SQE 的 FUSE 位决定融合原子性
- [完成队列条目与状态](completion-queue-entry-and-status.md) - CID 字段对应 SQ 命令
- [通用控制器特性](common-controller-features.md) - FID 01h 仲裁权重参数
- [特性值与作用域](feature-values-and-scope.md) - Set Features 切仲裁作用域
