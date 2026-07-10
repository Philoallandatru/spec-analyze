# PCIe 设备枚举

## 概述

PCIe 设备枚举是系统启动时由 BIOS/UEFI 或操作系统执行的过程，用于发现总线上的所有设备、分配资源（内存地址、I/O 地址、中断）并建立设备树结构。

## 枚举过程概览

```
系统启动
    |
    v
+------------------+
| 1. 链路训练      |  LTSSM 完成，链路进入 L0
+------------------+
    |
    v
+------------------+
| 2. 扫描总线      |  从 Bus 0 开始递归扫描
+------------------+
    |
    v
+------------------+
| 3. 读取配置空间  |  Vendor ID, Device ID, Class Code
+------------------+
    |
    v
+------------------+
| 4. 分配 BDF      |  Bus:Device.Function
+------------------+
    |
    v
+------------------+
| 5. 配置 Bridge   |  设置 Primary/Secondary/Subordinate Bus
+------------------+
    |
    v
+------------------+
| 6. 资源分配      |  BAR, IRQ, Bus Number
+------------------+
    |
    v
+------------------+
| 7. 启用设备      |  设置 Command Register
+------------------+
    |
    v
设备可用
```

## BDF 地址

### 地址格式

```
+---------+---------+-----------+
|  Bus    | Device  | Function  |
|  8 bits |  5 bits |  3 bits   |
+---------+---------+-----------+
    |         |          |
    v         v          v
  0-255     0-31       0-7

示例：01:00.0
- Bus = 1
- Device = 0
- Function = 0

完整地址：Domain:Bus:Device.Function
例如：0000:01:00.0
```

### 配置空间访问

**传统 PCI 配置机制（IO 端口）**：
```
CONFIG_ADDRESS (0xCF8):  选择配置空间寄存器
CONFIG_DATA (0xCFC):     读写数据

CONFIG_ADDRESS 格式：
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|31|30|29|28|27|26|25|24|23|22|21|20|19|18|17|16|
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|En| Reserved          |  Bus Number[7:0]       |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|15|14|13|12|11|10| 9| 8| 7| 6| 5| 4| 3| 2| 1| 0|
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|Device[4:0]|Func[2:0]|  Register[7:2]  |0 0|
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```

**PCIe 增强配置机制（MMIO）**：
```
ECAM (Enhanced Configuration Access Mechanism)

基地址：从 ACPI MCFG 表获取
配置空间地址 = Base + (Bus << 20) + (Device << 15) + (Function << 12) + Offset

示例：
Base = 0xE0000000
访问 Bus 1, Device 0, Function 0, Offset 0x10:
地址 = 0xE0000000 + (1 << 20) + (0 << 15) + (0 << 12) + 0x10
     = 0xE0100010
```

## 扫描算法

### 深度优先搜索

```python
def scan_bus(bus_num):
    for device in range(32):
        for function in range(8):
            vendor_id = read_config(bus_num, device, function, 0x00)
            
            # 检查设备是否存在
            if vendor_id == 0xFFFF:
                if function == 0:
                    break  # 该 device 不存在
                continue   # 该 function 不存在
            
            # 读取设备信息
            device_id = read_config(bus_num, device, function, 0x02)
            header_type = read_config(bus_num, device, function, 0x0E)
            
            print(f"Found device: {bus_num:02x}:{device:02x}.{function}")
            
            # 如果是 Bridge，递归扫描下级总线
            if (header_type & 0x7F) == 0x01:  # PCI-to-PCI Bridge
                secondary_bus = allocate_bus_number()
                configure_bridge(bus_num, device, function, secondary_bus)
                scan_bus(secondary_bus)
            
            # 如果不是多功能设备，跳过其他 function
            if function == 0 and (header_type & 0x80) == 0:
                break

def configure_bridge(bus, device, function, secondary_bus):
    # 设置 Primary Bus Number
    write_config(bus, device, function, 0x18, bus)
    
    # 设置 Secondary Bus Number
    write_config(bus, device, function, 0x19, secondary_bus)
    
    # 设置 Subordinate Bus Number (临时设置为 0xFF)
    write_config(bus, device, function, 0x1A, 0xFF)
    
    # 扫描完成后更新 Subordinate Bus Number
```

### 配置 Type 0 vs Type 1

**Type 0 Configuration Request**：
- 目标设备在**当前总线**上
- 直接通过 IDSEL 信号选择设备

**Type 1 Configuration Request**：
- 目标设备在**下游总线**上
- 通过 Bridge 转发
- Bridge 根据 Bus Number 范围决定是否转发

```
Root Complex (Bus 0)
    |
    +-- Type 0 --> Device on Bus 0
    |
    +-- Bridge --> Type 1
         |
         +-- Type 0 --> Device on Bus 1
```

## Bridge 配置

### Bridge 配置空间

```
+0x00: Vendor ID / Device ID
+0x04: Command / Status
+0x08: Revision ID / Class Code
+0x0C: Cache Line Size / Latency Timer / Header Type / BIST

Bridge 特有部分：
+0x18: Primary Bus Number
+0x19: Secondary Bus Number
+0x1A: Subordinate Bus Number
+0x1B: Secondary Latency Timer

+0x1C: I/O Base / I/O Limit
+0x20: Memory Base / Memory Limit
+0x24: Prefetchable Memory Base / Prefetchable Memory Limit

+0x28: Prefetchable Base Upper 32 Bits
+0x2C: Prefetchable Limit Upper 32 Bits

+0x30: I/O Base Upper 16 Bits / I/O Limit Upper 16 Bits
+0x34: Capability Pointer
+0x38: Expansion ROM Base Address
+0x3C: Interrupt Line / Interrupt Pin / Bridge Control
```

### Bus Number 分配示例

```
+----------------+
| Root Complex  |
| Bus 0         |
+----------------+
    |
    +-- Switch (Bus 1)
         |
         +-- Upstream Port (01:00.0)
         |   Primary Bus: 0
         |   Secondary Bus: 1
         |   Subordinate Bus: 5
         |
         +-- Downstream Port 1 (02:00.0)
         |   Primary Bus: 1
         |   Secondary Bus: 2
         |   Subordinate Bus: 2
         |   |
         |   +-- Endpoint (02:00.0)
         |
         +-- Downstream Port 2 (02:01.0)
             Primary Bus: 1
             Secondary Bus: 3
             Subordinate Bus: 5
             |
             +-- Bridge (03:00.0)
                 Primary Bus: 3
                 Secondary Bus: 4
                 Subordinate Bus: 5
                 |
                 +-- Endpoint 1 (04:00.0)
                 +-- Endpoint 2 (04:01.0)
```

## 资源分配

### BAR 大小探测

```c
// 探测 BAR 大小
uint32_t size = probe_bar_size(bus, device, function, bar_offset) {
    uint32_t original, mask;
    
    // 1. 读取原始值
    original = read_config_dword(bus, device, function, bar_offset);
    
    // 2. 写入全 1
    write_config_dword(bus, device, function, bar_offset, 0xFFFFFFFF);
    
    // 3. 读回值
    mask = read_config_dword(bus, device, function, bar_offset);
    
    // 4. 恢复原始值
    write_config_dword(bus, device, function, bar_offset, original);
    
    // 5. 计算大小
    if (mask == 0 || mask == 0xFFFFFFFF) {
        return 0;  // BAR 未实现
    }
    
    // 清除类型位
    if (mask & 0x01) {
        // I/O Space BAR
        mask &= 0xFFFFFFFC;
    } else {
        // Memory Space BAR
        mask &= 0xFFFFFFF0;
    }
    
    // 大小 = ~mask + 1
    return ~mask + 1;
}
```

### 内存窗口分配

```
Root Complex
    |
    +-- Bridge 1 (Bus 0 -> Bus 1-3)
    |   Memory Window: 0xF8000000 - 0xFBFFFFFF (64 MB)
    |
    |   +-- Device 1 (Bus 1)
    |   |   BAR0: 0xF8000000 - 0xF8FFFFFF (16 MB)
    |   |
    |   +-- Device 2 (Bus 2)
    |       BAR0: 0xF9000000 - 0xF9FFFFFF (16 MB)
    |
    +-- Bridge 2 (Bus 0 -> Bus 4-5)
        Memory Window: 0xFC000000 - 0xFDFFFFFF (32 MB)
        
        +-- Device 3 (Bus 4)
            BAR0: 0xFC000000 - 0xFC00FFFF (64 KB)
            BAR1: 0xFC010000 - 0xFC01FFFF (64 KB)
```

### 资源分配算法

```c
struct resource_range {
    uint64_t start;
    uint64_t end;
    uint64_t available;
};

void allocate_resources(struct pci_device *dev) {
    struct resource_range mem_range = {
        .start = 0xF8000000,
        .end = 0xFBFFFFFF,
        .available = 0xF8000000
    };
    
    // 1. 探测所有 BAR
    for (int i = 0; i < 6; i++) {
        uint32_t bar_offset = 0x10 + i * 4;
        uint32_t size = probe_bar_size(dev->bus, dev->device, 
                                        dev->function, bar_offset);
        
        if (size == 0) continue;
        
        // 2. 对齐地址
        uint64_t aligned_addr = ALIGN_UP(mem_range.available, size);
        
        // 3. 检查是否有足够空间
        if (aligned_addr + size > mem_range.end) {
            printf("ERROR: Not enough memory space!\n");
            return;
        }
        
        // 4. 分配地址
        write_config_dword(dev->bus, dev->device, dev->function,
                           bar_offset, aligned_addr);
        
        printf("BAR%d: 0x%08lx - 0x%08lx (%lu KB)\n",
               i, aligned_addr, aligned_addr + size - 1, size / 1024);
        
        // 5. 更新可用地址
        mem_range.available = aligned_addr + size;
        
        // 6. 如果是 64-bit BAR，跳过下一个 BAR
        uint32_t bar = read_config_dword(dev->bus, dev->device,
                                          dev->function, bar_offset);
        if ((bar & 0x06) == 0x04) {
            i++;  // 64-bit BAR 占用两个 BAR 寄存器
        }
    }
}
```

## 中断路由

### Legacy INTx

```
+----------+     INTA#     +--------+     IRQ 11
| Device 1 | ------------> |        |------------>
+----------+               |        |
                           | Bridge |
+----------+     INTB#     |        |     IRQ 10
| Device 2 | ------------> |        |------------>
+----------+               +--------+

中断路由表：
Device 1, INTA# --> Bridge INTA# --> IRQ 11
Device 2, INTB# --> Bridge INTB# --> IRQ 10
```

### MSI/MSI-X

PCIe 设备优先使用 MSI/MSI-X，无需路由表：
- MSI/MSI-X 通过 Memory Write TLP 实现
- 设备直接写入中断控制器的地址
- 无需硬件中断线

## FEMU 实现

### 设备枚举模拟

```c
// hw/pci/pci.c
static void pci_bus_reset(PCIBus *bus)
{
    PCIDevice *dev;
    
    // 重置总线上所有设备
    QLIST_FOREACH(dev, &bus->children, sibling) {
        pci_do_device_reset(dev);
    }
}

// 设备注册
int pci_register_device(PCIDevice *pci_dev, PCIBus *bus,
                         const char *name, int devfn)
{
    // 分配 devfn (Device << 3 | Function)
    if (devfn < 0) {
        for (devfn = bus->devfn_min; devfn < ARRAY_SIZE(bus->devices);
             devfn += PCI_FUNC_MAX) {
            if (!bus->devices[devfn])
                goto found;
        }
        return -ENOSPC;
    }
    
found:
    pci_dev->bus = bus;
    pci_dev->devfn = devfn;
    bus->devices[devfn] = pci_dev;
    
    return 0;
}
```

### BAR 注册

```c
// hw/block/nvme.c
static void nvme_realize(PCIDevice *pci_dev, Error **errp)
{
    NvmeCtrl *n = NVME(pci_dev);
    
    // 初始化 Memory Region
    memory_region_init_io(&n->iomem, OBJECT(n), &nvme_mmio_ops, n,
                          "nvme", n->reg_size);
    
    // 注册 BAR0
    pci_register_bar(pci_dev, 0,
                     PCI_BASE_ADDRESS_SPACE_MEMORY |
                     PCI_BASE_ADDRESS_MEM_TYPE_64,
                     &n->iomem);
}

void pci_register_bar(PCIDevice *pci_dev, int region_num,
                      uint8_t type, MemoryRegion *memory)
{
    PCIIORegion *r = &pci_dev->io_regions[region_num];
    uint32_t addr;
    uint64_t wmask;
    
    r->addr = PCI_BAR_UNMAPPED;
    r->size = memory_region_size(memory);
    r->type = type;
    r->memory = memory;
    
    // 设置 BAR 寄存器的可写掩码
    wmask = ~(r->size - 1);
    
    if (region_num == PCI_ROM_SLOT) {
        addr = PCI_ROM_ADDRESS;
        wmask |= PCI_ROM_ADDRESS_ENABLE;
    } else {
        addr = PCI_BASE_ADDRESS_0 + region_num * 4;
    }
    
    pci_set_long(pci_dev->config + addr, type);
    pci_set_long(pci_dev->wmask + addr, wmask & 0xffffffff);
    
    // 64-bit BAR
    if (type & PCI_BASE_ADDRESS_MEM_TYPE_64) {
        pci_set_long(pci_dev->wmask + addr + 4, wmask >> 32);
    }
}
```

### 配置空间访问

```c
// hw/pci/pci_host.c
uint32_t pci_host_config_read_common(PCIDevice *pci_dev,
                                      uint32_t addr, uint32_t limit,
                                      int len)
{
    uint32_t ret;
    
    ret = pci_read_config(pci_dev, addr, len);
    trace_pci_cfg_read(pci_dev->name, PCI_SLOT(pci_dev->devfn),
                       PCI_FUNC(pci_dev->devfn), addr, ret);
    
    return ret;
}

void pci_host_config_write_common(PCIDevice *pci_dev,
                                   uint32_t addr, uint32_t limit,
                                   uint32_t val, int len)
{
    trace_pci_cfg_write(pci_dev->name, PCI_SLOT(pci_dev->devfn),
                        PCI_FUNC(pci_dev->devfn), addr, val);
    
    pci_write_config(pci_dev, addr, val, len);
}
```

## Linux 枚举流程

### 内核代码路径

```c
// drivers/pci/probe.c

// 主入口
pci_scan_root_bus()
  |
  +-> pci_scan_child_bus()
       |
       +-> pci_scan_slot()
            |
            +-> pci_scan_single_device()
                 |
                 +-> pci_scan_device()
                 |    |
                 |    +-> pci_bus_read_config_word()  // 读取 Vendor ID
                 |    +-> pci_setup_device()
                 |         |
                 |         +-> pci_read_bases()      // 读取 BAR
                 |
                 +-> pci_device_add()
                      |
                      +-> device_add()               // 注册设备

// 资源分配
pci_assign_unassigned_resources()
  |
  +-> pci_assign_unassigned_bridge_resources()
       |
       +-> __pci_bus_assign_resources()
            |
            +-> pci_assign_resource()
```

### 查看枚举结果

```bash
# 列出所有 PCIe 设备
lspci -tv

# 查看详细配置
lspci -vv -s 01:00.0

# 查看配置空间
lspci -xxx -s 01:00.0

# 查看系统资源分配
cat /proc/iomem
cat /proc/ioports

# 查看设备树
ls -l /sys/bus/pci/devices/
ls -l /sys/devices/pci0000:00/
```

## 热插拔枚举

### Hot-Plug 流程

```
1. 检测设备插入
    |
    v
2. Power On Slot
    |
    v
3. 等待链路训练完成 (LTSSM -> L0)
    |
    v
4. 分配 Bus Number (如果是 Bridge)
    |
    v
5. 读取配置空间
    |
    v
6. 分配资源 (BAR, IRQ)
    |
    v
7. 加载驱动
    |
    v
8. 设备可用
```

### Native PCIe Hot-Plug

```c
// drivers/pci/hotplug/pciehp_ctrl.c
static int pciehp_enable_slot(struct slot *slot)
{
    struct controller *ctrl = slot->ctrl;
    int ret;
    
    // 1. Power on
    pciehp_power_on_slot(slot);
    
    // 2. 等待链路建立
    pciehp_check_link_status(ctrl);
    
    // 3. 枚举设备
    ret = pci_scan_slot(ctrl->pcie->port->subordinate, 
                        PCI_DEVFN(0, 0));
    
    // 4. 分配资源
    pci_assign_unassigned_bridge_resources(ctrl->pcie->port);
    
    // 5. 启动设备
    pci_bus_add_devices(ctrl->pcie->port->subordinate);
    
    return 0;
}
```

## 调试技巧

### BIOS/UEFI 日志

```
POST Code: Enumerating PCI devices...
Bus 0, Device 0, Function 0: 8086:1910 (Host Bridge)
Bus 0, Device 28, Function 0: 8086:A110 (PCIe Root Port)
  Assigning Bus 1
  Bus 1, Device 0, Function 0: 144D:A802 (NVMe Controller)
    BAR0: 64-bit, size=16KB
    Assigned: 0xF8000000-0xF8003FFF
    MSI-X: 8 vectors
```

### 内核参数

```bash
# 详细 PCI 调试信息
pci=debug

# 禁用资源重分配
pci=nocrs

# 使用传统配置机制
pci=conf1
```

## 总结

PCIe 设备枚举的关键步骤：

1. **链路训练**：LTSSM 完成，链路进入 L0 状态
2. **总线扫描**：深度优先搜索，递归扫描所有总线
3. **BDF 分配**：为每个设备分配唯一的 Bus:Device.Function
4. **Bridge 配置**：设置 Primary/Secondary/Subordinate Bus Number
5. **资源分配**：探测 BAR 大小，分配内存/IO 地址
6. **中断配置**：设置 MSI/MSI-X 或路由 Legacy INTx
7. **启用设备**：设置 Command Register，设备开始工作

参见：
- [PCIe 拓扑结构](../introduction/topology.md)
- [配置空间](../configuration/config-space.md)
- [BAR 寄存器](../configuration/bars.md)
- [热插拔](../advanced-features/hot-plug.md)
