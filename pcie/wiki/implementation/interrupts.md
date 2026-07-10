# PCIe 中断实现

## 概述

PCIe 支持三种中断机制：Legacy INTx、MSI 和 MSI-X。本文介绍这些中断机制的硬件实现、软件配置和调试方法。

## 中断机制对比

| 特性 | Legacy INTx | MSI | MSI-X |
|------|------------|-----|-------|
| 中断线 | 4 根物理线 (INTA#-INTD#) | 无（Message） | 无（Message） |
| 最大向量数 | 4 | 32 | 2048 |
| 中断共享 | 是 | 否 | 否 |
| 性能 | 低 | 中 | 高 |
| 地址/数据对 | - | 1 | 每个向量独立 |
| 硬件复杂度 | 低 | 中 | 高 |
| PCIe 要求 | 可选 | 必需（Gen 1+） | 必需（Gen 2+） |

## Legacy INTx 实现

### 硬件信号

```
PCIe 设备                         Root Complex
+----------+                      +------------+
|          |                      |            |
|  INTA#   |--------------------->|   APIC     |---> CPU
|  INTB#   |--------------------->|            |
|  INTC#   |--------------------->|            |
|  INTD#   |--------------------->|            |
|          |                      |            |
+----------+                      +------------+

物理信号（低电平有效）：
- Assert: 拉低信号线
- Deassert: 释放信号线
```

### 软件实现

```c
// 注册 INTx 中断
static int setup_intx_interrupt(struct pci_dev *pdev,
                                 struct my_device *mydev)
{
    int err;
    
    // 读取中断引脚
    u8 irq_pin;
    pci_read_config_byte(pdev, PCI_INTERRUPT_PIN, &irq_pin);
    
    if (irq_pin == 0) {
        dev_err(&pdev->dev, "No interrupt pin\n");
        return -ENODEV;
    }
    
    dev_info(&pdev->dev, "Using IRQ %d (Pin INT%c#)\n",
             pdev->irq, 'A' + irq_pin - 1);
    
    // 启用 INTx
    pci_intx(pdev, 1);
    
    // 注册中断处理程序（共享中断）
    err = request_irq(pdev->irq, my_intx_handler,
                      IRQF_SHARED, "mydevice", mydev);
    if (err) {
        dev_err(&pdev->dev, "Failed to request IRQ\n");
        return err;
    }
    
    return 0;
}

// INTx 中断处理程序
static irqreturn_t my_intx_handler(int irq, void *data)
{
    struct my_device *mydev = data;
    u32 status;
    
    // 读取中断状态
    status = ioread32(mydev->bar0 + INT_STATUS_REG);
    
    // 检查是否是我们的中断
    if (!(status & INT_PENDING))
        return IRQ_NONE;  // 不是我们的中断
    
    // 清除中断状态（写 1 清除）
    iowrite32(status, mydev->bar0 + INT_STATUS_REG);
    
    // 处理中断
    handle_device_interrupt(mydev, status);
    
    return IRQ_HANDLED;
}

// 禁用 INTx
static void disable_intx(struct pci_dev *pdev, struct my_device *mydev)
{
    free_irq(pdev->irq, mydev);
    pci_intx(pdev, 0);
}
```

### INTx 路由表

```
设备            Pin     路由      IRQ
-----------------------------------------
00:1f.0 (SATA)  INTA#   -->      IRQ 19
01:00.0 (GPU)   INTA#   -->      IRQ 16
02:00.0 (NIC)   INTA#   -->      IRQ 17
03:00.0 (NVMe)  INTA#   -->      IRQ 11

查看路由：
$ cat /proc/interrupts
  11:  my-nvme
  16:  nvidia
  17:  e1000e
  19:  ahci
```

## MSI 实现

### Capability 结构

```
MSI Capability (32-bit 地址):
+0x00: [7:0]   Capability ID = 0x05
       [15:8]  Next Pointer
+0x02: [15:0]  Message Control
       [0]     MSI Enable
       [3:1]   Multiple Message Capable (MMC)
       [6:4]   Multiple Message Enable (MME)
       [7]     64-bit Address Capable
       [8]     Per-Vector Masking Capable
+0x04: [31:0]  Message Address (Lower)
+0x08: [31:0]  Message Data
+0x0C: [31:0]  Mask Bits (可选)
+0x10: [31:0]  Pending Bits (可选)

MSI Capability (64-bit 地址):
+0x04: [31:0]  Message Address (Lower)
+0x08: [31:0]  Message Address (Upper)
+0x0C: [15:0]  Message Data
+0x10: [31:0]  Mask Bits (可选)
+0x14: [31:0]  Pending Bits (可选)
```

### 软件配置

```c
// 配置 MSI
static int setup_msi_interrupts(struct pci_dev *pdev,
                                 struct my_device *mydev)
{
    int nvec, err, i;
    
    // 查询设备支持的向量数
    nvec = pci_msi_vec_count(pdev);
    if (nvec < 0) {
        dev_err(&pdev->dev, "MSI not supported\n");
        return nvec;
    }
    
    // 请求向量（2 的幂次）
    mydev->num_vectors = roundup_pow_of_two(min(nvec, 8));
    
    err = pci_enable_msi_exact(pdev, mydev->num_vectors);
    if (err) {
        dev_err(&pdev->dev, "Failed to enable MSI\n");
        return err;
    }
    
    // 注册中断处理程序
    for (i = 0; i < mydev->num_vectors; i++) {
        err = request_irq(pdev->irq + i, my_msi_handler,
                          0, "mydevice-msi", mydev);
        if (err)
            goto free_irqs;
    }
    
    dev_info(&pdev->dev, "Enabled %d MSI vectors\n",
             mydev->num_vectors);
    
    return 0;

free_irqs:
    while (--i >= 0)
        free_irq(pdev->irq + i, mydev);
    pci_disable_msi(pdev);
    return err;
}

// MSI 中断处理程序
static irqreturn_t my_msi_handler(int irq, void *data)
{
    struct my_device *mydev = data;
    int vector;
    
    // 计算向量号
    vector = irq - mydev->pdev->irq;
    
    // 处理该向量的中断
    handle_vector_interrupt(mydev, vector);
    
    return IRQ_HANDLED;
}
```

### MSI 地址/数据格式

```
Message Address (x86/x64):
+---+---+---+---+---+---+---+---+
|31           20|19   12|11    0|
+---+---+---+---+---+---+---+---+
| 0xFEE      | Dest ID |  Flags |
+---+---+---+---+---+---+---+---+

Bits [19:12]: Destination ID (APIC ID)
Bits [3]: Redirection Hint (0=Direct, 1=Lowest Priority)
Bits [2]: Destination Mode (0=Physical, 1=Logical)

Message Data:
+---+---+---+---+---+---+---+---+
|31   15|14 |13 |12 |11  10|9  8|7      0|
+---+---+---+---+---+---+---+---+
|Reserved|Lvl|Trig|Rsvd|Del Mode|Vector |
+---+---+---+---+---+---+---+---+

Bits [7:0]: Interrupt Vector (0-255)
Bits [10:8]: Delivery Mode (0=Fixed, 1=Lowest Priority)
Bit [14]: Trigger Mode (0=Edge, 1=Level)
Bit [15]: Level (0=Deassert, 1=Assert)
```

## MSI-X 实现

### Capability 结构

```
MSI-X Capability:
+0x00: [7:0]   Capability ID = 0x11
       [15:8]  Next Pointer
+0x02: [15:0]  Message Control
       [0]     Reserved
       [14]    Mask All
       [15]    MSI-X Enable
       [10:0]  Table Size (N-1)
+0x04: [31:0]  Table Offset
       [31:3]  Table Offset
       [2:0]   BIR (BAR Indicator)
+0x08: [31:0]  PBA Offset
       [31:3]  PBA Offset
       [2:0]   BIR
```

### MSI-X Table 结构

```
MSI-X Table Entry (16 bytes per entry):
+0x00: [31:0]  Message Address (Lower)
+0x04: [31:0]  Message Address (Upper)
+0x08: [31:0]  Message Data
+0x0C: [31:0]  Vector Control
       [0]     Mask Bit

示例（8 个向量）：
+----------+----------+----------+----------+
| Entry 0  |          |          |          |  16 bytes
+----------+----------+----------+----------+
| Entry 1  |          |          |          |  16 bytes
+----------+----------+----------+----------+
|   ...    |          |          |          |
+----------+----------+----------+----------+
| Entry 7  |          |          |          |  16 bytes
+----------+----------+----------+----------+
Total: 128 bytes
```

### 软件配置

```c
// 配置 MSI-X
static int setup_msix_interrupts(struct pci_dev *pdev,
                                  struct my_device *mydev)
{
    int nvec, err, i;
    
    // 查询向量数
    nvec = pci_msix_vec_count(pdev);
    if (nvec < 0) {
        dev_err(&pdev->dev, "MSI-X not supported\n");
        return nvec;
    }
    
    // 分配 MSI-X entries
    mydev->msix_entries = kcalloc(nvec, sizeof(struct msix_entry),
                                   GFP_KERNEL);
    if (!mydev->msix_entries)
        return -ENOMEM;
    
    // 初始化 entry 索引
    for (i = 0; i < nvec; i++)
        mydev->msix_entries[i].entry = i;
    
    // 启用 MSI-X
    err = pci_enable_msix_exact(pdev, mydev->msix_entries, nvec);
    if (err) {
        dev_err(&pdev->dev, "Failed to enable MSI-X: %d\n", err);
        goto free_entries;
    }
    
    // 注册中断处理程序
    for (i = 0; i < nvec; i++) {
        err = request_irq(mydev->msix_entries[i].vector,
                          my_msix_handler,
                          0, "mydevice-msix", mydev);
        if (err)
            goto free_irqs;
        
        // 设置中断亲和性
        irq_set_affinity_hint(mydev->msix_entries[i].vector,
                              get_cpu_mask(i % num_online_cpus()));
    }
    
    mydev->num_vectors = nvec;
    dev_info(&pdev->dev, "Enabled %d MSI-X vectors\n", nvec);
    
    return 0;

free_irqs:
    while (--i >= 0) {
        irq_set_affinity_hint(mydev->msix_entries[i].vector, NULL);
        free_irq(mydev->msix_entries[i].vector, mydev);
    }
    pci_disable_msix(pdev);
free_entries:
    kfree(mydev->msix_entries);
    return err;
}

// MSI-X 中断处理程序
static irqreturn_t my_msix_handler(int irq, void *data)
{
    struct my_device *mydev = data;
    int vector;
    
    // 查找向量号
    for (vector = 0; vector < mydev->num_vectors; vector++) {
        if (irq == mydev->msix_entries[vector].vector)
            break;
    }
    
    if (vector == mydev->num_vectors)
        return IRQ_NONE;
    
    // 处理该向量的中断
    handle_vector_interrupt(mydev, vector);
    
    return IRQ_HANDLED;
}

// 掩码/解除掩码向量
static void msix_mask_vector(struct pci_dev *pdev, int vector)
{
    pci_msix_mask_irq(pdev, vector);
}

static void msix_unmask_vector(struct pci_dev *pdev, int vector)
{
    pci_msix_unmask_irq(pdev, vector);
}
```

### 动态分配 MSI-X

```c
// Linux 4.10+ 新 API
static int setup_msix_modern(struct pci_dev *pdev,
                              struct my_device *mydev)
{
    int nvec, err;
    
    // 请求 MSI-X 向量（自动分配）
    nvec = pci_alloc_irq_vectors(pdev, 1, 32,
                                  PCI_IRQ_MSIX | PCI_IRQ_AFFINITY);
    if (nvec < 0) {
        dev_err(&pdev->dev, "Failed to allocate MSI-X vectors\n");
        return nvec;
    }
    
    mydev->num_vectors = nvec;
    
    // 注册中断（自动获取 IRQ 号）
    for (int i = 0; i < nvec; i++) {
        int irq = pci_irq_vector(pdev, i);
        
        err = request_irq(irq, my_msix_handler, 0,
                          "mydevice-msix", mydev);
        if (err)
            goto free_irqs;
    }
    
    return 0;

free_irqs:
    pci_free_irq_vectors(pdev);
    return err;
}
```

## FEMU 实现

### MSI-X 初始化

```c
// hw/block/nvme.c
static int nvme_init_msix(NvmeCtrl *n)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    int ret;
    
    // 创建 MSI-X 表和 PBA 的内存区域
    ret = msix_init_exclusive_bar(pci_dev, n->params.msix_qsize,
                                   NVME_MSIX_BAR, NULL);
    if (ret < 0) {
        return ret;
    }
    
    // 标记所有向量为已使用
    for (int i = 0; i < n->params.msix_qsize; i++) {
        msix_vector_use(pci_dev, i);
    }
    
    return 0;
}
```

### 中断触发

```c
// 触发 MSI-X 中断
static void nvme_irq_assert(NvmeCtrl *n, NvmeCQueue *cq)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    
    if (!cq->irq_enabled) {
        return;
    }
    
    // 检查中断掩码
    if (n->bar.intms & (1 << cq->vector)) {
        return;
    }
    
    trace_nvme_irq_assert(cq->cqid, cq->vector);
    
    // 发送 MSI-X 中断
    if (msix_enabled(pci_dev)) {
        msix_notify(pci_dev, cq->vector);
    } else if (msi_enabled(pci_dev)) {
        msi_notify(pci_dev, cq->vector);
    } else {
        // Legacy INTx
        pci_set_irq(pci_dev, 1);
    }
}

// MSI-X 通知实现
void msix_notify(PCIDevice *dev, unsigned vector)
{
    MSIMessage msg;
    
    if (vector >= dev->msix_entries_nr || !dev->msix_entry_used[vector]) {
        return;
    }
    
    // 检查向量是否被掩码
    if (msix_is_masked(dev, vector)) {
        msix_set_pending(dev, vector);
        return;
    }
    
    // 读取 MSI-X 表条目
    msg = msix_get_message(dev, vector);
    
    // 发送 Memory Write TLP
    pci_dma_write(dev, msg.address, &msg.data, sizeof(msg.data));
}
```

### MSI/MSI-X 模拟

```c
// hw/pci/msi.c
void msi_notify(PCIDevice *dev, unsigned int vector)
{
    MSIMessage msg;
    
    if (!msi_enabled(dev)) {
        return;
    }
    
    // 获取 Message Address/Data
    msg = msi_get_message(dev, vector);
    
    // 发送 Memory Write TLP
    pci_dma_write(dev, msg.address, &msg.data, sizeof(msg.data));
}

MSIMessage msi_get_message(PCIDevice *dev, unsigned int vector)
{
    uint16_t flags = pci_get_word(dev->config + dev->msi_cap + PCI_MSI_FLAGS);
    bool msi64bit = flags & PCI_MSI_FLAGS_64BIT;
    MSIMessage msg;
    
    // 读取 Message Address
    if (msi64bit) {
        msg.address = pci_get_quad(dev->config + dev->msi_cap + PCI_MSI_ADDRESS_LO);
        uint16_t data_offset = PCI_MSI_DATA_64;
        msg.data = pci_get_word(dev->config + dev->msi_cap + data_offset);
    } else {
        msg.address = pci_get_long(dev->config + dev->msi_cap + PCI_MSI_ADDRESS_LO);
        msg.data = pci_get_word(dev->config + dev->msi_cap + PCI_MSI_DATA_32);
    }
    
    // 修改 data 以包含向量号
    msg.data &= ~0x7;  // 清除低 3 位
    msg.data |= vector & 0x7;
    
    return msg;
}
```

## 中断亲和性

### 绑定到特定 CPU

```bash
# 查看中断亲和性
cat /proc/irq/125/smp_affinity_list
0-7

# 绑定到 CPU 0
echo 0 > /proc/irq/125/smp_affinity_list

# 绑定到 CPU 0-3
echo 0-3 > /proc/irq/125/smp_affinity_list

# 使用十六进制掩码
echo ff > /proc/irq/125/smp_affinity  # CPU 0-7
```

### 代码设置亲和性

```c
// 设置中断亲和性
static void set_irq_affinity(struct my_device *mydev)
{
    int i;
    
    for (i = 0; i < mydev->num_vectors; i++) {
        int cpu = i % num_online_cpus();
        const struct cpumask *mask = get_cpu_mask(cpu);
        
        irq_set_affinity_hint(mydev->msix_entries[i].vector, mask);
        
        dev_info(&mydev->pdev->dev,
                 "Vector %d -> CPU %d\n", i, cpu);
    }
}
```

## 中断合并

### NAPI (网络设备)

```c
// 使用 NAPI 轮询模式减少中断
static int my_napi_poll(struct napi_struct *napi, int budget)
{
    struct my_device *mydev = container_of(napi, struct my_device, napi);
    int work_done = 0;
    
    // 处理完成队列
    while (work_done < budget) {
        if (!has_pending_work(mydev))
            break;
        
        process_completion(mydev);
        work_done++;
    }
    
    // 如果处理完了，重新启用中断
    if (work_done < budget) {
        napi_complete(napi);
        enable_interrupts(mydev);
    }
    
    return work_done;
}

// 中断处理：禁用中断，调度 NAPI
static irqreturn_t my_napi_irq(int irq, void *data)
{
    struct my_device *mydev = data;
    
    // 禁用中断
    disable_interrupts(mydev);
    
    // 调度 NAPI 轮询
    napi_schedule(&mydev->napi);
    
    return IRQ_HANDLED;
}
```

### 中断调节（Coalescing）

```c
// 配置中断调节
static void set_interrupt_coalescing(struct my_device *mydev,
                                      u32 usecs, u32 frames)
{
    // 时间阈值（微秒）
    iowrite32(usecs, mydev->bar0 + INT_COAL_TIME);
    
    // 帧数阈值
    iowrite32(frames, mydev->bar0 + INT_COAL_FRAMES);
    
    dev_info(&mydev->pdev->dev,
             "Interrupt coalescing: %u us, %u frames\n",
             usecs, frames);
}
```

## 调试技巧

### 查看中断信息

```bash
# 查看所有中断
cat /proc/interrupts

# 查看特定设备的中断
cat /proc/interrupts | grep nvme

# 查看 MSI/MSI-X 信息
lspci -vv -s 01:00.0 | grep -A 10 "MSI-X"

# 实时监控中断计数
watch -n 1 'cat /proc/interrupts | grep nvme'
```

### 内核日志

```bash
# 查看中断相关日志
dmesg | grep -i "msi\|irq"

# 启用中断调试
echo 8 > /proc/sys/kernel/printk
echo 'module pci +p' > /sys/kernel/debug/dynamic_debug/control
```

### 追踪中断

```bash
# 追踪中断事件
trace-cmd record -e irq:irq_handler_entry -e irq:irq_handler_exit

# 查看追踪
trace-cmd report
```

## 性能优化

### 最佳实践

1. **优先使用 MSI-X**：性能最高，无共享
2. **合理分配向量**：每个 CPU 核心一个向量
3. **设置亲和性**：减少 CPU 间通信
4. **中断合并**：降低中断频率
5. **NAPI 轮询**：高负载时减少中断开销

### 性能对比

```
测试场景：NVMe 4K 随机读
------------------------------
Legacy INTx:  100K IOPS
MSI (8 vecs): 400K IOPS
MSI-X (32 vecs): 1000K IOPS
MSI-X + 轮询: 2000K IOPS
```

## 总结

PCIe 中断实现的关键点：

1. **Legacy INTx**：共享中断，需要检查中断状态，性能较低
2. **MSI**：Message 方式，最多 32 个向量，无需物理线
3. **MSI-X**：独立 Table，最多 2048 个向量，每个可独立配置
4. **中断亲和性**：绑定到特定 CPU，提高缓存局部性
5. **中断合并**：减少中断频率，提高吞吐量
6. **FEMU 模拟**：通过 Memory Write 实现中断注入

参见：
- [中断机制概述](../interrupts/overview.md)
- [MSI-X 详解](../interrupts/msix.md)
- [MSI 详解](../interrupts/msi.md)
- [驱动开发指南](driver-guide.md)
