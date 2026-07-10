# Base Address Registers (BARs) 详解

Base Address Registers (BARs) 是 PCIe 设备向系统声明所需地址空间的核心机制，通过 BARs，操作系统可以为设备分配内存或 I/O 地址范围，实现设备寄存器和缓冲区的访问。

---

## 概述

**BAR 的作用**：
- **地址空间映射**：将设备内部寄存器映射到系统地址空间
- **资源分配**：告知操作系统设备需要多大的地址空间
- **访问接口**：提供 CPU 访问设备的入口

**BAR 在配置空间中的位置**：
```
PCIe 配置空间 (Type 0 Header)
┌────────────────────────────────────┐
│ 0x00: Device ID / Vendor ID        │
│ 0x04: Status / Command             │
│ 0x08: Class Code / Revision ID     │
│ 0x0C: BIST / Header Type / ...     │
├────────────────────────────────────┤
│ 0x10: BAR 0                        │◄── 6 个 BARs
│ 0x14: BAR 1                        │
│ 0x18: BAR 2                        │
│ 0x1C: BAR 3                        │
│ 0x20: BAR 4                        │
│ 0x24: BAR 5                        │
├────────────────────────────────────┤
│ 0x28: Cardbus CIS Pointer          │
│ 0x2C: Subsystem ID / Subsystem VID │
│ 0x30: Expansion ROM Base Address   │
│ ...                                │
└────────────────────────────────────┘
```

**BAR 的类型**：

| 类型 | 位宽 | 地址空间 | 常见用途 |
|------|------|---------|---------|
| Memory BAR (32-bit) | 32 位 | 4 GB 以内 | 寄存器、小缓冲区 |
| Memory BAR (64-bit) | 64 位 | 全地址空间 | 大内存映射、DMA 缓冲区 |
| I/O BAR | 32 位 | I/O 空间 | 传统设备兼容（少用） |

---

## BAR 格式详解

### Memory BAR (32-bit)

**格式**：
```
31                                4  3  2  1  0
┌──────────────────────────────────┬──┬──┬───┬─┐
│    Base Address [31:4]           │Pf│Tp│ 0 │0│
│    (16-byte 对齐)                │  │  │   │ │
└──────────────────────────────────┴──┴──┴───┴─┘
Bit 0:    0 = Memory Space (非 I/O)
Bit 2-1:  Type
          00 = 32-bit 地址
          10 = 64-bit 地址（需要两个 BAR）
Bit 3:    Prefetchable
          1 = 可预取（仅数据，无副作用）
          0 = 非预取（有副作用，如寄存器）
Bit 31-4: Base Address（实际分配的地址）
```

**示例**：
```
BAR 值: 0xF000_0008
  Bit 0 = 0 → Memory Space
  Bit 2-1 = 10b → 64-bit
  Bit 3 = 1 → Prefetchable
  Bit 31-4 = 0xF000_000 → Base Address = 0xF000_0000
```

### Memory BAR (64-bit)

64-bit BAR **占用两个连续的 BAR 寄存器**：

**格式**：
```
BAR N (低 32 位):
31                                4  3  2  1  0
┌──────────────────────────────────┬──┬──┬───┬─┐
│    Base Address [31:4]           │Pf│10│ 0 │0│
└──────────────────────────────────┴──┴──┴───┴─┘

BAR N+1 (高 32 位):
31                                            0
┌───────────────────────────────────────────────┐
│    Base Address [63:32]                       │
└───────────────────────────────────────────────┘

完整地址 = (BAR N+1 << 32) | (BAR N & 0xFFFFFFF0)
```

**示例**（NVMe 设备的 64-bit BAR）：
```
BAR 0: 0xF800_000C  (Bit 2-1 = 10b → 64-bit, Bit 3 = 1 → Prefetchable)
BAR 1: 0x0000_0000  (高 32 位)

完整地址: 0x0000_0000_F800_0000
BAR 大小: 通过探测机制确定（见后文）
```

### I/O BAR

**格式**：
```
31                          2  1  0
┌─────────────────────────────┬──┬─┐
│    Base Address [31:2]      │R │1│
└─────────────────────────────┴──┴─┘
Bit 0:    1 = I/O Space
Bit 1:    Reserved
Bit 31-2: Base Address（4-byte 对齐）
```

**特点**：
- **已过时**：现代设备极少使用 I/O BAR
- **地址空间小**：x86 仅 64 KB I/O 空间
- **访问方式**：使用 `inb/outb` 指令（x86）而非内存访问

---

## Prefetchable 属性

### 什么是 Prefetchable？

**定义**：标记内存区域是否可以被 CPU/总线安全地预取。

**Prefetchable (Bit 3 = 1)**：
- **纯数据区域**：如 DMA 缓冲区、帧缓冲区
- **无副作用**：读取不会改变设备状态
- **可缓存**：可被 CPU Cache
- **可合并**：多个读写可以合并
- **可乱序**：CPU 可以乱序执行

**Non-Prefetchable (Bit 3 = 0)**：
- **寄存器区域**：控制/状态寄存器
- **有副作用**：读取可能清除中断标志、改变 FIFO 指针等
- **强序**：必须按程序顺序执行
- **不可缓存**：通常映射为 Uncacheable

**对比表**：

| 特性 | Prefetchable | Non-Prefetchable |
|------|-------------|------------------|
| CPU Cache | 允许 | 禁止（UC/WC） |
| 预取 | 允许 | 禁止 |
| 写合并 | 允许 | 取决于设备 |
| 读副作用 | 无 | 可能有 |
| 典型用途 | DMA 缓冲区、VRAM | 控制寄存器 |

**示例**（NVMe 设备）：
```
BAR 0 (寄存器): Non-Prefetchable
  - Admin/IO 队列寄存器
  - Doorbell 寄存器
  - 读取状态寄存器可能清除标志

BAR 4 (CMB - 可选): Prefetchable
  - Controller Memory Buffer（DMA 缓冲区）
  - 纯数据，无副作用
```

---

## BAR 大小探测机制

### 探测原理

操作系统通过 **写全 1，读回，计算** 的方式探测 BAR 大小：

**步骤**：
```
1. 读取原始 BAR 值（保存）
   BAR = 0xF800_000C

2. 写入全 1 (0xFFFF_FFFF)
   write_config_dword(BAR_offset, 0xFFFFFFFF)

3. 读回 BAR
   BAR = 0xFFFF_000C
   (低位为 0，表示不可配置的位)

4. 计算大小
   可配置位 = BAR & 0xFFFFFFF0 = 0xFFFF_0000
   大小 = ~可配置位 + 1 = ~0xFFFF_0000 + 1 = 0x0001_0000 = 64 KB

5. 恢复原始值
   write_config_dword(BAR_offset, original_bar)
```

**原理**：
- 设备将 BAR 的低位硬连线为 0，对应 BAR 大小的对齐要求
- 写全 1 后，只有可配置的位保持为 1
- 最低的 1 位对应 BAR 大小

**示例**：

| 读回值 | 可配置位掩码 | 大小 | 对齐 |
|-------|-------------|------|------|
| 0xFFFF_FFF0 | 0xFFFF_FFF0 | 16 B | 16 B |
| 0xFFFF_FF00 | 0xFFFF_FF00 | 256 B | 256 B |
| 0xFFFF_F000 | 0xFFFF_F000 | 4 KB | 4 KB |
| 0xFFFF_0000 | 0xFFFF_0000 | 64 KB | 64 KB |
| 0xFFF0_0000 | 0xFFF0_0000 | 1 MB | 1 MB |
| 0xFF00_0000 | 0xFF00_0000 | 16 MB | 16 MB |

### 64-bit BAR 大小探测

对于 64-bit BAR，需要同时探测两个 BAR：

```
BAR 0 探测:
  写: 0xFFFFFFFF
  读: 0xFFFF_000C (64-bit 标志保留)
  低 32 位可配置: 0xFFFF_0000

BAR 1 探测:
  写: 0xFFFFFFFF
  读: 0xFFFF_FFFF
  高 32 位可配置: 0xFFFF_FFFF

完整大小:
  可配置位 = 0xFFFF_FFFF_FFFF_0000
  大小 = ~0xFFFF_FFFF_FFFF_0000 + 1 = 0x0000_0000_0001_0000 = 64 KB

如果高 32 位不全为 1:
  读: BAR 1 = 0xFFFFF000 (假设)
  可配置位 = 0xFFFFF000_FFFF_0000
  大小 = ~0xFFFFF000_FFFF_0000 + 1 = 0x00000FFF_0001_0000 = ~16 GB
```

---

## BAR 分配流程

### 系统启动时的 BAR 配置

**BIOS/UEFI 阶段**：
```
1. 枚举设备
   ├─> 扫描 PCI 总线
   └─> 发现设备

2. 探测 BAR 大小
   ├─> 对每个 BAR 写全 1
   ├─> 读回并计算大小
   └─> 记录需求

3. 分配地址空间
   ├─> 规划内存布局
   ├─> 避免冲突
   └─> 写入 BAR 基地址

4. 启用设备
   └─> 设置 Command 寄存器
       Bit 0: I/O Space Enable
       Bit 1: Memory Space Enable
```

**Linux 内核阶段**：
```
1. 重新扫描（可选）
   pci_scan_bus()

2. 资源分配
   pci_assign_unassigned_resources()
   ├─> 读取 BIOS 分配的地址
   ├─> 如果冲突或未分配，重新分配
   └─> 更新内核资源树

3. 设备驱动映射
   pci_ioremap_bar(pdev, bar_num)
   ├─> 获取物理地址
   ├─> 建立虚拟地址映射
   └─> 返回虚拟地址供驱动使用
```

**地址空间布局示例**：
```
物理内存布局（x86_64）
┌─────────────────────────────────┐ 0xFFFFFFFFFFFFFFFF
│   Reserved                      │
├─────────────────────────────────┤ 0x100000000 (4 GB)
│   PCIe Memory Space (64-bit)    │◄── 64-bit BARs 分配在这里
│   - NVMe BAR 0: 0xF8000000      │
│   - GPU VRAM:   0xE0000000      │
├─────────────────────────────────┤ 0xC0000000 (3 GB)
│   PCIe Memory Space (32-bit)    │◄── 32-bit BARs 分配在这里
│   - Device registers            │
├─────────────────────────────────┤ 0x100000 (1 MB)
│   Low Memory / BIOS             │
├─────────────────────────────────┤ 0x0
│   System RAM                    │
└─────────────────────────────────┘
```

---

## FEMU 代码实现

### BAR 定义与初始化

```c
// hw/block/nvme.c (NVMe 设备为例)
static void nvme_realize(PCIDevice *pci_dev, Error **errp)
{
    NvmeCtrl *n = NVME(pci_dev);
    
    // BAR 0: 控制器寄存器（Non-Prefetchable, 32-bit）
    // 大小: 8 KB (NVMe 规范最小要求)
    memory_region_init_io(&n->iomem, OBJECT(n), &nvme_mmio_ops, n,
                          "nvme", 8192);  // 8 KB
    
    // 注册 BAR 0 (Memory Space, Non-Prefetchable)
    pci_register_bar(pci_dev, 0,
                     PCI_BASE_ADDRESS_SPACE_MEMORY |
                     PCI_BASE_ADDRESS_MEM_TYPE_64,  // 64-bit BAR
                     &n->iomem);
    
    // BAR 4: CMB (Controller Memory Buffer, 可选, Prefetchable)
    if (n->params.cmb_size_mb) {
        memory_region_init_io(&n->cmb, OBJECT(n), &nvme_cmb_ops, n,
                              "nvme-cmb", n->params.cmb_size_mb * MiB);
        
        // 注册 BAR 4 (Prefetchable, 64-bit)
        pci_register_bar(pci_dev, 4,
                         PCI_BASE_ADDRESS_SPACE_MEMORY |
                         PCI_BASE_ADDRESS_MEM_TYPE_64 |
                         PCI_BASE_ADDRESS_MEM_PREFETCH,  // Prefetchable
                         &n->cmb);
    }
}
```

### BAR 类型标志

```c
// include/hw/pci/pci_regs.h
// BAR 寄存器位定义

// Bit 0: Space Type
#define PCI_BASE_ADDRESS_SPACE_MEMORY   0x00  // Memory Space
#define PCI_BASE_ADDRESS_SPACE_IO       0x01  // I/O Space

// Bit 2-1: Memory Type (仅 Memory Space)
#define PCI_BASE_ADDRESS_MEM_TYPE_32    0x00  // 32-bit
#define PCI_BASE_ADDRESS_MEM_TYPE_64    0x04  // 64-bit
#define PCI_BASE_ADDRESS_MEM_TYPE_1M    0x02  // 1MB 以下 (废弃)

// Bit 3: Prefetchable
#define PCI_BASE_ADDRESS_MEM_PREFETCH   0x08

// 地址掩码
#define PCI_BASE_ADDRESS_MEM_MASK       (~0x0fULL)  // Memory BAR 地址部分
#define PCI_BASE_ADDRESS_IO_MASK        (~0x03ULL)  // I/O BAR 地址部分
```

### BAR 注册函数

```c
// hw/pci/pci.c
void pci_register_bar(PCIDevice *pci_dev, int region_num,
                      uint8_t type, MemoryRegion *memory)
{
    PCIIORegion *r;
    uint32_t addr;
    uint64_t wmask;
    
    assert(region_num >= 0 && region_num < PCI_NUM_REGIONS);
    
    r = &pci_dev->io_regions[region_num];
    r->addr = PCI_BAR_UNMAPPED;
    r->size = memory_region_size(memory);
    r->type = type;
    r->memory = memory;
    
    // 计算 BAR 地址（配置空间偏移）
    addr = pci_bar_address(pci_dev, region_num);
    
    // 设置写掩码（用于大小探测）
    wmask = ~(r->size - 1);  // 大小对应的掩码
    
    if (type & PCI_BASE_ADDRESS_SPACE_IO) {
        // I/O BAR
        wmask &= 0xFFFFFFFC;  // 保留低 2 位
        wmask |= PCI_BASE_ADDRESS_SPACE_IO;
    } else {
        // Memory BAR
        wmask &= 0xFFFFFFF0;  // 保留低 4 位
        wmask |= type;  // 添加类型标志
        
        if (type & PCI_BASE_ADDRESS_MEM_TYPE_64) {
            // 64-bit BAR: 设置下一个 BAR 为高 32 位
            pci_set_long(pci_dev->wmask + addr + 4, 0xFFFFFFFF);
        }
    }
    
    pci_set_long(pci_dev->wmask + addr, wmask);
    pci_set_long(pci_dev->config + addr, type);
}
```

### BAR 地址映射

```c
// hw/pci/pci.c
static void pci_update_mappings(PCIDevice *d)
{
    PCIIORegion *r;
    int i;
    pcibus_t new_addr;
    
    for (i = 0; i < PCI_NUM_REGIONS; i++) {
        r = &d->io_regions[i];
        
        // 跳过未使用的 BAR
        if (r->size == 0) {
            continue;
        }
        
        // 读取配置空间中的 BAR 地址
        new_addr = pci_bar_address(d, i, r->type, r->size);
        
        // 如果地址改变，重新映射
        if (new_addr != r->addr) {
            if (r->addr != PCI_BAR_UNMAPPED) {
                // 取消旧映射
                memory_region_del_subregion(r->address_space, r->memory);
            }
            
            r->addr = new_addr;
            
            if (r->addr != PCI_BAR_UNMAPPED) {
                // 建立新映射
                memory_region_add_subregion_overlap(
                    r->address_space, r->addr, r->memory, 1);
            }
        }
    }
}
```

### BAR 访问示例

```c
// NVMe 驱动中的 BAR 映射
static int nvme_probe(struct pci_dev *pdev,
                     const struct pci_device_id *id)
{
    struct nvme_dev *dev;
    int result;
    
    // 启用设备
    result = pci_enable_device_mem(pdev);
    if (result)
        return result;
    
    // 请求 BAR 0 的内存区域
    result = pci_request_mem_regions(pdev, "nvme");
    if (result)
        goto disable;
    
    // 映射 BAR 0 到虚拟地址空间
    dev->bar = ioremap(pci_resource_start(pdev, 0),
                       pci_resource_len(pdev, 0));
    if (!dev->bar) {
        result = -ENOMEM;
        goto release;
    }
    
    // 读取控制器寄存器
    u64 cap = readq(&dev->bar->cap);  // Capabilities
    u32 vs = readl(&dev->bar->vs);    // Version
    
    pr_info("NVMe controller CAP=0x%llx, VS=0x%x\n", cap, vs);
    
    // ... 设备初始化 ...
    
    return 0;
    
release:
    pci_release_mem_regions(pdev);
disable:
    pci_disable_device(pdev);
    return result;
}
```

---

## 实用技巧

### 查看设备 BAR 信息

**使用 lspci**：
```bash
# 查看所有 BAR
lspci -vvv -s 01:00.0 | grep -A20 "Region"
  Region 0: Memory at f8000000 (64-bit, non-prefetchable) [size=16K]
  Region 4: Memory at f8100000 (64-bit, prefetchable) [size=1M]

# 详细解读:
# - Region 0 (BAR 0): 
#   地址 0xf8000000, 64-bit, Non-prefetchable, 16 KB
#   → 控制寄存器
# - Region 4 (BAR 4):
#   地址 0xf8100000, 64-bit, Prefetchable, 1 MB
#   → CMB 缓冲区
```

**使用 setpci 读取原始值**：
```bash
# 读取 BAR 0
setpci -s 01:00.0 10.l
  f800000c
  # Bit 0 = 0 → Memory
  # Bit 2-1 = 10b → 64-bit
  # Bit 3 = 1 → Prefetchable
  # Address = 0xf8000000

# 读取 BAR 1 (BAR 0 的高 32 位)
setpci -s 01:00.0 14.l
  00000000

# 探测 BAR 0 大小
setpci -s 01:00.0 10.l=ffffffff  # 写全 1
setpci -s 01:00.0 10.l            # 读回
  ffffc00c
  # 可配置位: 0xffffc000
  # 大小: ~0xffffc000 + 1 = 0x4000 = 16 KB
setpci -s 01:00.0 10.l=f800000c  # 恢复原值
```

**查看内核资源分配**：
```bash
# /proc/iomem 显示物理地址分配
cat /proc/iomem | grep -A5 nvme
  f8000000-f8003fff : 0000:01:00.0 (BAR 0)
    f8000000-f8003fff : nvme
  f8100000-f81fffff : 0000:01:00.0 (BAR 4)

# /sys 接口
cat /sys/bus/pci/devices/0000:01:00.0/resource
  0x00000000f8000000 0x00000000f8003fff 0x0000000000140204
  0x0000000000000000 0x0000000000000000 0x0000000000000000
  0x0000000000000000 0x0000000000000000 0x0000000000000000
  0x0000000000000000 0x0000000000000000 0x0000000000000000
  0x00000000f8100000 0x00000000f81fffff 0x0000000000140204
  # 格式: start end flags
  # flags: 见 include/linux/ioport.h
```

### 手动重新分配 BAR

**通过 sysfs**（需要 root 权限，危险操作）：
```bash
# 1. 移除设备驱动
echo "0000:01:00.0" > /sys/bus/pci/drivers/nvme/unbind

# 2. 重新分配资源
echo 1 > /sys/bus/pci/devices/0000:01:00.0/remove
echo 1 > /sys/bus/pci/rescan

# 3. 重新绑定驱动
echo "0000:01:00.0" > /sys/bus/pci/drivers/nvme/bind
```

### 调试 BAR 映射问题

**检查 Command 寄存器**：
```bash
# 确保 Memory Space 已启用
setpci -s 01:00.0 04.w
  0547
  # Bit 1 = 1 → Memory Space Enable
  # Bit 2 = 1 → Bus Master Enable

# 如果 Bit 1 = 0，手动启用:
setpci -s 01:00.0 04.w=0547
```

**检查 BAR 冲突**：
```bash
dmesg | grep -i "resource conflict\|BAR"
  pci 0000:01:00.0: BAR 0: assigned [mem 0xf8000000-0xf8003fff 64bit]
  pci 0000:01:00.0: BAR 4: assigned [mem 0xf8100000-0xf81fffff 64bit pref]
```

---

## 总结

### 关键要点

1. ✅ **BARs 提供设备到系统地址空间的映射**，支持 Memory 和 I/O 两种类型
2. ✅ **32-bit BAR** 最大 4 GB，**64-bit BAR** 支持全地址空间，占用两个 BAR 槽位
3. ✅ **Prefetchable** 标记影响 CPU Cache 行为：数据缓冲区可预取，控制寄存器不可预取
4. ✅ **BAR 大小探测**通过写全 1 读回实现，设备硬连线低位为 0
5. ✅ **BIOS/OS 负责分配** BAR 地址，设备通过配置空间声明需求
6. ✅ **FEMU 通过 pci_register_bar()** 注册 BAR，支持动态内存映射

**设计建议**：
- 寄存器：使用 Non-Prefetchable Memory BAR
- DMA 缓冲区：使用 Prefetchable Memory BAR
- 大于 4 GB 或需要高地址：使用 64-bit BAR
- 避免使用 I/O BAR（已过时）

---

## 下一步学习

- [配置空间详解](config-space.md) - 完整配置空间布局
- [DMA 映射](../advanced-features/dma.md) - 如何使用 BAR 进行 DMA
- [MSI-X 配置](../interrupts/msix.md) - MSI-X Table 通过 BAR 访问
- [内存映射 I/O](mmio.md) - BAR 的使用细节

---

## 参考资料

- **规范**：PCIe Base Spec Chapter 7.5 (Base Address Registers)
- **代码**：`hw/pci/pci.c` (FEMU BAR 实现)
- **NVMe 示例**：`hw/block/nvme.c` (NVMe BAR 配置)
- **Linux**：`drivers/pci/setup-res.c` (BAR 分配)

---

**相关页面**：
- [← 配置空间概述](overview.md)
- [Capabilities →](capabilities.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
