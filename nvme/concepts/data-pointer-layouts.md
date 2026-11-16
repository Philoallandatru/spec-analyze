# 数据指针布局（Data Pointer Layouts）

数据指针布局用于在命令的 `DPTR`（16 字节数据指针区域）中描述主机内存与控制器之间传输的数据和元数据位置。NVMe 提供两种机制：**PRP**（Physical Region Page，物理页指针，简单高效但受限于页对齐）和 **SGL**（Scatter Gather List，分散聚集列表，用带类型描述符支持任意长度与内存布局）。[PDF 第 173-175 页](../_source/pages/page-173.md)

## 心智模型

在命令的 `PSDT` 字段选择模式后，`DPTR` 会切换为对应布局：

```text
PRP 模式（PSDT = 00b）                     SGL 模式（PSDT = 01b / 10b）
command.DPTR                               command.DPTR
├── PRP1 -> 第 1 页 + 可选页内偏移         └── SGL1 (16 字节描述符)
└── PRP2 -> 第 2 页                            ├── Data Block / Bit Bucket
            或 打包的 PRP List 页              └── Segment -> 下一段 SGL
                    └── next List 页（如需）           └── Last Segment（终点）
```

- **PRP** 像按固定楼层（如 4 KiB 页）搬运：总页数由“命令传输字节数 + 页大小”反推。
- **SGL** 像带明细的流水单：每个 16 字节描述符显式写明本段地址与长度，支持跳过不需要的字节。

## PRP 布局与规则

每个 PRP 条目占 8 字节（64 位），dword 对齐，其中 `CC.MPS` 决定页大小及页内偏移（Offset）的位宽（`n = log2(MPS)`）：[PDF 第 173-174 页](../_source/pages/page-173.md)

```text
63                         n+1 n                         0
+-----------------------------+--------------------------+
| Page Base Address           | offset within CC.MPS page|
+-----------------------------+--------------------------+
```

| PRP 位置 | 页内偏移规则 | 作用 |
|---|---|---|
| 命令中 `PRP1` | 允许非零；低 2 位（bit 1:0）必须为 `00b`（dword 对齐） | 指向第 1 页数据起始地址（或特定命令的 List） |
| 命令中 `PRP2` | 必须为 `0` | 若剩余数据 ≤ 1 页则指向第 2 页；否则指向首个 PRP List 页 |
| PRP List 内条目 | 必须为 `0` | 指向连续物理页；**不得重复** `PRP1/PRP2` 已包含的数据页 |
| 链向下一段 List 的条目 | 必须页对齐（偏移为 `0`） | 仅占用本页最后一条可用槽位，指向下一页 PRP List |

> 若 `PRP1` bit 1:0 非零或要求页对齐的位置出现非零偏移，控制器可报错 `PRP Offset Invalid`（不报错时按 0 处理）。

## SGL 描述符与段规则

SGL 由一段或多段 qword 对齐的连续数组组成，数组元素为 16 字节描述符（对应规范 Figures 113-116）：[PDF 第 175-177 页](../_source/pages/page-175.md)

```text
16 字节通用 SGL 描述符
+-----------------------------------------------+------+
| 字节 0-14：Descriptor Type Specific (DTS)     | ID   |
+-----------------------------------------------+------+
                                                 |    |
                                           Type(7:4) SubType(3:0)
```

| 描述符 Type | 名称 | 核心用途与约束 |
|---|---|---|
| `0h` | **Data Block** | 真实搬运数据；全零描述符（Address=0, Length=0）是合法的 NULL 描述符 |
| `1h` | **Bit Bucket** | 占位跳过指定字节数（仅对主机接收/写向主机方向有意义）；SubType 必须为 `0h` |
| `2h` | **Segment** | 链向下一段 SGL（长度非零且为 16 的倍数）；**只能是当前段的最后一个描述符** |
| `3h` | **Last Segment** | 链向最后一段 SGL；Last Segment 内不得再包含段链接 |
| `4h` | **Keyed Data Block** | 带 32 位 Key 与 24 位长度（0 ~ 16 MiB-1），用于 Key-Per-I/O 或 KV 场景 |
| `5h` | **Transport Data Block** | 将传输缓冲机制交由具体 NVMe-oF 传输绑定层定义 |

- **SubType**：`0h` 为绝对地址（Address），`1h` 为相对偏移（Offset，PCIe 上不可用）。
- **总长度契约**：所有 `Data Block` 与 `Bit Bucket` 长度之和必须**精确等于**命令请求的传输长度，控制器不能超额搬运；若不匹配将报 `SGL Data Block Count Mismatch` 或 `Data SGL Length Invalid`。
- **对齐与传输限制**：Identify Controller 中的 `SGLS` 字段声明控制器支持字节对齐还是仅支持 dword 对齐。**PCIe 上 Admin 命令强制使用 PRP；NVMe-oF 上 Admin 与 I/O 命令强制使用 SGL。**

## 关联与引申概念

- **通用命令格式（DPTR / PSDT）**：每个 64 字节 SQE 中 `DPTR` 占字节 24-39，`PSDT` 决定采用 PRP 还是 SGL，见 [通用命令格式](common-command-format.md)。
- **Key Per I/O**：基于 `Keyed Data Block`（SGL Type `4h`）为每次 I/O 传递加密密钥标签，见 [Key Per I/O](key-per-io.md)。
- **主机内存缓冲区（HMB）** & **控制器内存窗口（CMW）**：其他主机与控制器共享内存寻址机制，见 [主机内存缓冲区](host-memory-buffer.md) 与 [控制器内存窗口](controller-memory-windows.md)。
- **元数据区域（Metadata Region）**：用于存放端到端数据保护（PI）等元数据，Base 规范仅定义概念边界，具体格式由 I/O 命令集规范定义。[PDF 第 181 页](../_source/pages/page-181.md)

## 规范依据

- [PRP Entry 与 offset 定义 Figures 109-110，PDF 第 173-174 页](../_source/pages/page-173.md)
- [PRP List 结构 Figures 111-112，PDF 第 174 页](../_source/pages/page-174.md)
- [SGL 段与描述符总述，PDF 第 175 页](../_source/pages/page-175.md)
- [SGL 描述符与子类型 Figures 114-116，PDF 第 176 页](../_source/pages/page-176.md)
- [Keyed / Transport SGL 描述符 Figures 119-122，PDF 第 177-179 页](../_source/pages/page-177.md)
- [链式 SGL + Bit Bucket 示例 Figure 123，PDF 第 179-180 页](../_source/pages/page-179.md)
- [Metadata Region 边界，PDF 第 181 页](../_source/pages/page-181.md)
