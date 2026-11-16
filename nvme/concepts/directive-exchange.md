# 指令交换（Directive Exchange）

`Directive Send`（主机→控制器）和 `Directive Receive`（控制器→主机）是一对配套的 Admin 命令，充当通用的“信封”来搬运特定功能的控制数据；至于“信纸”里写什么，则完全由所选的 Directive Type（`DTYPE`）在其扩展能力规范（基础规范 8.1.8 节）中定义。[PDF 第 212-214 页](../_source/pages/page-212.md)

## 心智模型：快递收发室

- **通用命令 = 收发室**：只负责按地址收发包裹，不拆开检查内容，也不为参数赋予通用业务含义。
- **三联标签（`DTYPE` + `DOPER` + `DSPEC`）**：
  - `DTYPE`（8 bit）：选择功能大类（例如 Streams Directive）。
  - `DOPER`（8 bit）：在该大类内选择具体操作（读状态、启用、释放资源等）。
  - `DSPEC`（16 bit）：该类型内部的二级定位参数。

## 工作流程与通用字段

```text
主机 (Host)                                    控制器 (Controller)
  |                                                  |
  |-- Directive Send (DTYPE + DOPER + DSPEC) ------->| 解析三联标签 -> 交给对应模块
  |   Data Pointer + NUMD dwords                     |
  |<-- Admin CQE ------------------------------------|
  |                                                  |
  |-- Directive Receive (DTYPE + DOPER + DSPEC) ---->| 取出对应结构 -> 通过 Data Pointer 返回
  |   Data Pointer + NUMD dwords                     |
  |<-- payload + Admin CQE --------------------------|
```

`Directive Send` 与 `Directive Receive` 在 `CDW10`–`CDW11` 上**共用完全相同的字段布局**，唯一区别是数据传输方向：[PDF 第 212-213 页](../_source/pages/page-212.md)

| 字段 | 含义 |
|---|---|
| **Data Pointer** | 传输缓冲区的起始地址（PRP 或 SGL） |
| **`NUMD`**（32 bit） | **0-based** 的 dword 计数，决定本次传输的最大长度 |
| **`DTYPE`**（8 bit） | 选择 Directive Type（若选了未实现的类型，直接报命令级错误） |
| **`DOPER`**（8 bit） | 在所选 Type 内指定具体操作 |
| **`DSPEC`**（16 bit） | Directive-Type 依赖的二级标识 |
| **`CDW12`–`CDW13`** | 可选参数，仅在所选 `DTYPE + DOPER` 需要时有意义，否则为保留字段 |

## 核心规则与易混点

- **Receive 长度截断与不补零规则**：当请求的 `NUMD` 小于控制器返回结构的大小时，控制器**只传前 `NUMD` 个 dword**（截断属于正常行为）；当 `NUMD` 大于结构大小时，控制器传完整结构，**不补零、不追加填充**。[PDF 第 212 页](../_source/pages/page-212.md)
- **完成与状态码边界**：两条命令均通过 **Admin Completion Queue** 返回完成项（CQE）；命令特定状态码由所选 `DTYPE` 定义，通用命令只保证通用错误码（如 `Invalid Opcode`、`Invalid Field in Command`）。[PDF 第 213-214 页](../_source/pages/page-213.md)
- **Directive vs Get Log Page / Set Features**：`Get Log Page` 走日志页 ID（LID）注册表，`Set Features` 走特性 ID（FID）；而 `Directive` 专用于需要双向交互、带复杂子操作和二级参数的扩展机制（最典型应用是 **Streams Directive**，用于向 SSD 提示写入数据流生命周期以减少写放大）。

## 关联与引申概念

- **命令集（Command Sets）**：Directive Type 的启用往往依赖控制器支持的命令集与特性，见 [命令集](command-sets.md)。
- **命令效果与支持（Commands Supported and Effects）**：主机可通过该日志查询 Send/Receive 是否需要排空 I/O 再执行，见 [命令效果与支持](command-effects-and-support.md)。
- **Identify 命令模型**：用于发现控制器是否支持 Directives 能力，见 [Identify 命令模型](identify-command-model.md)。
- **常见 I/O 控制命令**：包含 Streams 等与主机 I/O 提示协同的机制，见 [常见 I/O 控制命令](common-io-control-commands.md)。

## 规范依据

- [Directive Receive 缓冲与长度行为，PDF 第 212 页](../_source/pages/page-212.md)
- [Directive Receive/Send 通用字段布局，PDF 第 213 页](../_source/pages/page-213.md)
- [Directive Send 完成边界，PDF 第 214 页](../_source/pages/page-214.md)
