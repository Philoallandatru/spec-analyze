# PCIe 配置空间实现

## 概述

PCIe 配置空间的实现涉及硬件设计、固件初始化和软件访问三个层面。本文介绍如何在实际系统中实现和访问 PCIe 配置空间。

## 配置空间结构回顾

### 标准配置空间（256 字节）

```
+0x00: Device ID / Vendor ID
+0x04: Status / Command
+0x08: Class Code / Revision ID
+0x0C: BIST / Header Type / Latency Timer / Cache Line Size
+0x10: BAR0
+0x14: BAR1
+0x18: BAR2
+0x1C: BAR3
+0x20: BAR4
+0x24: BAR5
+0x28: Cardbus CIS Pointer
+0x2C: Subsystem ID / Subsystem Vendor ID
+0x30: Expansion ROM Base Address
+0x34: Reserved / Capabilities Pointer
+0x38: Reserved
+0x3C: Max_Lat / Min_Gnt / Interrupt Pin / Interrupt Line
+0x40-0xFF: Capability Structures
```

### 扩展配置空间（4096 字节）

```
+0x000-0x0FF: 标准配置空间
+0x100-0xFFF: 扩展 Capability 结构
```

## 硬件实现

### FPGA/ASIC 实现

```verilog
// PCIe 配置空间寄存器实现
module pcie_config_space (
    input wire clk,
    input wire rst_n,
    
    // 配置空间访问接口
    input wire [11:0] cfg_addr,
    input wire [3:0]  cfg_be,
    input wire        cfg_wr,
    input wire        cfg_rd,
    input wire [31:0] cfg_wdata,
    output reg [31:0] cfg_rdata,
    output reg        cfg_ack
);

    // 配置寄存器
    reg [15:0] vendor_id;
    reg [15:0] device_id;
    reg [15:0] command;
    reg [15:0] status;
    reg [7:0]  revision_id;
    reg [23:0] class_code;
    reg [31:0] bar[0:5];
    
    // 只读寄存器初始化
    initial begin
        vendor_id = 16'h8086;
        device_id = 16'h5845;
        revision_id = 8'h01;
        class_code = 24'h010802;  // NVM Express
    end
    
    // 配置空间读写
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            command <= 16'h0000;
            status <= 16'h0010;  // Capabilities List
            bar[0] <= 32'h00000004;  // 64-bit, Non-Prefetchable
            bar[1] <= 32'h00000000;
            bar[2] <= 32'h00000000;
            bar[3] <= 32'h00000000;
            bar[4] <= 32'h00000000;
            bar[5] <= 32'h00000000;
            cfg_rdata <= 32'h0;
            cfg_ack <= 1'b0;
        end else begin
            cfg_ack <= cfg_rd | cfg_wr;
            
            if (cfg_rd) begin
                case (cfg_addr[11:2])
                    10'h000: cfg_rdata <= {device_id, vendor_id};
                    10'h001: cfg_rdata <= {status, command};
                    10'h002: cfg_rdata <= {class_code, revision_id};
                    10'h004: cfg_rdata <= bar[0];
                    10'h005: cfg_rdata <= bar[1];
                    10'h006: cfg_rdata <= bar[2];
                    10'h007: cfg_rdata <= bar[3];
                    10'h008: cfg_rdata <= bar[4];
                    10'h009: cfg_rdata <= bar[5];
                    10'h00D: cfg_rdata <= {24'h000000, 8'h40};  // Cap Pointer
                    default: cfg_rdata <= 32'h00000000;
                endcase
            end
            
            if (cfg_wr) begin
                case (cfg_addr[11:2])
                    10'h001: begin
                        if (cfg_be[0]) command[7:0] <= cfg_wdata[7:0];
                        if (cfg_be[1]) command[15:8] <= cfg_wdata[15:8];
                    end
                    10'h004: bar[0] <= cfg_wdata & 32'hFFFFC004;
                    10'h005: bar[1] <= cfg_wdata;
                    10'h006: bar[2] <= cfg_wdata;
                    10'h007: bar[3] <= cfg_wdata;
                    10'h008: bar[4] <= cfg_wdata;
                    10'h009: bar[5] <= cfg_wdata;
                endcase
            end
        end
    end

endmodule
```

### BAR 大小硬编码

```verilog
// BAR 大小掩码实现
module bar_sizing (
    input wire [31:0] cfg_wdata,
    input wire [31:0] bar_size_mask,
    output wire [31:0] bar_value
);

    // BAR0: 64-bit, 16KB
    parameter BAR0_SIZE_MASK = 32'hFFFFC004;  // ~(16KB-1) | Type
    
    // 写入时应用掩码
    assign bar_value = cfg_wdata & BAR0_SIZE_MASK;

endmodule
```

## 软件访问方法

### 1. Legacy 方法（IO 端口）

```c
// x86 架构特有方法
#define CONFIG_ADDRESS  0xCF8
#define CONFIG_DATA     0xCFC

uint32_t pci_config_read_legacy(uint8_t bus, uint8_t device,
                                 uint8_t function, uint8_t offset)
{
    uint32_t address;
    
    // 构造配置地址
    address = (1 << 31) |                    // Enable bit
              (bus << 16) |
              (device << 11) |
              (function << 8) |
              (offset & 0xFC);
    
    // 写配置地址寄存器
    outl(address, CONFIG_ADDRESS);
    
    // 读配置数据寄存器
    return inl(CONFIG_DATA);
}

void pci_config_write_legacy(uint8_t bus, uint8_t device,
                              uint8_t function, uint8_t offset,
                              uint32_t value)
{
    uint32_t address;
    
    address = (1 << 31) |
              (bus << 16) |
              (device << 11) |
              (function << 8) |
              (offset & 0xFC);
    
    outl(address, CONFIG_ADDRESS);
    outl(value, CONFIG_DATA);
}
```

### 2. ECAM 方法（MMIO）

```c
// PCIe 增强配置访问机制
struct ecam_region {
    void __iomem *base;
    uint8_t start_bus;
    uint8_t end_bus;
};

// 计算 ECAM 地址
static inline void __iomem *ecam_addr(struct ecam_region *ecam,
                                       uint8_t bus, uint8_t device,
                                       uint8_t function, uint16_t offset)
{
    // 地址 = Base + (Bus << 20) + (Device << 15) + (Function << 12) + Offset
    return ecam->base + 
           ((bus - ecam->start_bus) << 20) +
           (device << 15) +
           (function << 12) +
           offset;
}

uint32_t pci_config_read_ecam(struct ecam_region *ecam,
                               uint8_t bus, uint8_t device,
                               uint8_t function, uint16_t offset)
{
    void __iomem *addr;
    
    if (bus < ecam->start_bus || bus > ecam->end_bus)
        return 0xFFFFFFFF;
    
    addr = ecam_addr(ecam, bus, device, function, offset);
    return readl(addr);
}

void pci_config_write_ecam(struct ecam_region *ecam,
                            uint8_t bus, uint8_t device,
                            uint8_t function, uint16_t offset,
                            uint32_t value)
{
    void __iomem *addr;
    
    if (bus < ecam->start_bus || bus > ecam->end_bus)
        return;
    
    addr = ecam_addr(ecam, bus, device, function, offset);
    writel(value, addr);
}
```

### 3. Linux 内核 API

```c
// 使用内核提供的配置空间访问函数
int pci_read_config_byte(struct pci_dev *dev, int where, u8 *val);
int pci_read_config_word(struct pci_dev *dev, int where, u16 *val);
int pci_read_config_dword(struct pci_dev *dev, int where, u32 *val);

int pci_write_config_byte(struct pci_dev *dev, int where, u8 val);
int pci_write_config_word(struct pci_dev *dev, int where, u16 val);
int pci_write_config_dword(struct pci_dev *dev, int where, u32 val);

// 示例：读取设备 ID
u16 vendor_id, device_id;
pci_read_config_word(pdev, PCI_VENDOR_ID, &vendor_id);
pci_read_config_word(pdev, PCI_DEVICE_ID, &device_id);

// 启用 Bus Master
u16 command;
pci_read_config_word(pdev, PCI_COMMAND, &command);
command |= PCI_COMMAND_MASTER | PCI_COMMAND_MEMORY;
pci_write_config_word(pdev, PCI_COMMAND, command);
```

## 用户空间访问

### 1. sysfs 接口

```bash
# 读取配置空间
hexdump -C /sys/bus/pci/devices/0000:01:00.0/config

# 读取特定偏移
dd if=/sys/bus/pci/devices/0000:01:00.0/config bs=1 skip=0 count=4 | hexdump -C

# 写入配置空间（需要 root）
echo -ne '\x07\x05' | dd of=/sys/bus/pci/devices/0000:01:00.0/config bs=1 seek=4 count=2
```

```c
// C 代码访问 sysfs
int read_config_sysfs(const char *bdf, uint16_t offset, 
                      void *buf, size_t len)
{
    char path[256];
    int fd;
    ssize_t ret;
    
    snprintf(path, sizeof(path), 
             "/sys/bus/pci/devices/%s/config", bdf);
    
    fd = open(path, O_RDONLY);
    if (fd < 0)
        return -1;
    
    lseek(fd, offset, SEEK_SET);
    ret = read(fd, buf, len);
    close(fd);
    
    return ret;
}
```

### 2. libpci 库

```c
#include <pci/pci.h>

void read_device_config(void)
{
    struct pci_access *pacc;
    struct pci_dev *dev;
    
    // 初始化 libpci
    pacc = pci_alloc();
    pci_init(pacc);
    pci_scan_bus(pacc);
    
    // 遍历所有设备
    for (dev = pacc->devices; dev; dev = dev->next) {
        pci_fill_info(dev, PCI_FILL_IDENT | PCI_FILL_BASES);
        
        printf("Device: %04x:%02x:%02x.%d\n",
               dev->domain, dev->bus, dev->dev, dev->func);
        printf("  Vendor: %04x, Device: %04x\n",
               dev->vendor_id, dev->device_id);
        
        // 读取配置空间
        u8 revision = pci_read_byte(dev, PCI_REVISION_ID);
        u16 command = pci_read_word(dev, PCI_COMMAND);
        
        printf("  Revision: %02x, Command: %04x\n",
               revision, command);
        
        // 读取 BAR
        for (int i = 0; i < 6; i++) {
            if (dev->base_addr[i]) {
                printf("  BAR%d: 0x%lx\n", i, dev->base_addr[i]);
            }
        }
    }
    
    pci_cleanup(pacc);
}
```

### 3. /dev/mem 直接访问

```c
// 通过 /dev/mem 访问 ECAM
void *map_ecam_region(uint64_t ecam_base, size_t size)
{
    int fd;
    void *addr;
    
    fd = open("/dev/mem", O_RDWR | O_SYNC);
    if (fd < 0) {
        perror("open /dev/mem");
        return NULL;
    }
    
    addr = mmap(NULL, size, PROT_READ | PROT_WRITE,
                MAP_SHARED, fd, ecam_base);
    close(fd);
    
    if (addr == MAP_FAILED) {
        perror("mmap");
        return NULL;
    }
    
    return addr;
}

// 访问配置空间
uint32_t read_config_devmem(void *ecam_base, uint8_t bus,
                             uint8_t dev, uint8_t func,
                             uint16_t offset)
{
    volatile uint32_t *addr;
    
    addr = (volatile uint32_t *)((char *)ecam_base +
                                  (bus << 20) +
                                  (dev << 15) +
                                  (func << 12) +
                                  offset);
    
    return *addr;
}
```

## FEMU 实现

### 配置空间读写

```c
// hw/pci/pci.c
uint32_t pci_default_read_config(PCIDevice *d, uint32_t address, int len)
{
    uint32_t val = 0;
    
    // 边界检查
    if (address >= pci_config_size(d)) {
        return 0;
    }
    
    // 从配置空间数组读取
    memcpy(&val, d->config + address, len);
    
    trace_pci_cfg_read(d->name, PCI_SLOT(d->devfn),
                       PCI_FUNC(d->devfn), address, val);
    
    return val;
}

void pci_default_write_config(PCIDevice *d, uint32_t addr,
                               uint32_t val, int len)
{
    int i;
    
    trace_pci_cfg_write(d->name, PCI_SLOT(d->devfn),
                        PCI_FUNC(d->devfn), addr, val);
    
    // 应用写掩码
    for (i = 0; i < len; i++) {
        uint8_t wmask = d->wmask[addr + i];
        uint8_t w1cmask = d->w1cmask[addr + i];
        
        d->config[addr + i] = 
            (d->config[addr + i] & ~wmask) |
            (val >> (i * 8) & wmask);
        
        // Write-1-to-clear 位
        d->config[addr + i] &= ~(val >> (i * 8) & w1cmask);
    }
    
    // BAR 更新检查
    if (ranges_overlap(addr, len, PCI_BASE_ADDRESS_0, 24) ||
        ranges_overlap(addr, len, PCI_ROM_ADDRESS, 4)) {
        pci_update_mappings(d);
    }
    
    // Command 寄存器更新
    if (ranges_overlap(addr, len, PCI_COMMAND, 2)) {
        pci_update_irq_status(d);
    }
}
```

### 设置只读/可写掩码

```c
// hw/block/nvme.c
static void nvme_init_pci_config(NvmeCtrl *n)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    uint8_t *pci_conf = pci_dev->config;
    
    // 设置标准配置头
    pci_config_set_vendor_id(pci_conf, PCI_VENDOR_ID_INTEL);
    pci_config_set_device_id(pci_conf, 0x5845);
    pci_config_set_class(pci_conf, PCI_CLASS_STORAGE_EXPRESS);
    pci_config_set_prog_interface(pci_conf, 0x02);
    
    // 设置可写位掩码
    pci_set_word(pci_dev->wmask + PCI_COMMAND,
                 PCI_COMMAND_IO | PCI_COMMAND_MEMORY |
                 PCI_COMMAND_MASTER | PCI_COMMAND_INTX_DISABLE);
    
    pci_set_word(pci_dev->wmask + PCI_STATUS, 0);  // Status 只读
    
    // BAR0 写掩码
    pci_set_long(pci_dev->wmask + PCI_BASE_ADDRESS_0,
                 0xFFFFC000);  // 16KB, 底 14 位只读
    pci_set_long(pci_dev->wmask + PCI_BASE_ADDRESS_1,
                 0xFFFFFFFF);  // 64-bit BAR 高 32 位
}
```

### Capability 实现

```c
// 添加 PCIe Capability
static int nvme_add_pcie_cap(PCIDevice *pci_dev)
{
    int offset;
    
    // 分配 Capability 空间
    offset = pci_add_capability(pci_dev, PCI_CAP_ID_EXP,
                                 0, PCI_EXP_SIZEOF_V2, &error_abort);
    
    // 设置 PCIe Capability 寄存器
    pci_set_word(pci_dev->config + offset + PCI_EXP_FLAGS,
                 (PCI_EXP_TYPE_ENDPOINT << 4) |  // Device Type
                 PCI_EXP_FLAGS_VER2);             // Version 2
    
    pci_set_long(pci_dev->config + offset + PCI_EXP_DEVCAP,
                 PCI_EXP_DEVCAP_RBER |           // Role-Based Error
                 PCI_EXP_DEVCAP_FLRESET |        // FLR
                 (0 << 26));                      // Max Payload: 128B
    
    pci_set_word(pci_dev->config + offset + PCI_EXP_DEVCTL, 0);
    
    // Link Capabilities
    pci_set_long(pci_dev->config + offset + PCI_EXP_LNKCAP,
                 (QEMU_PCI_EXP_LNKCAP_MLW_X4 << 4) |  // x4 width
                 QEMU_PCI_EXP_LNKCAP_MLS_8GT);         // Gen3 speed
    
    return offset;
}
```

## 配置空间虚拟化

### SR-IOV 配置

```c
// 物理功能（PF）配置空间
struct pf_config {
    uint16_t vendor_id;
    uint16_t device_id;
    // ... 标准配置空间 ...
    
    // SR-IOV Capability (Offset 0x160)
    uint16_t sriov_cap_id;      // 0x0010
    uint16_t sriov_control;
    uint16_t total_vfs;         // 总 VF 数量
    uint16_t initial_vfs;
    uint16_t num_vfs;           // 当前启用的 VF 数量
    uint32_t vf_bar[6];         // VF BAR
};

// 虚拟功能（VF）配置空间
struct vf_config {
    uint16_t vendor_id;         // 与 PF 相同
    uint16_t vf_device_id;      // VF Device ID
    // ... 简化的配置空间 ...
};
```

### VFIO 配置空间模拟

```c
// 用户空间驱动访问配置空间
ssize_t vfio_pci_read_config(int device_fd, void *buf,
                              size_t count, off_t offset)
{
    return pread(device_fd, buf, count,
                 VFIO_PCI_CONFIG_REGION_INDEX * VFIO_REGION_SIZE + offset);
}

ssize_t vfio_pci_write_config(int device_fd, const void *buf,
                               size_t count, off_t offset)
{
    return pwrite(device_fd, buf, count,
                  VFIO_PCI_CONFIG_REGION_INDEX * VFIO_REGION_SIZE + offset);
}
```

## 调试和监控

### lspci 详细输出

```bash
# 显示配置空间十六进制
lspci -xxx -s 01:00.0

# 显示完整 4096 字节配置空间
lspci -xxxx -s 01:00.0

# 解析的配置信息
lspci -vvv -s 01:00.0
```

### setpci 修改配置

```bash
# 读取 Command 寄存器
setpci -s 01:00.0 COMMAND

# 启用 Memory Space
setpci -s 01:00.0 COMMAND=0x0002

# 启用 Bus Master
setpci -s 01:00.0 COMMAND=0x0004

# 读取 BAR0
setpci -s 01:00.0 10.L

# 读取 MSI-X Capability
setpci -s 01:00.0 CAP_MSIX+0.W
```

### 内核追踪

```bash
# 追踪配置空间访问
trace-cmd record -e pci:pci_read_config -e pci:pci_write_config

# 查看追踪
trace-cmd report
```

## 安全考虑

### 配置空间保护

```c
// 限制用户空间修改关键寄存器
static bool is_writable_offset(uint16_t offset)
{
    // 允许写入的寄存器
    if (offset >= PCI_BASE_ADDRESS_0 && offset <= PCI_BASE_ADDRESS_5)
        return true;  // BAR
    
    if (offset == PCI_COMMAND)
        return true;
    
    // 禁止写入
    if (offset == PCI_VENDOR_ID || offset == PCI_DEVICE_ID)
        return false;  // 只读
    
    if (offset >= 0x100)
        return false;  // 扩展配置空间保护
    
    return false;
}
```

### IOMMU 保护

```c
// 配置 IOMMU 保护配置空间访问
int setup_iommu_protection(struct pci_dev *pdev)
{
    struct iommu_domain *domain;
    
    domain = iommu_get_domain_for_dev(&pdev->dev);
    if (!domain)
        return -ENODEV;
    
    // 限制设备访问配置空间
    iommu_set_attr(domain, DOMAIN_ATTR_PCIE_NO_SNOOP, &no_snoop);
    
    return 0;
}
```

## 总结

PCIe 配置空间实现的关键点：

1. **硬件层**：使用寄存器阵列实现，应用写掩码控制可写位
2. **访问方法**：Legacy IO 端口、ECAM MMIO、内核 API
3. **用户空间**：sysfs、libpci、/dev/mem
4. **虚拟化**：VFIO 提供用户空间直接访问
5. **安全性**：保护关键寄存器，使用 IOMMU

参见：
- [配置空间详解](../configuration/config-space.md)
- [BAR 寄存器](../configuration/bars.md)
- [设备枚举](../architecture/enumeration.md)
- [驱动开发指南](driver-guide.md)
