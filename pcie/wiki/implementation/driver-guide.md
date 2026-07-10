# PCIe 驱动开发指南

## 概述

PCIe 驱动程序是操作系统与 PCIe 设备之间的桥梁，负责设备的初始化、配置、中断处理、DMA 管理以及设备的运行时控制。本文介绍 PCIe 驱动开发的核心概念、关键步骤和最佳实践。

## 驱动架构

### 分层结构

```
+---------------------------+
|      应用程序              |
+---------------------------+
|      系统调用接口          |
+---------------------------+
|   PCIe 设备驱动 (你的代码)  |
+---------------------------+
|   PCIe 核心子系统          |
|   - 配置空间访问           |
|   - 资源分配               |
|   - 中断管理               |
+---------------------------+
|   PCI 总线驱动             |
+---------------------------+
|   硬件抽象层               |
+---------------------------+
```

### Linux 驱动框架

```c
// 驱动结构
struct pci_driver {
    char *name;
    const struct pci_device_id *id_table;
    int (*probe)(struct pci_dev *dev, const struct pci_device_id *id);
    void (*remove)(struct pci_dev *dev);
    int (*suspend)(struct pci_dev *dev, pm_message_t state);
    int (*resume)(struct pci_dev *dev);
    void (*shutdown)(struct pci_dev *dev);
    const struct pci_error_handlers *err_handler;
};

// 设备 ID 表
static const struct pci_device_id my_pci_ids[] = {
    { PCI_DEVICE(VENDOR_ID, DEVICE_ID) },
    { 0, }
};
MODULE_DEVICE_TABLE(pci, my_pci_ids);
```

## 驱动初始化流程

### 1. 设备探测 (Probe)

```c
static int my_pci_probe(struct pci_dev *pdev,
                        const struct pci_device_id *id)
{
    int err;
    struct my_device *mydev;
    
    // 1. 启用设备
    err = pci_enable_device(pdev);
    if (err) {
        dev_err(&pdev->dev, "Cannot enable PCI device\n");
        return err;
    }
    
    // 2. 请求内存区域
    err = pci_request_regions(pdev, DRV_NAME);
    if (err) {
        dev_err(&pdev->dev, "Cannot request regions\n");
        goto err_disable;
    }
    
    // 3. 设置 DMA 掩码
    err = dma_set_mask_and_coherent(&pdev->dev, DMA_BIT_MASK(64));
    if (err) {
        err = dma_set_mask_and_coherent(&pdev->dev, DMA_BIT_MASK(32));
        if (err) {
            dev_err(&pdev->dev, "No usable DMA configuration\n");
            goto err_release;
        }
    }
    
    // 4. 映射 BAR 空间
    mydev = kzalloc(sizeof(*mydev), GFP_KERNEL);
    if (!mydev) {
        err = -ENOMEM;
        goto err_release;
    }
    
    mydev->bar0 = pci_ioremap_bar(pdev, 0);
    if (!mydev->bar0) {
        dev_err(&pdev->dev, "Cannot map BAR0\n");
        err = -ENOMEM;
        goto err_free;
    }
    
    // 5. 设置 Bus Master
    pci_set_master(pdev);
    
    // 6. 配置 MSI-X 中断
    err = setup_msix_interrupts(pdev, mydev);
    if (err)
        goto err_unmap;
    
    // 7. 初始化设备硬件
    err = init_device_hw(mydev);
    if (err)
        goto err_irq;
    
    // 8. 注册字符设备/块设备/网络设备等
    err = register_device(mydev);
    if (err)
        goto err_hw;
    
    pci_set_drvdata(pdev, mydev);
    dev_info(&pdev->dev, "Device initialized successfully\n");
    
    return 0;

err_hw:
    cleanup_device_hw(mydev);
err_irq:
    free_msix_interrupts(pdev, mydev);
err_unmap:
    iounmap(mydev->bar0);
err_free:
    kfree(mydev);
err_release:
    pci_release_regions(pdev);
err_disable:
    pci_disable_device(pdev);
    return err;
}
```

### 2. 配置空间访问

```c
// 读取配置空间
u16 vendor_id;
pci_read_config_word(pdev, PCI_VENDOR_ID, &vendor_id);

u32 bar0;
pci_read_config_dword(pdev, PCI_BASE_ADDRESS_0, &bar0);

// 写入配置空间
u16 command;
pci_read_config_word(pdev, PCI_COMMAND, &command);
command |= PCI_COMMAND_MEMORY | PCI_COMMAND_MASTER;
pci_write_config_word(pdev, PCI_COMMAND, command);

// 查找能力
int pos = pci_find_capability(pdev, PCI_CAP_ID_MSIX);
if (pos) {
    u16 control;
    pci_read_config_word(pdev, pos + PCI_MSIX_FLAGS, &control);
    int num_vectors = (control & PCI_MSIX_FLAGS_QSIZE) + 1;
}
```

### 3. BAR 空间映射

```c
// 读取 BAR 信息
resource_size_t bar_start = pci_resource_start(pdev, 0);
resource_size_t bar_len = pci_resource_len(pdev, 0);
unsigned long bar_flags = pci_resource_flags(pdev, 0);

if (!(bar_flags & IORESOURCE_MEM)) {
    dev_err(&pdev->dev, "BAR0 is not memory resource\n");
    return -EINVAL;
}

// 映射到内核虚拟地址空间
void __iomem *mmio = pci_ioremap_bar(pdev, 0);
if (!mmio) {
    dev_err(&pdev->dev, "Failed to map BAR0\n");
    return -ENOMEM;
}

// 访问寄存器
u32 reg_value = ioread32(mmio + REG_OFFSET);
iowrite32(0x12345678, mmio + REG_OFFSET);

// 使用完后释放
iounmap(mmio);
```

## 中断处理

### MSI-X 配置

```c
static int setup_msix_interrupts(struct pci_dev *pdev,
                                  struct my_device *mydev)
{
    int nvec, err;
    
    // 请求中断向量
    nvec = pci_msix_vec_count(pdev);
    if (nvec < 0) {
        dev_err(&pdev->dev, "MSI-X not supported\n");
        return nvec;
    }
    
    mydev->msix_entries = kcalloc(nvec, sizeof(struct msix_entry),
                                   GFP_KERNEL);
    if (!mydev->msix_entries)
        return -ENOMEM;
    
    for (int i = 0; i < nvec; i++)
        mydev->msix_entries[i].entry = i;
    
    err = pci_enable_msix_exact(pdev, mydev->msix_entries, nvec);
    if (err) {
        dev_err(&pdev->dev, "Failed to enable MSI-X\n");
        kfree(mydev->msix_entries);
        return err;
    }
    
    // 注册中断处理程序
    for (int i = 0; i < nvec; i++) {
        err = request_irq(mydev->msix_entries[i].vector,
                          my_irq_handler,
                          0, DRV_NAME, mydev);
        if (err) {
            dev_err(&pdev->dev, "Failed to request IRQ %d\n", i);
            goto err_free_irqs;
        }
    }
    
    mydev->num_vectors = nvec;
    return 0;

err_free_irqs:
    while (--i >= 0)
        free_irq(mydev->msix_entries[i].vector, mydev);
    pci_disable_msix(pdev);
    kfree(mydev->msix_entries);
    return err;
}

// 中断处理函数
static irqreturn_t my_irq_handler(int irq, void *data)
{
    struct my_device *mydev = data;
    u32 status;
    
    // 读取中断状态
    status = ioread32(mydev->bar0 + INT_STATUS_REG);
    if (!status)
        return IRQ_NONE;  // 不是我们的中断
    
    // 清除中断状态
    iowrite32(status, mydev->bar0 + INT_STATUS_REG);
    
    // 处理中断
    if (status & INT_COMPLETION)
        handle_completion(mydev);
    
    if (status & INT_ERROR)
        handle_error(mydev);
    
    return IRQ_HANDLED;
}
```

### Legacy INTx（后备方案）

```c
static int setup_intx_interrupt(struct pci_dev *pdev,
                                 struct my_device *mydev)
{
    int err;
    
    // 启用 INTx 中断
    pci_intx(pdev, 1);
    
    // 注册中断处理程序
    err = request_irq(pdev->irq, my_irq_handler,
                      IRQF_SHARED, DRV_NAME, mydev);
    if (err) {
        dev_err(&pdev->dev, "Failed to request INTx IRQ\n");
        return err;
    }
    
    return 0;
}
```

## DMA 管理

### 一致性 DMA 映射

```c
// 分配一致性 DMA 缓冲区（用于控制结构）
dma_addr_t dma_handle;
void *cpu_addr;

cpu_addr = dma_alloc_coherent(&pdev->dev, size, &dma_handle,
                               GFP_KERNEL);
if (!cpu_addr) {
    dev_err(&pdev->dev, "Failed to allocate DMA buffer\n");
    return -ENOMEM;
}

// 使用缓冲区
struct command_desc *cmd = cpu_addr;
cmd->opcode = CMD_READ;
cmd->address = data_dma_addr;
cmd->length = data_len;

// 通知设备（写入物理地址）
iowrite64(dma_handle, mydev->bar0 + CMD_QUEUE_REG);

// 释放
dma_free_coherent(&pdev->dev, size, cpu_addr, dma_handle);
```

### 流式 DMA 映射

```c
// 映射单个缓冲区（用于数据传输）
dma_addr_t dma_addr;
void *buffer = kmalloc(size, GFP_KERNEL);

dma_addr = dma_map_single(&pdev->dev, buffer, size,
                          DMA_TO_DEVICE);
if (dma_mapping_error(&pdev->dev, dma_addr)) {
    dev_err(&pdev->dev, "DMA mapping failed\n");
    kfree(buffer);
    return -ENOMEM;
}

// 使用 dma_addr
submit_dma_transfer(mydev, dma_addr, size);

// 传输完成后解映射
dma_unmap_single(&pdev->dev, dma_addr, size, DMA_TO_DEVICE);

// scatter-gather 映射
struct scatterlist *sg;
int nents, mapped_nents;

// 创建 scatter-gather 列表
sg = kcalloc(nr_pages, sizeof(*sg), GFP_KERNEL);
sg_init_table(sg, nr_pages);
for (int i = 0; i < nr_pages; i++) {
    sg_set_page(&sg[i], pages[i], PAGE_SIZE, 0);
}

// 映射
mapped_nents = dma_map_sg(&pdev->dev, sg, nr_pages, DMA_BIDIRECTIONAL);
if (!mapped_nents) {
    dev_err(&pdev->dev, "Failed to map sg list\n");
    kfree(sg);
    return -ENOMEM;
}

// 使用 scatter-gather
for_each_sg(sg, s, mapped_nents, i) {
    submit_sg_entry(mydev, sg_dma_address(s), sg_dma_len(s));
}

// 解映射
dma_unmap_sg(&pdev->dev, sg, nr_pages, DMA_BIDIRECTIONAL);
kfree(sg);
```

## 电源管理

### 挂起和恢复

```c
static int my_pci_suspend(struct pci_dev *pdev, pm_message_t state)
{
    struct my_device *mydev = pci_get_drvdata(pdev);
    
    // 1. 停止设备活动
    stop_device_operations(mydev);
    
    // 2. 保存设备状态
    save_device_state(mydev);
    
    // 3. 释放中断
    free_msix_interrupts(pdev, mydev);
    
    // 4. 保存 PCI 配置空间
    pci_save_state(pdev);
    
    // 5. 设置电源状态
    pci_set_power_state(pdev, pci_choose_state(pdev, state));
    
    return 0;
}

static int my_pci_resume(struct pci_dev *pdev)
{
    struct my_device *mydev = pci_get_drvdata(pdev);
    int err;
    
    // 1. 恢复到 D0 状态
    pci_set_power_state(pdev, PCI_D0);
    
    // 2. 恢复 PCI 配置空间
    pci_restore_state(pdev);
    
    // 3. 重新启用设备
    err = pci_enable_device(pdev);
    if (err)
        return err;
    
    // 4. 重新设置 Bus Master
    pci_set_master(pdev);
    
    // 5. 重新配置中断
    err = setup_msix_interrupts(pdev, mydev);
    if (err)
        return err;
    
    // 6. 恢复设备状态
    restore_device_state(mydev);
    
    // 7. 重启设备活动
    restart_device_operations(mydev);
    
    return 0;
}
```

### Runtime PM

```c
static int my_runtime_suspend(struct device *dev)
{
    struct pci_dev *pdev = to_pci_dev(dev);
    struct my_device *mydev = pci_get_drvdata(pdev);
    
    // 空闲时自动挂起
    save_minimal_state(mydev);
    return 0;
}

static int my_runtime_resume(struct device *dev)
{
    struct pci_dev *pdev = to_pci_dev(dev);
    struct my_device *mydev = pci_get_drvdata(pdev);
    
    // 有活动时自动恢复
    restore_minimal_state(mydev);
    return 0;
}

static const struct dev_pm_ops my_pm_ops = {
    .suspend = my_pci_suspend,
    .resume = my_pci_resume,
    .runtime_suspend = my_runtime_suspend,
    .runtime_resume = my_runtime_resume,
};
```

## 错误处理

### AER 错误处理

```c
static pci_ers_result_t my_error_detected(struct pci_dev *pdev,
                                           pci_channel_state_t state)
{
    struct my_device *mydev = pci_get_drvdata(pdev);
    
    if (state == pci_channel_io_perm_failure)
        return PCI_ERS_RESULT_DISCONNECT;
    
    // 停止设备活动
    stop_device_operations(mydev);
    
    // 请求重置
    return PCI_ERS_RESULT_NEED_RESET;
}

static pci_ers_result_t my_slot_reset(struct pci_dev *pdev)
{
    struct my_device *mydev = pci_get_drvdata(pdev);
    int err;
    
    // 重新启用设备
    err = pci_enable_device(pdev);
    if (err)
        return PCI_ERS_RESULT_DISCONNECT;
    
    // 重新初始化设备
    err = reinit_device(mydev);
    if (err)
        return PCI_ERS_RESULT_DISCONNECT;
    
    return PCI_ERS_RESULT_RECOVERED;
}

static void my_error_resume(struct pci_dev *pdev)
{
    struct my_device *mydev = pci_get_drvdata(pdev);
    
    // 恢复正常操作
    restart_device_operations(mydev);
}

static const struct pci_error_handlers my_err_handlers = {
    .error_detected = my_error_detected,
    .slot_reset = my_slot_reset,
    .resume = my_error_resume,
};
```

## 驱动卸载

```c
static void my_pci_remove(struct pci_dev *pdev)
{
    struct my_device *mydev = pci_get_drvdata(pdev);
    
    // 1. 注销设备
    unregister_device(mydev);
    
    // 2. 停止设备
    stop_device_hw(mydev);
    
    // 3. 释放中断
    free_msix_interrupts(pdev, mydev);
    
    // 4. 解除 BAR 映射
    iounmap(mydev->bar0);
    
    // 5. 释放内存资源
    kfree(mydev);
    
    // 6. 释放 PCI 资源
    pci_release_regions(pdev);
    
    // 7. 禁用设备
    pci_disable_device(pdev);
    
    dev_info(&pdev->dev, "Device removed\n");
}
```

## 调试技巧

### 1. 使用 lspci

```bash
# 查看设备
lspci -v
lspci -vv  # 更详细
lspci -xxx  # 显示配置空间十六进制

# 查看特定设备
lspci -s 01:00.0 -vvv

# 查看 MSI-X 表
lspci -s 01:00.0 -vvv | grep -A 10 "MSI-X"
```

### 2. sysfs 接口

```bash
# 配置空间
cat /sys/bus/pci/devices/0000:01:00.0/config | hexdump -C

# 资源信息
cat /sys/bus/pci/devices/0000:01:00.0/resource

# 启用/禁用设备
echo 1 > /sys/bus/pci/devices/0000:01:00.0/enable
echo 0 > /sys/bus/pci/devices/0000:01:00.0/enable

# 移除设备
echo 1 > /sys/bus/pci/devices/0000:01:00.0/remove

# 重新扫描
echo 1 > /sys/bus/pci/rescan
```

### 3. 内核日志

```c
// 使用正确的日志级别
dev_err(&pdev->dev, "Error: %d\n", err);
dev_warn(&pdev->dev, "Warning: ...\n");
dev_info(&pdev->dev, "Info: ...\n");
dev_dbg(&pdev->dev, "Debug: ...\n");  // 需要 DEBUG 宏

// 动态调试
// 启用: echo 'file mydriver.c +p' > /sys/kernel/debug/dynamic_debug/control
```

### 4. tracepoints

```c
#include <trace/events/pci.h>

// PCIe 核心已有的 tracepoints
trace_pci_read_config_dword(pdev, offset, value);
trace_pci_write_config_dword(pdev, offset, value);
```

## 最佳实践

### 1. 资源管理

- 使用 devm_* 系列函数自动管理资源
- 在 probe 失败路径中正确释放资源
- 使用 RAII 模式避免资源泄漏

```c
// 使用 devm 版本
void __iomem *mmio = devm_ioremap(&pdev->dev, start, len);
int *data = devm_kzalloc(&pdev->dev, size, GFP_KERNEL);
// 驱动卸载时自动释放
```

### 2. 错误处理

- 检查所有 PCI 操作的返回值
- 实现完整的错误恢复路径
- 使用 AER 处理硬件错误

### 3. 性能优化

- 使用 DMA 而不是 PIO
- 批量处理中断
- 使用 NAPI 轮询（网络设备）
- 利用 MSI-X 多队列

### 4. 安全考虑

- 验证设备返回的所有数据
- 使用 IOMMU 保护 DMA
- 实现适当的访问控制
- 防止 DMA 攻击

## FEMU 示例

FEMU 中 NVMe 设备的 PCIe 驱动实现：

```c
// hw/block/nvme.c
static void nvme_realize(PCIDevice *pci_dev, Error **errp)
{
    NvmeCtrl *n = NVME(pci_dev);
    
    // 设置 PCI 配置空间
    pci_config_set_vendor_id(pci_dev->config, PCI_VENDOR_ID_INTEL);
    pci_config_set_device_id(pci_dev->config, 0x5845);
    pci_config_set_class(pci_dev->config, PCI_CLASS_STORAGE_EXPRESS);
    
    // 注册 BAR0（控制寄存器）
    memory_region_init_io(&n->iomem, OBJECT(n), &nvme_mmio_ops, n,
                          "nvme", n->reg_size);
    pci_register_bar(pci_dev, 0,
                     PCI_BASE_ADDRESS_SPACE_MEMORY |
                     PCI_BASE_ADDRESS_MEM_TYPE_64,
                     &n->iomem);
    
    // 配置 MSI-X
    if (msix_init(pci_dev, n->params.msix_qsize,
                  &n->iomem, 0, PCI_MSIX_TABLE_OFFSET,
                  &n->iomem, 0, PCI_MSIX_PBA_OFFSET, 0, errp)) {
        return;
    }
}

// 中断注入
static void nvme_irq_assert(NvmeCtrl *n, NvmeCQueue *cq)
{
    if (cq->irq_enabled) {
        if (msix_enabled(&n->parent_obj)) {
            msix_notify(&n->parent_obj, cq->vector);
        } else {
            pci_irq_assert(&n->parent_obj);
        }
    }
}
```

## 相关资源

- Linux 内核文档：Documentation/PCI/
- `drivers/pci/` 核心代码
- `include/linux/pci.h` API 定义
- LWN.net PCI 文章系列

## 总结

PCIe 驱动开发的关键要点：

1. **初始化顺序**：启用设备 → 映射资源 → 配置中断 → 初始化硬件
2. **中断处理**：优先使用 MSI-X，提供 INTx 后备
3. **DMA 管理**：正确映射/解映射，设置合适的 DMA 掩码
4. **电源管理**：实现挂起/恢复，支持 Runtime PM
5. **错误处理**：使用 AER，实现恢复路径
6. **调试工具**：lspci、sysfs、内核日志、tracepoints

参见：
- [PCIe 配置空间](../configuration/config-space.md)
- [中断机制](../interrupts/overview.md)
- [错误处理](../error-handling/aer.md)
- [NVMe 实现](nvme.md)
