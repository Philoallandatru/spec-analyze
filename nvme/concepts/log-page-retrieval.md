# 日志页读取（Log Page Retrieval）

**Get Log Page（Admin OPC `02h`）** 是 NVMe 的通用只读诊断命令，用于读取控制器、命名空间、Domain（域）、Endurance Group（耐久度组）或 NVM 子系统层级的运行日志。命令通过 **LID（Log Page Identifier，日志页标识符）** 决定返回的数据结构与作用范围，通过 `RAE`、`OT`、`LPO` 等字段控制事件清除与分段读取。[PDF pp. 223-227](../_source/pages/page-223.md)

可以把 `Get Log Page` 想象成**一台万能日志取件机**：`LID` 是取件码（`01h` 错误日志、`02h` SMART 健康、`03h` 固件槽位、`05h` 命令支持等）；`NUMDL:NUMDU` 指定取多少个 4 字节 dword；`LPO` 配合 `OT` 决定是从第几个字节还是第几条记录开始取；而 `RAE` 决定取完后是否把对应的异步事件提醒（AER）清掉。

## 命令流程与核心字段契约

```text
Host                         Controller interface                 Log object
 |                                   |                               |
 |-- Get Log Page (LID + NSID) ----->|                               |
 |   NUMDU:NUMDL (0基 dword 计数)    |-- 选择支持的 LID ------------>|
 |   LPOU:LPOL + OT (字节/索引偏移)   |-- 定位起始位置 --------------->|
 |   RAE (是否保留异步事件)           |<-- 返回请求的 dword 范围 ------|
 |<---------------- data buffer -----|                               |
 |   仅当命令成功且 RAE=0 时清除对应异步事件                            |
```

| 命令字段 / 关注点 | 规范契约与边界规则 |
|---|---|
| **传输长度（`NUMDU:NUMDL`）** | 0 基的 dword（4 字节）计数；若请求范围**超出日志末尾**，控制器返回完整日志及未定义的尾部 dword（不补零、不报错） |
| **事件保留（`RAE`）** | 仅当命令**成功完成且 `RAE=0`** 时清除对应的异步事件；`RAE=1` 保留；命令失败时**总是保留** |
| **字节偏移（`OT=0`）** | `LPOU:LPOL` 为字节偏移，按 dword 对齐（低 2 位非零可能被拒绝，若接受则视为 0）；偏移超出日志范围返回 `Invalid Field in Command` |
| **索引偏移（`OT=1`）** | `LPOU:LPOL` 为条目索引；仅当所选 LID 在 Supported Log Pages 中声明 `IOS=1` 时合法，否则或索引越界返回 `Invalid Field in Command` |
| **命令集与 UUID（`CDW14`）** | `CDW14.CSI` 仅在支持它的 LID 配合 `CC.CSS=110b` 时生效；`CDW14.UIDX` 仅在支持 UUID 选择的日志中生效；其余情况均被忽略 |
| **不支持的 LID** | 返回 `Invalid Log Page`（规范注明的命令专属例外除外） |

## 支持的日志发现（LID `00h`）与作用范围

**Supported Log Pages（LID `00h`）** 返回 1,024 字节（256 个 4 字节条目，对应 `LID 00h–FFh`），报告当前提交接口上每个 LID 的支持与效果位。注意**不同 Admin Queue 或管理端点（Management Endpoint）实例暴露的支持集合可能不同**，且可能随 `CC.CSS` 选中的 I/O 命令集变化。[PDF pp. 227-228](../_source/pages/page-227.md)

```text
Supported Log Pages (LID 00h, 1,024 字节)
+---------+---------+-----+---------+
| LID 00h | LID 01h | ... | LID FFh |   每个 LID 占 32 位 (Figure 204):
+---------+---------+-----+---------+
  31             16 15          2 1   0
 +-----------------+-------------+---+---+
 |  LID-specific   |  reserved   |IOS|SUP|
 +-----------------+-------------+---+---+
  - SUP=0：不支持该 LID，主机应忽略其余位
  - IOS=1：支持索引偏移 (OT=1)；扩展 Get Log Page 不支持时 IOS 必为 0
```

**作用范围与 `NSID` 规则**：[PDF pp. 226-227](../_source/pages/page-226.md)
- 日志作用范围由日志自身定义（子系统级、Domain 级、控制器级或命名空间级）。
- 对于控制器或子系统级日志，`NSID` 只能是 `0h` 或 `FFFFFFFFh`，其他值无效；多作用范围日志则结合 Domain 支持情况或 `NSID` 解析实际范围。

## 规范依据

- [Get Log Page 命令行为、数据指针、RAE、LPO/OT/CSI/UIDX，PDF pp. 223-225](../_source/pages/page-223.md)
- [LID 注册表与作用范围规则，PDF pp. 226-227](../_source/pages/page-226.md)
- [Supported Log Pages 结构（LID 00h, Figures 203–204），PDF pp. 227-228](../_source/pages/page-227.md)

## 关联与引申概念

- **异步事件上报（AER）**：AER 通知主机发生事件后，主机通常用 `Get Log Page(..., RAE=0)` 读取详情并清除事件掩码以接收后续通知；若只想偷窥而不影响其他主机，可设 `RAE=1`。详见 [异步事件上报](asynchronous-event-reporting.md)。
- **错误与健康日志（LID `01h`/`02h`）**：控制器级与命名空间级日志读取的典型应用。详见 [错误与健康日志](error-and-health-logs.md)。
- **持久化事件日志（LID `0Dh`）**：与普通 `Get Log Page` 读取实时当前视图不同，Persistent Event Log 需先建立快照再分段读取以保证代数一致。详见 [持久化事件日志](persistent-event-log.md)。
- **遥测与命令效果日志**：Telemetry（LID `07h`/`08h`）与 Commands Supported and Effects（LID `05h`）均基于 `Get Log Page` 机制。详见 [遥测捕获日志](telemetry-capture-logs.md) 与 [命令效果与支持](command-effects-and-support.md)。
