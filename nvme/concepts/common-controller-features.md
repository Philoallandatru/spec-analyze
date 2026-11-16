# 控制器通用特性（Common Controller Features）

**控制器通用特性（Common Controller Features）** 是 NVMe 跨越所有传输协议的基础配置项（Feature ID），主机通过 `Set Features` 和 `Get Features` 命令读写它们，用来调整控制器的仲裁权重、功耗状态、温度阈值、写缓存和热管理等核心策略。[PDF pp. 395-398](../_source/pages/page-395.md)

> **类比理解**：把通用特性想成汽车仪表盘上的常用旋钮——仲裁（`01h`）是各档位动力分配，电源管理（`02h`）是手动切档，APST（`0Ch`）是等红灯自动启停，温度阈值（`04h`）与 HCTM（`10h`）分别是水温报警线与发动机过热降功率，易失性写缓存（`06h`）则是"暂存缓冲换速度"的开关。

---

## 生效模型与核心 FID 总览

当 `Set Features` 成功完成后，新配置会立即影响**后续**命令或行为（已在执行中的命令不一定会受影响）：[PDF p. 395](../_source/pages/page-395.md)

| Feature ID | 特性名称 | 核心参数与关键约束 |
|---|---|---|
| **`01h`** | **Arbitration**（仲裁） | `CDW11`：`HPW`[31:24]、`MPW`[23:16]、`LPW`[15:8]（0-based 权重）；`AB`[2:0]（仲裁突发为 2 的幂，`111b` = 无限制）[PDF p. 396](../_source/pages/page-396.md) |
| **`02h`** | **Power Management**（电源管理） | `CDW11`：`PS`[4:0]（目标功耗状态索引）、`WH`[7:5]（Workload Hint）。CQE DW0 返回当前或正在转入的 PS；多控制器冲突请求同一 Domain 时最终状态未指定 [PDF pp. 394-396](../_source/pages/page-394.md) |
| **`04h`** | **Temperature Threshold**（温度阈值） | `TMPSEL`[19:16]（`0h`=Composite，`1h..8h`=Sensor 1..8）、`THSEL`[21:20]（`00b` 过温/`01b` 低温）、`TMPTHH`[24:22]（迟滞，开尔文）。**复合温度过温阈值强制支持**；已实现传感器均需支持过温与低温 [PDF pp. 397-398](../_source/pages/page-397.md) |
| **`06h`** | **Volatile Write Cache**（易失性写缓存） | 仅 `WCE`（bit 0）。禁用（`0`）时写入 NS 的数据必须立即持久化。非易失缓存不归此特性管；若无易失缓存，Get/Set 均报 `Invalid Field in Command` [PDF p. 398](../_source/pages/page-398.md) |
| **`0Ch`** | **APST**（自主功耗状态转换） | `CDW11.APSTE`（bit 0）启用 + 256B 物理连续表（32 项×8B，每项含空闲超时 `ITPT`[31:8] 毫秒与目标非操作态 `ITPS`[7:3]；不支持的 PS 项清零）[PDF pp. 400-401](../_source/pages/page-400.md) |
| **`0Eh`** | **Timestamp**（时间戳） | 48 位毫秒计数器；`Origin` 指示复位/主机初始化，`SYNC=1` 提示存在未计入的间隔（如深度休眠）；复位后可能倒退，**不可用于安全/防重放场景** [PDF pp. 402-403](../_source/pages/page-402.md) |
| **`10h`** | **HCTM**（主机控制热管理） | 两级复合温度阈值：`TMT1`（轻度降温）与 `TMT2`（强制降温）；设为 0 禁用该级，两者均非零时必须 `TMT1 < TMT2` [PDF pp. 403-404](../_source/pages/page-403.md) |
| **`12h`** | **Read Recovery Level**（读取恢复等级） | 支持 NVM Set 时作用于单个 NVM Set，否则作用于整个子系统；Namespace 继承所属 NVM Set 设置 [PDF p. 405](../_source/pages/page-405.md) |
| **`1Ah`** | **Spinup Control**（启动控制） | 仅适用于含旋转介质（HDD）的 Endurance Group（主机控制/自动启动），配置持久化；无旋转介质返回错误 [PDF p. 411](../_source/pages/page-411.md) |
| **`1Bh`** | **Power Loss Signaling**（掉电信号） | 配置模式：`Disabled` / `Emergency Power Fail` / `Forced Quiescence`；**在 `Forced Quiescence` 处理中禁止修改** [PDF p. 412](../_source/pages/page-412.md) |
| **`80h`** | **Software Progress Marker**（软件进度标记） | 8 位持久化饱和计数器（上限 255）；预启动环境成功初始化后递增，OS 启动成功后清零，用于诊断反复启动失败 [PDF p. 422](../_source/pages/page-422.md) |

---

## 关键交互与边界机制

### 温度迟滞（Hysteresis）防抖规则（FID `04h`）

```text
过温告警：温度 >= 阈值 时触发，直到 温度 < (阈值 - TMPTHH) 才恢复
低温告警：温度 <= 阈值 时触发，直到 温度 > (阈值 + TMPTHH) 才恢复
```

温度越限时 SMART Critical Warning 对应位置 `1`；恢复正常后清零并可触发迟滞恢复异步事件。[PDF pp. 397-398](../_source/pages/page-397.md)

### APST（`0Ch`）与 NOPPME（`11h`）的功耗交互（Figure 394）

`NOPPME`（Non-Operational Power State Max Power Enable）控制控制器处于非操作性低功耗状态时，后台维护任务（如 GC、磨损均衡）能否短暂突破当前功耗上限：[PDF pp. 401, 404-405](../_source/pages/page-401.md)

| `APSTE` | `NOPPME` | 进入非操作态方式 | 后台工作的功耗限制 |
|:---:|:---:|---|---|
| `1` | `1` | 主机请求或空闲超时 | **宽松模式**：允许短暂超出当前功耗上限（不超过最近操作态限制），利于热管理与性能恢复 |
| `0` | `1` | 仅主机请求 | 允许后台工作超限 |
| `1` | `0` | 主机请求或空闲超时 | 不允许后台工作超出功耗限制 |
| `0` | `0` | 仅主机请求 | **严格模式**：禁止超限，热管理与性能恢复可能受限 |

---

## 易混概念对比

| 易混概念 | 实际区别 |
|---|---|
| **`HPW/MPW/LPW` vs `AB`** | 权重决定各优先级队列的调度比例；`AB`（Arbitration Burst）限制单次连续从同一队列取出的命令数 |
| **`Power Mgmt PS` vs `APST ITPS`** | `PS` 是主机手动请求的功耗状态；`ITPS` 是空闲超时后自动跳入的目标非操作性状态 |
| **`WCE = 0` vs 无易失性写缓存** | `WCE = 0` 表示控制器有易失性缓存但当前禁用；完全没有易失性缓存时 Get/Set `06h` 均会报错 |
| **`Timestamp (0Eh)` vs `Keep Alive (0Fh)`** | `Timestamp` 是 48 位毫秒时钟；`Keep Alive` 是检测主机-控制器关联存活的看门狗定时机制 |

---

## 关联与引申概念

- **Feature ID & 作用域**：所有特性共用的 Set/Get 语义、Save 位、UUID 选择与复位持久性，详见 [feature-values-and-scope.md](feature-values-and-scope.md)。
- **WRR 仲裁**：`FID 01h` 配合队列优先级 `QPRIO` 控制命令调度，详见 [command-ordering-and-arbitration.md](command-ordering-and-arbitration.md)。
- **SMART / Health Log**：温度阈值与掉电/磨损告警最终体现在 SMART 日志与 AER 通知中，详见 [error-and-health-logs.md](error-and-health-logs.md)。
- **延伸阅读**：
  - [power-state-descriptors.md](power-state-descriptors.md) — Identify 报告的功耗状态描述符格式
  - [host-memory-buffer.md](host-memory-buffer.md) — `FID 0Dh` 主机内存缓冲区配置
  - [power-loss-signaling.md](power-loss-signaling.md) — `FID 1Bh` 掉电信号与静默流程
  - [host-metadata-and-management-addresses.md](host-metadata-and-management-addresses.md) — `FID 7Dh..7Fh` 主机元数据特性

---

## 规范依据

- [通用特性生效边界、仲裁与电源管理，PDF pp. 394-396](../_source/pages/page-394.md)
- [温度阈值、迟滞与易失性写缓存，PDF pp. 397-398](../_source/pages/page-397.md)
- [APST 表结构与 APSTE×NOPPME 交互（Figure 394），PDF pp. 400-401](../_source/pages/page-400.md)
- [Timestamp、HCTM 与 Read Recovery Level，PDF pp. 402-405](../_source/pages/page-402.md)
- [Spinup Control 与 Power Loss Signaling，PDF pp. 411-412](../_source/pages/page-411.md)
- [Software Progress Marker，PDF p. 422](../_source/pages/page-422.md)
