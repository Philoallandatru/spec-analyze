# MSI 中断详解

**MSI（Message Signaled Interrupts，消息信号中断）允许 PCIe 设备通过发送一笔普通的“内存写 TLP（Memory Write）”来触发 CPU 中断**，彻底摆脱了传统 INTx 的共享轮询开销。所有支持 PCIe 的设备都必须支持 MSI 或 MSI-X。

> **直观类比**：如果 INTx 是拉响公共警铃，那么 **MSI** 就像给 CPU 的收件箱（特定物理地址 `0xFEExxxxx`）直接投递一张写着“中断号（Message Data）”的快递单。Root Complex 看到这笔写往 `0xFEExxxxx` 的内存写请求时，不会真的去写内存条，而是立刻按快递单上的编号把对应的 CPU 叫醒。

---

## 1. MSI 的工作原理与特点

1. **配置阶段**：OS 在配置空间的 **MSI Capability（ID = `0x05`）** 中填入目标地址（`Message Address`）和基准中断数据（`Message Data`），并置位 `MSI Enable`。
2. **触发阶段**：当事件发生时，设备构造一笔 **Posted Memory Write TLP**（写往 `Message Address`，载荷为 `Message Data`）。
3. **直达 CPU**：由于写往 `0xFEExxxxx`（x86 Local APIC 窗口），Root Complex 直接将它路由给目标 CPU 的 Local APIC，无需经过 IOAPIC 和共享中断链表。

**MSI 的主要局限（与 MSI-X 对比）**：
- **向量数量受限**：最多只支持 **32 个**向量，且必须是 2 的幂（1, 2, 4, 8, 16, 32）。
- **必须连续且同目标**：所有向量共用同一组 `Message Address`，这意味着无法把不同向量分散到不同 CPU 核心；且 OS 必须分配连续的中断向量号。
- **掩码可选**：Per-Vector Masking（单向量屏蔽）在 MSI 中是可选特性，而 MSI-X 中强制支持。

---

## 2. MSI Capability 结构（ID = `0x05`）

MSI Capability 位于配置空间 `0x40–0xFF` 链表中，大小随是否支持 **64 位地址** 和 **Per-Vector Masking** 从 10 字节到 24 字节不等：

```
偏移     31                            16 15            8 7             0
+0x00   ┌────────────────────────────────┬───────────────┬───────────────┐
        │     Message Control (16b)      │   Next Ptr    │ Cap ID = 0x05 │
+0x04   ├────────────────────────────────┴───────────────┴───────────────┤
        │                  Message Address Lower (32b)                   │
+0x08   ├────────────────────────────────────────────────────────────────┤
        │   Message Address Upper (32b)  [仅 64-bit Capable=1 时存在]     │
+0x0C   ├────────────────────────────────┬───────────────────────────────┤
        │          Reserved              │      Message Data (16b)       │ (32-bit 时在 +0x08)
+0x10   ├────────────────────────────────┴───────────────────────────────┤
        │     Mask Bits (32b)            [仅 Per-Vector Masking=1 时存在] │ (32-bit 时在 +0x0C)
+0x14   ├────────────────────────────────────────────────────────────────┤
        │     Pending Bits (32b)         [仅 Per-Vector Masking=1 时存在] │ (32-bit 时在 +0x10)
        └────────────────────────────────────────────────────────────────┘
```

### Message Control 寄存器（`+0x02`，16-bit）

| 位 | 名称 | 读写 | 说明 |
|----|------|------|------|
| `Bit 0` | **MSI Enable** | RW | `1` = 启用 MSI（同时应关闭 Command[Bit 10] 的 INTx）。 |
| `Bits 3:1` | **MMC (Multiple Message Capable)** | RO | 硬件支持的向量数编码：`2^MMC`（`000`=1, `001`=2, ..., `101`=32）。 |
| `Bits 6:4` | **MME (Multiple Message Enable)** | RW | OS 实际分配的向量数编码：`2^MME`（必须 $\le$ MMC）。 |
| `Bit 7` | **64-bit Address Capable** | RO | `1` = 支持 64 位 Message Address。 |
| `Bit 8` | **Per-Vector Masking Capable** | RO | `1` = 支持 Mask Bits / Pending Bits 寄存器。 |

---

## 3. x86/x64 地址、数据格式与多向量编码

### 3.1 Message Address 与 Message Data（x86 示例）

```
Message Address (32-bit):
31              20 19            12 11         4  3   2  1 0
┌─────────────────┬────────────────┬────────────┬───┬───┬───┐
│   0xFEE (固定)  │ Destination ID │  Reserved  │RH │DM │0 0│
└─────────────────┴────────────────┴────────────┴───┴───┴───┘
  - 0xFEE: x86 Local APIC 固定内存窗口前缀
  - Destination ID: 目标 CPU 的 APIC ID
  - DM (Bit 2): 0=Physical APIC ID, 1=Logical

Message Data (16-bit):
15   14  13 11 10       8 7                               0
┌───┬───┬─────┬──────────┬─────────────────────────────────┐
│Lvl│TM │Rsvd │Deliv Mode│      Interrupt Vector (0-255)   │
└───┴───┴─────┴──────────┴─────────────────────────────────┘
  - Vector [7:0]: CPU 中断向量号；TM=0 表示边沿触发 (Edge)
  - Delivery Mode [10:8]: 000b=Fixed（定向投递）
```

> **提示**：在开启 **IOMMU 中断重映射（Interrupt Remapping）** 时，Message Address/Data 中填入的不再是原始 APIC ID 和 Vector，而是经过重映射编码的句柄（详见 [中断路由详解](routing.md)）。

### 3.2 多向量工作方式

当 OS 分配 $N = 2^{\text{MME}}$ 个向量时，设备会根据触发的中断源，自动将 `Message Data` 的**最低 `MME` 位**替换为中断源编号（`0` 到 `N-1`）：

- 例：OS 配置 `Message Data = 0x4020`（低 2 位为 `00b`），`MME = 010b`（4 个向量）。
- 设备发出的 4 个中断分别为：`0x4020`（向量 0）、`0x4021`（向量 1）、`0x4022`（向量 2）、`0x4023`（向量 3）。

### 3.3 Per-Vector Masking 与 Pending Bits

若设备支持该特性，驱动可通过写 `Mask Bits` 的第 `i` 位为 `1` 来临时屏蔽第 `i` 个向量：屏蔽期间设备若产生中断，**不会发送 TLP**，而是将 `Pending Bits` 的第 `i` 位置 `1`；当驱动解除屏蔽（清零 Mask 位）后，硬件自动补发该中断。

---

## 4. 驱动接入与 CLI 调试

在现代 Linux 内核驱动中，无需手动拼接配置空间字节，只需调用 `pci_alloc_irq_vectors(pdev, min, max, PCI_IRQ_MSI)` 分配向量，再通过 `pci_irq_vector(pdev, i)` 获取 Linux IRQ 号并调用 `request_irq()` 注册即可。

```bash
# 1. 查看 MSI Capability 的启用状态、向量数、地址与数据
lspci -vvv -s 01:00.0 | grep -A 3 "MSI:"
# 示例: Capabilities: [50] MSI: Enable+ Count=8/32 Maskable+ 64bit+
#       Address: 00000000fee00000  Data: 4022

# 2. 用 setpci 读取 MSI Message Control（假设 Capability 位于 0x50）
setpci -s 01:00.0 CAP_MSI+2.W
```

---

## 关联与引申概念

| 概念 | 简要说明 | 相关文档 |
|------|----------|----------|
| **MSI-X** | MSI 的全面升级版，将表搬到 BAR 空间，支持 2048 个独立路由的中断向量。 | [MSI-X 详解](msix.md) |
| **Legacy INTx** | 传统共享电平中断的 PCIe 消息模拟，性能低于 MSI。 | [Legacy INTx 详解](intx.md) |
| **Local APIC / 中断重映射** | MSI 的 `0xFEExxxxx` 地址直接指向 x86 Local APIC，虚拟化下由 VT-d IRTE 重映射。 | [中断路由详解](routing.md) |
| **Capability Linked List** | MSI (`0x05`) 作为标准能力挂在配置空间 `0x40–0xFF` 链表中。 | [能力结构详解](../configuration/capabilities.md) |
