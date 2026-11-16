# 通用命令格式（Common Command Format）

**通用命令格式（Common Command Format）** 定义了 NVMe 中所有 Admin 命令与主流 I/O 命令共用的 **64 字节提交队列条目（SQE）** 信封。无论具体操作是读写数据还是管理控制器，前 40 字节的信封头与指针布局保持统一，仅后 24 字节（`CDW10..CDW15`）留给各命令自定义。[PDF pp. 155-158](../_source/pages/page-155.md)

> **类比理解**：把 64 字节 SQE 想成一张标准快递面单——面单大小固定，左上角是服务类型（`CDW0`）、收件人编号（`NSID`）、取件地址清单（`PRP` 或 `SGL`），最下面的备注栏（`CDW10..CDW15`）则由不同业务各自填写 LBA、长度或管理参数。

---

## 64 字节 SQE 布局与 `CDW0` 位域

```text
字节偏移  0       4       8      16      24              40              64
        +-------+-------+-------+-------+---------------+---------------+
        | CDW0  | NSID  |CDW2/3 | MPTR  | DPTR          | CDW10..CDW15  |
        +-------+-------+-------+-------+---------------+---------------+
        | OPC   | 命名  | 命令  | 元数据| PRP1+PRP2     | 命令特定字段  |
        | FUSE  | 空间ID| 相关  | 指针  | 或 SGL1       | (24 字节)     |
        | PSDT  |       |       |       |               |               |
        +-------+-------+-------+-------+---------------+---------------+
```

### `CDW0`（Command Dword 0，Figure 91）

| 位域 | 字段 | 说明 |
|---|---|---|
| `31:16` | **`CID`** | 命令标识符，与 SQID 共同唯一标识在途命令；**应避免使用 `FFFFh`**（Error Info Log 用它表示"无对应命令"） |
| `15:14` | **`PSDT`** | 数据/元数据指针类型：`00b` = PRP；`01b` = SGL（`MPTR` 为连续缓冲区）；`10b` = SGL（`MPTR` 为单描述符段）；`11b` = 保留 |
| `09:08` | **`FUSE`** | 融合操作标识：`00b` = 普通；`01b` = 融合第一条；`10b` = 融合第二条；`11b` = 保留 |
| `07:02` | **`OPC.FN`** | 6 位命令功能码 |
| `01:00` | **`OPC.DTD`** | 2 位数据传输方向：`00b` 无传输、`01b` Host→Ctrl、`10b` Ctrl→Host、`11b` 双向 |

---

## `NSID` 与指针（`MPTR` / `DPTR`）约定

### 命名空间标识符（`NSID`）

- **`0`**：命令不使用命名空间（若不需要 NSID 的命令填入非零值，返回 `Invalid Field in Command`）。
- **`1..FFFFFFFEh`**：指定具体的命名空间（对非活跃 NS 返回 `Invalid Field in Command`；对不存在/无效 NS 返回 `Invalid Namespace or Format`）。
- **`FFFFFFFFh`**：广播标识，仅在命令明确支持时对所有相关命名空间生效。[PDF p. 156](../_source/pages/page-156.md)

### `PSDT` 协议约束与指针布局

- **传输层约束**：**NVMe over PCIe** 的 Admin 命令**必须**使用 PRP（不得用 SGL）；**NVMe over Fabrics** 的 Admin 与 I/O 命令**必须**使用 SGL（`PSDT = 01b`，受传输层限制）。[PDF pp. 155-156](../_source/pages/page-155.md)
- **PRP 模式（`PSDT = 00b`）**：`MPTR` 为 dword 对齐的连续元数据缓冲区地址；`DPTR` 为 `PRP1 + PRP2`：
  - 未跨越页边界：`PRP2` 保留；
  - 跨越 **1 个**页边界：`PRP2` 指向第 2 页；
  - 跨越**多个**页边界：`PRP2` 指向 **PRP List** 物理地址。[PDF pp. 157-158](../_source/pages/page-157.md)
- **SGL 模式（`PSDT = 01b` / `10b`）**：`DPTR` 存放第一个 SGL 段（`SGL1`）；`PSDT = 01b` 时 `MPTR` 为按 Identify Controller `SGLS.MBA` 对齐的连续缓冲区，`PSDT = 10b` 时 `MPTR` 为 qword 对齐的单个元数据 SGL 描述符段。[PDF pp. 157-158](../_source/pages/page-157.md)

---

## Fabrics SQE 与格式边界

- **Fabrics Command Common SQE（Figures 94-95）**：同样是 64 字节，但 `CDW0` 操作码固定为 **`7Fh`**，字节 4 为 **`FCTYPE`**（6 位功能码 + 2 位方向），字节 `24:39` 为完整传输的 `SGL1`，字节 `40:63` 为 Fabrics 类型特定字段；**没有融合操作形式**；`PSDT = 10b` 显式标记一次传输，`00b` 表示无传输。[PDF pp. 159-160](../_source/pages/page-159.md)
- **厂商扩展与未来演进**：厂商自定义命令可选用 `CDW10..11` 作为传输长度并保留公共指针布局；此外，"Common" 表示当前最广泛使用而非永久唯一——未来的新 I/O 命令集可定义不同大小或布局的 SQE。[PDF pp. 156, 158-159](../_source/pages/page-156.md)

---

## 易混概念对比

| 易混概念 | 实际区别 |
|---|---|
| **PRP vs SGL** | `PRP` 按固定内存页（如 4 KiB）描述地址；`SGL` 是支持任意长度分散-聚集区域的描述符链 |
| **`PRP2` 跨 1 页 vs 跨多页** | 跨 1 页时 `PRP2` 直接放第 2 页地址；跨 2 页及以上时 `PRP2` 放 PRP List 地址（常见越界报错根源） |
| **`CDW2/3` vs `CDW10..15`** | 字节 `8..15`（`CDW2/3`）与字节 `40..63`（`CDW10..15`）均由具体命令定义，但各自位置固定 |

---

## 关联与引申概念

- **CDW0 / CID**：Command Dword 0 中的 `CID` 与 `SQID` 配合，用于在 Completion Queue Entry 中把结果精确匹配回原命令。
- **PRP / SGL（Physical Region Page / Scatter Gather List）**：NVMe 描述主机内存数据缓冲区的两种指针机制，详见 [data-pointer-layouts.md](data-pointer-layouts.md)。
- **Fused Operation（融合操作）**：通过 `CDW0.FUSE` 将两条相邻命令绑定为原子单元，详见 [command-ordering-and-arbitration.md](command-ordering-and-arbitration.md)。
- **延伸阅读**：
  - [completion-queue-entry-and-status.md](completion-queue-entry-and-status.md) — CQE 如何通过 `CID` 与 `SQID` 匹配 SQE
  - [identify-command-model.md](identify-command-model.md) — 使用通用 SQE 格式的典型 Admin 命令
  - [admin-command-model.md](admin-command-model.md) — `CDW10..15` 在 Admin 命令中的具体分配

---

## 规范依据

- [Figure 91：Command Dword 0 详细定义，PDF pp. 155-156](../_source/pages/page-155.md)
- [Figure 92：Common Command Format 完整布局，PDF pp. 156-158](../_source/pages/page-156.md)
- [PRP2 规则与 SGL 描述符，PDF pp. 157-158](../_source/pages/page-157.md)
- [Figures 93-95：格式边界与 Fabrics SQE 定义，PDF pp. 159-160](../_source/pages/page-159.md)
