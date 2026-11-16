# 持久事件日志（Persistent Event Log, LID `0Dh`）

**持久事件日志（Persistent Event Log，日志页 `0Dh`）** 是 NVM 子系统级别的**跨断电历史事件记录**，用于追踪与具体 I/O 命令无关的关键状态变化和硬件/管理事件。事件在断电与重置后依然保留，并通过“**建立上下文冻结快照 → 分段读取 → 代数校验**”的三段式协议保证多主机并发读取时的数据一致性。[PDF pp. 252-255](../_source/pages/page-252.md)

> **类比**：把持久事件日志想成**档案室的永久流水账**——新事件只增不改地写入账本（断电不丢）；当你发起读取时，管理员先为当前账本拍一张**快照照片（Reporting Context）**，你分几次抄完后再核对照片**版本号（Generation Number）**，确认中途没人重拍过快照，这份日志才算有效。

## 报告上下文（Reporting Context）与一致性协议

```text
[无上下文] ── Establish & Read / Establish ──→ [上下文存在 (冻结快照，后续新事件不进入)]
    ▲                                                    │
    │                                                    ├── Read (按字节偏移分段读取)
    │                                                    │
    └──── 显式 Release / 子系统或控制器 Reset / 超时 ─────┘

一致性校验流程：
  1. Establish 创建快照，从 512B 头部获取 TNEV、TLL 与初始 Generation Number (G1)
  2. 分多次 Get Log Page 按字节偏移读取全部事件
  3. 重读头部校验 Generation Number：仍为 G1 则快照有效；若变为 G2 则整批作废重来
```

| 上下文操作（Action） | 行为与约束 |
|---|---|
| **Establish and Read** | 冻结当前事件集合与长度，并返回请求偏移处的数据；若上下文已存在则失败。[PDF pp. 253-255](../_source/pages/page-253.md) |
| **Read** | 读取已冻结快照的数据；若当前无上下文，返回 `Command Sequence Error`。[PDF pp. 253-255](../_source/pages/page-253.md) |
| **Establish/Check Header** | 建立新上下文或检查现有上下文，始终返回 512 字节头部并报告上下文是否存在（NVMe 2.1 由 `ECRH` 能力位公布支持）。[PDF pp. 253-260](../_source/pages/page-253.md) |
| **Release** | 删除现有上下文（不返回数据，幂等操作；即使无上下文也成功）。[PDF pp. 253-255](../_source/pages/page-253.md) |

- **代数编号（Generation Number）**：仅在新建上下文且返回不同事件集合时递增；重复读取相同事件集合时不变。[PDF p. 254](../_source/pages/page-254.md)
- **记录抑制与存储压力**：控制器对短时间高频重复事件可按厂商阈值抑制；容量/条数满时按厂商策略淘汰旧记录；影响多控制器的事件仅由厂商选定一个控制器记录；执行 `Sanitize` 可能清除或修改事件以防信息泄露。[PDF pp. 252-253](../_source/pages/page-252.md)

## 日志结构与事件帧格式

```text
┌─────────────── 512 字节快照头部 ───────────────┐
│ TNEV (事件数) | TLL (总字节长) | LREV (03h)    │
│ Generation Number | 上下文建立时间 | 电源历史  │
│ 控制器 ID | 子系统 NQN | Reporting Context Info│
│ Supported Events Bitmap (支持事件类型位图)     │
└────────────────────────────────────────────────┘
  ├── 事件条目 0: [事件头 (长=EHL+3) | 可选厂商信息 (VLI) | 类型特定数据]  (总长 EL)
  └── 事件条目 1..N (默认最新事件优先)
```

- **通用事件头部**：包含事件类型、事件版本（独立于日志版本演进）、`EHL`（头部长度 = `EHL + 3`）、控制器 ID、时间戳、端口关联（子系统端口 / Management Endpoint / 无端口）、厂商信息长度 `VLI` 与事件数据长度 `EL`。[PDF pp. 258-260](../_source/pages/page-258.md)

## 标准事件类型概览

| 事件族 | 关键记录内容与触发条件 |
|---|---|
| **基础强制/条件事件** | • **SMART/Health 快照**：每控制器至少每 24 通电小时一次（虚拟化下每主控制器一次）。<br>• **Firmware Commit**：旧/新版本、动作、槽位、CQE 状态。<br>• **Timestamp Change**：先前时间戳与自 CLR 起的毫秒数。<br>• **Power-on / Reset**：生效固件版本及每控制器重置描述符。[PDF pp. 260-262](../_source/pages/page-260.md) |
| **硬件错误（`05h`/`02h`）** | 2 字节错误码 + 特定证据：PCIe 错误（设备状态、AER 状态/掩码、TLP 头/前缀日志）、链路状态变更/未激活、临界警告（含 EG ID）、意外断电（断电次数+带外标志）、致命控制器状态、介质/数据完整性错误（捕获 CQE）、**控制器就绪超时**（CNR/Admin/NS 就绪标志及 CRIME 模式）。[PDF pp. 263-266](../_source/pages/page-263.md) |
| **命名空间与破坏性操作** | • **Change Namespace**：记录管理操作的大小、格式、共享、ANA 组、NVM Set、EG 及 NSID（全删用 `FFFFFFFFh`）。<br>• **Format NVM / Sanitize 配对事件**：“开始”事件（介质修改前/进入处理时记录参数意图）与“完成”事件（记录进度、错误标志与 CQE 结果）。[PDF pp. 266-269](../_source/pages/page-266.md) |
| **Feature 变更** | 受支持且可记录的 Feature 通过 `Set Features` 成功变更时必记（载荷含 CDW10 起连续 dword、可选完成 DW0 及内存缓冲字节）。[PDF pp. 270-271](../_source/pages/page-270.md) |
| **诊断与扩展事件** | • **Telemetry Created**：主机/控制器遥测日志的首个 512B 头部。<br>• **Thermal Excursion**：过温/低温/节流启停及开尔文温差。<br>• **Sanitize Media Verification**：进入介质验证阶段的仅头部标记。<br>• **Vendor-specific**：支持按 UUID 索引过滤的类型化描述符（名称/ASCII/二进制/整数）。<br>• **TCG-defined**：安全相关事件载荷。[PDF pp. 272-275](../_source/pages/page-272.md) |

## 关联与引申概念

- **PCIe AER（Advanced Error Reporting，高级错误报告）**：PCIe 规范定义的可纠正/不可纠正错误诊断寄存器组，发生硬件错误时会被持久化到事件日志中。
- **CRIME（Controller Ready Independent of Media Enable）**：允许控制器在底层介质完全就绪前先行响应 Admin 命令的就绪模式。
- **相关文档**：
  - [log-page-retrieval.md](log-page-retrieval.md) — Get Log Page 字节偏移与分段读取机制
  - [error-and-health-logs.md](error-and-health-logs.md) — Error Information (`01h`) 与 SMART (`02h`) 日志
  - [telemetry-capture-logs.md](telemetry-capture-logs.md) — 深度诊断 Telemetry 数据块
  - [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) — 触发上下文释放的重置边界

## 规范依据

- [全局持久化、抑制与代数一致性协议，PDF pp. 252-254](../_source/pages/page-252.md)
- [报告上下文操作、512B 头部与事件帧格式，PDF pp. 255-260](../_source/pages/page-255.md)
- [基础事件与硬件错误事件格式，PDF pp. 260-266](../_source/pages/page-260.md)
- [命名空间、Format/Sanitize、Feature、温度与厂商事件，PDF pp. 266-275](../_source/pages/page-266.md)
