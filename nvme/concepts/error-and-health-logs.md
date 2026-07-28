# 错误与健康日志（Error and Health Logs）

## 一句话说明

NVMe 暴露两个核心健康相关日志：**Error Information（01h）记录控制器全局、按时间倒序的扩展错误历史**；**SMART / Health（02h）记录控制器的当前关键告警与终生计数器**。两本日志互不替代——前者是"事故台账"，后者是"健康仪表盘"。

## 生活化类比

把这两本日志想成**急诊台账 vs 健康档案**：

- **Error Information = 急诊台账**
  - 谁出的问题（`SQID/CID/OPC/CSI/NSID`）
  - 问题什么时候出的（`ECNT` 单调递增）
  - 哪里出错了（`STS/PEL`，告诉你"出错参数最末位字节/位"）
  - 急诊台账容量固定（`ELPE` 项），满则**丢掉最老的**、插最新的。
- **SMART / Health = 体检档案**
  - 现在"哪里亮红灯"（Critical Warning）
  - 一辈子"跑了多少公里"（Data Units Read/Written、Host Commands、Controller Busy Time）
  - "环境怎么样"（温度、Power Cycle、Unsafe Shutdowns、错误累计）
  - 体检档案**跨断电保留**，除非字段自身说明例外。

急诊台账会在**断电和控制器级复位时被清空**；体检档案不会。

## 工作流程

```text
   命令错误 / 异步错误                  控制器/介质状态
         |                                  |
         v                                  v
   +--------------------+         +--------------------+
   | Error Information  |         | SMART / Health     |
   | 最新条目优先        |         | 当前告警 + 终生计数 |
   | 64 字节/条          |         | 512 字节           |
   +---------+----------+         +---------+----------+
             \----- Get Log Page -----/
             \--- (RAE=0 成功清事件) ---/

   触发路径:
   1) CQE 的 M=1 -> Error Information 必有扩展细节
   2) AER Error 类型异步事件 -> Error Information
   3) 健康状态轮询 -> SMART / Health
```

简化说明：上图是事件源与日志消费的简化映射。

### Error Information 条目（64 字节）关键字段

| 字段 | 含义 |
|------|------|
| `ECNT` | 持久唯一错误计数器，从 1 起；`ECNT=0` 表示"槽位无效/已丢失" |
| `SQID` / `CID` | 关联命令；`FFFFh/FFFFh` 表示"无具体命令的错误" |
| `STS` | 复制完成队列的状态字段 |
| `PEL` | 出错参数最末位字节/位定位（`FFFFh` 表示无关联） |
| `NSID` / `CSI` / `OPC` | 命令的命名空间、命令集、opcode（部分版本生效） |
| 命令专属数据 / 厂商日志 | 关联上下文与厂商扩展 |
| `LPVER` | 2.1 定义条目版本为 `1h` |

### SMART / Health 关键信号

| 信号 | 含义 |
|------|------|
| Critical Warning | 备件、温度、可靠性、全介质只读、易失备份失败、PMR 状态等的当前位图 |
| Composite Temperature | 实现定义的 Kelvin 值，反映控制器及关联命名空间 |
| Available Spare / Threshold | 剩余备件归一化值与异步事件阈值 |
| Percentage Used | 厂商估计的寿命消耗；100 = 寿命耗尽，**未必故障**，可能超过 100 直到饱和 255 |
| 终生计数器 | Data Units / Host Commands / Controller Busy Time（分钟）/ Power Cycles / Power-On Hours / Unsafe Shutdowns / 错误累计 |
| 温度历史 | 高/低阈值跨越分钟数、最多 8 个温度传感器读数（不支持则为 0）、热管理转换计数与时长 |

## 初学者案例

**场景：一条读命令返回了 `Generic Command Status` 错误**

1. 主机发现 CQE 的 `M=1`，意味着"有扩展错误细节"。
2. 主机发 `GetLogPage(LID=01h, NUMDL=N)` 读取 Error Information。
3. 控制器返回**最新** N 条 64 字节记录：`ECNT` 最大的在最前。
4. 主机在第一条里看 `STS`、`PEL`：知道错误类型与出错参数最末位字节/位。
5. 主机再去 `Identify Controller` 找到 `ELPE`（最大条目数），理解容量上限与"满则丢老"的策略。
6. 同一时间主机也会读 `LID=02h` 看 SMART：是否有 Critical Warning 亮灯、Percentage Used 是不是过高。
7. 关联路径：若 CQE 的 M=1 同时伴随 AER 异步事件，Async Event Information 字段会指向同一错误条目。

> 故障速查：遇到 `ECNT=0` 的条目 = 槽位已被回收或从未写入，**不要把它当第一条错误**。`M=0` 表示"没有扩展错误"，**不要为了完整性反复读 Error Information**。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| `M=1` 才有用 | CQE 的 More 位 = 1 时 Error Information 才有针对该命令的扩展细节 |
| AER Error 入口 | 异步事件 + Error 状态类型 → 主机应读 Error Information |
| 倒序排列 | Error Information 按时间倒序（最新在前） |
| 容量循环 | 容量由 `ELPE` 决定；满时**丢最老插最新** |
| 复位清空 | 断电与 Controller Level Reset 都会清空 Error Information |
| `ECNT` 持久 | 错误计数本身跨 power-off 保留；值 0 表示无效/丢失 |
| SMART 跨电 | SMART/Health 字段跨断电保留，**除非字段自身说明例外** |
| NSID 选择 | `NSID=0h` 或 `FFFFFFFFh` = 控制器级；其他值需要支持按 NSID 的 SMART |
| 2.1 视图统一 | 修订版 2.1 下，命名空间级 SMART 与控制器级 SMART **内容相同** |
| 旧版兼容 | 想兼容修订版 1.4 及以前，请用 `NSID=FFFFFFFFh` |
| 温度单位 | 所有温度字段都是**开尔文（K）**；不支持的传感器返回 0 |
| 计数单位 | Data Units 以 1000×512 字节为单位、向上取整、**不含元数据** |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Error Information vs SMART | 一个记**事件历史**、一个记**当前+终生状态** |
| `ECNT=0` vs `ECNT=1` | 0 = 槽位无效/已丢；1 是"第一个真实错误" |
| 控制器级 SMART vs 命名空间级 SMART | 2.1 下两者**内容相同**；旧版必须用 `FFFFFFFFh` |
| `M=1` vs AER | `M=1` 是 CQE 携带的标志；AER 是独立异步事件，但**两类入口**都指 Error Information |
| `PEL` 含义 | "出错参数最末位字节/位"，**不是"参数序号"** |
| Percentage Used 100 | 100 表示"寿命耗尽"，但**不**代表立即故障；可能继续上涨到 255 饱和 |
| Critical Warning 实时性 | 反映"Get Log Page 处理时"的状态，**可能**与产生 AER 时的状态不同 |
| 错误日志 vs Telemetry | Telemetry 是**详细诊断数据块**；Error Information 是**事件条目** |
| Power Cycles vs Unsafe Shutdowns | 一次 Unsafe Shutdown **同时**增加 Power Cycle 与 Unsafe Shutdown 计数 |
| 复位清空 vs 字段保留 | Error Information 清空但**ECNT 计数保留**；SMART 整体保留 |

## 进阶细节

- **错误历史计数器**（SMART 段）：
  - Media and Data Integrity Errors
  - Number of Error Information Log Entries（与 Error Information 联动）
- **寿命估算与饱和**：Percentage Used 厂商定义；超过 100 仍然合法，255 视为饱和上限。
- **Power History 范围**：Unsafe Shutdowns 的判定依据 shutdown-notification 规则，可能涉及带外掉电原因标志（与 Persistent Event Log 联动）。
- **热管理转换计数**：进出节流状态各记一次，配合 Thermal Excursion 事件可还原温度策略。
- **`STS`/`PEL` 配合**：`STS` 保留完成状态（包含 SCT/SC），`PEL` 在错误源自命令参数时定位到具体字节/位；非命令错误 `PEL=FFFFh`。
- **跨日志协同**：
  - Error Information 与 AER 用 Asynchronous Event Information 字段相互指认。
  - SMART 的 Critical Warning 配合 Asynchronous Event Configuration 控制是否上抛 AER。
- **命名空间级 SMART 的限制**：修订版 2.1 即便支持 `NSID=具体值` 的请求形式，**返回内容仍与控制器级一致**；需要按命名空间细分的字段在 Identify Namespace 数据结构中。
- **`LPVER` 字段**：当前为 `1h`；未来版本扩展时，老主机会按版本字段决定字段解析范围。
- **LID 02h 与命名空间状态字段**：包含按 Endurance Group 汇总的 Critical Warning Summary 字节，反应多个 EG 的整体健康。

## 规范依据

- [Error Information 总述与 Figure 205 前半，PDF 第 229 页](../_source/pages/page-229.md)
- [Error Information 续、SMART 作用域与 NSID 选择，PDF 第 230 页](../_source/pages/page-230.md)
- [SMART 告警与初始健康字段，PDF 第 231 页](../_source/pages/page-231.md)
- [SMART 终生计数、错误与热历史，PDF 第 232–234 页](../_source/pages/page-232.md)

## 相关阅读

- [log-page-retrieval.md](log-page-retrieval.md) - 通用 Get Log Page 读取机制
- [asynchronous-event-reporting.md](asynchronous-event-reporting.md) - AER 触发错误日志读取
- [persistent-event-log.md](persistent-event-log.md) - 跨断电事件流的补全
- [telemetry-capture-logs.md](telemetry-capture-logs.md) - 更详细的诊断数据块
