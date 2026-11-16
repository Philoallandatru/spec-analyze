# 错误信息与健康日志（Error and Health Logs）

NVMe 通过两份核心日志暴露设备的健康与异常状态：**错误信息日志（Error Information Log，`LID 01h`）** 是按时间倒序排列的控制器级“事故台账”，记录扩展的命令错误与异步错误；**SMART / 健康信息日志（SMART / Health Information Log，`LID 02h`）** 则是跨断电保留的“健康仪表盘”，提供当前告警位图与终生累计计数器。[PDF 第 229-231 页](../_source/pages/page-229.md)

## 心智模型：急诊台账 vs 体检档案

```text
命令错误 (CQE.M=1) / 异步错误 (AER)       控制器与介质运行状态
                |                                   |
                v                                   v
      +--------------------+              +--------------------+
      | Error Information  |              | SMART / Health     |
      | 事故台账 (64B/条)  |              | 体检档案 (512B)    |
      | 最新在前，掉电清空 |              | 瞬时告警+终生计数  |
      +---------+----------+              +---------+----------+
                \--------- Get Log Page ------------/
```

## 错误信息日志（`LID 01h`）

当命令完成项（CQE）中 **`M = 1`**（More）或控制器上报 Error 类型异步事件（AER）时，主机应通过 `Get Log Page` 读取 `LID 01h` 获取详细上下文。日志容量由 Identify Controller 中的 `ELPE` 字段决定，写满后自动覆盖最老条目。[PDF 第 229-230 页](../_source/pages/page-229.md)

```text
错误信息条目（64 字节，简化结构）
+--------+------+------+------+-----+------+-----+-----+----------+
| ECNT   | SQID | CID  | STS  | PEL | NSID | CSI | OPC | 详情/厂商|
|  0..7  | 8..9 |10..11|12..13|14..15|24..27| 30 | 31  | 剩余字节 |
+--------+------+------+------+-----+------+-----+-----+----------+
```

| 关键字段 | 含义与规则 |
|---|---|
| **`ECNT`** | 持久化的唯一错误计数器，从 `1` 开始单调递增（全 1 后回绕到 `1`）；**`ECNT=0` 表示槽位无效或已丢失** |
| **`SQID` / `CID`** | 关联到出错命令；若与具体命令无关，则 `SQID=CID=FFFFh` |
| **`STS` / `PEL`** | 保留完成状态（Status）；`PEL` 定位出错命令参数的最末位字节/位（非参数错误则为 `FFFFh`） |
| **`NSID` / `CSI` / `OPC`** | 关联命名空间、命令集标识与操作码（受日志版本 `LPVER=1h` 约束） |

- **清空规则**：`ECNT` 计数器本身跨断电保留，但控制器在**断电和控制器级复位（CLR）后应清空**错误信息日志条目。

## SMART / 健康信息日志（`LID 02h`）

SMART 数据默认跨断电保留（除非个别字段另有说明）。请求控制器级数据时可使用 `NSID=0h` 或 `FFFFFFFFh`（为兼容 1.4 及更早版本，**主机应优先使用 `FFFFFFFFh`**）；在修订版 2.1 中，即便支持按命名空间请求，其返回内容也与控制器级完全相同。[PDF 第 230-234 页](../_source/pages/page-230.md)

### 1. 瞬时告警与状态

| 字段 | 含义 |
|---|---|
| **Critical Warning（关键告警）** | 位图，反映处理 `Get Log Page` 瞬间的状态：备件不足、温度异常、可靠性下降、全介质只读、易失备份失败、PMR 异常等；可配置为触发 AER |
| **Composite Temperature（综合温度）** | 实现定义的开尔文（Kelvin）温度，代表控制器及关联命名空间 |
| **Available Spare / Threshold** | 归一化剩余备件百分比及触发告警的阈值 |
| **Percentage Used（已用百分比）** | 厂商估计的寿命消耗；`100` 表示设计寿命耗尽，但**不等于立即故障**，允许继续递增至饱和值 `255` |

### 2. 终生累计计数器

- **Data Units Read / Written**：以 `1000 × 512` 字节为单位向上取整计数，**不含元数据**。
- **Host Read / Write Commands** & **Controller Busy Time**：已完成的读写命令数，以及控制器忙于处理 I/O 的累计分钟数。
- **Power History**：上下电次数（Power Cycles）、累计通电小时数、非安全关机次数（Unsafe Shutdowns；一次非安全关机同时增加 Power Cycle 与 Unsafe Shutdown 计数）。
- **Error & Thermal History**：介质/数据完整性错误累计、错误日志条目终生累计数、高/低温度阈值跨越分钟数、热管理转换计数，以及最多 8 个温度传感器读数（Kelvin，不支持则返回 `0`）。

## 关联与引申概念

- **完成队列项与状态（CQE）**：提供 `M=1` 提示位和复制进错误日志的 `STS` 字段，见 [完成队列项与状态](completion-queue-entry-and-status.md)。
- **异步事件报告（AER）**：在发生严重错误或 Critical Warning 亮灯时主动通知主机去读日志，见 [异步事件报告](asynchronous-event-reporting.md)。
- **日志页读取（Get Log Page）**：定义 `LID`、`RAE` 清事件等通用读取机制，见 [日志页读取](log-page-retrieval.md)。
- **持久化事件日志 & Telemetry**：用于跨断电事件流归档与深度固件诊断，见 [持久化事件日志](persistent-event-log.md) 与 [Telemetry 捕获日志](telemetry-capture-logs.md)。

## 规范依据

- [Error Information 总述与 Figure 205 前半，PDF 第 229 页](../_source/pages/page-229.md)
- [Error Information 续与 SMART 作用域规则，PDF 第 230 页](../_source/pages/page-230.md)
- [SMART 告警与初始健康字段，PDF 第 231 页](../_source/pages/page-231.md)
- [SMART 终生计数器、关机/错误与温度历史，PDF 第 232-234 页](../_source/pages/page-232.md)
