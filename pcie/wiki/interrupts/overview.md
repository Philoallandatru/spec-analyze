# PCIe 中断机制概述

PCIe 提供了三种中断机制：传统的 INTx、MSI (Message Signaled Interrupts) 和 MSI-X (Extended Message Signaled Interrupts)。从 INTx 到 MSI-X 的演进极大地提升了中断性能和灵活性。

---

## 概述

**为什么需要中断？**
- 设备通知 CPU 事件发生（I/O 完成、错误、状态变化）
- 避免轮询，节省 CPU 资源
- 实现异步操作，提高系统吞吐量

**三种中断机制**：

| 机制 | 引入时间 | 中断数量 | 共享性 | 性能 | 复杂度 |
|------|---------|---------|--------|------|--------|
| Legacy INTx | PCI 1.0 (1992) | 4 个 (A/B/C/D) | 共享 | 低 | 低 |
| MSI | PCI 2.2 (1998) | 1/2/4/8/16/32 | 独占 | 中 | 中 |
| MSI-X | PCI 3.0 (2002) | 最多 2048 | 独占 | 高 | 高 |

**中断在系统中的流程**：
```
设备事件发生
    │
    ├─> INTx: 物理中断线电平变化
    │         ↓
    │   中断控制器 (8259/IOAPIC)
    │         ↓
    │   CPU 接收中断
    │
    ├─> MSI/MSI-X: Memory Write TLP
    │         ↓
    │   写入特定内存地址 (中断控制器映射)
    │         ↓
    │   中断控制器解析地址和数据
    │         ↓
    │   CPU 接收中断
    │
    └─> 中断处理程序执行
```

---

## Legacy INTx 中断

### 工作原理

INTx 使用 **物理中断线** 传递信号，继承自传统 PCI。

**4 条中断线**：
```
设备可使用 4 条中断线之一:
  INTA#, INTB#, INTC#, INTD#
  
多设备共享中断线:
┌────────┐
│ Device │────> INTA# ─┐
└────────┘             │
┌────────┐             ├───> 中断控制器 (IOAPIC) ───> CPU
│ Device │────> INTA# ─┘
└────────┘
  ↑
共享同一中断线，需要软件轮询
```

**信号方式**：
- **电平触发** (Level-Triggered)：中断线保持低电平直到清除
- **边沿触发** (Edge-Triggered)：某些旧系统，已不推荐

### INTx 消息机制

PCIe **没有物理中断线**，通过 **Message** 模拟 INTx：

**断言中断**：
```
发送 Assert_INTx Message:
┌────────────────────────────────┐
│ TLP Type: Message (Msg)        │
│ Message Code: 0x20 (Assert_A)  │
│              0x21 (Assert_B)   │
│              0x22 (Assert_C)   │
│              0x23 (Assert_D)   │
│ Routing: Implicit (到 RC)      │
└────────────────────────────────┘
```

**取消中断**：
```
发送 Deassert_INTx Message:
┌────────────────────────────────┐
│ TLP Type: Message (Msg)        │
│ Message Code: 0x24 (Deassert_A)│
│              0x25 (Deassert_B) │
│              0x26 (Deassert_C) │
│              0x27 (Deassert_D) │
└────────────────────────────────┘
```

### 配置

**Interrupt Line/Pin 寄存器**：
```
配置空间 0x3C-0x3F:
┌──────────┬──────────┬──────────┬──────────┐
│  Max_Lat │ Min_Gnt  │ Int Pin  │ Int Line │
│  (0x3F)  │  (0x3E)  │  (0x3D)  │  (0x3C)  │
└──────────┴──────────┴──────────┴──────────┘

Interrupt Pin (只读):
  0x00 = 不使用中断
  0x01 = INTA#
  0x02 = INTB#
  0x03 = INTC#
  0x04 = INTD#

Interrupt Line (读写):
  系统分配的 IRQ 号 (如 IRQ 11)
  OS 特定，BIOS/OS 写入
```

### 优缺点

**优点**：
- ✅ 简单，无需特殊配置
- ✅ 所有设备都支持
- ✅ 兼容传统 PCI

**缺点**：
- ❌ **共享中断**：多个设备共享，需要轮询所有处理程序
- ❌ **性能差**：每次中断需要读取设备寄存器确认
- ❌ **仅 4 条线**：无法支持多队列
- ❌ **消息开销**：PCIe 需要发送 Message TLP

**共享中断示例**：
```c
// Linux 中断处理程序
irqreturn_t nvme_irq_handler(int irq, void *data)
{
    struct nvme_queue *nvmeq = data;
    
    // 检查是否是本设备的中断
    if (!nvme_process_cq(nvmeq)) {
        return IRQ_NONE;  // 不是本设备，返回未处理
    }
    
    return IRQ_HANDLED;  // 处理完成
}

// 当 IRQ 11 触发时，所有共享 IRQ 11 的处理程序都会被调用
// 每个处理程序需要检查自己的设备
```

---

## MSI (Message Signaled Interrupts)

### 工作原理

MSI 通过 **Memory Write TLP** 发送中断，无需物理中断线。

**基本流程**：
```
1. OS 配置 MSI Capability:
   - 分配中断地址 (如 0xFEE00000)
   - 分配中断数据 (如 0x4000)

2. 设备发送中断:
   构造 Memory Write TLP:
   ┌─────────────────────────────┐
   │ Type: Memory Write (Posted) │
   │ Address: 0xFEE00000         │
   │ Data: 0x4000                │
   │ Length: 1 DW                │
   └─────────────────────────────┘
   
3. Root Complex 解析:
   - 识别为 MSI 地址范围
   - 从 Data 提取中断向量
   - 触发 CPU 中断

4. CPU 执行中断处理程序:
   - 直接跳转到对应向量
   - 无需轮询
```

**地址空间**：
```
x86/x64 MSI 地址格式:
  0xFEExxxxx (固定前缀)
  
ARM GICv3 MSI 地址:
  由 GIC 配置确定
```

### MSI Capability 结构

**Capability 寄存器布局**：
```
Offset 0x00: Message Control (16-bit)
┌───┬────────┬────────┬───────┬────────┬────────┐
│En │ MMC    │  MME   │ 64bit │Per-Vec │ Rsvd   │
│(1)│ (3)    │  (3)   │ (1)   │Mask(1) │        │
└───┴────────┴────────┴───────┴────────┴────────┘
Bit 0:    MSI Enable (1=启用)
Bit 3-1:  Multiple Message Capable (设备支持的中断数)
          000b = 1, 001b = 2, 010b = 4, 011b = 8,
          100b = 16, 101b = 32
Bit 6-4:  Multiple Message Enable (OS 分配的中断数)
Bit 7:    64-bit Address Capable
Bit 8:    Per-Vector Masking Capable

Offset 0x04: Message Address (32-bit) / Lower (64-bit)
┌────────────────────────────────────────────────┐
│         Message Address [31:2] | 00            │
└────────────────────────────────────────────────┘

Offset 0x08: Message Upper Address (仅 64-bit)
┌────────────────────────────────────────────────┐
│         Message Address [63:32]                │
└────────────────────────────────────────────────┘

Offset 0x0C (32-bit) / 0x08 (64-bit): Message Data
┌────────────────────────────────────────────────┐
│         Message Data (16-bit)                  │
└────────────────────────────────────────────────┘

Offset 0x10 (64-bit): Mask Bits (可选)
┌────────────────────────────────────────────────┐
│   Per-Vector Mask (每位对应一个中断)           │
└────────────────────────────────────────────────┘
```

### 多中断支持

MSI 支持 **1, 2, 4, 8, 16, 32** 个中断：

**中断向量编码**：
```
配置:
  Message Address: 0xFEE00000
  Message Data: 0x4000
  Multiple Message Enable: 010b (4 个中断)

设备发送中断:
  中断 0: Data = 0x4000 (基数)
  中断 1: Data = 0x4001 (基数 + 1)
  中断 2: Data = 0x4002 (基数 + 2)
  中断 3: Data = 0x4003 (基数 + 3)
  
Data 低位被设备修改，高位保持不变
```

**限制**：
- 中断数量必须是 2 的幂
- 所有中断地址相同，仅 Data 不同
- OS 必须分配连续的中断向量

### 优缺点

**优点**：
- ✅ **独占中断**：无需共享，无轮询
- ✅ **无物理线**：通过 Memory Write 实现
- ✅ **多中断**：最多 32 个
- ✅ **Posted**：Memory Write 是 Posted，低延迟

**缺点**：
- ❌ **中断数限制**：最多 32 个，不足以支持多队列 NVMe
- ❌ **必须连续**：OS 必须分配连续向量
- ❌ **共享地址**：所有中断使用相同地址
- ❌ **掩码可选**：不是所有设备都支持 Per-Vector Masking

---

## MSI-X (Extended MSI)

### 工作原理

MSI-X 是 MSI 的增强版，支持 **更多中断**和 **独立配置**。

**关键改进**：
- **最多 2048 个中断**（实际受限于设备实现）
- **每个中断独立配置**：独立的地址和数据
- **独立掩码控制**：每个中断可单独屏蔽
- **Table 存储在 BAR 中**：通过内存访问配置

**MSI-X 架构**：
```
┌──────────────────────────────────────┐
│  MSI-X Capability (配置空间)         │
│  - Table Size: 16                   │
│  - Table Offset: 0x2000 (在 BAR 0) │
│  - PBA Offset: 0x3000 (在 BAR 0)   │
└──────────────────────────────────────┘
            │
            ▼
┌──────────────────────────────────────┐
│  BAR 0 (设备内存空间)                │
│  ┌────────────────────────────────┐  │
│  │ 0x0000: 设备寄存器             │  │
│  ├────────────────────────────────┤  │
│  │ 0x2000: MSI-X Table            │◄─── 中断配置
│  │   Entry 0: Addr=0xFEE..., Data │  │
│  │   Entry 1: Addr=0xFEE..., Data │  │
│  │   ...                          │  │
│  ├────────────────────────────────┤  │
│  │ 0x3000: PBA (Pending Bit Array)│◄─── 中断挂起状态
│  │   Bit 0-63: 中断 0-63 挂起状态 │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
```

### MSI-X 优势

**与 MSI 对比**：

| 特性 | MSI | MSI-X |
|------|-----|-------|
| 最大中断数 | 32 | 2048 |
| 中断地址 | 共享 | 独立 |
| 中断数据 | 部分可变 | 完全独立 |
| 掩码控制 | 可选 | 必须支持 |
| 配置方式 | 配置空间 | BAR 内存 |
| 运行时修改 | 困难 | 容易 |
| 中断分配 | 必须连续 | 可以分散 |

**典型应用**：
- **多队列设备**：NVMe, 高性能网卡
- **每队列一个中断**：避免队列竞争
- **NUMA 优化**：将中断绑定到特定 CPU

---

## 中断机制对比

### 性能对比

**中断延迟**：
```
INTx:  设备发送 Message → RC 转换为电平中断 → IOAPIC → CPU
       延迟: ~5-10 μs (包括共享中断轮询)

MSI:   设备发送 Memory Write → RC 解析 → 直接触发 CPU
       延迟: ~1-2 μs

MSI-X: 与 MSI 相同，但支持更好的 CPU 亲和性
       延迟: ~1-2 μs (可优化到 <1 μs)
```

**吞吐量**：
```
INTx:  共享导致处理程序开销大，~50K 中断/秒
MSI:   独占，~200K 中断/秒
MSI-X: 多队列并行，>1M 中断/秒 (每队列 100K+)
```

### 功能对比

| 功能 | INTx | MSI | MSI-X |
|------|------|-----|-------|
| 中断共享 | 是 | 否 | 否 |
| 中断数量 | 1 | 1-32 | 1-2048 |
| CPU 亲和性 | 差 | 中 | 优秀 |
| 运行时掩码 | 困难 | 部分支持 | 完全支持 |
| 虚拟化支持 | 差 | 中 | 优秀 |
| 多队列支持 | 不支持 | 有限 | 完全支持 |
| 配置复杂度 | 低 | 中 | 高 |

### 选择建议

**何时使用 INTx**：
- 传统设备兼容
- 简单设备（低中断率）
- 不支持 MSI/MSI-X 的系统

**何时使用 MSI**：
- 需要独占中断
- 中断数量 ≤ 32
- 不需要运行时重新配置

**何时使用 MSI-X**（推荐）：
- 高性能设备
- 多队列架构
- 需要 CPU 亲和性优化
- 现代设备（首选）

---

## FEMU 代码示例

### 检测中断支持

```c
// hw/block/nvme.c
static void nvme_init_pci(NvmeCtrl *n, PCIDevice *pci_dev)
{
    uint8_t *pci_conf = pci_dev->config;
    
    // 配置 Interrupt Pin (INTx)
    pci_conf[PCI_INTERRUPT_PIN] = 1;  // INTA#
    
    // 添加 MSI Capability (可选)
    if (msi_present(pci_dev)) {
        msi_init(pci_dev, 0x50,  // Capability offset
                 1,               // 1 个中断向量
                 true,            // 64-bit 地址
                 false,           // 无 Per-Vector Masking
                 NULL);
    }
    
    // 添加 MSI-X Capability (推荐)
    if (msix_present(pci_dev)) {
        msix_init(pci_dev, n->params.msix_qsize,
                  &n->iomem,     // Table BAR
                  0,             // Table BAR 号
                  0x2000,        // Table Offset
                  &n->iomem,     // PBA BAR
                  0,             // PBA BAR 号
                  0x3000,        // PBA Offset
                  0x50,          // Capability offset
                  NULL);
    }
}
```

### 发送中断

```c
// 发送 INTx 中断
static void nvme_irq_assert_intx(NvmeCtrl *n)
{
    PCIDevice *pci = PCI_DEVICE(n);
    
    if (!msix_enabled(pci) && !msi_enabled(pci)) {
        pci_irq_assert(pci);  // 断言 INTx
    }
}

// 发送 MSI 中断
static void nvme_irq_notify_msi(NvmeCtrl *n, NvmeQueue *q)
{
    PCIDevice *pci = PCI_DEVICE(n);
    
    if (msi_enabled(pci)) {
        msi_notify(pci, q->vector);  // 发送 MSI, vector 0-31
    }
}

// 发送 MSI-X 中断
static void nvme_irq_notify_msix(NvmeCtrl *n, NvmeQueue *q)
{
    PCIDevice *pci = PCI_DEVICE(n);
    
    if (msix_enabled(pci)) {
        msix_notify(pci, q->vector);  // 发送 MSI-X
    }
}

// 统一中断通知
static void nvme_irq_notify(NvmeCtrl *n, NvmeQueue *q)
{
    PCIDevice *pci = PCI_DEVICE(n);
    
    if (msix_enabled(pci)) {
        msix_notify(pci, q->vector);
    } else if (msi_enabled(pci)) {
        msi_notify(pci, q->vector);
    } else {
        pci_irq_assert(pci);
    }
}
```

---

## 实用技巧

### 查看设备中断配置

```bash
# 查看中断类型
lspci -vvv -s 01:00.0 | grep -i "interrupt\|msi"
  Interrupt: pin A routed to IRQ 11
  Capabilities: [50] MSI: Enable- Count=1/1 Maskable- 64bit+
  Capabilities: [70] MSI-X: Enable+ Count=16 Masked-
  
# MSI-X Enable+ → 使用 MSI-X
# MSI Enable- → MSI 未启用

# 查看 MSI-X 中断分配
cat /proc/interrupts | grep nvme
  128:  0  0  0  0  IR-PCI-MSI-0000:01:00.0   0-edge  nvme0q0
  129:  100 0  0  0  IR-PCI-MSI-0000:01:00.0   1-edge  nvme0q1
  130:  0  200 0  0  IR-PCI-MSI-0000:01:00.0   2-edge  nvme0q2
  ...
```

### 测试中断性能

```bash
# 使用 fio 测试中断率
fio --name=iops --rw=randread --bs=4k --numjobs=4 \
    --iodepth=32 --filename=/dev/nvme0n1 --direct=1 \
    --runtime=10

# 监控中断
watch -n1 'cat /proc/interrupts | grep nvme'
```

---

## 总结

### 关键要点

1. ✅ **INTx 简单但共享**，适合传统兼容，性能差
2. ✅ **MSI 独占中断**，最多 32 个，性能中等
3. ✅ **MSI-X 最强大**，最多 2048 个，支持多队列和 CPU 亲和性
4. ✅ **现代设备应优先使用 MSI-X**，提供最佳性能
5. ✅ **中断通过 Memory Write TLP** 实现（MSI/MSI-X），无需物理线
6. ✅ **FEMU 提供完整的中断模拟**，支持三种机制

---

## 下一步学习

- [MSI-X 详解](msix.md) - MSI-X Table 和 PBA 详细结构
- [中断亲和性优化](affinity.md) - 绑定中断到特定 CPU
- [中断合并](coalescing.md) - 减少中断频率优化

---

## 参考资料

- **规范**：PCIe Base Spec Chapter 6.1 (Interrupt and PME Support)
- **MSI**：PCI Local Bus Specification 2.2+
- **MSI-X**：PCI Local Bus Specification 3.0+
- **实现**：`hw/pci/msi.c`, `hw/pci/msix.c` (FEMU)

---

**相关页面**：
- [← TLP 格式](../transaction-layer/tlp-format.md)
- [MSI-X 详解 →](msix.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
