# 设备自检（Device Self-test）

设备自检（Device Self-test，DST）是一条 Admin 命令，用于触发 SSD 控制器或命名空间的内部诊断测试（短测试、扩展测试、厂商测试），或执行主机发起的刷新（Host-Initiated Refresh），也可中止正在运行的测试。测试进度与最近 20 条历史结果保存在自检日志页（Log Page `06h`）中。[PDF 第 210-212 页](../_source/pages/page-210.md)

## 心智模型：硬盘体检

- **短测试（Short test）**：快速抽检（通常数分钟），检查控制器与关键介质路径。
- **扩展测试（Extended test）**：全面深度扫描（可能耗时数十分钟），不能用短测试替代。
- **主机发起的刷新（Host-Initiated Refresh）**：不测试硬件介质，仅让设备重建/刷新内部健康快照。
- **自检日志（Log Page `06h`）**：保存当前运行状态 + **最近 20 次**已完成或中止的结果（槽位 1 最新，旧结果依次向后推至槽位 20）。

## 核心状态机与命令参数

```text
   [无测试运行]
        |  启动 (STC = 1h / 2h / 3h / Eh)
        v
   [测试正在运行] <──── 中止命令 (STC = Fh) ────┐
        |                                       |
        | 尝试再启动新标准测试                    | 终止操作 -> 写新结果到槽位 1
        v                                       | -> 清除当前状态 -> 成功返回
   拒绝: Device Self-test in Progress           |
        +───────────────────────────────────────┘
```

### 1. 测试类型（`STC` 字段）

| `STC` 值 | 含义 | 备注 |
|---|---|---|
| `1h` | 短测试（Short test） | — |
| `2h` | 扩展测试（Extended test） | — |
| `3h` | 主机发起的刷新（Host-Initiated Refresh） | **忽略 `NSID` 字段**，作用于整个控制器 |
| `Eh` | 厂商特定测试 | 仅此时命令双字 15（`CDW15`）有意义，其余情况保留 |
| `Fh` | 中止当前测试 | 若无测试在运行，直接成功返回且不修改日志 |

### 2. 测试范围（`NSID` 字段）与作用域（`SDSO`）

| `NSID` 取值 | 测试覆盖范围 |
|---|---|
| `00000000h` | 仅测试控制器（Controller），不含任何命名空间 |
| `00000001h`–`FFFFFFFEh` | 测试单个有效且活跃的命名空间 |
| `FFFFFFFFh` | 测试所有当前已挂接且可访问的命名空间 |

- **作用域控制（`SDSO`）**：来自 Identify Controller 的 `DSTO` 字段。`SDSO=0` 表示自检运行状态仅当前控制器可见；`SDSO=1` 表示整个 NVM 子系统内所有控制器均可见。[PDF 第 212 页](../_source/pages/page-212.md)

## 结果日志结构与原子更新规则

自检日志包含“当前操作状态（`none` / `short` / `extended`）+ 完成百分比（仅测试活跃时有效）+ 20 条 28 字节结果条目”：[PDF 第 238-241 页](../_source/pages/page-238.md)

- **原子更新顺序**：当测试完成或被中止时，控制器**必须先将新结果写入槽位 1，再清零当前操作字段**，防止主机读到“测试已停但结果还没写入”的空窗状态。
- **未使用槽位**：结果不足 20 条时，未使用的条目 `STC` 标记为 `Fh`，其余字段清零。
- **28 字节结果条目要点**：包含测试类型、测试结果（成功、显式中止、复位、NSID 移除、格式化、执行失败、段失败等）、通电时间（Power On Hours，不计低功耗时间）、`NSID` 与失败 LBA（`FLBA`）、状态码与类型、失败段编号及厂商数据。其中 `NSID`、`FLBA`、状态码均受**独立有效性位**门控，有效位为 1 时才可解析。

## 常见易混点

- **自检 vs Sanitize**：自检是“只读体检”，不破坏用户数据；Sanitize 是“彻底安全擦除”。
- **与其它日志的关系**：自检**不会**写入 SMART/Health 计数，也不直接触发异步事件（AER），但自检中发现的严重错误可记录到 Error Information 日志。
- **与 Format / Reset 的联动**：自检期间若发生 Format NVM 或控制器复位，正在进行的自检会中止并记录为 `NSID Removed` 或复位中止结果。

## 关联与引申概念

- **Get Log Page**：主机通过 `Get Log Page (LID=06h)` 轮询自检进度与结果，见 [日志页读取](log-page-retrieval.md)。
- **错误与健康日志**：自检失败的诊断细节可进入 Error Information，见 [错误信息与健康日志](error-and-health-logs.md)。
- **持久化事件日志 & Telemetry**：用于跨断电归档自检事件和拉取深度固件诊断数据，见 [持久化事件日志](persistent-event-log.md) 与 [Telemetry 捕获日志](telemetry-capture-logs.md)。

## 规范依据

- [命名空间作用域与命令边界，PDF 第 210 页](../_source/pages/page-210.md)
- [操作码与处理矩阵，PDF 第 211 页](../_source/pages/page-211.md)
- [作用域说明与命令完成，PDF 第 212 页](../_source/pages/page-212.md)
- [设备自检日志头部与结果列表，PDF 第 238-239 页](../_source/pages/page-238.md)
- [自检结果结构与有效性位门控，PDF 第 240-241 页](../_source/pages/page-240.md)
