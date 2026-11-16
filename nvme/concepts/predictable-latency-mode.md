# 可预测延迟模式（Predictable Latency Mode, PLM）

可预测延迟模式（PLM）是 NVMe 提供的服务质量（QoS）特性：它将每个支持的 **NVM Set** 在时间轴上划分为交替运行的**确定性窗口（DTWIN）**和**非确定性窗口（NDWIN）**，使主机在 DTWIN 内获得有界的 I/O 延迟，而控制器将垃圾回收（GC）等后台维护集中到 NDWIN 中执行。[PDF pp. 248-250](../_source/pages/page-248.md)

可以把 PLM 想成**牙科诊所的排班表**：
- **DTWIN（确定性窗口）** = 正常营业接诊时间，预约患者按排期就诊，等待时间有上限（延迟可预测）。
- **NDWIN（非确定性窗口）** = 营业结束后的清洁消毒时段，诊所集中做设备维护（控制器做 GC、刷 FTL 等后台操作），此时仍可接待急诊但无时间保证。
- **NVM Set 日志页（LID `0Ah`）** = 诊所排班表，实时显示当前窗口状态与剩余名额。

---

## 窗口状态机与工作流程

PLM 状态机（规范 8.1.18.1）包含 **Disabled**、**DTWIN** 和 **NDWIN** 三种状态，按 NVM Set 独立运行，互不干扰：

```text
  Set Features (FID 13h) 启用 PLM
        |
        v (后台准备完成，初始必进 NDWIN)
  +-----------+    最小 NDWIN 驻留时间(NTWTML)满足     +-----------+
  |   NDWIN   | -------------------------------------> |   DTWIN   |
  | 非确定性  |   (主机 FID 14h 请求或控制器转换)       |  确定性   |
  |   窗口    | <------------------------------------- |   窗口    |
  +-----------+    读/写/时间预算耗尽 或 确定性偏差     +-----------+
```

**端到端使用流程**：
1. **启用与阈值配置**：主机通过 `Set Features`（FID `13h`）为指定 NVM Set 启用 PLM，并独立配置读/写/时间警告事件阈值及偏差/超限事件。命令仅在后台准备完成后返回成功，成功后该 NVM Set **初始进入 NDWIN**。[PDF pp. 405-406](../_source/pages/page-405.md)
2. **窗口切换**：当最小 NDWIN 驻留时间（`NTWTML`）满足后，主机可通过 `Set Features`（FID `14h`）请求进入 DTWIN（若未满 `NTWTML`，命令完成会被延迟至条件满足，命令完成即标志着切换边界）。[PDF pp. 406-407](../_source/pages/page-406.md)
3. **运行与监控**：在 DTWIN 内执行延迟敏感 I/O；当读/写操作数或停留时间达到预算，或发生确定性偏差（Deterministic Excursion）时，控制器自主切回 NDWIN。主机可通过 `Get Log Page`（LID `0Ah`）查询当前窗口状态与剩余预算。

---

## 日志报告与事件聚合机制

### 单 NVM Set 日志（LID `0Ah`）

| 报告类别 | 核心字段与说明 |
|----------|----------------|
| **窗口状态** | 当前状态：已禁用（Disabled）、确定性窗口（DTWIN）或非确定性窗口（NDWIN） |
| **事件位** | 读/写/时间警告事件（估计值跌破阈值时触发），以及自主 DTWIN→NDWIN 转换事件（偏差或超限） |
| **静态值（典型值）** | `DTWRT`（典型读预算）、`DTWW`（典型写预算）、`DTWTM`（最大 DTWIN 时间）、`NTWTML`/`NTWTMH`（NDWIN 最小时间上下限）。基于终身最坏情况，子系统生命周期内不变 |
| **可靠估计值** | `DTWRE`、`DTWWE`、`DTWTE`：当前活动负载下 DTWIN 剩余可用的读次数、写次数和毫秒时间，随操作动态递减 |

**计数单位规则**：
- **读预算**：按 **4 KiB 随机读**计数（不是通用 IOPS）。
- **写预算**：按该 NVM Set 的 **Optimal Write Size**（最优写入大小）计数。

### 聚合日志（Aggregate Log）

为了避免主机轮询所有 NVM Set，NVMe 提供了按升序排列的非零 NVM Set ID 聚合日志：[PDF pp. 249-250](../_source/pages/page-249.md)

```text
  事件发生 ─> 对应 NVM Set ID 加入有序聚合日志 ─> 发送异步事件通知 (AER)
                                                         |
  对应条目从聚合日志移除 <─ 用 RAE=0 读该 NVM Set 日志 <─ 主机读取聚合日志定位 Set ID
```

- 新的 NVM Set 加入聚合日志时会触发异步事件；读取超过列表长度的部分返回零值。
- 主机必须使用 `RAE=0`（Retain Asynchronous Event = 0）读取对应 NVM Set 的日志页，才能清除其事件位并从聚合日志移除该条目；`RAE=1` 只读不清除。

---

## 实践要点与易混淆概念

**初学者排错提示**：
- **混淆静态值与动态估计**：`DTWRT/DTWW/DTWTM` 是设备终身最坏情况的静态参考，不会随当前负载变化；真正用于判断本次窗口还剩多少预算的是 `DTWRE/DTWWE/DTWTE`。
- **DTWIN 不等于高吞吐**：DTWIN 承诺的是**延迟有界**，为了避免后台干扰，其吞吐量有时反而低于普通模式。

| 容易混淆的概念 | 实际区别 |
|----------------|----------|
| **Feature `13h` vs `14h`** | `13h`（PLM Config）用于启用模式并设置事件阈值；`14h`（PLM Window）用于在 DTWIN/NDWIN 间切换（模式禁用时 `14h` 无效） |
| **确定性偏差 vs 预算耗尽** | 偏差（Excursion）指控制器未能满足承诺的延迟上限；预算耗尽指读次数、写次数或停留时间三者之一先到顶（"或"关系） |
| **NDWIN 内的 I/O** | NDWIN 内依然可以正常发送 I/O，只是控制器同时在做后台维护，**不提供延迟保证** |

---

## 关联与引申概念

- **NVM Set 与 Endurance Group**：NVM Set 是 Endurance Group 内部的容量与隔离划分单元；PLM 以 NVM Set 为独立调度时间域，而底层磨损/寿命统计仍归属 Endurance Group。[PDF p. 99](../_source/pages/page-099.md)
- **AER（Asynchronous Event Reporting，异步事件报告）**：控制器通过异步事件主动通知主机读取 PLM 聚合日志，避免盲目轮询。[PDF pp. 195-202](../_source/pages/page-195.md)
- **FDP（Flexible Data Placement）**：与 PLM 同属改善 QoS 尾延迟的技术，FDP 侧重通过写入放置减少 GC 写放大，PLM 则通过时间窗口隔离 GC。
- **相关文档**：
  - [NVM 集与耐久度组](nvm-sets-and-endurance-groups.md) — PLM 作用的资源粒度
  - [异步事件报告](asynchronous-event-reporting.md) — 聚合日志的事件通知与 `RAE` 确认机制
  - [命名空间访问模型](namespace-access-models.md) — 访问模型与 QoS 背景
  - [控制器支持要求](controller-support-requirements.md) — PLM 特性的合规支持要求

## 规范依据

- [每个 NVM Set 的日志选择器与 Endurance Group 边界，PDF p. 248](../_source/pages/page-248.md)
- [PLM 窗口状态、事件与静态/可靠估计值字段，PDF pp. 249-250](../_source/pages/page-249.md)
- [有序聚合列表与事件清除机制，PDF p. 250](../_source/pages/page-250.md)
- [PLM Config（FID 13h）与 PLM Window（FID 14h），PDF pp. 405-407](../_source/pages/page-405.md)
