# MSI-X 详解

MSI-X (Message Signaled Interrupts - Extended) 是 PCIe 最强大的中断机制，支持多达 2048 个独立配置的中断向量，是现代高性能设备（如 NVMe、高速网卡）的首选中断方案。

---

## 概述

**MSI-X 的核心特性**：
- **海量中断**：最多 2048 个独立中断向量
- **独立配置**：每个中断有独立的地址和数据
- **独立掩码**：每个中断可单独启用/禁用
- **基于内存**：配置存储在 BAR 空间，易于访问
- **灵活分配**：中断向量可以非连续分配

**MSI-X 架构**：
```
┌─────────────────────────────────────────────┐
│  配置空间 MSI-X Capability                  │
│  ┌──────────────────────────────────────┐   │
│  │ Control: Enable, Function Mask       │   │
│  │ Table: BAR=0, Offset=0x2000, Size=16 │   │
│  │ PBA: BAR=0, Offset=0x3000            │   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  BAR 0 (设备内存映射)                       │
│  ┌──────────────────────────────────────┐   │
│  │ 0x0000: 设备寄存器区域               │   │
│  ├──────────────────────────────────────┤   │
│  │ 0x2000: MSI-X Table                  │◄──┐
│  │ ┌────────────────────────────────┐   │   │
│  │ │ Entry 0:                       │   │   │
│  │ │  + Msg Addr Low:  0xFEE00000   │   │   ├─ OS 配置
│  │ │  + Msg Addr High: 0x00000000   │   │   │
│  │ │  + Msg Data:      0x4000       │   │   │
│  │ │  + Vector Control: 0 (Unmasked)│   │   │
│  │ ├────────────────────────────────┤   │   │
│  │ │ Entry 1: ...                   │   │   │
│  │ │ Entry 2: ...                   │   │   │
│  │ └────────────────────────────────┘   │   │
│  ├──────────────────────────────────┤   │   │
│  │ 0x3000: PBA (Pending Bit Array)  │◄──┘
│  │ ┌────────────────────────────────┐   │
│  │ │ QWORD 0: Bits 0-63             │   │
│  │ │ QWORD 1: Bits 64-127           │   │
│  │ │ ...                            │   │
│  │ └────────────────────────────────┘   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

**典型应用场景**：
- **NVMe 设备**：每个 I/O 队列独立中断
- **多队列网卡**：每个 RX/TX 队列独立中断
- **GPU**：多引擎异步通知
- **存储控制器**：多通道并行处理

---

## MSI-X Capability 结构

### Capability Header

**位于配置空间**，标识 MSI-X 能力的位置和大小：

```
Offset 0x00: Capability ID (0x11) + Next Pointer
┌─────────────────┬──────────────────────────┐
│   Next (8 bit)  │  Cap ID = 0x11 (8 bit)   │
└─────────────────┴──────────────────────────┘

Offset 0x02: Message Control (16-bit)
15              11   10       0
┌─────────────────┬───┬────────────────┐
│   Reserved      │ FM│ Table Size - 1 │
│                 │   │   (N-1)        │
└─────────────────┴───┴────────────────┘
Bit 10-0:  Table Size - 1
           实际大小 = 此值 + 1
           范围: 0-2047 (表示 1-2048 个中断)
Bit 14:    Function Mask (全局掩码)
           1 = 屏蔽所有中断
           0 = 根据各 Entry 的 Vector Control
Bit 15:    MSI-X Enable
           1 = 启用 MSI-X
           0 = 禁用 MSI-X

Offset 0x04: Table Offset/BIR (32-bit)
31                           3   2    0
┌──────────────────────────────┬───────┐
│   Table Offset [31:3]        │  BIR  │
│   (8-byte 对齐)              │       │
└──────────────────────────────┴───────┘
BIR (BAR Indicator Register): 0-5 指示哪个 BAR
Table Offset: 在该 BAR 内的偏移（8-byte 对齐）

Offset 0x08: PBA Offset/BIR (32-bit)
31                           3   2    0
┌──────────────────────────────┬───────┐
│   PBA Offset [31:3]          │  BIR  │
│   (8-byte 对齐)              │       │
└──────────────────────────────┴───────┘
```

**示例**（NVMe 设备）：
```bash
lspci -vvv -s 01:00.0 | grep -A5 MSI-X
  Capabilities: [70] MSI-X: Enable+ Count=33 Masked-
    Vector table: BAR=0 offset=00002000
    PBA: BAR=0 offset=00003000

解读:
  Enable+: MSI-X 已启用
  Count=33: 33 个中断向量 (Table Size = 32)
  Vector table: 在 BAR 0, 偏移 0x2000
  PBA: 在 BAR 0, 偏移 0x3000
```

---

## MSI-X Table 结构

### Table Entry 格式

每个 Table Entry **16 字节**，包含 4 个 DWORD：

```
Entry N (16 字节 = 4 DW):

Offset 0x00: Message Address (Lower 32-bit)
31                                         2  0
┌─────────────────────────────────────────┬──┐
│     Msg Addr [31:2]                     │00│
│     (4-byte 对齐)                       │  │
└─────────────────────────────────────────┴──┘

Offset 0x04: Message Address (Upper 32-bit)
┌───────────────────────────────────────────┐
│     Msg Addr [63:32]                      │
└───────────────────────────────────────────┘

Offset 0x08: Message Data (32-bit)
┌───────────────────────────────────────────┐
│     Message Data (16-bit 有效)            │
└───────────────────────────────────────────┘

Offset 0x0C: Vector Control (32-bit)
31                                  1     0
┌───────────────────────────────────┬──────┐
│          Reserved                 │ Mask │
└───────────────────────────────────┴──────┘
Bit 0: Mask Bit
       1 = 屏蔽此中断
       0 = 启用此中断
```

**完整 Table 布局**：
```
Table Base (如 BAR 0 + 0x2000):

Entry 0:  [0x00-0x0F]  ← 中断向量 0
  +0x00: Msg Addr Low
  +0x04: Msg Addr High
  +0x08: Msg Data
  +0x0C: Vector Control

Entry 1:  [0x10-0x1F]  ← 中断向量 1
  +0x10: Msg Addr Low
  +0x14: Msg Addr High
  +0x18: Msg Data
  +0x1C: Vector Control

Entry 2:  [0x20-0x2F]  ← 中断向量 2
  ...

Entry N:  [0xN*16 - 0xN*16+15]
```

### Message Address 格式

**x86/x64 架构的 MSI 地址**：
```
固定前缀: 0xFEE00000
完整格式:
┌────────┬──────┬────┬──────┬────────┬──┐
│ 0xFEE  │ Dest │ RH │ DM   │ Rsvd   │00│
│        │ (8)  │(1) │ (1)  │        │  │
└────────┴──────┴────┴──────┴────────┴──┘
Bit 31-20: 0xFEE (固定)
Bit 19-12: Destination ID (目标 CPU APIC ID)
Bit 3:     Redirection Hint
           0 = 定向到 Destination ID
           1 = 由中断控制器选择
Bit 2:     Destination Mode
           0 = Physical (物理 APIC ID)
           1 = Logical (逻辑模式)

示例:
  0xFEE00000: CPU 0
  0xFEE01000: CPU 1
  0xFEE02000: CPU 2
```

### Message Data 格式

```
15    14  11  10    8  7             0
┌─────┬─────┬───────┬────────────────┐
│ TM  │Rsvd │  DM   │   Vector       │
│ (1) │     │  (3)  │   (8)          │
└─────┴─────┴───────┴────────────────┘
Bit 7-0:   Interrupt Vector (中断向量号)
           范围: 0x20-0xFF (32-255)
Bit 10-8:  Delivery Mode
           000b = Fixed (定向中断)
           001b = Lowest Priority
           010b = SMI
           100b = NMI
           101b = INIT
           111b = ExtINT
Bit 15:    Trigger Mode
           0 = Edge (边沿触发，推荐)
           1 = Level (电平触发)

示例:
  0x0040: Vector 0x40, Fixed, Edge
  0x4123: Vector 0x23, Fixed, Edge
```

---

## PBA (Pending Bit Array)

### 作用

**Pending Bit Array** 记录哪些中断正在挂起（设备已触发但被屏蔽）。

**使用场景**：
```
1. 中断被 Mask (Vector Control Bit 0 = 1)
2. 设备触发中断
3. 设备在 PBA 中设置对应位
4. OS 稍后 Unmask 中断
5. OS 检查 PBA，发现挂起中断
6. OS 重新触发或处理
```

### PBA 格式

**位图数组**，每个中断一个位：
```
PBA Base (如 BAR 0 + 0x3000):

QWORD 0 (Offset 0x00):  Bits 0-63
┌───┬───┬───┬───┬───┬───┬───┬───┐
│63 │...│ 2 │ 1 │ 0 │   │   │   │
└───┴───┴───┴───┴───┴───┴───┴───┘
  ↑               ↑
  中断 63 挂起    中断 0 挂起

QWORD 1 (Offset 0x08):  Bits 64-127
┌───┬───┬───┬───┬───┬───┬───┬───┐
│127│...│66 │65 │64 │   │   │   │
└───┴───┴───┴───┴───┴───┴───┴───┘

...

总大小 = ceil(中断数量 / 64) * 8 字节
示例: 33 个中断 → 需要 1 个 QWORD (8 字节)
      256 个中断 → 需要 4 个 QWORD (32 字节)
```

**读取示例**：
```c
// 读取 PBA，检查中断 5 是否挂起
uint64_t pba_qword = read_pba(0);  // 读取 QWORD 0
bool pending = (pba_qword & (1ULL << 5)) != 0;

if (pending) {
    // 中断 5 在被屏蔽期间触发过
    handle_pending_interrupt(5);
}
```

---

## MSI-X 配置流程

### 设备初始化

**1. 设备声明 MSI-X 支持**（固件/硬件）：
```c
// FEMU: hw/block/nvme.c
static void nvme_realize(PCIDevice *pci_dev, Error **errp)
{
    NvmeCtrl *n = NVME(pci_dev);
    int msix_vectors = n->num_queues + 1;  // Admin + I/O 队列
    
    // 初始化 MSI-X
    // Table 和 PBA 都在 BAR 0
    int ret = msix_init(pci_dev,
                        msix_vectors,      // 中断数量
                        &n->iomem,         // Table BAR
                        0,                 // Table BAR 号 (BAR 0)
                        0x2000,            // Table Offset
                        &n->iomem,         // PBA BAR
                        0,                 // PBA BAR 号 (BAR 0)
                        0x3000,            // PBA Offset
                        0x70,              // Capability offset
                        NULL);
    if (ret) {
        error_setg(errp, "msix_init failed");
        return;
    }
}
```

### OS 配置流程

**2. OS 枚举并配置**：
```
步骤 1: 扫描配置空间
  ├─> 发现 MSI-X Capability (ID 0x11)
  └─> 读取 Table Size, Table/PBA 位置

步骤 2: 映射 BAR
  ├─> 映射包含 Table 和 PBA 的 BAR
  └─> 获取 Table/PBA 虚拟地址

步骤 3: 分配中断向量
  ├─> 为每个中断分配 IRQ 号
  ├─> 计算 Message Address (目标 CPU)
  └─> 计算 Message Data (中断向量)

步骤 4: 配置 MSI-X Table
  for each entry:
    ├─> 写入 Message Address
    ├─> 写入 Message Data
    └─> 清除 Vector Control (Unmask)

步骤 5: 启用 MSI-X
  └─> 设置 Capability Control 的 Enable 位

步骤 6: 注册中断处理程序
  └─> 绑定 IRQ 到处理函数
```

**Linux 配置示例**：
```c
// 驱动程序中的 MSI-X 配置
static int nvme_setup_msix(struct nvme_dev *dev)
{
    int nr_vectors = dev->num_queues;
    struct pci_dev *pdev = to_pci_dev(dev->dev);
    
    // 1. 启用 MSI-X
    nr_vectors = pci_alloc_irq_vectors(pdev, 1, nr_vectors,
                                       PCI_IRQ_MSIX);
    if (nr_vectors < 0)
        return nr_vectors;
    
    // 2. 为每个队列分配中断
    for (i = 0; i < nr_vectors; i++) {
        int irq = pci_irq_vector(pdev, i);
        
        // 3. 注册中断处理程序
        err = request_irq(irq, nvme_irq_handler,
                         0, "nvme_queue", &dev->queues[i]);
        if (err)
            goto out_free_irqs;
        
        // 4. 设置中断亲和性 (可选)
        irq_set_affinity_hint(irq, get_cpu_mask(i % num_cpus));
    }
    
    return 0;
    
out_free_irqs:
    // 错误处理...
    return err;
}
```

---

## MSI-X 中断发送

### 设备发送中断

**流程**：
```
1. 设备检测到事件 (如 I/O 完成)
2. 确定中断向量号 (如队列 3 → 向量 3)
3. 读取 MSI-X Table Entry 3:
   - Message Address: 0xFEE01000
   - Message Data: 0x0043
   - Vector Control: 0 (Unmasked)
4. 如果 Masked (Bit 0 = 1):
   ├─> 设置 PBA Bit 3
   └─> 不发送 TLP
   否则:
   └─> 构造 Memory Write TLP
5. 发送 TLP:
   ┌─────────────────────────────┐
   │ Type: Memory Write (Posted) │
   │ Address: 0xFEE01000         │
   │ Data: 0x0043                │
   │ Length: 1 DW                │
   └─────────────────────────────┘
6. Root Complex 接收:
   ├─> 识别为 MSI 地址范围
   ├─> 提取目标 CPU (Addr[19:12] = 1)
   ├─> 提取中断向量 (Data[7:0] = 0x43)
   └─> 触发 CPU 1 的中断 0x43
```

### FEMU 实现

```c
// hw/pci/msix.c
void msix_notify(PCIDevice *dev, unsigned vector)
{
    MSIMessage msg;
    
    // 检查 MSI-X 是否启用
    if (!msix_enabled(dev)) {
        return;
    }
    
    // 检查向量是否被屏蔽
    if (msix_is_masked(dev, vector)) {
        // 设置 PBA 位
        msix_set_pending(dev, vector);
        return;
    }
    
    // 读取 Table Entry
    msg = msix_get_message(dev, vector);
    
    // 发送 MSI Memory Write
    // 实际上调用 address_space_write() 写入中断地址
    address_space_write(&address_space_memory,
                       msg.address,
                       MEMTXATTRS_UNSPECIFIED,
                       &msg.data,
                       sizeof(msg.data));
}

// 获取 Table Entry 内容
MSIMessage msix_get_message(PCIDevice *dev, unsigned vector)
{
    uint8_t *table_entry = dev->msix_table + vector * PCI_MSIX_ENTRY_SIZE;
    MSIMessage msg;
    
    // 读取 Message Address (64-bit)
    msg.address = pci_get_quad(table_entry + PCI_MSIX_ENTRY_LOWER_ADDR);
    
    // 读取 Message Data (32-bit, 实际 16-bit 有效)
    msg.data = pci_get_long(table_entry + PCI_MSIX_ENTRY_DATA);
    
    return msg;
}
```

---

## 性能优势

### 多队列并行

**传统单队列 vs MSI-X 多队列**：
```
单队列 + 1 个中断:
┌──────┐      ┌───────┐
│Queue │─────>│ CPU 0 │ (处理所有 I/O)
└──────┘      └───────┘
瓶颈: CPU 0 处理所有中断，串行化

MSI-X 多队列:
┌────────┐      ┌───────┐
│ Queue 0│─────>│ CPU 0 │
├────────┤      ├───────┤
│ Queue 1│─────>│ CPU 1 │
├────────┤      ├───────┤
│ Queue 2│─────>│ CPU 2 │
├────────┤      ├───────┤
│ Queue 3│─────>│ CPU 3 │
└────────┘      └───────┘
优势: 并行处理，无锁竞争
```

**性能提升**：
```
NVMe 设备 (4 核 CPU):
  1 队列 + 1 中断:  ~500K IOPS
  4 队列 + 4 中断:  ~1.8M IOPS (3.6x)
  
高速网卡 (16 核 CPU):
  1 队列:  10 Gbps
  16 队列: 80+ Gbps (线性扩展)
```

### CPU 亲和性

**绑定中断到特定 CPU**：
```c
// 设置中断亲和性
irq_set_affinity_hint(irq, cpumask_of(cpu));

// 示例: 将队列 N 绑定到 CPU N
for (i = 0; i < num_queues; i++) {
    int cpu = i % num_online_cpus();
    irq_set_affinity_hint(queue[i].irq, cpumask_of(cpu));
}
```

**NUMA 优化**：
```
NUMA Node 0:          NUMA Node 1:
  CPU 0-7               CPU 8-15
  Memory 0-32GB         Memory 32-64GB
  NVMe Queues 0-7       NVMe Queues 8-15
      ↓                     ↓
  本地内存访问          本地内存访问
  (低延迟)              (低延迟)
```

---

## 实用技巧

### 查看 MSI-X 配置

**读取 Table 内容**：
```bash
# 使用 devmem 或 /sys/bus/pci/devices/.../resourceN
# 假设 BAR 0 映射到 0xf8000000, Table 在 0x2000

# 读取 Entry 0
devmem 0xf8002000 64  # Msg Addr
devmem 0xf8002008 32  # Msg Data
devmem 0xf800200c 32  # Vector Control

# 或使用 Python
python3 << EOF
import mmap
import struct

with open('/sys/bus/pci/devices/0000:01:00.0/resource0', 'r+b') as f:
    mm = mmap.mmap(f.fileno(), 0)
    
    # 读取 Entry 0 (offset 0x2000)
    addr_low = struct.unpack('<I', mm[0x2000:0x2004])[0]
    addr_high = struct.unpack('<I', mm[0x2004:0x2008])[0]
    msg_data = struct.unpack('<I', mm[0x2008:0x200c])[0]
    vec_ctrl = struct.unpack('<I', mm[0x200c:0x2010])[0]
    
    print(f"Entry 0:")
    print(f"  Address: 0x{addr_high:08x}{addr_low:08x}")
    print(f"  Data:    0x{msg_data:08x}")
    print(f"  Control: 0x{vec_ctrl:08x} ({'Masked' if vec_ctrl & 1 else 'Unmasked'})")
EOF
```

**查看中断统计**：
```bash
# 查看每个中断的触发次数
cat /proc/interrupts | grep nvme
  130:  123456  0      0      0      IR-PCI-MSI 524288-edge nvme0q0
  131:  0       234567 0      0      IR-PCI-MSI 524289-edge nvme0q1
  132:  0       0      345678 0      IR-PCI-MSI 524290-edge nvme0q2
  133:  0       0      0      456789 IR-PCI-MSI 524291-edge nvme0q3

# 观察负载均衡
watch -n1 'cat /proc/interrupts | grep nvme'
```

### 调试 MSI-X 问题

**检查 MSI-X 是否启用**：
```bash
lspci -vvv -s 01:00.0 | grep MSI-X
  Capabilities: [70] MSI-X: Enable+ Count=33 Masked-

# Enable- → MSI-X 未启用，检查:
# 1. 内核是否支持 (CONFIG_PCI_MSI=y)
# 2. 驱动是否请求 MSI-X
# 3. 系统是否有足够的中断向量
```

**测试中断屏蔽**：
```c
// 屏蔽中断向量 5
uint32_t *table = (uint32_t *)(bar0_base + 0x2000);
table[5 * 4 + 3] |= 1;  // 设置 Vector Control Bit 0

// 触发 I/O，观察 PBA
uint64_t *pba = (uint64_t *)(bar0_base + 0x3000);
if (pba[0] & (1ULL << 5)) {
    printf("Interrupt 5 is pending\n");
}

// 取消屏蔽
table[5 * 4 + 3] &= ~1;
```

---

## 总结

### 关键要点

1. ✅ **MSI-X 支持多达 2048 个独立中断**，远超 MSI 的 32 个
2. ✅ **Table 存储在 BAR 中**，每个 Entry 16 字节，包含地址、数据和掩码
3. ✅ **PBA 记录挂起中断**，当中断被屏蔽时使用
4. ✅ **独立配置**允许每个中断绑定到不同 CPU，实现负载均衡
5. ✅ **多队列设备的最佳选择**，提供最高性能和灵活性
6. ✅ **FEMU 完整实现** MSI-X，包括 Table、PBA 和中断发送

**设计建议**：
- 高性能设备：优先使用 MSI-X
- 中断数量 = 队列数量（通常）
- Table 和 PBA 可放在同一 BAR 或不同 BAR
- 合理设置中断亲和性以优化 NUMA

---

## 下一步学习

- [中断亲和性](affinity.md) - 绑定中断到 CPU
- [中断合并](coalescing.md) - 减少中断频率
- [中断概述](overview.md) - 三种中断机制对比

---

## 参考资料

- **规范**：PCI Local Bus Specification 3.0, Section 6.8 (MSI-X)
- **PCIe**：PCIe Base Spec Chapter 6.1.4 (MSI/MSI-X)
- **实现**：`hw/pci/msix.c` (FEMU)
- **示例**：`hw/block/nvme.c` (NVMe MSI-X 配置)

---

**相关页面**：
- [← 中断概述](overview.md)
- [BARs 详解 →](../configuration/bars.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
