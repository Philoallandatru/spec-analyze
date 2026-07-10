# PCIe 中断路由

## 概述

中断路由决定了设备中断如何从设备传递到 CPU。PCIe 支持三种中断机制（INTx、MSI、MSI-X），每种都有不同的路由方式。

## 中断路由架构

### 整体架构

```
设备层
  ├── INTx 消息
  ├── MSI 内存写
  └── MSI-X 内存写
      ↓
Root Complex
  ├── INTx → IOAPIC/PIC
  ├── MSI → LAPIC
  └── MSI-X → LAPIC
      ↓
中断控制器 (APIC)
  ├── Local APIC (每个 CPU)
  └── I/O APIC (系统)
      ↓
CPU 中断处理
```

## INTx 路由

### 路由表 (_PRT)

ACPI _PRT 方法定义 INTx 路由：

```
设备地址 + 中断引脚 → IRQ 号

示例：
Device 0x1C (Bridge)
  INTA# → IRQ 16
  INTB# → IRQ 17
  INTC# → IRQ 18
  INTD# → IRQ 19

Device 0x00 (下游设备)
  INTA# → IRQ 16 (与 Bridge 共享)
```

### 内核实现

```c
// 查询 ACPI 路由表
static int query_acpi_prt(struct pci_dev *pdev)
{
    acpi_handle handle;
    struct acpi_prt_entry *entry;
    u8 pin;
    int irq;
    
    // 读取 Interrupt Pin
    pci_read_config_byte(pdev, PCI_INTERRUPT_PIN, &pin);
    if (pin == 0)
        return -EINVAL;
    
    // 获取 ACPI handle
    handle = ACPI_HANDLE(&pdev->dev);
    
    // 查询 _PRT
    entry = acpi_pci_irq_lookup(pdev, pin - 1);
    if (!entry) {
        dev_err(&pdev->dev, "No _PRT entry found\n");
        return -ENODEV;
    }
    
    irq = entry->index;
    
    // 写入 Interrupt Line
    pci_write_config_byte(pdev, PCI_INTERRUPT_LINE, irq);
    
    dev_info(&pdev->dev, "INT%c# routed to IRQ %d\n",
             'A' + pin - 1, irq);
    
    return irq;
}
```

## MSI/MSI-X 路由

### 地址映射

```
MSI/MSI-X Message Address 格式 (x86):

0xFEE00000 + [Destination ID] + [Flags]

示例：
目标 CPU 0 (APIC ID = 0):
  Address = 0xFEE00000

目标 CPU 4 (APIC ID = 4):
  Address = 0xFEE04000

Message Data:
  Vector = 中断向量号 (0-255)
  Delivery Mode = 通常是 Fixed (000)
```

### 内核分配

```c
// 为 MSI 分配地址和向量
static int allocate_msi_vector(struct pci_dev *pdev, int vector_index)
{
    struct msi_desc *desc;
    struct msi_msg msg;
    int vector;
    
    // 分配中断向量
    vector = alloc_interrupt_vector();
    
    // 构造 MSI Message
    msg.address_hi = 0;
    msg.address_lo = 0xFEE00000;  // APIC base
    
    // 设置目标 CPU (Destination ID)
    int cpu = vector_index % num_online_cpus();
    int apic_id = per_cpu(x86_cpu_to_apicid, cpu);
    msg.address_lo |= (apic_id << 12);
    
    // 设置 Message Data
    msg.data = vector;  // 中断向量
    msg.data |= (0 << 8);  // Delivery Mode: Fixed
    
    // 写入设备配置
    write_msi_msg(pdev, &msg);
    
    dev_info(&pdev->dev,
             "MSI vector %d: addr=0x%08x data=0x%04x CPU=%d\n",
             vector_index, msg.address_lo, msg.data, cpu);
    
    return vector;
}
```

## IOAPIC 路由

### IOAPIC 架构

```
Legacy 设备 → IOAPIC → Local APIC → CPU

IOAPIC 重定向表项 (RTE):
+-------+--------+--------+--------+
| Vector| Dest  | Mode   | Status |
| 8-bit | 8-bit | 3-bit  | 2-bit  |
+-------+--------+--------+--------+

示例配置：
IRQ 16:
  Vector = 0x30 (48)
  Destination = 0x00 (CPU 0)
  Delivery Mode = 000 (Fixed)
  Trigger = Level
```

### 配置 IOAPIC

```c
// 配置 IOAPIC RTE
void setup_ioapic_routing(int irq, int vector, int dest_cpu)
{
    struct io_apic_irq_attr attr;
    int apic_id = per_cpu(x86_cpu_to_apicid, dest_cpu);
    
    attr.ioapic = 0;  // 第一个 IOAPIC
    attr.pin = irq;
    attr.trigger = IOAPIC_LEVEL;  // Level-triggered
    attr.polarity = IOAPIC_POL_LOW;  // Active low
    
    // 设置 RTE
    io_apic_set_pci_routing(&attr, irq, vector, apic_id);
    
    pr_info("IRQ %d → Vector %d → CPU %d (APIC %d)\n",
            irq, vector, dest_cpu, apic_id);
}
```

## 中断亲和性

### CPU 亲和性设置

```c
// 设置中断亲和性
static void set_interrupt_affinity(struct pci_dev *pdev, int vector, int cpu)
{
    int irq = pci_irq_vector(pdev, vector);
    const struct cpumask *mask = cpumask_of(cpu);
    
    // 设置硬件亲和性
    irq_set_affinity_hint(irq, mask);
    
    // 强制设置（需要权限）
    irq_set_affinity(irq, mask);
    
    dev_info(&pdev->dev, "IRQ %d pinned to CPU %d\n", irq, cpu);
}

// 按 NUMA 节点分配
static void distribute_irqs_by_numa(struct pci_dev *pdev, int nvec)
{
    int node = dev_to_node(&pdev->dev);
    const struct cpumask *node_cpus = cpumask_of_node(node);
    int cpu, i = 0;
    
    for_each_cpu(cpu, node_cpus) {
        if (i >= nvec)
            break;
        
        set_interrupt_affinity(pdev, i, cpu);
        i++;
    }
}
```

### 自动负载均衡

```bash
# 启用 irqbalance 服务
systemctl start irqbalance

# 配置策略
# /etc/sysconfig/irqbalance
IRQBALANCE_ONESHOT=0
IRQBALANCE_BANNED_CPUS=

# 查看当前分配
cat /proc/irq/*/smp_affinity_list
```

## 中断重映射 (Interrupt Remapping)

### VT-d 中断重映射

```
传统路由：
设备 → [MSI Addr/Data] → APIC → CPU

中断重映射：
设备 → [Remapped Addr/Data] → VT-d → [Real Addr/Data] → APIC → CPU

优势：
- 安全性：设备不能直接指定任意地址
- 灵活性：软件可以动态重映射
- 虚拟化：支持设备直通
```

### 重映射配置

```c
// Intel VT-d 中断重映射
struct irte {
    u64 present     : 1;
    u64 fpd         : 1;  // Fault Processing Disable
    u64 dst_mode    : 1;  // Destination Mode
    u64 rh          : 1;  // Redirection Hint
    u64 tm          : 1;  // Trigger Mode
    u64 dlvry_mode  : 3;  // Delivery Mode
    u64 avail       : 4;
    u64 reserved    : 3;
    u64 irte_mode   : 1;
    u64 vector      : 8;
    u64 reserved2   : 8;
    u64 dest_id     : 32;
};

// 配置 IRTE
static int setup_irte(struct intel_iommu *iommu,
                       int index, struct irte *irte)
{
    struct irq_2_iommu *irq_iommu;
    
    // 分配 IRTE 索引
    irq_iommu = alloc_irte(iommu, index);
    
    // 配置 IRTE
    irte->present = 1;
    irte->dst_mode = 0;  // Physical
    irte->tm = 0;        // Edge
    irte->dlvry_mode = 0;  // Fixed
    irte->vector = 0x30;  // 中断向量
    irte->dest_id = 0;    // CPU 0
    
    // 写入 IRTE 表
    write_irte(iommu, index, irte);
    
    return 0;
}
```

## 多队列中断分发

### 队列与 CPU 绑定

```c
// NVMe 多队列中断分发示例
static int nvme_setup_irq_distribution(struct nvme_dev *dev)
{
    int nr_queues = dev->online_queues;
    int nr_cpus = num_online_cpus();
    int i;
    
    // 为每个队列分配中断向量
    for (i = 0; i < nr_queues; i++) {
        int cpu = i % nr_cpus;
        int vector = i;
        
        // 设置亲和性
        set_interrupt_affinity(dev->pdev, vector, cpu);
        
        // 配置队列
        dev->queues[i]->cpu = cpu;
        dev->queues[i]->vector = vector;
        
        dev_info(&dev->pdev->dev,
                 "Queue %d → Vector %d → CPU %d\n",
                 i, vector, cpu);
    }
    
    return 0;
}
```

### RSS (Receive Side Scaling)

```c
// 网卡 RSS 配置
struct rss_config {
    u32 hash_key[10];  // 320-bit key
    u8  indirection_table[128];  // CPU 映射表
    u32 hash_types;    // 启用的哈希类型
};

static void configure_rss(struct net_device *netdev, int num_queues)
{
    struct rss_config rss;
    int i;
    
    // 生成随机哈希密钥
    netdev_rss_key_fill(rss.hash_key, sizeof(rss.hash_key));
    
    // 配置间接表（轮询分配到队列）
    for (i = 0; i < 128; i++) {
        rss.indirection_table[i] = i % num_queues;
    }
    
    // 启用哈希类型
    rss.hash_types = RSS_HASH_TYPE_TCP_IPV4 |
                     RSS_HASH_TYPE_TCP_IPV6 |
                     RSS_HASH_TYPE_UDP_IPV4 |
                     RSS_HASH_TYPE_UDP_IPV6;
    
    // 写入硬件
    write_rss_config(netdev, &rss);
}
```

## 虚拟化中的中断路由

### 设备直通

```
Guest VM 中的中断路由：

VM 中的设备 → VT-d 重映射 → Host APIC → vCPU

Posted Interrupts (Posted-interrupt processing):
1. 设备产生中断
2. VT-d 将中断记录到 Posted Interrupt Descriptor
3. 发送通知向量到 pCPU
4. pCPU 在 VM-Entry 时注入中断到 vCPU
```

### VFIO 中断路由

```c
// VFIO 设备中断配置
static int vfio_pci_set_msi_trigger(struct vfio_pci_device *vdev,
                                     int index, int start, int count)
{
    struct vfio_irq_set *hdr;
    int *fds;
    int i, ret;
    
    // 分配 eventfd
    fds = kmalloc_array(count, sizeof(int), GFP_KERNEL);
    
    for (i = 0; i < count; i++) {
        // 创建 eventfd
        fds[i] = eventfd_create(0, EFD_CLOEXEC);
        
        // 配置 MSI-X
        ret = vfio_msi_set_vector_signal(vdev, start + i, fds[i]);
        if (ret)
            goto err_cleanup;
    }
    
    return 0;

err_cleanup:
    while (--i >= 0)
        eventfd_ctx_put(eventfd_ctx_fdget(fds[i]));
    kfree(fds);
    return ret;
}
```

## 调试和监控

### 查看中断路由

```bash
# 查看中断路由表
cat /proc/interrupts

# 查看特定设备的中断
lspci -vv -s 01:00.0 | grep "Interrupt:"

# 查看 IOAPIC 配置
cat /proc/ioapic

# 查看 MSI/MSI-X 路由
cat /sys/kernel/debug/pci/0000:01:00.0/msi_irqs
```

### 性能分析

```bash
# 中断统计
perf stat -e irq:* -a sleep 10

# 中断延迟
trace-cmd record -e irq_handler_entry -e irq_handler_exit

# 中断不均衡检测
watch -n 1 'cat /proc/interrupts | grep mydriver'
```

## 最佳实践

1. **MSI-X 优先**：提供最好的性能和灵活性
2. **CPU 亲和性**：绑定中断到特定 CPU 提高缓存命中率
3. **NUMA 感知**：将中断路由到设备所在的 NUMA 节点
4. **负载均衡**：使用 irqbalance 或手动分配
5. **避免中断风暴**：合理配置中断合并

## 总结

PCIe 中断路由的关键点：

1. **三种机制**：INTx（IOAPIC）、MSI/MSI-X（LAPIC）
2. **地址映射**：MSI 通过特殊地址直接写 APIC
3. **亲和性控制**：将中断绑定到特定 CPU
4. **中断重映射**：VT-d 提供安全和灵活性
5. **多队列支持**：RSS、多队列网卡/存储

参见：
- [中断概述](overview.md)
- [MSI 详解](msi.md)
- [MSI-X 详解](msix.md)
- [Legacy INTx](intx.md)
