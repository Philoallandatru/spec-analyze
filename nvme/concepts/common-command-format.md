# 通用命令格式（Common Command Format）

## 概述

通用命令格式是 NVMe 协议中所有命令的基础数据结构。无论是管理命令（Admin Command）还是各种 I/O 命令集（I/O Command Set）中的 I/O 命令，都共享这个 64 字节的提交队列条目（Submission Queue Entry, SQE）格式。

这个统一的格式定义了以下关键字段的位置：
- 命令标识信息
- 命名空间标识符（Namespace ID）
- 元数据指针（Metadata Pointer）
- 数据指针（Data Pointer）
- 命令特定字段（Command-Specific Fields）

[规范参考：PDF pp. 155-158](../_source/pages/page-155.md)

## 结构示意图

为了帮助理解，我们可以将 64 字节的命令格式想象成以下布局：

```text
字节偏移  0       4       8      16      24              40              64
        +-------+-------+-------+-------+---------------+---------------+
        | CDW0  | NSID  |CDW2/3 | MPTR  | DPTR          | CDW10..CDW15  |
        +-------+-------+-------+-------+---------------+---------------+
        | 操作码| 命名  |       | 元数据| PRP1+PRP2     | 命令特定字段  |
        | 等控制| 空间ID|       | 指针  | 或 SGL1       |               |
        +-------+-------+-------+-------+---------------+---------------+
```

**说明：**
- 这是对规范 Figure 92 的简化表示
- 指针的具体含义取决于 `CDW0.PSDT` 字段的值
- CDW 表示 Command Dword（命令双字），每个双字为 4 字节

[规范参考：PDF pp. 156-158](../_source/pages/page-156.md)

## 命令双字 0（Command Dword 0）详解

命令双字 0 包含了命令的核心控制信息，其各个位段的定义如下：

| 位域 | 字段名称 | 说明 |
|------|----------|------|
| `31:16` | **CID**<br>（Command Identifier，命令标识符） | 与提交队列标识符共同唯一标识一条命令；应避免使用 `FFFFh`（该值在错误信息日志中有特殊含义） |
| `15:14` | **PSDT**<br>（PRP or SGL for Data Transfer） | 数据传输方式选择器：<br>• `00b` - 使用 PRP（Physical Region Page）<br>• `01b` - 使用 SGL，`MPTR` 为缓冲区地址<br>• `10b` - 使用 SGL，`MPTR` 为单描述符段<br>• `11b` - 保留 |
| `09:08` | **FUSE**<br>（Fused Operation） | 融合操作标识：<br>• `00b` - 普通命令<br>• `01b` - 融合操作的第一条命令<br>• `10b` - 融合操作的第二条命令<br>• `11b` - 保留 |
| `07:02` | **OPC.FN**<br>（Opcode Function） | 命令功能码 |
| `01:00` | **OPC.DTD**<br>（Data Transfer Direction） | 数据传输方向：<br>• 无传输<br>• 主机到控制器<br>• 控制器到主机<br>• 双向传输 |

[规范参考：PDF pp. 155-156](../_source/pages/page-155.md)

### 传输方式的差异

**PRP vs SGL：**
- **PRP（Physical Region Page）**：基于物理页的传输方式，主要用于 PCIe 传输
- **SGL（Scatter Gather List）**：分散-聚集列表，更加灵活，支持非连续内存区域

**传输协议约束：**
- **NVMe over PCIe**：管理命令使用 PRP，**不得**使用 SGL
- **NVMe over Fabrics**：管理命令和 I/O 命令使用 `PSDT=01b` 的 SGL，具体支持的值受传输层限制

## 命名空间与指针约定

### 命名空间标识符（NSID）的使用规则

| 值 | 含义 | 使用场景 |
|---|------|----------|
| `0` | 未使用 | 不需要指定命名空间的命令 |
| `1` ~ `FFFFFFFEh` | 具体的命名空间 ID | 针对特定命名空间的操作 |
| `FFFFFFFFh` | 广播标识 | 仅在命令明确支持时使用，表示对所有相关命名空间执行操作 |

**错误处理：**
- 对非活跃（inactive）的 NSID：通常返回 `Invalid Field in Command`
- 对无效的 NSID：返回 `Invalid Namespace or Format`
- 对不使用 NSID 的命令，如果提供了非零 NSID：返回 `Invalid Field in Command`

[规范参考：PDF p. 156](../_source/pages/page-156.md)

### 指针布局根据 PSDT 的变化

不同的 `PSDT` 值决定了元数据指针（`MPTR`）和数据指针（`DPTR`）的解释方式：

| PSDT 值 | MPTR（字节 16:23） | DPTR（字节 24:39） |
|---------|--------------------|--------------------|
| `00b`<br>（PRP 模式） | 双字（dword）对齐的连续元数据缓冲区地址 | `PRP1` 加上 `PRP2`<br>（根据页边界跨越情况，`PRP2` 可能是：保留位、第二页地址或 PRP 列表指针） |
| `01b`<br>（SGL 缓冲区地址） | 按 Identify Controller 指定对齐的连续元数据缓冲区 | 第一个 SGL 段（`SGL1`） |
| `10b`<br>（SGL 描述符） | 四字（qword）对齐的段，仅包含一个元数据 SGL 描述符 | 第一个 SGL 段（`SGL1`） |

**PRP 模式的 PRP2 使用规则：**
- 未跨越页边界时：`PRP2` 为保留位
- 跨越一个页边界时：`PRP2` 指向第二页
- 跨越多个页边界时：`PRP2` 指向 PRP 列表

**SGL 模式的描述符规则：**
- 数据块描述符可以描述整个传输
- 否则，第一个描述符链接到下一个段（Segment）或最后一个段（Last Segment）

[规范参考：PDF pp. 157-158](../_source/pages/page-157.md)

## 格式边界与扩展

### 命令特定字段（字节 40:63）

字节 40 至 63 的 24 字节空间对应命令双字 10 至双字 15（CDW10 ~ CDW15），由各个具体命令自行定义使用。

### 可选的厂商特定命令格式

对于可选的厂商特定（Vendor-Specific）命令，可以采用以下布局：
- **双字 10 和 11** 用作传输长度字段
- 保留公共指针的布局

这种设计在保持标准指针结构的同时，允许厂商自定义传输语义。

[规范参考：PDF pp. 156, 158-159](../_source/pages/page-156.md)

### 未来扩展性

需要注意的是，未来的 I/O 命令集可能会定义不同的命令大小或格式，因此当前的通用命令格式是"通用"（common）而非"普适"（universal）的——它是当前最广泛使用的格式，但不是唯一可能的格式。

## Fabrics 命令的通用格式

NVMe over Fabrics 使用一种不同的 64 字节布局，称为 **Fabrics Command Common SQE**：

```text
字节偏移  0       4   5                 24              40              64
        +-------+---+-----------------+---------------+---------------+
        | CDW0  |TYP| 保留            | SGL1          | FCTS          |
        +-------+---+-----------------+---------------+---------------+
        |OPC=7Fh|FCTYPE               | 完整传输      | 类型特定字段  |
        +-------+---+-----------------+---------------+---------------+
```

**字段说明：**
- **字节 4**：`FCTYPE`（Fabrics Command Type），由 6 位功能码和 2 位传输方向组成
- **字节 5:23**：保留
- **字节 24:39**：包含一个传输（Transport）或键控（Keyed）SGL 描述符，用于完整的数据传输
- **字节 40:63**：Fabrics 命令类型特定字段

**Fabrics 命令的特点：**
- 使用操作码 `7Fh`
- 没有融合操作形式
- 使用 SGL 进行数据传输
- `PSDT=10b` 显式标记一次传输
- `PSDT=00b` 表示无传输值，但在需要传输的命令中仍接受 SGL

[规范参考：PDF pp. 159-160](../_source/pages/page-159.md)

## 字段关系总结

理解通用命令格式中各字段的相互关系，有助于掌握 NVMe 命令的执行机制：

### 命令标识与完成匹配
- **CID（命令标识符）** 与 **提交队列标识符（Submission Queue ID）** 共同组成命令的唯一标识
- 控制器使用这个组合将命令与其对应的完成队列条目（Completion Queue Entry）关联起来

[规范参考：PDF p. 155](../_source/pages/page-155.md)

### 融合操作的标识
- **FUSE 字段** 将融合操作（Fused Operation）的位置信息嵌入通用提交队列条目中
- 融合操作允许两条命令作为一个原子单元执行

[规范参考：PDF p. 155](../_source/pages/page-155.md)

### 指针类型的联动选择
- **PSDT 字段** 同时控制元数据指针和数据指针的解释方式
- 该字段的有效值受传输协议（PCIe 或 Fabrics）的约束
- 这种联动设计确保了元数据和数据的传输方式保持一致

[规范参考：PDF pp. 155-158](../_source/pages/page-155.md)

## 规范依据

本文档基于以下 NVMe 规范内容整理：

- [Figure 91: Command Dword 0 详细定义，PDF pp. 155-156](../_source/pages/page-155.md)
- [Figure 92: Common Command Format 完整布局，PDF pp. 156-158](../_source/pages/page-156.md)
- [Figures 93-94: 格式边界说明，PDF p. 159](../_source/pages/page-159.md)
- [Figures 94-95: Fabrics SQE 与 CDW0 定义，PDF pp. 159-160](../_source/pages/page-159.md)
