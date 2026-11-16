# 控制器就绪模式与超时（Controller Ready Modes and Timeouts）

NVMe 控制器“就绪（`CSTS.RDY=1`）”有两种契约：**Ready With Media**（命令处理、介质与挂载命名空间全部就绪）和 **Ready Independent of Media**（不依赖介质的命令先就绪，介质与命名空间稍后到位）。`CAP.CRMS` 声明支持情况，`CC.CRIME` 在使能时选择模式，`CRTO.CRIMT` 与 `CRTO.CRWMT` 限定等待上限。[PDF pp. 128-131](../_source/pages/page-128.md)

可以把它想成**新开的快递分拣中心**：Ready With Media 是等货架全装满再开门，顾客来了就能取件；Ready Independent of Media 是先开“客服窗口”接寄件和查单（非介质命令），后台货架（介质）在稍后规定时间内补齐。

## 两种就绪模式与超时时间线

```text
CC.EN: 0 -> 1
   |
   +-- Ready With Media (CC.CRIME = 0)
   |      `-- CRTO.CRWMT 之前：CSTS.RDY=1，命令、介质与挂载命名空间全部就绪
   |
   `-- Ready Independent of Media (CC.CRIME = 1)
          |-- CRTO.CRIMT 之前：CSTS.RDY=1，不依赖介质/命名空间的命令可处理
          `-- CRTO.CRWMT 之前：挂载的命名空间与 Admin 所需介质就绪
```

[PDF pp. 128-131](../_source/pages/page-128.md)

| `CAP.CRMS` | 支持的就绪模式 | `CC.CRIME` 行为 |
|---|---|---|
| `00b` | 传统模式（无选择，遵循 Rev < 2.0，由 `CAP.TO` 限定） | 只读为 `0`，主机忽略 |
| `01b` | 仅 Ready With Media | 只读为 `0` |
| `11b` | 两种模式都支持（Rev 2.1 控制器必选） | RW；由 `CC.EN: 0→1` 时的值决定本轮模式 |

- **超时单位与关系**：`CRTO.CRIMT` 与 `CRTO.CRWMT` 均以 **500 ms** 为单位，且 `CRWMT ≥ CRIMT`。[PDF p. 92](../_source/pages/page-092.md)
- **修改 `CC.CRIME` 的影响**：切换 `CC.CRIME` 可能改变 `CAP.TO` 报告的传统超时值，但**不会**改变 `CRTO.CRIMT` 或 `CRTO.CRWMT`。[PDF pp. 130-131](../_source/pages/page-130.md)

## 命令可用性与超时违规处理

- **Ready With Media**：`CSTS.RDY=1` 即代表所有挂载的命名空间、Admin 所需介质及命令处理全部就绪。
- **Ready Independent of Media**：
  - 在 `CRTO.CRIMT` 之前置 `RDY=1` 后，不依赖挂载命名空间或介质的命令必须可用。
  - 在 `CRTO.CRWMT` 之前，依赖介质/命名空间的命令可能返回**可重试**的 `Namespace Not Ready`（I/O 命令）或 `Admin Command Media Not Ready`（Admin 命令，具体适用白名单见规范 Figure 84，其中 Device Self-test 与 Get Log Page 有更窄限制）。[PDF pp. 129-130](../_source/pages/page-129.md)
- **超时违规处理**：若 `CAP.CRMS != 0` 的控制器错过了适用的就绪截止时间，它**仍然必须**在 `CRTO.CRWMT` 之前将 `CSTS.RDY` 置 `1`；若支持 Persistent Event Log，还需记录一条 `NVM Subsystem Hardware Error` 事件（事件码：`Controller Ready Timeout Exceeded`）。[PDF pp. 131-132](../_source/pages/page-131.md)

## 概念辨析

| 容易混淆 | 实际区别 |
|---|---|
| `CAP.CRMS` vs `CC.CRIME` | `CRMS` 是静态能力（是否支持双模式）；`CRIME` 是启用时选择的动态模式 |
| `CAP.TO` vs `CRTO` | `CAP.TO` 是传统单段超时（封顶 127.5 秒）；`CRTO` 是 Rev 2.1 引入的两段式超时（可超 127.5 秒） |
| `Admin Command Media Not Ready` vs `Namespace Not Ready` | 前者针对 Admin 命令介质未就绪；后者针对 I/O 命令访问未就绪命名空间；两者均**可重试** |

## 关联与引申概念

- **控制器启用与重置（[Controller Enable, Shutdown, and Reset](controller-enable-shutdown-reset.md)）**：`CC.EN`、`CSTS.RDY` 握手及 `CRTO` 寄存器的完整定义。
- **控制器初始化（[Controller Initialization](controller-initialization.md)）**：控制器就绪后主机如何继续 Identify、枚举命名空间与建队列。
- **格式化生命周期（[Format NVM Lifecycle](format-nvm-lifecycle.md)）**：Format/Sanitize 等操作期间也可能触发介质/命名空间未就绪的可重试状态。
- **Keep Alive（[Keep Alive](keep-alive.md)）**：控制器启用与就绪过程中的心跳保活激活条件。

## 规范依据

- [CRTO 寄存器字段定义，PDF 第 92 页](../_source/pages/page-092.md)
- [就绪模式定义与 Discovery 初始化，PDF 第 128 页](../_source/pages/page-128.md)
- [Ready Independent of Media 命令限制与 Figure 84，PDF 第 129-130 页](../_source/pages/page-129.md)
- [超时映射与违规失败处理，PDF 第 131-132 页](../_source/pages/page-131.md)
