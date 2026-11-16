# NVM 核心 I/O 命令详解（Read / Write / DSM / Copy）

在 NVMe 规范族中，Base 规范只定义了 64 字节 SQE 的公共头（`CDW0–CDW9`），而具体对逻辑块（LBA）进行读写、清零、回收与拷贝的语义（`CDW10–CDW15`），则全部由 **NVM Command Set Specification** 定义 `[NVM CmdSet §3]`。

---

## 高频 NVM I/O 命令一览

| 命令名称 | Opcode | 数据方向 | 核心用途 |
| :--- | :--- | :--- | :--- |
| **Flush** | `00h` | 无数据 | 强制把控制器易失性写缓存中属于该 NSID 的数据刷入非易失性介质 |
| **Write** | `01h` | Host → Ctrl | 向指定 `SLBA` 起连续写入 `NLB + 1` 个逻辑块 |
| **Read** | `02h` | Ctrl → Host | 从指定 `SLBA` 起连续读取 `NLB + 1` 个逻辑块 |
| **Write Uncorrectable** | `04h` | 无数据 | 人为将指定 LBA 标记为“不可纠正错误”（用于注入测试验证容错） |
| **Compare** | `05h` | Host → Ctrl | 将主机内存数据与介质上指定 LBA 内容比对（常与 Write 组成原子 Fused 操作） |
| **Write Zeroes** | `08h` | 无数据 | 无需主机 DMA 传零数据，由控制器直接将目标 LBA 范围逻辑置零（常配合 `DEAC` 回收空间） |
| **Dataset Management (DSM)** | `09h` | Host → Ctrl | 向控制器传递 LBA 范围提示，最常用 **`AD=1`（Deallocate / TRIM）** 通知 SSD 回收废弃块 |
| **Copy** | `19h` | Host → Ctrl | 控制器内部搬运：主机传入源 LBA 范围列表，SSD 直接在内部把数据复制到目标 `SLBA`，无需经过 PCIe 总线 |

---

## Read (`02h`) / Write (`01h`) 核心字段（`CDW10–12`）

```text
  CDW10 : SLBA[31:0]   (起始逻辑块地址低 32 位)
  CDW11 : SLBA[63:32]  (起始逻辑块地址高 32 位)
  CDW12 : [31] LR | [30] FUA | [29:26] PRINFO | [25:20] DTYPE | [15:0] NLB (0-based!)
```

- **`NLB`（Number of Logical Blocks, `CDW12[15:0]`）**：**0-based** 值。填 `0` 表示传输 **1 个逻辑块**，填 `7` 表示 **8 个逻辑块**。单条命令最大传输长度还受 `Identify Controller.MDTS`（Maximum Data Transfer Size）限制，超限会返回 `Invalid Field in Command`。
- **`FUA`（Force Unit Access, `CDW12[30]`）**：置 `1` 时绕过控制器易失性写缓存。对 `Write`，保证数据落盘后才返回 CQE；对 `Read`，保证从非易失介质读取最新数据。数据库 WAL 日志与文件系统 Journal 极度依赖此位。
- **`LR`（Limited Retry, `CDW12[31]`）**：置 `1` 时指示控制器减少 ECC/读重试次数，以“宁可早报错、不要拖延迟”保障尾部延迟（Tail Latency）。

---

## Dataset Management (`09h`) 与 `AD`（TRIM）

文件系统删除文件后，SSD FTL 并不知道哪些 LBA 已经无效，仍会在垃圾回收（GC）时搬运这些死数据，推高写放大（WAF）。操作系统通过 **DSM 命令** 通知 SSD `[NVM CmdSet §3.2.2]`：
- **`CDW10.NR`（Bits `[7:0]`）**：Range 描述符数量（**0-based**，最多 256 个 Range，每个 Range 16 字节：`Context Attributes` + `Length in Logical Blocks` + `Starting LBA`）。
- **`CDW11.AD`（Bit `2`, Attribute Deallocate）**：置 `1` 表示释放这些 LBA 范围。释放后的 LBA 再被读取时，返回值由 `Identify Namespace.DLFEAT` 决定（全 0、全 FF 或不确定）。
- **`CDW11.IDW` / `IDR`**：建议性提示（Integral Dataset for Write/Read），提示 SSD 将这批范围视为同一冷热/生命周期的数据流。

```bash
# 使用 nvme-cli 查看支持的最大传输块数 (MDTS) 与发送 Deallocate (TRIM)
nvme id-ctrl /dev/nvme0 | grep -i mdts
nvme dsm /dev/nvme0n1 -s 0 -b 1024 -d
```

---

## 关联与引申概念

- **融合操作（Fused Operations）**：`Compare` + `Write` 如何作为不可分割的原子操作执行，详见 [命令排序与仲裁](../nvme/concepts/command-ordering-and-arbitration.md)。
- **公共 I/O 控制命令**：不涉及具体块读写的跨命令集控制命令，详见 [公共 I/O 控制命令](../nvme/concepts/common-io-control-commands.md)。
- **数据指针**：`Read`/`Write` 的数据缓冲区如何通过 PRP/SGL 描述，详见 [PRP / SGL 内存描述符与对齐规则](../nvme-pcie-transport/prp-sgl-and-alignment.md)。
