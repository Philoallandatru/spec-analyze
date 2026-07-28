# 异步事件上报（Asynchronous Event Reporting）

## 一句话说明

异步事件请求（Asynchronous Event Request, AER）是 NVMe 控制器主动通知主机发生重要事件的长寿命命令：主机预先提交多个 AER，控制器在事件发生时通过完成队列条目（Completion Queue Entry, CQE）回报事件类型，并由 Get Log Page 读取详情并解除屏蔽。

## 生活化类比

把异步事件上报想成**医院的护士呼叫系统**：

- **护士站的主机** = 主机（Host）
- **床头呼叫器** = 控制器（Controller）
- **预先按下的"待命"按钮** = AER 命令（长寿命，等着被响应）
- **不同床位的报警灯** = 事件族（Error / SMART / Notice / I/O-specific / Immediate / One-Shot / Vendor）
- **护士去床头看病情记录** = Get Log Page 读取日志详情
- **"读完日志 = 故障已处理"** = 解除事件屏蔽

护士（主机）不会一直守在每个床位旁，而是预先按下"待命"按钮（AER）；病人一有情况（事件），床头呼叫器亮灯（CQE）；护士去翻病历本（Get Log Page），处理完后这一类报警才会重新"被允许"触发。

## 工作流程

```text
主机（Host）                  控制器（Controller）              日志页（Log Page）
    |                                |                              |
    |---- 提交 AER ① --------------->|                              |
    |---- 提交 AER ② --------------->|  (持有，不立即完成)             |
    |                                |                              |
    |                                |<---- 事件发生（如温度过高）     |
    |                                |                              |
    |<---- CQE: AET + AEI + LID -----|  (返回事件概要)                |
    |                                |---- 屏蔽同类型事件              |
    |                                |                              |
    |---- Get Log Page (RAE=0) -------------------------->|         |
    |                                                     |         |
    |<---- 返回事件详情 -------------------------------|         |
    |                                |                              |
    |                                |<---- 事件类型解除屏蔽          |
    |                                |                              |
    |---- 提交新 AER 补回 ---------->|  (维持未完成请求数)            |
    |                                |                              |
```

简化说明：上图是"提交 → 触发 → 读取 → 解除 → 补回"的最小循环；多个 AER 并行持有时，只有最先匹配上的事件占先返回 CQE，其他 AER 继续等待。

## 初学者案例

**场景：SSD 温度过高，主机怎么知道？**

1. 主机启动后通过 `nvme aer` 提交若干 AER 命令，保持在 Admin 提交队列里"挂着"。
2. 控制器内部温度传感器读数越过阈值。
3. 控制器把 AER 命令之一标记为完成，CQE DW0 写入：`AET=001b`（SMART/Health）、`AEI=0x03`（温度阈值跨越）、`LID=02h`（SMART/Health log）。
4. 主机收到中断，解析 CQE，发现是 SMART 事件。
5. 主机用 `nvme get-log /dev/nvme0 -i 2` 读取 SMART/Health 日志，确认当前温度与临界温度。
6. 主机把 RAE 位清零（默认就为 0）提交 Get Log Page，控制器在数据搬运成功后解除该事件类型的屏蔽。
7. 主机再次提交 AER 补回，确保事件流不中断。

> 速查：若发现事件"刷屏"，往往是底层条件未消除（温度还没降下来）。规范建议主机在清除事件前先调阈值或临时屏蔽。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| AER 没有超时 | AER 命令可无限期等待事件（"长寿命"命令） |
| 主机可提交多个 AER | 数量受 Identify Controller 通告的上限限制（超出返回状态码 05h） |
| 事件屏蔽 | 同一事件类型在报告后被屏蔽，直到 Get Log Page（RAE=0）成功完成 |
| 即时事件（Immediate） | 仅在事件发生时存在未完成 AER 才上报，不保留 |
| 一次性事件（One-Shot） | 报告后由控制器自动清除，无需读取日志 |
| 事件合并/排队 | 相同事件响应可合并；不同事件响应应排队等待 |
| 介质未就绪阻塞 | 若清除事件所需日志暂时无法读（media-not-ready），控制器阻塞该事件 |
| 控制器重置中止 | Reset 中止所有未完成 AER，且**不返回 CQE** |
| 启用已成立的条件 | 通过异步事件配置（Asynchronous Event Configuration）启用某事件时，若条件已成立，立即上报 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| AER vs 普通 Admin 命令 | 普通命令有超时、AER 没有；AER 是"挂着等事件" |
| AET vs AEI | AET（3 位）= 事件族（族级）；AEI（8 位）= 族内具体事件（个例） |
| 屏蔽 vs 禁用 | 屏蔽是事件族内的临时阻断（清除后自动解除）；禁用通过 Set Features 关闭整族通知 |
| 持久条件 vs 瞬态事件 | 持续存在的条件在清除后会立刻再触发；规范建议先调阈值再清除 |
| LID vs 事件类型 | LID 是日志页标识（用来 Get Log Page 读详情），不等于事件类型 |
| One-Shot vs Immediate | One-Shot 报告后自动清除；Immediate 仅在有 AER 时报告、不保留 |

## 进阶细节

- **CQE DW0 字段布局**（Figure 147）：
  - `AET`（bit 2:0）：异步事件类型（Event family）
  - `AEI`（bit 15:8）：异步事件信息（具体事件）
  - `LID`（bit 23:16）：日志标识符（指向 Get Log Page 目标）
  - 剩余位：保留
- **CQE DW1**：32 位事件特定参数（Event-specific Parameter），由各事件族定义。
- **事件族与典型成员**：
  - **Error**（日志支持型）：所有命令错误状态。
  - **SMART/Health**（日志支持型）：温度、可靠性、降级警告。
  - **Notice**（日志支持型）：命名空间属性变更、固件激活、ANA 状态变更、Discovery Log 变更。
  - **I/O-specific**（日志支持型）：预留（Reservation）状态、清理（Sanitize）状态。
  - **Immediate**（发生时上报型）：子系统关机（Subsystem Shutdown）。
  - **One-Shot**（自动清除型）：控制器数据队列（Controller Data Queue）尾指针/满载通知。
  - **Vendor**：厂商自定义事件。
- **Asynchronous Event Configuration Feature**（规范第 5 章 Feature 定义）：按事件族粒度启用/禁用通知；启用已成立的条件时控制器立即补发。
- **保留事件**：除即时事件外，已启用但无未完成 AER 时发生的事件，控制器会保留到下次 AER 完成时回报。
- **错误码**：`05h` Asynchronous Event Request Limit Exceeded（超出控制器允许的未完成 AER 数）。

## 规范依据

- [AER 命令与异步事件基础，PDF 第 195 页](../_source/pages/page-195.md)
- [CQE 字段与命令特定状态值，PDF 第 197 页](../_source/pages/page-197.md)
- [事件族定义 Figures 149-150（Error / SMART/Health），PDF 第 198 页](../_source/pages/page-198.md)
- [Notice / I/O-specific / Immediate / One-Shot 事件 Figures 151-155，PDF 第 199 页](../_source/pages/page-199.md)
- [Asynchronous Event Configuration Feature，PDF 第 399 页](../_source/pages/page-399.md)

## 相关阅读

- [admin-command-model.md](admin-command-model.md) - AER 在 Admin opcode 表中
- [keep-alive.md](keep-alive.md) - 关联终止的 AEN 触发源
- [controller-initialization.md](controller-initialization.md) - 初始化末尾配 AER 流程
- [firmware-update-lifecycle.md](firmware-update-lifecycle.md) - Firmware Activation AEN 来源
