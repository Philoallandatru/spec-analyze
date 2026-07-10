# PCIe 地址空间管理

## 概述

PCIe 设备使用多种地址空间来访问系统资源和设备寄存器。理解这些地址空间的组织、映射和管理对于 PCIe 系统设计和驱动开发至关重要。

## 地址空间类型

### 1. 内存地址空间 (Memory Address Space)

**特点**：
- 64-bit 地址空间（理论最大 2^64 字节）
- 实际使用通常在 4GB 以下或高地址区域
- 支持预取 (Prefetchable) 和非预取 (Non-Prefetchable)
- 最常用的地址空间类型

**用途**：
- 设备控制寄存器 (MMIO)
- DMA 缓冲区
- 显存 (GPU)
- NVMe 命令队列

### 2. I/O 地址空间 (I/O Address Space)

**特点**：
- 32-bit 地址空间（实际使用通常 16-bit，64KB）
- x86 架构特有（通过 IN/OUT 指令访问）
- 不支持预取
- 逐渐被淘汰

**用途**：
- 传统设备兼容
- 配置访问（传统 PCI）

### 3. 配置地址空间 (Configuration Space)

**特点**：
- 每个 Function 有独立的配置空间
- 标准配置空间：256 字节
- 扩展配置空间：4096 字节（PCIe）
- 通过特殊机制访问（ECAM 或 IO 端口）

**用途**：
- 设备识别（Vendor ID, Device ID）
- BAR 寄存器
- Capability 结构
- 设备配置

### 4. Message 地址空间

**特点**：
- 使用 Memory Write 事务实现
- 无需专用地址空间
- 用于中断和系统消息

**用途**：
- MSI/MSI-X 中断
- 电源管理消息
- 错误报告消息

## 地址空间布局

### 典型系统内存映射

```
+---------------------------+ 0xFFFFFFFFFFFFFFFF
|                           |
|   未使用                   |
|                           |
+---------------------------+ 0x100000000 (4GB)
|                           |
|   PCIe 设备 (64-bit BAR)  |
|   - GPU 显存              |
|   - NVMe 大容量 BAR       |
|                           |
+---------------------------+ 0x80000000 (2GB)
|   PCIe 设备 (32-bit BAR)  |
|   - NVMe 寄存器           |
|   - 网卡寄存器            |
+---------------------------+ 0xC0000000 (3GB)
|   系统保留                 |
|   - APIC                  |
|   - HPET                  |
+---------------------------+ 0x100000000
|                           |
|   系统内存 (DRAM)          |
|   (由 BIOS/UEFI 管理)     |
|                           |
+---------------------------+ 0x100000 (1MB)
|   传统 BIOS 区域           |
+---------------------------+ 0x0

I/O 地址空间 (独立的，非内存映射):
+---------------------------+ 0x10000 (64KB)
|   PCIe 设备 I/O BAR       |
+---------------------------+ 0x1000 (4KB)
|   传统设备                 |
|   - 串口 (0x3F8)          |
|   - PCI Config (0xCF8)    |
+---------------------------+ 0x0
```

### PCIe 层次结构中的地址窗口

```
+------------------+
| Root Complex    |
| Memory: 全部     |
| I/O: 全部        |
+------------------+
        |
        v
+------------------+
| Switch          |
| Memory Window:  |
| 0x80000000-     |
| 0xBFFFFFFF      |
+------------------+
    |           |
    v           v
+--------+  +--------+
| Port 1 |  | Port 2 |
| Memory:|  | Memory:|
| 0x8000-|  | 0xA000-|
| 0x9FFF |  | 0xBFFF |
+--------+  +--------+
    |           |
    v           v
+--------+  +--------+
| NVMe   |  | NIC    |
| BAR0:  |  | BAR0:  |
| 0x8000-|  | BAR1:  |
| 0x8003 |  | 0xA000-|
|        |  | 0xA0FF |
+--------+  +--------+
```

## BAR (Base Address Register)

### BAR 类型

**32-bit Memory BAR**：
```
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|31                                           4|3|2|1|0|
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|          Base Address[31:4]                  |P|T|0|
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

Bit 0: 0 = Memory Space
Bit [2:1]: Type (00=32-bit, 10=64-bit)
Bit 3: Prefetchable (1=Yes, 0=No)
Bits [31:4]: Base Address (16-byte aligned)
```

**64-bit Memory BAR**：
```
BAR[n]:   [31:4] Base Address[31:4]  [3]P [2:1]10 [0]0
BAR[n+1]: [31:0] Base Address[63:32]
```

**I/O BAR**：
```
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|31                                           2|1|0|
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|          Base Address[31:2]                  |R|1|
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

Bit 0: 1 = I/O Space
Bit 1: Reserved
Bits [31:2]: Base Address (4-byte aligned)
```

### BAR 大小探测

```c
uint64_t probe_bar_size(uint8_t bus, uint8_t device, 
                        uint8_t function, uint8_t bar_offset)
{
    uint32_t bar_low, bar_high = 0;
    uint32_t size_low, size_high = 0;
    uint64_t size;
    
    // 1. 读取原始 BAR 值
    bar_low = pci_read_config_dword(bus, device, function, bar_offset);
    
    // 2. 写入全 1
    pci_write_config_dword(bus, device, function, bar_offset, 0xFFFFFFFF);
    
    // 3. 读回掩码
    size_low = pci_read_config_dword(bus, device, function, bar_offset);
    
    // 4. 恢复原始值
    pci_write_config_dword(bus, device, function, bar_offset, bar_low);
    
    // 5. 检查 BAR 类型
    if (size_low == 0 || size_low == 0xFFFFFFFF) {
        return 0;  // BAR 未实现
    }
    
    if (size_low & 0x01) {
        // I/O Space BAR
        size_low &= 0xFFFFFFFC;
        size = ~size_low + 1;
    } else {
        // Memory Space BAR
        size_low &= 0xFFFFFFF0;
        
        // 检查是否为 64-bit BAR
        if ((bar_low & 0x06) == 0x04) {
            // 64-bit BAR
            bar_high = pci_read_config_dword(bus, device, function, 
                                             bar_offset + 4);
            pci_write_config_dword(bus, device, function, 
                                   bar_offset + 4, 0xFFFFFFFF);
            size_high = pci_read_config_dword(bus, device, function,
                                              bar_offset + 4);
            pci_write_config_dword(bus, device, function,
                                   bar_offset + 4, bar_high);
            
            uint64_t mask = ((uint64_t)size_high << 32) | size_low;
            size = ~mask + 1;
        } else {
            // 32-bit BAR
            size = ~size_low + 1;
        }
    }
    
    return size;
}
```

### BAR 分配示例

```c
// NVMe 设备 BAR 分配
struct nvme_bars {
    void __iomem *bar0;    // 控制寄存器，64-bit, 16KB
    uint64_t bar0_phys;
    uint32_t bar0_size;
};

void allocate_nvme_bars(struct pci_dev *pdev, struct nvme_bars *bars)
{
    // BAR0: 64-bit Memory, Non-Prefetchable, 16KB
    bars->bar0_phys = pci_resource_start(pdev, 0);
    bars->bar0_size = pci_resource_len(pdev, 0);
    
    if (!(pci_resource_flags(pdev, 0) & IORESOURCE_MEM)) {
        dev_err(&pdev->dev, "BAR0 is not memory space\n");
        return;
    }
    
    // 映射到内核虚拟地址空间
    bars->bar0 = pci_ioremap_bar(pdev, 0);
    if (!bars->bar0) {
        dev_err(&pdev->dev, "Failed to map BAR0\n");
        return;
    }
    
    dev_info(&pdev->dev, 
             "BAR0: phys=0x%llx, size=%uKB, virt=%p\n",
             bars->bar0_phys, bars->bar0_size / 1024, bars->bar0);
}
```

## Bridge 窗口配置

### Memory Window

```
Bridge Memory Base/Limit 寄存器 (0x20-0x22):

+0x20: Memory Base
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|15                                           4|3  0|
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|     Memory Base Address[31:20]               | 0  |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

+0x22: Memory Limit
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|15                                           4|3  0|
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|     Memory Limit Address[31:20]              | 0  |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

地址范围：[Base << 20, (Limit << 20) | 0xFFFFF]
粒度：1 MB
```

### Prefetchable Memory Window

```
Bridge Prefetchable Memory Base/Limit (0x24-0x28):

+0x24: Prefetchable Memory Base (Lower)
+0x28: Prefetchable Memory Base (Upper 32-bit)
+0x26: Prefetchable Memory Limit (Lower)
+0x2C: Prefetchable Memory Limit (Upper 32-bit)

支持 64-bit 地址
```

### I/O Window

```
Bridge I/O Base/Limit (0x1C-0x1E):

+0x1C: I/O Base
+--+--+--+--+--+--+--+--+
|7                    4|3  0|
+--+--+--+--+--+--+--+--+
| I/O Base[15:12]      |Cap|
+--+--+--+--+--+--+--+--+

+0x1D: I/O Limit
+--+--+--+--+--+--+--+--+
|7                    4|3  0|
+--+--+--+--+--+--+--+--+
| I/O Limit[15:12]     |Cap|
+--+--+--+--+--+--+--+--+

Cap: 0=16-bit, 1=32-bit
粒度：4 KB
```

## DMA 地址映射

### CPU 视角 vs 设备视角

```
CPU 物理地址空间:
+------------------+ 0x100000000 (4GB)
|   系统内存        |
|   0-2GB          |
+------------------+ 0x0

PCIe 设备视角 (无 IOMMU):
+------------------+ 0x100000000
|   系统内存        |
|   0-2GB          |
+------------------+ 0x0
(相同的物理地址)

PCIe 设备视角 (有 IOMMU):
+------------------+ 0x100000000
|   IOVA 空间      |
|   (由 IOMMU      |
|    转换到物理)   |
+------------------+ 0x0
(设备使用 IOVA，IOMMU 转换)
```

### DMA 映射类型

**一致性 DMA (Coherent DMA)**：
```c
// 用于控制结构、命令队列等
dma_addr_t dma_handle;
void *cpu_addr;

cpu_addr = dma_alloc_coherent(&pdev->dev, size, 
                               &dma_handle, GFP_KERNEL);

// cpu_addr: CPU 虚拟地址
// dma_handle: 设备使用的总线地址
```

**流式 DMA (Streaming DMA)**：
```c
// 用于大块数据传输
dma_addr_t dma_addr;
void *buffer = kmalloc(size, GFP_KERNEL);

dma_addr = dma_map_single(&pdev->dev, buffer, size, 
                          DMA_TO_DEVICE);

// 使用 dma_addr
do_dma_transfer(dma_addr, size);

// 传输完成后
dma_unmap_single(&pdev->dev, dma_addr, size, DMA_TO_DEVICE);
```

## IOMMU/VT-d

### 地址转换

```
CPU 发起 DMA:
kmalloc() --> 虚拟地址 (VA)
    |
    v (MMU 转换)
物理地址 (PA)
    |
    v (传给设备)
设备 DMA 地址

设备发起 DMA (无 IOMMU):
设备使用物理地址 (PA) 直接访问内存

设备发起 DMA (有 IOMMU):
设备使用 IOVA (I/O Virtual Address)
    |
    v (IOMMU 转换)
物理地址 (PA)
    |
    v
访问内存
```

### IOMMU 页表

```
设备访问 IOVA 0x8000_0000:

1. PCIe TLP: Memory Read, Addr=0x80000000
2. IOMMU 拦截请求
3. 查找设备的页表
   - Requester ID (Bus:Dev.Func) 作为索引
4. 转换 IOVA -> PA
   - IOVA 0x8000_0000 -> PA 0x1234_5000
5. 修改 TLP 地址为 PA
6. 转发到内存控制器
```

## FEMU 实现

### 地址空间管理

```c
// hw/pci/pci.c
static void pci_update_mappings(PCIDevice *d)
{
    PCIIORegion *r;
    int i;
    
    for (i = 0; i < PCI_NUM_REGIONS; i++) {
        r = &d->io_regions[i];
        
        if (r->size == 0) {
            continue;
        }
        
        // 读取 BAR 值
        pcibus_t new_addr;
        if (i == PCI_ROM_SLOT) {
            new_addr = pci_get_long(d->config + PCI_ROM_ADDRESS);
            new_addr &= ~PCI_ROM_ADDRESS_ENABLE;
        } else {
            new_addr = pci_bar_address(d, i, r->type, r->size);
        }
        
        // 如果地址改变，重新映射
        if (new_addr != r->addr) {
            if (r->addr != PCI_BAR_UNMAPPED) {
                memory_region_del_subregion(r->address_space, r->memory);
            }
            
            r->addr = new_addr;
            
            if (r->addr != PCI_BAR_UNMAPPED) {
                memory_region_add_subregion_overlap(
                    r->address_space, r->addr, r->memory, 1);
            }
        }
    }
}
```

### DMA 映射

```c
// hw/pci/pci.c
int pci_dma_read(PCIDevice *dev, dma_addr_t addr, 
                 void *buf, dma_addr_t len)
{
    // 通过 PCIe 的地址空间访问内存
    return dma_memory_read(pci_get_address_space(dev), 
                           addr, buf, len);
}

int pci_dma_write(PCIDevice *dev, dma_addr_t addr,
                  const void *buf, dma_addr_t len)
{
    return dma_memory_write(pci_get_address_space(dev),
                            addr, buf, len);
}
```

### 内存区域层次

```c
// FEMU 中的内存层次
system_memory (MemoryRegion)
    |
    +-- ram (系统内存)
    |
    +-- pci_memory (PCIe 内存空间)
         |
         +-- bridge_memory_window
              |
              +-- nvme_bar0 (设备 BAR)
```

## 地址对齐要求

### BAR 对齐

```
BAR 大小      对齐要求
---------    ---------
4 KB         4 KB
16 KB        16 KB
64 KB        64 KB
1 MB         1 MB
16 MB        16 MB
256 MB       256 MB

规则：BAR 大小必须是 2 的幂，且对齐到其大小
```

### DMA 对齐

```c
// NVMe 对齐要求示例
#define NVME_PAGE_SIZE      4096
#define NVME_SQ_ENTRY_SIZE  64
#define NVME_CQ_ENTRY_SIZE  16

// Admin Queue 必须页对齐
dma_addr_t admin_sq_addr;
admin_sq_addr = dma_alloc_coherent(&pdev->dev, 
                                    NVME_PAGE_SIZE,
                                    &dma_handle, GFP_KERNEL);
assert((dma_handle & (NVME_PAGE_SIZE - 1)) == 0);

// PRP (Physical Region Page) 对齐
// PRP 条目必须指向页对齐的内存
dma_addr_t data_addr;
data_addr = dma_map_single(&pdev->dev, buffer, size, direction);
assert((data_addr & (NVME_PAGE_SIZE - 1)) == 0);
```

## 地址空间冲突处理

### 检测冲突

```c
bool check_address_conflict(uint64_t addr1, uint64_t size1,
                             uint64_t addr2, uint64_t size2)
{
    uint64_t end1 = addr1 + size1 - 1;
    uint64_t end2 = addr2 + size2 - 1;
    
    // 检查是否重叠
    return !(end1 < addr2 || end2 < addr1);
}

// 示例
if (check_address_conflict(bar0_addr, bar0_size,
                            bar1_addr, bar1_size)) {
    printf("ERROR: BAR0 and BAR1 overlap!\n");
}
```

### 资源重分配

```bash
# Linux 内核参数
pci=realloc          # 重新分配所有 PCIe 资源
pci=nocrs            # 忽略 BIOS 的资源分配

# 手动重新分配
echo 1 > /sys/bus/pci/devices/0000:01:00.0/remove
echo 1 > /sys/bus/pci/rescan
```

## 调试技巧

### 查看地址分配

```bash
# lspci 显示 BAR
lspci -vv -s 01:00.0 | grep "Memory at"

# sysfs 查看资源
cat /sys/bus/pci/devices/0000:01:00.0/resource
0x00000000f8000000 0x00000000f8003fff 0x0000000000040200

# /proc/iomem 查看内存映射
cat /proc/iomem | grep nvme
f8000000-f8003fff : nvme
```

### QEMU 监控

```bash
# QEMU Monitor
(qemu) info mtree
address-space: memory
  0000000000000000-ffffffffffffffff (prio 0, i/o): system
    0000000000000000-000000007fffffff (prio 0, ram): ram
    00000000f8000000-00000000f8003fff (prio 1, i/o): nvme-bar0
```

## 总结

PCIe 地址空间管理的关键点：

1. **多种地址空间**：Memory, I/O, Configuration, Message
2. **层次结构**：Root Complex -> Bridge -> Endpoint
3. **BAR 机制**：设备通过 BAR 映射到地址空间
4. **窗口管理**：Bridge 使用窗口限制下游设备的地址范围
5. **DMA 映射**：CPU 地址 ↔ 设备地址转换
6. **IOMMU**：提供地址隔离和保护
7. **对齐要求**：BAR 和 DMA 都有严格的对齐要求

参见：
- [BAR 寄存器详解](../configuration/bars.md)
- [设备枚举](enumeration.md)
- [PCIe 拓扑结构](../introduction/topology.md)
- [DMA 传输](../transaction-layer/transaction-types.md)
