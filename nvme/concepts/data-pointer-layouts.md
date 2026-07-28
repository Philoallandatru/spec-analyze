# 数据指针布局（Data Pointer Layouts）

## 一句话说明

NVMe 用两种"指针"告诉控制器"主机数据放在哪"：PRP 按固定 4KB 页地址描述（简单但只能描述页对齐内存），SGL 用任意大小和类型的描述符链（灵活、支持分散聚集和跳过）。

## 生活化类比

把数据指针布局想成**搬家时给搬运工的"取件清单"**：

- **PRP 模式** = 一张楼层平面图："东西在 A 楼 3 层、3.5 层、B 楼 1 层"，每层就是 4KB。清单要自己算每层装多少字节，跨多页还得再附一张"清单的清单"（PRP List）。
- **SGL 模式** = 一张手写流水单："先装 1.5KB 到 A 箱，跳过 0.5KB 不用装，再装 4KB 到 B 箱"，可以详细写每一段怎么搬。
- **Bit Bucket 描述符** = 单子上"此处丢弃 0.5KB"，帮搬运工跳过不感兴趣的部分。
- **Segment 描述符** = "本张单子用完了，翻到清单第 N 张继续"——支持超长的搬运动作。

## 工作流程

```text
  通用 SQE 中 DPTR 区域（16 字节）按 PSDT 切换：

  PRP 模式（PSDT=00b）                       SGL 模式（PSDT=01b/10b）
  +---------------+---------------+           +-------------------------------+
  |     PRP1      |     PRP2      |           |  SGL1 (16 字节描述符)          |
  +---------------+---------------+           +-------------------------------+
  | 第 1 页 + 偏移 | 第 2 页 / List|           |  Data Block / Bit Bucket      |
  +---------------+---------------+           |  Segment / Last Segment …     |
                                             +-------------------------------+
                                                       |
                                              当一个段装不下时
                                                       v
                                             +-------------------------------+
                                             |  Segment 描述符 ──> 下一段     |
                                             +-------------------------------+

  段（SGL Segment）：1 个或多个 16 字节描述符的 qword 对齐数组
  末段（Last Segment）：含本命令最后一个描述符，无进一步链接
```

## 初学者案例

**场景：写命令要传 13KB，但主机内存不是连续的。**

1. 主机源缓冲布局：4KB 缓冲 + 2KB 缓冲 + 2KB 缓冲 + 5KB 缓冲，跨多个不连续页。
2. 用 PRP 几乎不可能描述（PRP 假设页对齐且按页走）。
3. 改用 SGL：建一个 SGL 段，含 4 个 Data Block 描述符，每块长度与偏移严格对应。
4. 如果单段 16KB 装不下，再加一个 Segment 描述符链到下一段；最后一段用 Last Segment 收尾。
5. 如果某段希望"控制器跳过 2KB 不写盘"，用 Bit Bucket 描述符（subtype=0h Address），长度为 2KB。
6. 提交后控制器按描述符顺序搬运；总长度必须等于命令指定传输长度，否则回 `SGL Data Block Count Mismatch` / `SGL Data Length Mismatch` / `SGL Descriptor Invalid`。

> 排错提示：SGL 错误码很细：先看状态码是不是 `Data SGL Length Invalid` 或 `SGL Descriptor Type Invalid`，再回头检查描述符链。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| PRP 大小 | 每个 PRP 是 8 字节（64 位），dword 对齐 |
| PRP 页大小 | 由 `CC.MPS` 决定 offset 位宽（offset 位 = log2(MPS)） |
| PRP1 偏移 | 可非零；bit 1:0 必须为 `00b`（低 2 位是 0） |
| PRP2 偏移 | 必须为 0（充当第二页或 List 指针） |
| PRP List 偏移 | 每个条目内偏移必须为 0 |
| PRP List 链 | 最后一个可用条目用作 next-list 指针时必须页对齐 |
| 偏移非法 | PRP 偏移非法可报 `PRP Offset Invalid`；或被控制器视为 0 |
| SGL 描述符 | 16 字节；段内 qword 对齐；末段不再含段链接 |
| SGL 描述符类型 | `0h`=Data Block；`1h`=Bit Bucket；`2h`=Segment；`3h`=Last Segment；`4h`=Keyed Data Block；`5h`=Transport Data Block；`6h-Eh`=保留；`Fh`=Vendor Specific |
| SGL 描述符子类型 | `0h`=Address；`1h`=Offset（仅对 0h/2h/3h/4h 等类型合法，Type 1h 必须为 0h） |
| 全零描述符 | 是合法 Data Block（Address=0，Length=0），可当 NULL 描述符 |
| 非法描述符 | 保留/不支持的 type/subtype → `SGL Descriptor Type error` |
| 传输覆盖 | Data Block + Bit Bucket 总和 = 命令传输长度；不允许"超额传输" |
| 对齐 | 控制器在 Identify Controller 声明是否仅支持 dword 对齐；非 dword 对齐 Address/Length 控制器可中止 |
| 段链接 | 只有"非末段"的最后描述符可链接到下一段；链长度非零且 16 的倍数 |
| 协议约束 | Fabrics 上 Admin/I/O 命令强制用 SGL；PCIe 上 Admin 命令强制用 PRP |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| PRP1 vs PRP2 | PRP1 可带页内偏移；PRP2 必须 0 |
| PRP2 vs PRP List | PRP2 是命令内字段；List 是 PRP2 指向的"整页多 PRP 数组" |
| PRP List 中的 PRP vs 数据 PRP | List 内每个 PRP 指一页数据，**不是**下一个 List |
| Data Block vs Bit Bucket | Data Block 真搬数据；Bit Bucket 仅"占位跳过"（仅目的地操作有意义） |
| Segment vs Last Segment | Segment 链到下一段；Last Segment 是终点 |
| Address subtype vs Offset subtype | Address 给出 64-bit 绝对地址；Offset 给出"自某基址的偏移"（PCIe 上偏移子类型指向 0） |
| Data Block Length=0 vs 全零描述符 | 都是合法 NULL 描述符；表示"此处长度为 0，不搬运" |
| Keyed Data Block vs Data Block | Keyed 多 8 字节 key（32 位）+ 24 位长度，KV 场景使用 |
| Transport SGL Data Block vs Data Block | Transport 把缓冲/传输机制交由绑定层决定；Base 中地址字节保留 |
| PRP 总长度 vs SGL 总长度 | PRP 由命令参数+页大小推算；SGL 由描述符累加 |

## 进阶细节

- **PRP Entry 位域**（规范 Figures 109-110）：
  - `63:n+1` Page Base Address（n = log2(CC.MPS)）
  - `n:0` Offset within page
- **PRP List 链规则**：每个 PRP List 占用 1 个连续内存页，从 entry 0 开始；最后可用 entry 用于链到下一个 List 页（要求页对齐）。
- **PRP List 不得重复**：List 内不应出现已由命令本身 PRP1/PRP2 描述的数据页地址。
- **SGL 通用描述符**（Figure 114）：
  - 字节 0-14：Descriptor Type Specific（DTS）
  - 字节 15：SGL Identifier（高 4 位 = Type，低 4 位 = Sub Type）
- **Descriptor Sub Type**（Figure 116）：0h=Address；1h=Offset（PCIe 上不可用）；其他保留。Type 1h（Bit Bucket）必须 Sub Type=0h。
- **SGL Keyed Data Block**：长度域 24 位（0-16 MiB-1），高位强制为 0；Key 32 位。
- **SGL Transport Data Block**：地址字节在 Base 规范中保留，由具体 NVMe Transport binding 规定。
- **示例：链式 SGL + Bit Bucket**（Figure 123 风格）：逻辑 13 KiB 跨 3 段描述，主机只接收 11 KiB 真实数据，2 KiB 用 Bit Bucket 跳过。
- **SGL 对齐**：`SGLS` 字段声明对齐粒度（字节或 dword）；控制器只支持 dword 时，任何 Address/Length 低位为 1 都应中止。
- **段长对齐**：`Segment` 描述符中的 length 非零且为 16 的倍数。
- **保留 type/subtype 错误**：导致 `SGL Descriptor Type error`（完成状态 `0Ch` 子类型）。
- **Metadata Region**：Base 规范只定义"概念边界"，具体格式由各 I/O 命令集规范定义。

## 规范依据

- [PRP Entry 与 offset 定义 Figures 109-110，PDF 第 173-174 页](../_source/pages/page-173.md)
- [PRP List 结构 Figures 111-112，PDF 第 174 页](../_source/pages/page-174.md)
- [SGL 段与描述符总述，PDF 第 175 页](../_source/pages/page-175.md)
- [SGL 描述符与子类型 Figures 114-116，PDF 第 176 页](../_source/pages/page-176.md)
- [Keyed / Transport SGL 描述符 Figures 119-122，PDF 第 177-179 页](../_source/pages/page-177.md)
- [链式 SGL + Bit Bucket 示例 Figure 123，PDF 第 179-180 页](../_source/pages/page-179.md)
- [Metadata Region 边界，PDF 第 181 页](../_source/pages/page-181.md)

## 相关阅读

- [通用命令格式](common-command-format.md) - DPTR 字段位置和 PSDT 选择
- [Key Per I/O](key-per-io.md) - Keyed Data Block 是 SGL 类型 4h
- [主机内存缓冲区](host-memory-buffer.md) - HMB 描述符与 SGL 结构类似
- [控制器内存窗口](controller-memory-windows.md) - 另一种主机内存引用机制
