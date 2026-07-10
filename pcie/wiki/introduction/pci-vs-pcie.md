# PCI vs PCIe：并行到串行的演进

## 概述

PCI（Peripheral Component Interconnect）和 PCIe（PCI Express）代表了计算机扩展总线技术的两个重要时代。从 PCI 的并行架构到 PCIe 的串行架构，这一转变不仅仅是技术细节的改变，而是对现代计算需求的根本性回应。本文将详细比较这两种技术，帮助理解为什么 PCIe 成为了现代计算的基础。

## 架构对比

### PCI：共享并行总线

PCI 使用传统的并行总线架构：

```
         PCI 总线（共享）
    ═══════════════════════════════
    ║         ║         ║         ║
┌───▼───┐ ┌──▼────┐ ┌──▼────┐ ┌──▼────┐
│ 网卡  │ │ 声卡  │ │ GPU   │ │ SCSI  │
└───────┘ └───────┘ └───────┘ └───────┘

特点：
- 32/64 位并行数据线
- 所有设备共享同一总线
- 半双工通信
- 总线仲裁机制
```

**PCI 总线信号：**
- **地址/数据复用**：AD[31:0] 或 AD[63:0]
- **控制信号**：FRAME#, IRDY#, TRDY#, DEVSEL#
- **仲裁信号**：REQ#, GNT#
- **中断信号**：INTA#, INTB#, INTC#, INTD#
- **时钟**：单一共享时钟（33/66 MHz）

### PCIe：点对点串行链路

PCIe 采用现代的串行点对点架构：

```
        Root Complex
            │
    ┌───────┼───────┬──────┐
    │       │       │      │
┌───▼──┐ ┌──▼──┐ ┌──▼──┐ ┌─▼────┐
│ 网卡 │ │ GPU │ │ SSD │ │Switch│
└──────┘ └─────┘ └─────┘ └──┬───┘
                            │
                        ┌───▼────┐
                        │ 声卡   │
                        └────────┘

特点：
- 串行差分信号对（Lane）
- 点对点专用连接
- 全双工通信
- 无需总线仲裁
```

**PCIe Lane 信号：**
- **发送差分对**：TX+, TX-
- **接收差分对**：RX+, RX-
- **时钟**：嵌入式时钟（无需独立时钟线）
- **边带信号**：PERST#, WAKE#, CLKREQ#

## 关键技术差异

### 1. 并行 vs 串行

#### PCI 并行传输的挑战

```
PCI 33MHz, 32-bit:
Clock  ┐   ┌───┐   ┌───┐   ┌───┐   ┌───
       └───┘   └───┘   └───┘   └───┘
       
Data[0]────[Bit 0]────────────────────
Data[1]────[Bit 1]────────────────────
Data[2]────[Bit 2]────────────────────
...
Data[31]───[Bit 31]───────────────────

问题：
1. 时钟偏斜（Clock Skew）
2. 信号完整性问题
3. 电磁干扰（EMI）
4. 频率提升困难
```

#### PCIe 串行传输的优势

```
PCIe Gen1 (2.5 GT/s), 1 Lane:
       
TX+/TX- ═══╤═══╤═══╤═══╤═══╤═══╤═══
           │ 1 │ 0 │ 1 │ 1 │ 0 │ 1 │
           └───┴───┴───┴───┴───┴───┘

特点：
1. 差分信号抗干扰
2. 嵌入式时钟（8b/10b 编码）
3. 易于提升频率
4. 可扩展 Lane 数量
```

### 2. 带宽对比

#### PCI 带宽演进

| 版本 | 时钟频率 | 数据宽度 | 理论带宽 | 实际带宽 |
|------|----------|----------|----------|----------|
| PCI 2.1 | 33 MHz | 32-bit | 133 MB/s | ~100 MB/s |
| PCI 2.2 | 66 MHz | 32-bit | 266 MB/s | ~200 MB/s |
| PCI 2.3 | 66 MHz | 64-bit | 533 MB/s | ~400 MB/s |
| PCI-X | 133 MHz | 64-bit | 1066 MB/s | ~800 MB/s |

**PCI 带宽瓶颈：**
- 所有设备共享带宽
- 半双工通信（不能同时读写）
- 总线仲裁开销
- 频率难以继续提升

#### PCIe 带宽演进

| 版本 | 速率 | 编码 | x1 带宽 | x16 带宽 |
|------|------|------|---------|----------|
| PCIe 1.0 | 2.5 GT/s | 8b/10b | 250 MB/s | 4 GB/s |
| PCIe 2.0 | 5.0 GT/s | 8b/10b | 500 MB/s | 8 GB/s |
| PCIe 3.0 | 8.0 GT/s | 128b/130b | ~1 GB/s | ~16 GB/s |
| PCIe 4.0 | 16.0 GT/s | 128b/130b | ~2 GB/s | ~32 GB/s |
| PCIe 5.0 | 32.0 GT/s | 128b/130b | ~4 GB/s | ~64 GB/s |
| PCIe 6.0 | 64.0 GT/s | 242b/256b | ~8 GB/s | ~128 GB/s |

**PCIe 带宽优势：**
- 每个设备独享带宽
- 全双工通信
- 易于扩展（增加 Lane）
- 持续的代际提升

### 3. 拓扑结构

#### PCI 总线拓扑

```c
// PCI 总线枚举（共享总线）
void pci_scan_bus(struct pci_bus *bus) {
    // 所有设备在同一总线上
    for (devfn = 0; devfn < 256; devfn++) {
        // 尝试读取每个设备/功能
        if (pci_bus_read_config_word(bus, devfn, 
                                     PCI_VENDOR_ID, &vendor) == 0) {
            if (vendor != 0xFFFF) {
                // 发现设备
                struct pci_dev *dev = pci_alloc_dev(bus);
                dev->devfn = devfn;
                // ...
            }
        }
    }
}
```

**PCI 总线限制：**
- 最多 32 个设备
- 单层扁平结构（或使用 Bridge 分段）
- 物理限制（插槽数量、布线复杂度）

#### PCIe 树形拓扑

```c
// PCIe 树形枚举
void pcie_scan_hierarchy(struct pci_bus *bus) {
    struct pci_dev *dev;
    
    // 扫描当前总线
    for_each_pci_bridge(dev, bus) {
        // 如果是 Bridge/Switch，递归扫描下游
        struct pci_bus *child = pci_alloc_child_bus(bus, dev);
        
        // 配置 Bridge 的总线号
        pci_write_config_byte(dev, PCI_SECONDARY_BUS, child->number);
        
        // 递归扫描子总线
        pcie_scan_hierarchy(child);
    }
}
```

**PCIe 拓扑优势：**
- 树形层次结构，易于扩展
- 使用 Switch 可连接多个设备
- 每条链路独立，无相互干扰
- 支持热插拔

### 4. 通信模式

#### PCI：基于事务的共享总线

```c
// PCI 总线事务（简化示例）
int pci_bus_transaction(struct pci_bus *bus, u32 address, u32 *data, 
                       bool is_write) {
    // 1. 请求总线（仲裁）
    if (!pci_bus_request(bus))
        return -EBUSY;
    
    // 2. 地址阶段（所有设备监听）
    pci_bus_drive_ad(bus, address);
    pci_bus_assert(bus, FRAME);
    
    // 3. 数据阶段
    if (is_write) {
        pci_bus_drive_ad(bus, *data);
        pci_bus_assert(bus, IRDY);
        pci_bus_wait(bus, TRDY);  // 等待目标就绪
    } else {
        pci_bus_assert(bus, IRDY);
        pci_bus_wait(bus, TRDY);
        *data = pci_bus_read_ad(bus);
    }
    
    // 4. 释放总线
    pci_bus_release(bus);
    
    return 0;
}
```

#### PCIe：基于包的点对点通信

```c
// PCIe TLP（Transaction Layer Packet）
struct pcie_tlp_header {
    u8 fmt:2;           // Format
    u8 type:5;          // Type
    u8 tc:3;            // Traffic Class
    u8 attr:3;          // Attributes
    u16 length:10;      // Length
    u16 requester_id;   // Requester ID
    u8 tag;             // Transaction tag
    u32 address;        // Address (for memory/IO)
};

// 发送 PCIe Memory Read 请求
int pcie_memory_read(struct pcie_port *port, u64 address, 
                    size_t length, u8 tag) {
    struct pcie_tlp tlp;
    
    // 构造 TLP 头
    tlp.header.fmt = 0b00;  // 3DW header, no data
    tlp.header.type = 0b00000;  // Memory Read
    tlp.header.length = (length + 3) / 4;  // DWORDs
    tlp.header.requester_id = port->bdf;
    tlp.header.tag = tag;
    tlp.header.address = address;
    
    // 通过数据链路层发送
    return pcie_dll_transmit(port, &tlp);
}
```

### 5. 中断机制

#### PCI：共享中断线

```c
// PCI 中断（共享 INTx）
#define PCI_INTERRUPT_LINE  0x3C
#define PCI_INTERRUPT_PIN   0x3D

void pci_setup_interrupt(struct pci_dev *dev) {
    u8 pin, line;
    
    // 读取中断引脚（INTA-INTD）
    pci_read_config_byte(dev, PCI_INTERRUPT_PIN, &pin);
    pci_read_config_byte(dev, PCI_INTERRUPT_LINE, &line);
    
    if (pin == 0) {
        dev_info(&dev->dev, "No interrupt pin\n");
        return;
    }
    
    // 注册共享中断处理程序
    request_irq(line, pci_interrupt_handler, IRQF_SHARED,
               dev_name(&dev->dev), dev);
    
    dev->irq = line;
}

// 中断处理程序需要检查是否是自己的中断
irqreturn_t pci_interrupt_handler(int irq, void *dev_id) {
    struct pci_dev *dev = dev_id;
    
    // 检查设备状态寄存器
    u16 status;
    pci_read_config_word(dev, PCI_STATUS, &status);
    
    if (!(status & PCI_STATUS_INTERRUPT)) {
        return IRQ_NONE;  // 不是我的中断
    }
    
    // 处理中断
    // ...
    
    return IRQ_HANDLED;
}
```

**PCI 中断问题：**
- 有限的中断线（4 个：INTA-INTD）
- 中断共享导致性能下降
- 需要软件轮询判断中断源

#### PCIe：MSI/MSI-X

```c
// PCIe MSI-X 配置
int pcie_setup_msix(struct pci_dev *dev, int nvec) {
    int ret;
    
    // 使能 MSI-X
    ret = pci_alloc_irq_vectors(dev, 1, nvec, PCI_IRQ_MSIX);
    if (ret < 0) {
        dev_err(&dev->dev, "Failed to enable MSI-X: %d\n", ret);
        return ret;
    }
    
    // 为每个向量注册处理程序
    for (int i = 0; i < ret; i++) {
        int irq = pci_irq_vector(dev, i);
        
        // 每个中断向量有独立的处理程序
        request_irq(irq, pcie_msix_handler, 0,
                   dev_name(&dev->dev), dev);
    }
    
    dev_info(&dev->dev, "Configured %d MSI-X vectors\n", ret);
    return 0;
}

// MSI-X 中断处理（无需共享）
irqreturn_t pcie_msix_handler(int irq, void *dev_id) {
    struct pci_dev *dev = dev_id;
    
    // 已知是自己的中断，直接处理
    // 不需要检查状态寄存器
    
    return IRQ_HANDLED;
}
```

**PCIe MSI-X 优势：**
- 支持多达 2048 个中断向量
- 无中断共享
- 内存写事务，无需独立信号线
- 可以绑定到特定 CPU 核心

## 性能对比

### 延迟对比

```c
// 性能测试代码
struct latency_test {
    ktime_t start;
    ktime_t end;
    u64 latency_ns;
};

void benchmark_read_latency(struct pci_dev *dev, int iterations) {
    struct latency_test tests[1000];
    u32 dummy;
    int i;
    
    for (i = 0; i < iterations; i++) {
        tests[i].start = ktime_get();
        
        // PCI/PCIe 配置读取
        pci_read_config_dword(dev, PCI_VENDOR_ID, &dummy);
        
        tests[i].end = ktime_get();
        tests[i].latency_ns = ktime_to_ns(ktime_sub(tests[i].end, 
                                                    tests[i].start));
    }
    
    // 计算统计数据
    u64 total = 0, min = ULLONG_MAX, max = 0;
    for (i = 0; i < iterations; i++) {
        total += tests[i].latency_ns;
        if (tests[i].latency_ns < min) min = tests[i].latency_ns;
        if (tests[i].latency_ns > max) max = tests[i].latency_ns;
    }
    
    pr_info("Read Latency: avg=%llu ns, min=%llu ns, max=%llu ns\n",
            total / iterations, min, max);
}
```

**典型延迟（配置空间读取）：**
- PCI 33MHz：~600-800 ns
- PCIe Gen1：~400-600 ns
- PCIe Gen3：~200-400 ns
- PCIe Gen5：~150-300 ns

### 吞吐量对比

```c
// DMA 吞吐量测试
struct throughput_test {
    void *buffer;
    dma_addr_t dma_handle;
    size_t size;
    ktime_t start;
    ktime_t end;
};

void benchmark_dma_throughput(struct pci_dev *dev) {
    struct throughput_test test;
    u64 duration_ns, throughput_mbps;
    
    test.size = 256 * 1024 * 1024;  // 256 MB
    
    // 分配 DMA 缓冲区
    test.buffer = dma_alloc_coherent(&dev->dev, test.size,
                                    &test.dma_handle, GFP_KERNEL);
    
    test.start = ktime_get();
    
    // 执行 DMA 传输
    // pcie_start_dma_transfer(dev, test.dma_handle, test.size);
    // wait_for_completion(&transfer_complete);
    
    test.end = ktime_get();
    
    duration_ns = ktime_to_ns(ktime_sub(test.end, test.start));
    throughput_mbps = (test.size * 8ULL * 1000) / duration_ns;
    
    pr_info("DMA Throughput: %llu Mbps (%.2f GB/s)\n",
            throughput_mbps, throughput_mbps / 8000.0);
    
    dma_free_coherent(&dev->dev, test.size, test.buffer, test.dma_handle);
}
```

**典型吞吐量（单向）：**

| 接口 | 理论带宽 | 实际吞吐量 | 效率 |
|------|----------|------------|------|
| PCI 33MHz 32-bit | 133 MB/s | ~100 MB/s | 75% |
| PCI-X 133MHz | 1066 MB/s | ~800 MB/s | 75% |
| PCIe 1.0 x16 | 4 GB/s | ~3.2 GB/s | 80% |
| PCIe 3.0 x16 | 16 GB/s | ~14 GB/s | 87% |
| PCIe 4.0 x16 | 32 GB/s | ~28 GB/s | 87% |
| PCIe 5.0 x16 | 64 GB/s | ~56 GB/s | 87% |

## 兼容性和迁移

### 软件兼容性

PCIe 保持了与 PCI 的软件兼容性：

```c
// PCI 和 PCIe 使用相同的配置空间访问方法
u32 read_vendor_id(struct pci_dev *dev) {
    u32 vendor_device;
    
    // 这段代码对 PCI 和 PCIe 都有效
    pci_read_config_dword(dev, PCI_VENDOR_ID, &vendor_device);
    
    return vendor_device & 0xFFFF;
}

// 驱动程序可以同时支持 PCI 和 PCIe
static struct pci_driver my_driver = {
    .name = "my_device",
    .id_table = my_pci_ids,  // 支持多种设备
    .probe = my_probe,
    .remove = my_remove,
};

static const struct pci_device_id my_pci_ids[] = {
    // PCI 设备
    { PCI_DEVICE(0x8086, 0x1234) },
    
    // PCIe 设备
    { PCI_DEVICE(0x8086, 0x5678) },
    
    { 0, }
};
```

### 检测设备类型

```c
// 判断设备是 PCI 还是 PCIe
bool is_pcie_device(struct pci_dev *dev) {
    return pci_is_pcie(dev);
}

// 获取 PCIe 能力信息
void detect_pcie_capabilities(struct pci_dev *dev) {
    if (!pci_is_pcie(dev)) {
        dev_info(&dev->dev, "Legacy PCI device\n");
        return;
    }
    
    // 读取 PCIe Capability
    int pos = pci_find_capability(dev, PCI_CAP_ID_EXP);
    u16 cap;
    
    pci_read_config_word(dev, pos + PCI_EXP_FLAGS, &cap);
    
    int type = (cap & PCI_EXP_FLAGS_TYPE) >> 4;
    int version = cap & PCI_EXP_FLAGS_VERS;
    
    dev_info(&dev->dev, "PCIe device: Type=%d, Version=%d\n", 
             type, version);
    
    // 读取链路能力
    u32 lnkcap;
    pci_read_config_dword(dev, pos + PCI_EXP_LNKCAP, &lnkcap);
    
    int max_speed = lnkcap & PCI_EXP_LNKCAP_SLS;
    int max_width = (lnkcap & PCI_EXP_LNKCAP_MLW) >> 4;
    
    dev_info(&dev->dev, "  Max Speed: Gen%d, Max Width: x%d\n",
             max_speed, max_width);
}
```

### 迁移策略

从 PCI 迁移到 PCIe 的步骤：

```c
// 1. 检查现有硬件
void audit_pci_devices(void) {
    struct pci_dev *dev = NULL;
    
    pr_info("=== PCI/PCIe Device Audit ===\n");
    
    for_each_pci_dev(dev) {
        const char *type = pci_is_pcie(dev) ? "PCIe" : "PCI";
        
        pr_info("%s: %04x:%04x (%s)\n",
                pci_name(dev),
                dev->vendor,
                dev->device,
                type);
        
        if (!pci_is_pcie(dev)) {
            pr_warn("  WARNING: Legacy PCI device - consider upgrade\n");
        }
    }
}

// 2. 渐进式迁移
struct migration_plan {
    struct pci_dev *old_dev;  // PCI 设备
    struct pci_dev *new_dev;  // PCIe 替代设备
    bool cutover_ready;
};

int perform_device_migration(struct migration_plan *plan) {
    // a. 安装新 PCIe 设备
    if (!plan->new_dev) {
        pr_err("New PCIe device not found\n");
        return -ENODEV;
    }
    
    // b. 加载驱动程序
    detect_pcie_capabilities(plan->new_dev);
    
    // c. 测试新设备
    if (!test_device_functionality(plan->new_dev)) {
        pr_err("New device failed functionality test\n");
        return -EIO;
    }
    
    // d. 切换工作负载
    pr_info("Switching workload to new PCIe device...\n");
    
    // e. 停用旧 PCI 设备
    if (plan->old_dev) {
        pci_disable_device(plan->old_dev);
        pr_info("Old PCI device disabled\n");
    }
    
    plan->cutover_ready = true;
    return 0;
}
```

## 使用场景选择

### 何时可以使用 PCI（已过时）

现代系统几乎不再使用 PCI，但以下场景可能仍存在：

- 工业控制系统（传统设备）
- 嵌入式系统（成本敏感）
- 维护旧设备

### 推荐使用 PCIe 的场景

所有现代应用都应使用 PCIe：

```c
// 高性能计算
struct hpc_system {
    struct pci_dev *gpu;        // PCIe 4.0/5.0 x16
    struct pci_dev *nvme;       // PCIe 4.0 x4
    struct pci_dev *ib_hca;     // PCIe 4.0 x16 (InfiniBand)
};

// 数据中心
struct datacenter_server {
    struct pci_dev *nic[4];     // 4x 100GbE (PCIe 4.0 x16)
    struct pci_dev *nvme[8];    // 8x NVMe SSD (PCIe 4.0 x4)
    struct pci_dev *fpga;       // FPGA 加速器 (PCIe 4.0 x16)
};

// 消费电子
struct desktop_pc {
    struct pci_dev *gpu;        // PCIe 4.0/5.0 x16
    struct pci_dev *nvme;       // PCIe 4.0 x4
    struct pci_dev *wifi;       // PCIe 3.0 x1
};
```

## 总结

### PCI 的遗产

PCI 为计算机扩展奠定了基础，其配置空间模型和软件接口被 PCIe 继承。然而，并行总线架构的物理限制使其无法满足现代计算需求。

### PCIe 的优势

| 方面 | PCIe 优势 |
|------|-----------|
| **性能** | 持续演进（Gen1 → Gen6），带宽提升 128 倍 |
| **可扩展性** | 灵活的 Lane 配置，易于扩展 |
| **延迟** | 点对点通信，无总线仲裁延迟 |
| **可靠性** | 差分信号，错误检测和纠正 |
| **功耗** | 链路电源管理（ASPM），降低功耗 |
| **兼容性** | 软件层面完全兼容 PCI |

### 展望未来

PCIe 将持续发展：

- **PCIe 6.0**：PAM4 编码，64 GT/s
- **PCIe 7.0**（规划中）：128 GT/s
- **CXL**（Compute Express Link）：基于 PCIe 物理层的缓存一致性互连
- **光互连**：未来可能采用光学传输

PCIe 已成为现代计算架构的基石，从个人电脑到数据中心，从嵌入式系统到超级计算机，都离不开 PCIe 技术的支撑。
