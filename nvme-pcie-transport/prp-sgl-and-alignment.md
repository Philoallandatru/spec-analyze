# PRP / SGL 内存描述符与对齐规则

在 PCIe 传输模型中，主机内存被操作系统切分成无数离散的物理页（或 IOVA 页）。NVMe 控制器通过 DMA 搬运数据时，必须依赖命令条目（SQE）中的 **DPTR（Data Pointer，Bytes `[39:24]`）** 来找到这些分散的内存碎片 `[PCIe Transport §4.1]`。

PCIe 传输支持两种描述符：**PRP（Physical Region Page）** 与 **SGL（Scatter-Gather List）**。可以把它们想成**两种寄件地址单**：
- **PRP** = **标准快递单**：强制要求除第一页外，后续每一页都必须从整页边界（如 4KB）开始，结构极简、硬件解析最快。
- **SGL** = **自定义货运清单**：支持任意字节长度与任意字节偏移，灵活性极高，但在 PCIe 上属于**可选特性**。

---

## PRP1 / PRP2 / PRP List 规则（PCIe 核心路径）

PRP 的页大小由 `CC.MPS` 决定（最小 `4KB`，公式 $2^{12 + \text{CC.MPS}}$），每个 PRP 条目占 **64 位（8 字节）**，低 `n` 位为页内偏移（4KB 页下为 Bits `[11:0]`） `[Base §4.2]`。

```text
情况 A：数据 <= 1 页（或跨 2 页但 <= 2 页）
  SQE.PRP1 ──▶ 物理页 1 + 页内偏移 (Offset != 0 允许)
  SQE.PRP2 ──▶ 物理页 2 基址 (Offset 必须为 0！)

情况 B：数据跨越 >= 3 页
  SQE.PRP1 ──▶ 物理页 1 + 页内偏移
  SQE.PRP2 ──▶ 指向一页 PRP List (存放 Page 2, Page 3, ... 的 64 位基址)
               └── 若一页 PRP List 装不下 (4KB 最多放 512 条，最后一格指向下一页 PRP List)
```

### 高频踩坑规则（`Invalid Field in Command`）

1. **PRP1 可以有偏移，PRP2/PRP List 条目必须页对齐**：只有 `PRP1` 允许页内偏移非零；`PRP2`（无论是数据页还是 PRP List 指针）以及 PRP List 内的所有数据页指针，其页内偏移位**必须全为 0**。
2. **PRP List 链接指针位置**：当需要多页 PRP List 串联时，**当前 PRP List 页的最后一个 8 字节条目**自动成为指向下一页 PRP List 的指针，不再代表数据页。
3. **跨页边界计算**：若 `PRP1` 偏移为 `2KB`，传输长度为 `4KB`（4KB 页下），数据跨越了两页，此时**必须使用 `PRP2`**，不能因为总长度 `<= 4KB` 就省略 `PRP2`。

---

## SGL 在 PCIe 下的支持与对齐差异

不同于 NVMe-oF（Fabrics 强制用 SGL），在 PCIe 传输下 **SGL 是可选的**。主机驱动必须先检查 `Identify Controller` 返回的 **`SGLS`（SGL Support，Bytes `[539:536]`）** 字段 `[PCIe Transport §4.2]`：

| 对比维度 | PRP（PCIe 必选） | SGL（PCIe 可选，看 `SGLS`） |
| :--- | :--- | :--- |
| **SQE 标志位** | `CDW0.PSDT = 00b` | `CDW0.PSDT = 01b`（Metadata 为连续缓冲）或 `10b`（Metadata 为 SGL） |
| **数据段对齐** | 后续页必须 **4KB（`CC.MPS`）页对齐** | 由 `SGLS` 决定：可能要求 **DWORD（4B）对齐**，或支持 **Byte 对齐** |
| **支持命令范围** | 所有 Admin 与 I/O 命令 | 传统上仅限 I/O 命令；Admin 命令支持与否取决于 `SGLS` 的对应位 |
| **适用场景** | 通用块设备 I/O（Linux `nvme-pci` 默认首选） | 用户态零拷贝、非页对齐网络包直接落盘、超大数据块聚合（减少链表跳数） |

```bash
# 查看 NVMe 控制器是否支持 PCIe SGL 及其对齐要求（SGLS 字段）
nvme id-ctrl /dev/nvme0 | grep -i sgls
```

---

## 关联与引申概念

- **描述符结构细节**：PRP 条目与 SGL Segment/Bit Bucket/Last Segment 的位域定义，详见 [数据指针布局](../nvme/concepts/data-pointer-layouts.md)。
- **公共命令格式**：`CDW0.PSDT` 与 `MPTR`（Metadata Pointer）在 64 字节 SQE 中的位置，详见 [公共命令格式](../nvme/concepts/common-command-format.md)。
- **IOMMU 协同**：PRP/SGL 里的地址在开启 IOMMU 时不是物理地址而是 IOVA，详见 [IOMMU、DMA 重映射与地址翻译](../host-system/iommu-and-dma.md)。
