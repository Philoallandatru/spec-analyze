# 可预测延迟模式（Predictable Latency Mode, PLM）

## 一句话说明

可预测延迟模式是 NVMe 提供的一种**服务质量（Quality of Service, QoS）特性**，它把每个支持该特性的 NVM Set（NVM 集合）在时间轴上切成两段交替的窗口——确定性窗口（Deterministic Window, DTWIN）和非确定性窗口（Non-Deterministic Window, NDWIN），让主机在 DTWIN 内能获得有界的 I/O 延迟。

## 生活化类比

把 NVM Set 想象成一家**牙科诊所**的诊室：

- **DTWIN** = 正常营业时间，挂号、就诊都有严格排期，医生专心给预约的病人看病（主机 I/O 延迟有上限）
- **NDWIN** = 营业结束后的清洁与设备维护时段，清洁阿姨、消毒员、巡检员轮番上阵（控制器做 GC、刷固件、刷新 FTL 表）
- **诊所不会在营业时段同时大扫除**——这就是 DTWIN 的延迟可预测性
- **诊所的"今日排班表"** = 每个 NVM Set 的日志页面（LID `0Ah`），上面写着当前是哪个窗口、还剩多少个挂号名额

## 工作流程

```text
                   读/写次数/时间预算耗尽
   +-----------+ -----------------------> +-----------+
   |  DTWIN    |                          |  NDWIN    |
   |  确定性    | <----------------------- |  非确定性  |
   |  窗口     |   最小 NDWIN 驻留时间满足   |  窗口     |
   +-----------+                          +-----------+
        ^                                       |
        |                                       |
        |            后台维护完成                 |
        +---------------------------------------+
   (典型参数：DTWIN Reads Typical / DTWRT，按 4 KiB 随机读计数)
   (写入单位：Optimal Write Size，按 NVM Set 的最佳写入粒度)
```

**端到端流程**：

1. 主机通过 `Set Features` Feature ID `13h` 在某个 NVM Set 上**启用** PLM。
2. 启用成功后，NVM Set 立即进入 NDWIN，做后台准备。
3. 准备完成后（且最小 NDWIN 驻留时间满足），进入 DTWIN，对外提供延迟可预测的 I/O。
4. 读/写次数或时间达到预算后，控制器自动切回 NDWIN。
5. 主机随时可通过 `Get Log Page` LID `0Ah` 读取当前窗口状态、剩余预算、典型值等。

## 初学者案例

**场景：数据库想保证 99% 的读请求延迟低于 1ms，但 GC 在时不时把延迟打飞，怎么办？**

1. DBA 找到 SSD 厂商，询问"这块盘支持可预测延迟模式吗？"
2. 厂商答复："支持，但前提是你的 NVM Set 在 NVM Sets 特性下被标识为可独立调度。"
3. DBA 用 `nvme get-feature /dev/nvme0n1 -f 0x13`（Predictable Latency Mode Config）查询该 NVM Set 是否启用了 PLM。
4. 若没启用，就用 `Set Features` 启用，并独立配置"读/写/时间"三个警告阈值。
5. 启用后，**最初的窗口一定是 NDWIN**——后台要准备干净的环境。
6. DBA 看到日志报 `PLMW=001b`（DTWIN）后，把应用流量切过来。
7. 应用每发一次 4 KiB 随机读，日志里的 `DTWRE`（DTWIN Reads Estimate）就减 1；当减到配置阈值时，触发异步事件——DBA 知道"该窗口快到顶了，别发新事务了"。
8. 写满或超时，控制器切到 NDWIN，**应用此时要避免在该 NVM Set 上发延迟敏感 I/O**。

> 注意：`DTWRT/DTWW/DTWTM` 是**终生最坏情况**的静态值，`DTWRE/DTWWE/DTWTE` 才是**当前活动负载下**还能用多少的动态值。混淆这两组数是初学者最常犯的错。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 窗口是 NVM Set 级的 | 一个 NVM Set 独立维护自己的 DTWIN/NDWIN 切换，不影响其他 NVM Set |
| 启用后必从 NDWIN 起 | 首次启用必须先做完后台准备才能进入 DTWIN；不会跳过 NDWIN |
| DTWIN 内只承诺延迟 | "可预测"指的是**延迟上限**，不保证吞吐量；DTWIN 仍可能比 NDWIN 慢 |
| 读按 4 KiB 随机读算 | DTWIN 读预算的单位是 4 KiB 随机读，不是 IOPS |
| 写按 Optimal Write Size 算 | DTWIN 写预算的单位是 NVM Set 的 Optimal Write Size，跨设备会变 |
| 异步事件 + 聚合日志 | 警告事件以 NVM Set ID 形式入聚合日志；用 `RAE=0` 读单 NVM Set 日志可清除其事件位 |
| DTWIN→NDWIN 自主切换 | 控制器可自主从 DTWIN 切到 NDWIN（典型值或最大读/写/时间超限时） |
| Feature `13h` 只启用 | Feature `13h`（PLM Config）只负责启用 + 配置事件阈值；不能选窗口 |
| Feature `14h` 选窗口 | Feature `14h`（PLM Window）才能在 DTWIN 与 NDWIN 之间请求切换 |
| 模式禁用时 `14h` 无效 | 控制器未启用 PLM 时，Feature `14h` 被视为无效 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 典型值 vs 可靠估计 | 典型值（DTWRT/DTWW/DTWTM 等）是**终生最坏**的静态值；可靠估计（DTWRE/DTWWE/DTWTE）是**当前**还能用多少的动态计数 |
| DTWIN vs 正常 I/O 性能 | DTWIN 不是"性能更好的模式"，而是"延迟可被预算约束的模式"，吞吐量可能反而更低 |
| 确定性偏差 vs 预算耗尽 | 偏差（Deterministic Excursion）= 控制器**没有**遵守承诺延迟；预算耗尽 = 读/写/时间到顶。两者都会触发 DTWIN→NDWIN |
| Feature `13h` vs `14h` | `13h`（Config）= 启用模式 + 设事件阈值；`14h`（Window）= 在 DTWIN/NDWIN 之间挑一个 |
| NDWIN 内的 I/O | NDWIN 内仍可发 I/O，但**无延迟保证**；只是后台也在跑维护 |
| `RAE=0` vs `RAE=1` | `RAE=0` 读完日志后会**清除**事件位并从聚合日志移除对应 NVM Set；`RAE=1` 只读不消费 |
| DTWIN 时间 vs DTWIN 次数 | DTWTM 是"最长可停留多久"（毫秒），DTWRT/DTWW 是"最多可做多少次"（按单位计数），三者是**或**的关系，任何一个到顶就切 |

## 进阶细节

- **典型值与可靠估计的来源**：DTWRT/DTWW/DTWTM/NTWTML/NTWTMH 是设备终生最坏情况，**NVM 子系统**生命周期内不变；DTWRE/DTWWE/DTWTE 由控制器根据近期主机活动与运行条件动态维护，读一次减一次，不会重置。
- **状态机（规范 8.1.18.1）**：PLM 状态机有 3 个值——Disabled、DTWIN、NDWIN；切换由控制器内部事件触发（自主切换）或主机用 Feature `14h` 请求触发；自主切换会写日志的 DEAT/MVEAT 位。
- **Feature `13h` 的事件配置**（规范 5.1.25.1.13）：可以独立启用"读警告""写警告""时间警告"三类警告事件，以及"偏差/超限"自主切换事件；事件位以 16 位 `ETYP` 字段编码。
- **Feature `14h` 的特殊行为**（规范 5.1.25.1.14）：请求进入 DTWIN 时若最小 NDWIN 驻留时间（NTWTML）未满足，命令完成会被**延迟**到条件满足；命令完成的那一刻才意味着窗口切换的"边界"。
- **聚合日志机制**：LID 字段未在 PLM 主题中显式指定时，可参考 Asynchronous Event Reporting 概念文档；主机只关心有序的 NVM Set ID 列表，新事件触发异步通知。
- **与 NVM Set 的关系**：NVM Set 是在创建 NVM 子系统时按容量与磨损均衡划分的；PLM 把每个 NVM Set 视为独立的"时间域"，互不干扰。
- **DPA/HMB 不参与**：PLM 是控制器内部逻辑，与 DPA（Direct Placement Accelerator）写入策略或 HMB（Host Memory Buffer）分配都无直接耦合。
- **NVM Sets 与 Endurance Groups 关系**：NVM Set ⊂ Endurance Group；PLM 按 NVM Set 调度，但底层寿命统计走 Endurance Group。

## 规范依据

- [每个 NVM Set 的日志选择器与 PLM 概述，PDF 第 248 页](../_source/pages/page-248.md)
- [PLM 窗口/事件/典型值/可靠估计字段定义，PDF 第 249 页](../_source/pages/page-249.md)
- [Predictable Latency Mode Config 特性（Feature `13h`），PDF 第 405 页](../_source/pages/page-405.md)
- [Predictable Latency Mode Window 特性（Feature `14h`），PDF 第 406 页](../_source/pages/page-406.md)
- [NVM Set 与 Endurance Group 资源模型，PDF 第 99 页](../_source/pages/page-099.md)

## 相关阅读

- [nvm-sets-and-endurance-groups.md](nvm-sets-and-endurance-groups.md) - NVM Set 是 PLM 作用单位
- [power-state-descriptors.md](power-state-descriptors.md) - 电源状态切换背景
- [namespace-access-models.md](namespace-access-models.md) - 访问模型与 QoS 背景
- [controller-support-requirements.md](controller-support-requirements.md) - PLM Feature 的支持要求
