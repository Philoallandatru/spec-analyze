# 异步事件上报（Asynchronous Event Reporting）

**异步事件请求（Asynchronous Event Request, AER）** 是一种"长寿命"Admin 命令，允许控制器在无需主机轮询的情况下，主动向主机报告错误、健康警告或状态变更。主机预先提交 AER 挂在队列中，控制器等到事件发生时才通过 CQE 返回概要，再由主机读取日志页获取详情并解除同类型事件屏蔽。[PDF pp. 195-202](../_source/pages/page-195.md)

> **类比理解**：把 AER 想成医院的床头呼叫系统——护士（主机）先给床位按下"待命"按钮（提交 AER），有情况时呼叫灯亮起（返回 CQE 概要），护士去翻病历本（`Get Log Page`）了解详情并确认处理，之后该类警报才解除屏蔽，护士再补按一次待命键。

- **护士站的主机** = 主机（Host）
- **床头呼叫器** = 控制器（Controller）
- **预先按下的"待命"按钮** = AER 命令（长寿命，等着被响应）
- **不同床位的报警灯** = 事件族（Error / SMART / Notice / I/O-specific / Immediate / One-Shot / Vendor）
- **护士去床头看病情记录** = Get Log Page 读取日志详情
- **"读完日志 = 故障已处理"** = 解除事件屏蔽

## 工作流程

```text
主机（Host）                  控制器（Controller）              日志页（Log Page）
    |                                |                              |
    |---- 提交 AER ① / ② ----------->|  (持有，不立即完成，无超时)     |
    |                                |                              |
    |                                |<---- 事件发生（如温度超限）     |
    |                                |                              |
    |<---- CQE: AET + AEI + LID -----|  (返回事件概要)                |
    |                                |---- 自动屏蔽同类型事件          |
    |                                |                              |
    |---- Get Log Page (RAE=0) ------------------------------------>|
    |<---- 返回事件详情 ---------------------------------------------|
    |                                |<---- 解除该事件类型屏蔽         |
    |                                |                              |
    |---- 提交新 AER 补充 ---------->|  (维持未完成 AER 数量)         |
```

1. **预先提交**：主机在初始化末尾提交一个或多个 AER（不超过控制器通告的上限）。
2. **等待与触发**：AER **没有超时限制**；当事件发生时，控制器挑选一个未完成的 AER 返回 CQE。
3. **自动屏蔽**：普通事件一旦报告，控制器立即屏蔽同一事件类型，防止在主机处理前反复刷屏。
4. **读取与清除**：主机根据 CQE 中的 `LID` 执行 `Get Log Page`（`RAE=0`，即不保留异步事件），读取完成后自动解除屏蔽。
5. **补充请求**：主机重新提交 AER 以保持事件监听不断档。

> 速查：若发现事件"刷屏"，往往是底层条件未消除（温度还没降下来）。规范建议主机在清除事件前先调阈值或临时屏蔽。

## CQE 格式与事件分类

### AER CQE 字段布局（Figures 147-148）

```text
AER CQE DW0（双字 0）
31        24 23        16 15         8 7       3 2       0
+-----------+------------+-------------+---------+---------+
| 保留      | LID        | AEI         | 保留    | AET     |
|           | 日志标识   | 事件信息    |         | 事件类型|
+-----------+------------+-------------+---------+---------+

AER CQE DW1（双字 1）：32 位事件特定参数（Event-specific Parameter）
```

- **AET（bit 2:0）**：异步事件类型（事件族），如 Error、SMART/Health、Notice、I/O-specific、Immediate、One-Shot、Vendor。
- **AEI（bit 15:8）**：异步事件信息，标识该族内的具体事件（Figures 149-155）。[PDF pp. 198-202](../_source/pages/page-198.md)
- **LID（bit 23:16）**：日志标识符，指向需要通过 `Get Log Page` 读取并清除事件的日志页。[PDF pp. 197-198](../_source/pages/page-197.md)

### 按清除与保留模型分类

| 事件族 | 模型 | 典型成员与处理方式 |
|---|---|---|
| **Error / SMART/Health / Notice / I/O-specific / Vendor** | 日志支持型 | 涵盖命令错误、温度/可靠性警告、命名空间/固件激活/ANA/Discovery 变更、Reservation/Sanitize 状态；需通过 `Get Log Page (RAE=0)` 读取并清除 |
| **Immediate（即时）** | 发生时上报型 | 如子系统关机（Subsystem Shutdown）；**仅在事件发生时刻存在未完成 AER 时才报告，不保留** |
| **One-Shot（一次性）** | 自动清除型 | 如控制器数据队列尾指针/满载通知；报告后由控制器自动清除，无需读日志 |

---

## 请求管理、屏蔽与配置规则

| 机制/场景 | 规范行为 |
|---|---|
| **未完成数量上限** | 主机可并发提交多个 AER；若超出控制器通告上限，控制器返回状态码 `05h`（Asynchronous Event Request Limit Exceeded）。[PDF p. 197](../_source/pages/page-197.md) |
| **控制器重置** | Controller Reset 会中止所有未完成的 AER，且**不返回 CQE**。[PDF p. 196](../_source/pages/page-196.md) |
| **事件保留、合并与排队** | 除 Immediate 外，启用但无未完成 AER 时发生的事件会被保留到下次 AER 提交时报告；相同事件响应可合并，不同事件应排队。[PDF p. 197](../_source/pages/page-197.md) |
| **介质未就绪阻塞** | 若清除事件所需的日志暂时无法读取（media-not-ready），控制器会阻止该事件上报。[PDF p. 196](../_source/pages/page-196.md) |
| **持久/瞬态条件防风暴** | 若底层触发条件（如温度过高）持续存在，直接清除会立即再次触发；主机应在清除前先调整阈值或临时屏蔽该事件类型。[PDF p. 197](../_source/pages/page-197.md) |
| **Asynchronous Event Configuration** | 通过 `Set Features` 按事件族粒度启用/禁用通知（不影响底层条件或日志更新）；若启用时持久条件已成立，控制器会立即发送事件；对不支持的资源特定事件族执行启用是非法的。[PDF pp. 399-400](../_source/pages/page-399.md) |

---

## 易混概念对比

| 易混概念 | 实际区别 |
|---|---|
| **AET vs AEI** | `AET`（3 位）是事件大类（族）；`AEI`（8 位）是族内的具体事件编号 |
| **屏蔽（Mask） vs 禁用（Disable）** | 屏蔽是事件报告后的临时自动阻断（读日志清除后解除）；禁用是通过 `Set Features` 关闭整族事件通知 |
| **Immediate vs One-Shot** | `Immediate` 不保留，发生时无 AER 即丢弃；`One-Shot` 报告后自动清除，无需 `Get Log Page` |
| **LID vs 事件类型** | `LID` 仅用于告诉主机去读哪张日志页，不代表事件本身的类型 |

| 容易混 | 实际区别 |
|--------|----------|
| AER vs 普通 Admin 命令 | 普通命令有超时、AER 没有；AER 是"挂着等事件" |
| AET vs AEI | AET（3 位）= 事件族（族级）；AEI（8 位）= 族内具体事件（个例） |
| 屏蔽 vs 禁用 | 屏蔽是事件族内的临时阻断（清除后自动解除）；禁用通过 Set Features 关闭整族通知 |
| 持久条件 vs 瞬态事件 | 持续存在的条件在清除后会立刻再触发；规范建议先调阈值再清除 |
| LID vs 事件类型 | LID 是日志页标识（用来 Get Log Page 读详情），不等于事件类型 |
| One-Shot vs Immediate | One-Shot 报告后自动清除；Immediate 仅在有 AER 时报告、不保留 |

## 关联与引申概念

- **CQE（Completion Queue Entry）**：控制器向主机返回命令执行结果与异步事件概要的 16 字节条目。
- **Get Log Page（RAE=0）**：读取控制器日志页的标准 Admin 命令；`RAE`（Retain Asynchronous Event）设为 0 表示读完后清除对应异步事件并解除屏蔽。
- **Set/Get Features**：用于配置或查询控制器特性（如 Asynchronous Event Configuration、温度阈值）。
- **ANA（Asymmetric Namespace Access）**：多路径/多控制器场景下命名空间的访问状态，状态变化会通过 Notice 类 AER 通知主机。
- **Firmware Activation / Sanitize**：固件激活与安全擦除操作，完成后常通过 AER 通知主机。
- **延伸阅读**：
  - [admin-command-model.md](admin-command-model.md) — AER 所属的 Admin 命令模型
  - [keep-alive.md](keep-alive.md) — 保活与关联超时相关的事件源
  - [controller-initialization.md](controller-initialization.md) — 控制器初始化末尾挂载 AER 的流程
  - [firmware-update-lifecycle.md](firmware-update-lifecycle.md) — Firmware Activation 异步事件来源

---

## 规范依据

- [AER 生命周期、屏蔽机制与事件族定义，PDF pp. 195-197](../_source/pages/page-195.md)
- [Figures 146-148：状态码与 CQE 编码格式，PDF pp. 197-198](../_source/pages/page-197.md)
- [Figures 149-150：Error 与 SMART/Health 事件，PDF p. 198](../_source/pages/page-198.md)
- [Figure 151：Notice 事件，PDF pp. 199-201](../_source/pages/page-199.md)
- [Figures 152-155：I/O-specific、Immediate 与 One-Shot 事件，PDF pp. 201-202](../_source/pages/page-201.md)
- [Asynchronous Event Configuration Feature，PDF pp. 399-400](../_source/pages/page-399.md)
