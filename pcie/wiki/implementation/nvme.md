# NVMe over PCIe 实现

## 概述

NVMe (Non-Volatile Memory Express) 是专为 PCIe 接口设计的高性能存储协议，相比传统的 AHCI/SATA 协议，NVMe 充分利用了 PCIe 的高带宽和低延迟特性，提供了多队列、并行处理和精简的命令集。

## NVMe 架构

### 整体结构

```
+------------------+
|   应用程序        |
+------------------+
|   文件系统        |
+------------------+
|   块设备层        |
+------------------+
|   NVMe 驱动      |
| +-------------+  |
| | Admin Queue |  |  <--- 管理命令
| +-------------+  |
| | I/O Queue 1 |  |  <--- 数据传输
| | I/O Queue 2 |  |
| |    ...      |  |
| | I/O Queue N |  |
| +-------------+  |
+------------------+
|   PCIe 层        |
+------------------+
|   NVMe SSD       |
+------------------+
```

### 队列架构

```
CPU 0           CPU 1           CPU 2           CPU 3
  |               |               |               |
  v               v               v               v
SQ 1            SQ 2            SQ 3            SQ 4  <-- Submission Queues
  |               |               |               |
  +---------------+---------------+---------------+
                        |
                        v
                  NVMe Controller
                        |
  +---------------+---------------+---------------+
  |               |               |               |
  v               v               v               v
CQ 1            CQ 2            CQ 3            CQ 4  <-- Completion Queues
  |               |               |               |
  v               v               v               v
IRQ 1           IRQ 2           IRQ 3           IRQ 4
```

**特点**：
- Admin Queue：1 对 SQ/CQ，用于管理命令
- I/O Queues：多对 SQ/CQ（最多 65535 对），用于 I/O 命令
- 每个队列可以绑定到特定的 CPU 核心
- 每个 CQ 可以有独立的中断向量

## PCIe 配置

### BAR 布局

NVMe 设备使用 **BAR0** 映射控制寄存器：

```
BAR0: 64-bit Memory Space
+---------------------------+  0x0000
|   Controller Capabilities  |  CAP (8 bytes)
|   - Max Queue Entries      |
|   - Doorbell Stride        |
+---------------------------+  0x0008
|   Version                  |  VS (4 bytes)
+---------------------------+  0x000C
|   Interrupt Mask Set       |  INTMS (4 bytes)
+---------------------------+  0x0010
|   Interrupt Mask Clear     |  INTMC (4 bytes)
+---------------------------+  0x0014
|   Controller Configuration |  CC (4 bytes)
+---------------------------+  0x001C
|   Controller Status        |  CSTS (4 bytes)
+---------------------------+  0x0020
|   Admin Queue Attributes   |  AQA (4 bytes)
+---------------------------+  0x0028
|   Admin SQ Base Address    |  ASQ (8 bytes)
+---------------------------+  0x0030
|   Admin CQ Base Address    |  ACQ (8 bytes)
+---------------------------+  0x1000
|   SQ 0 Tail Doorbell       |  (Doorbell stride)
+---------------------------+  0x1004
|   CQ 0 Head Doorbell       |
+---------------------------+
|   ...                      |
+---------------------------+
```

### 配置空间

```c
// PCI 配置空间标准头
Vendor ID:    0x144D (Samsung) / 0x8086 (Intel) / ...
Device ID:    设备特定
Class Code:   0x010802 (NVM Express)
Sub-Class:    0x08 (Non-Volatile Memory Controller)
Prog IF:      0x02 (NVMe)

// BAR0 配置
BAR0: 64-bit, Prefetchable Memory
Size: 通常 8KB - 16KB

// Capabilities
- Power Management Capability (必需)
- MSI-X Capability (必需，NVMe 1.0+)
- PCIe Capability
- AER Capability (推荐)
```

## 初始化流程

### 1. PCIe 层初始化

```c
static int nvme_pci_probe(struct pci_dev *pdev,
                          const struct pci_device_id *id)
{
    struct nvme_dev *dev;
    int result;
    
    // 1. 启用 PCIe 设备
    result = pci_enable_device_mem(pdev);
    if (result)
        return result;
    
    // 2. 请求内存区域
    result = pci_request_mem_regions(pdev, "nvme");
    if (result)
        goto disable;
    
    // 3. 设置 64-bit DMA
    result = dma_set_mask_and_coherent(&pdev->dev, DMA_BIT_MASK(64));
    if (result)
        goto release;
    
    // 4. 映射 BAR0
    dev = kzalloc(sizeof(*dev), GFP_KERNEL);
    if (!dev) {
        result = -ENOMEM;
        goto release;
    }
    
    dev->bar = pci_ioremap_bar(pdev, 0);
    if (!dev->bar) {
        result = -ENOMEM;
        goto free_dev;
    }
    
    // 5. 设置 Bus Master
    pci_set_master(pdev);
    
    // 6. 配置 MSI-X
    result = nvme_setup_irqs(dev);
    if (result)
        goto unmap;
    
    // 7. 初始化 NVMe 控制器
    result = nvme_dev_map(dev);
    if (result)
        goto free_irqs;
    
    return 0;

free_irqs:
    nvme_release_irqs(dev);
unmap:
    iounmap(dev->bar);
free_dev:
    kfree(dev);
release:
    pci_release_mem_regions(pdev);
disable:
    pci_disable_device(pdev);
    return result;
}
```

### 2. NVMe 控制器初始化

```c
static int nvme_dev_map(struct nvme_dev *dev)
{
    u64 cap;
    u32 cc;
    
    // 1. 读取 Controller Capabilities
    cap = readq(&dev->bar->cap);
    dev->dbbuf_stride = 1 << (2 + ((cap >> 32) & 0xf));
    dev->max_queue_depth = min_t(int, ((cap >> 0) & 0xffff) + 1,
                                  NVME_Q_DEPTH);
    
    // 2. 检查版本
    dev->ctrl.version = readl(&dev->bar->vs);
    if (dev->ctrl.version < NVME_VS(1, 2, 0)) {
        dev_warn(dev->ctrl.device, "Old NVMe version\n");
    }
    
    // 3. 重置控制器
    cc = readl(&dev->bar->cc);
    if (cc & NVME_CC_ENABLE) {
        // 禁用控制器
        writel(cc & ~NVME_CC_ENABLE, &dev->bar->cc);
        
        // 等待控制器就绪
        if (nvme_wait_ready(dev, false))
            return -ENODEV;
    }
    
    // 4. 创建 Admin Queue
    result = nvme_create_admin_queue(dev);
    if (result)
        return result;
    
    // 5. 配置控制器
    cc = NVME_CC_CSS_NVM;
    cc |= NVME_CC_MPS(PAGE_SHIFT - 12);  // Memory Page Size
    cc |= NVME_CC_AMS_RR;                // Arbitration: Round Robin
    cc |= NVME_CC_SHN_NONE;
    cc |= NVME_CC_IOSQES | NVME_CC_IOCQES;  // 6 bits each
    cc |= NVME_CC_ENABLE;
    
    writel(cc, &dev->bar->cc);
    
    // 6. 等待控制器就绪
    if (nvme_wait_ready(dev, true))
        return -ENODEV;
    
    // 7. 识别控制器
    result = nvme_identify_ctrl(dev);
    if (result)
        return result;
    
    // 8. 创建 I/O Queues
    result = nvme_setup_io_queues(dev);
    if (result)
        return result;
    
    return 0;
}
```

### 3. 队列创建

```c
// 创建 Admin Queue
static int nvme_create_admin_queue(struct nvme_dev *dev)
{
    struct nvme_queue *nvmeq;
    
    // 分配队列结构
    nvmeq = nvme_alloc_queue(dev, 0, NVME_AQ_DEPTH);
    if (!nvmeq)
        return -ENOMEM;
    
    // 分配 SQ (Submission Queue)
    nvmeq->sq_cmds = dma_alloc_coherent(dev->dev,
                                         SQ_SIZE(NVME_AQ_DEPTH),
                                         &nvmeq->sq_dma_addr,
                                         GFP_KERNEL);
    if (!nvmeq->sq_cmds)
        goto free_nvmeq;
    
    // 分配 CQ (Completion Queue)
    nvmeq->cqes = dma_alloc_coherent(dev->dev,
                                      CQ_SIZE(NVME_AQ_DEPTH),
                                      &nvmeq->cq_dma_addr,
                                      GFP_KERNEL);
    if (!nvmeq->cqes)
        goto free_sq;
    
    // 配置 Admin Queue 寄存器
    writel(NVME_AQ_DEPTH - 1, &dev->bar->aqa);
    writeq(nvmeq->sq_dma_addr, &dev->bar->asq);
    writeq(nvmeq->cq_dma_addr, &dev->bar->acq);
    
    dev->queues[0] = nvmeq;
    return 0;

free_sq:
    dma_free_coherent(dev->dev, SQ_SIZE(NVME_AQ_DEPTH),
                      nvmeq->sq_cmds, nvmeq->sq_dma_addr);
free_nvmeq:
    kfree(nvmeq);
    return -ENOMEM;
}

// 创建 I/O Queue
static int nvme_create_io_queue(struct nvme_dev *dev, u16 qid)
{
    struct nvme_queue *nvmeq = dev->queues[qid];
    struct nvme_command c;
    
    // 1. 创建 I/O Completion Queue
    memset(&c, 0, sizeof(c));
    c.create_cq.opcode = nvme_admin_create_cq;
    c.create_cq.prp1 = cpu_to_le64(nvmeq->cq_dma_addr);
    c.create_cq.cqid = cpu_to_le16(qid);
    c.create_cq.qsize = cpu_to_le16(nvmeq->q_depth - 1);
    c.create_cq.cq_flags = cpu_to_le16(NVME_CQ_IRQ_ENABLED |
                                        NVME_QUEUE_PHYS_CONTIG);
    c.create_cq.irq_vector = cpu_to_le16(nvmeq->cq_vector);
    
    result = nvme_submit_sync_cmd(dev->queues[0], &c, NULL, 0);
    if (result)
        return result;
    
    // 2. 创建 I/O Submission Queue
    memset(&c, 0, sizeof(c));
    c.create_sq.opcode = nvme_admin_create_sq;
    c.create_sq.prp1 = cpu_to_le64(nvmeq->sq_dma_addr);
    c.create_sq.sqid = cpu_to_le16(qid);
    c.create_sq.qsize = cpu_to_le16(nvmeq->q_depth - 1);
    c.create_sq.sq_flags = cpu_to_le16(NVME_QUEUE_PHYS_CONTIG);
    c.create_sq.cqid = cpu_to_le16(qid);
    
    result = nvme_submit_sync_cmd(dev->queues[0], &c, NULL, 0);
    if (result)
        goto delete_cq;
    
    return 0;

delete_cq:
    nvme_delete_cq(dev, qid);
    return result;
}
```

## I/O 路径

### 命令提交

```c
static blk_status_t nvme_queue_rq(struct blk_mq_hw_ctx *hctx,
                                   const struct blk_mq_queue_data *bd)
{
    struct nvme_queue *nvmeq = hctx->driver_data;
    struct request *req = bd->rq;
    struct nvme_command cmnd;
    blk_status_t ret;
    
    // 1. 构建 NVMe 命令
    ret = nvme_setup_cmd(req->q->queuedata, req, &cmnd);
    if (ret)
        return ret;
    
    // 2. 映射数据缓冲区
    ret = nvme_map_data(nvmeq->dev, req, &cmnd);
    if (ret)
        return ret;
    
    // 3. 提交命令到 SQ
    spin_lock_irq(&nvmeq->sq_lock);
    nvme_submit_cmd(nvmeq, &cmnd);
    spin_unlock_irq(&nvmeq->sq_lock);
    
    return BLK_STS_OK;
}

static void nvme_submit_cmd(struct nvme_queue *nvmeq,
                             struct nvme_command *cmd)
{
    u16 tail = nvmeq->sq_tail;
    
    // 1. 拷贝命令到 SQ
    memcpy(&nvmeq->sq_cmds[tail], cmd, sizeof(*cmd));
    
    // 2. 更新 tail 指针
    if (++tail == nvmeq->q_depth)
        tail = 0;
    nvmeq->sq_tail = tail;
    
    // 3. 写 Doorbell 寄存器通知设备
    writel(tail, nvmeq->q_db);
}
```

### 命令完成

```c
static irqreturn_t nvme_irq(int irq, void *data)
{
    struct nvme_queue *nvmeq = data;
    
    // 处理完成队列
    nvme_process_cq(nvmeq);
    
    return IRQ_HANDLED;
}

static void nvme_process_cq(struct nvme_queue *nvmeq)
{
    u16 head = nvmeq->cq_head;
    u16 phase = nvmeq->cq_phase;
    
    for (;;) {
        struct nvme_completion *cqe = &nvmeq->cqes[head];
        
        // 1. 检查 Phase Tag
        if ((le16_to_cpu(cqe->status) & 1) != phase)
            break;
        
        // 2. 处理完成条目
        nvme_handle_cqe(nvmeq, cqe);
        
        // 3. 更新 head 指针
        if (++head == nvmeq->q_depth) {
            head = 0;
            phase = !phase;
        }
    }
    
    // 4. 更新 CQ head doorbell
    if (head != nvmeq->cq_head) {
        nvmeq->cq_head = head;
        nvmeq->cq_phase = phase;
        writel(head, nvmeq->q_db + nvmeq->dev->db_stride);
    }
}

static void nvme_handle_cqe(struct nvme_queue *nvmeq,
                             struct nvme_completion *cqe)
{
    struct request *req;
    u16 status = le16_to_cpu(cqe->status) >> 1;
    u16 cid = le16_to_cpu(cqe->command_id);
    
    // 1. 根据 Command ID 找到请求
    req = blk_mq_tag_to_rq(nvmeq->tags, cid);
    if (!req) {
        dev_warn(nvmeq->dev->ctrl.device,
                 "Invalid command id %d\n", cid);
        return;
    }
    
    // 2. 解除 DMA 映射
    nvme_unmap_data(nvmeq->dev, req);
    
    // 3. 完成请求
    if (status)
        blk_mq_complete_request(req, BLK_STS_IOERR);
    else
        blk_mq_complete_request(req, BLK_STS_OK);
}
```

## 中断配置

### MSI-X 设置

```c
static int nvme_setup_irqs(struct nvme_dev *dev)
{
    int nr_io_queues = num_online_cpus();
    int result;
    
    // 1. 确定中断向量数量
    // Admin Queue (1) + I/O Queues (nr_io_queues)
    dev->num_vecs = 1 + nr_io_queues;
    
    // 2. 请求 MSI-X 向量
    result = pci_alloc_irq_vectors(dev->pdev, 1, dev->num_vecs,
                                    PCI_IRQ_MSIX | PCI_IRQ_AFFINITY);
    if (result < 0)
        return result;
    
    dev->num_vecs = result;
    
    // 3. 注册 Admin Queue 中断
    result = request_irq(pci_irq_vector(dev->pdev, 0),
                         nvme_irq, 0, "nvme-admin", dev->queues[0]);
    if (result)
        goto free_vectors;
    
    // 4. 注册 I/O Queue 中断
    for (int i = 1; i < dev->num_vecs; i++) {
        result = request_irq(pci_irq_vector(dev->pdev, i),
                             nvme_irq, 0, "nvme-io",
                             dev->queues[i]);
        if (result)
            goto free_irqs;
    }
    
    return 0;

free_irqs:
    while (--i >= 0)
        free_irq(pci_irq_vector(dev->pdev, i), dev->queues[i]);
    free_irq(pci_irq_vector(dev->pdev, 0), dev->queues[0]);
free_vectors:
    pci_free_irq_vectors(dev->pdev);
    return result;
}
```

### 中断亲和性

```c
// 绑定中断到特定 CPU
static void nvme_set_irq_affinity(struct nvme_dev *dev)
{
    int i;
    
    for (i = 0; i < dev->num_vecs; i++) {
        int cpu = cpumask_first(pci_irq_get_affinity(dev->pdev, i));
        if (cpu < nr_cpu_ids) {
            dev->queues[i]->cpu = cpu;
        }
    }
}
```

## FEMU 实现

FEMU 中 NVMe 设备的实现位于 `hw/block/nvme.c`：

### 设备初始化

```c
static void nvme_realize(PCIDevice *pci_dev, Error **errp)
{
    NvmeCtrl *n = NVME(pci_dev);
    
    // 1. 设置 PCI 配置空间
    pci_config_set_vendor_id(pci_dev->config, PCI_VENDOR_ID_INTEL);
    pci_config_set_device_id(pci_dev->config, 0x5845);
    pci_config_set_class(pci_dev->config, PCI_CLASS_STORAGE_EXPRESS);
    
    // 2. 注册 BAR0
    memory_region_init_io(&n->iomem, OBJECT(n), &nvme_mmio_ops, n,
                          "nvme", n->reg_size);
    pci_register_bar(pci_dev, 0,
                     PCI_BASE_ADDRESS_SPACE_MEMORY |
                     PCI_BASE_ADDRESS_MEM_TYPE_64,
                     &n->iomem);
    
    // 3. 配置 MSI-X
    if (msix_init(pci_dev, n->params.msix_qsize,
                  &n->iomem, 0, PCI_MSIX_TABLE_OFFSET,
                  &n->iomem, 0, PCI_MSIX_PBA_OFFSET, 0, errp)) {
        return;
    }
    msix_vector_use(pci_dev, 0);  // Admin Queue
    
    // 4. 初始化控制器寄存器
    n->bar.cap = 0x0020001f00028001ULL;  // Capabilities
    n->bar.vs = 0x00010200;               // Version 1.2
    n->bar.intms = 0;
    n->bar.intmc = 0;
    n->bar.cc = 0;
    n->bar.csts = 0;
}
```

### MMIO 操作

```c
static const MemoryRegionOps nvme_mmio_ops = {
    .read = nvme_mmio_read,
    .write = nvme_mmio_write,
    .endianness = DEVICE_LITTLE_ENDIAN,
    .impl = {
        .min_access_size = 2,
        .max_access_size = 8,
    },
};

static uint64_t nvme_mmio_read(void *opaque, hwaddr addr,
                                unsigned size)
{
    NvmeCtrl *n = opaque;
    
    if (addr < sizeof(n->bar)) {
        // 读取控制寄存器
        return *(uint64_t *)((char *)&n->bar + addr);
    } else {
        // Doorbell 寄存器
        return 0;
    }
}

static void nvme_mmio_write(void *opaque, hwaddr addr,
                             uint64_t data, unsigned size)
{
    NvmeCtrl *n = opaque;
    
    if (addr == NVME_REG_CC) {
        // Controller Configuration
        n->bar.cc = data;
        if (data & NVME_CC_ENABLE) {
            nvme_start_ctrl(n);
        } else {
            nvme_clear_ctrl(n);
        }
    } else if (addr >= 0x1000) {
        // Doorbell 写入
        uint16_t qid = (addr - 0x1000) / (4 << n->db_stride);
        bool is_cq = (addr - 0x1000) & (2 << n->db_stride);
        
        if (is_cq) {
            nvme_update_cq_head(n, qid, data);
        } else {
            nvme_process_sq(n, qid, data);
        }
    }
}
```

### 命令处理

```c
static void nvme_process_sq(NvmeCtrl *n, uint16_t qid, uint16_t new_tail)
{
    NvmeSQueue *sq = &n->sq[qid];
    NvmeCommand *cmd;
    
    // 处理 SQ 中的所有新命令
    while (sq->head != new_tail) {
        cmd = &sq->dma_buf[sq->head];
        
        // 分发命令
        if (qid == 0) {
            nvme_admin_cmd(n, cmd, qid);
        } else {
            nvme_io_cmd(n, cmd, qid);
        }
        
        // 更新 head
        sq->head = (sq->head + 1) % sq->size;
    }
}

static void nvme_io_cmd(NvmeCtrl *n, NvmeCommand *cmd, uint16_t qid)
{
    switch (cmd->opcode) {
    case NVME_CMD_READ:
        nvme_read(n, cmd, qid);
        break;
    case NVME_CMD_WRITE:
        nvme_write(n, cmd, qid);
        break;
    case NVME_CMD_FLUSH:
        nvme_flush(n, cmd, qid);
        break;
    default:
        nvme_post_cqe(n, qid, NVME_INVALID_OPCODE);
    }
}
```

### 中断注入

```c
static void nvme_post_cqe(NvmeCtrl *n, uint16_t qid, uint16_t status)
{
    NvmeCQueue *cq = &n->cq[qid];
    NvmeCompletion *cqe;
    
    // 1. 写入完成条目
    cqe = &cq->dma_buf[cq->tail];
    cqe->status = cpu_to_le16(status | (cq->phase << 15));
    cqe->sq_head = cpu_to_le16(n->sq[qid].head);
    cqe->sq_id = cpu_to_le16(qid);
    
    // 2. 更新 tail
    cq->tail = (cq->tail + 1) % cq->size;
    if (cq->tail == 0) {
        cq->phase = !cq->phase;
    }
    
    // 3. 触发中断
    if (cq->irq_enabled) {
        msix_notify(&n->parent_obj, cq->vector);
    }
}
```

## 性能优化

### 1. 多队列利用

```c
// 根据 CPU 数量创建队列
int nr_queues = min_t(int, num_online_cpus(), dev->max_io_queues);

// 绑定队列到 CPU
blk_mq_pci_map_queues(&set, dev->pdev, 0);
```

### 2. 轮询模式

```c
// 禁用中断，使用轮询
static int nvme_poll(struct blk_mq_hw_ctx *hctx)
{
    struct nvme_queue *nvmeq = hctx->driver_data;
    
    // 直接处理完成队列，不等待中断
    nvme_process_cq(nvmeq);
    
    return 0;
}
```

### 3. 批量提交

```c
// 批量提交命令，减少 doorbell 写入
static void nvme_queue_rqs(struct request **rqlist)
{
    struct nvme_queue *nvmeq = ...;
    
    // 提交多个命令
    while (*rqlist) {
        nvme_setup_cmd_and_queue(*rqlist);
        rqlist++;
    }
    
    // 一次性更新 doorbell
    writel(nvmeq->sq_tail, nvmeq->q_db);
}
```

## 调试技巧

### 查看 NVMe 信息

```bash
# nvme-cli 工具
nvme list
nvme id-ctrl /dev/nvme0
nvme id-ns /dev/nvme0n1

# 查看队列状态
cat /sys/block/nvme0n1/queue/nr_hw_queues

# 查看中断亲和性
cat /proc/irq/*/smp_affinity_list | grep nvme
```

### 性能测试

```bash
# fio 测试
fio --name=randread --ioengine=libaio --iodepth=32 \
    --rw=randread --bs=4k --direct=1 \
    --size=1G --numjobs=4 --runtime=60 \
    --filename=/dev/nvme0n1

# 查看 I/O 统计
iostat -x 1 /dev/nvme0n1
```

## 总结

NVMe over PCIe 的关键特性：

1. **多队列架构**：每个 CPU 核心独立队列，消除锁竞争
2. **精简命令集**：64 字节命令，16 字节完成，高效传输
3. **低延迟**：Doorbell 机制，无需 MMIO 读取
4. **高吞吐**：支持 65535 个队列，每队列 65536 条目
5. **PCIe 优化**：充分利用 PCIe 带宽和并行性

参见：
- [PCIe 驱动开发](driver-guide.md)
- [MSI-X 中断](../interrupts/msix.md)
- [配置空间](../configuration/config-space.md)
- [FEMU 概览](femu-overview.md)
