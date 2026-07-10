# FEMU 代码概览

## 概述

FEMU (Full-system Emulated NVMe) 是基于 QEMU 的 NVMe SSD 全系统模拟器，用于研究和开发 NVMe 相关的软件和固件。FEMU 实现了完整的 PCIe 和 NVMe 协议栈，提供了真实的 SSD 行为模拟。

## 架构总览

### FEMU 在系统中的位置

```
+------------------+
|   Guest OS       |
|   - NVMe Driver  |
+------------------+
       | PCIe TLP
       v
+------------------+
|   QEMU/KVM       |
|   - PCIe 模拟    |
|   - NVMe 控制器  |
|   - FEMU 后端    |
+------------------+
       | I/O
       v
+------------------+
|   Host Storage   |
|   - Image File   |
|   - Block Device |
+------------------+
```

### 代码结构

```
FEMU/
├── hw/
│   └── block/
│       ├── nvme.c              # NVMe 控制器实现（核心）
│       ├── nvme.h              # NVMe 数据结构
│       ├── nvme-ns.c           # Namespace 管理
│       └── femu/               # FEMU 扩展
│           ├── ftl.c           # Flash Translation Layer
│           ├── bbssd.c         # BlackBox SSD
│           └── oc12.c          # Open-Channel 1.2
├── include/
│   └── block/
│       └── nvme.h              # NVMe 规范定义
└── docs/
    └── specs/
        └── nvme.txt            # NVMe 规范说明
```

## PCIe 设备实现

### 设备类型定义

```c
// hw/block/nvme.c
static const TypeInfo nvme_info = {
    .name          = TYPE_NVME,
    .parent        = TYPE_PCI_DEVICE,
    .instance_size = sizeof(NvmeCtrl),
    .class_init    = nvme_class_init,
    .instance_init = nvme_instance_init,
    .interfaces = (InterfaceInfo[]) {
        { INTERFACE_PCIE_DEVICE },
        { }
    }
};

static void nvme_class_init(ObjectClass *oc, void *data)
{
    DeviceClass *dc = DEVICE_CLASS(oc);
    PCIDeviceClass *pc = PCI_DEVICE_CLASS(oc);
    
    // PCIe 设备类初始化
    pc->realize = nvme_realize;
    pc->exit = nvme_exit;
    pc->class_id = PCI_CLASS_STORAGE_EXPRESS;
    pc->vendor_id = PCI_VENDOR_ID_INTEL;
    pc->device_id = 0x5845;
    pc->revision = 2;
    pc->is_express = true;  // 标记为 PCIe 设备
    
    set_bit(DEVICE_CATEGORY_STORAGE, dc->categories);
    dc->desc = "Non-Volatile Memory Express";
}
```

### PCIe 配置空间设置

```c
static void nvme_realize(PCIDevice *pci_dev, Error **errp)
{
    NvmeCtrl *n = NVME(pci_dev);
    int64_t bs_size;
    
    // 1. 设置 PCI 配置空间标准头
    pci_conf = pci_dev->config;
    pci_config_set_vendor_id(pci_conf, PCI_VENDOR_ID_INTEL);
    pci_config_set_device_id(pci_conf, 0x5845);
    pci_config_set_class(pci_conf, PCI_CLASS_STORAGE_EXPRESS);
    pci_config_set_prog_interface(pci_conf, 0x02);  // NVMe
    
    // 2. PCIe Capability
    if (pcie_endpoint_cap_v1_init(pci_dev, 0x80) < 0) {
        error_setg(errp, "Failed to initialize PCIe capability");
        return;
    }
    
    // 3. Power Management Capability
    if (pci_add_capability(pci_dev, PCI_CAP_ID_PM, 0,
                           PCI_PM_SIZEOF, errp) < 0) {
        return;
    }
    
    // 4. MSI-X Capability
    n->params.msix_qsize = n->params.max_ioqpairs + 1;
    if (msix_init_exclusive_bar(pci_dev, n->params.msix_qsize,
                                 NVME_MSIX_BAR, errp)) {
        return;
    }
    
    // 5. 注册 BAR0 - 控制寄存器
    memory_region_init_io(&n->iomem, OBJECT(n), &nvme_mmio_ops, n,
                          "nvme", n->reg_size);
    pci_register_bar(pci_dev, NVME_REG_BAR,
                     PCI_BASE_ADDRESS_SPACE_MEMORY |
                     PCI_BASE_ADDRESS_MEM_TYPE_64,
                     &n->iomem);
    
    // 6. 初始化 NVMe 控制器
    nvme_init_ctrl(n, pci_dev);
}
```

### BAR 空间实现

```c
// NVMe 寄存器定义
typedef struct NvmeBar {
    uint64_t    cap;        // 0x00: Controller Capabilities
    uint32_t    vs;         // 0x08: Version
    uint32_t    intms;      // 0x0C: Interrupt Mask Set
    uint32_t    intmc;      // 0x10: Interrupt Mask Clear
    uint32_t    cc;         // 0x14: Controller Configuration
    uint32_t    rsvd1;
    uint32_t    csts;       // 0x1C: Controller Status
    uint32_t    nssr;       // 0x20: NVM Subsystem Reset
    uint32_t    aqa;        // 0x24: Admin Queue Attributes
    uint64_t    asq;        // 0x28: Admin SQ Base Address
    uint64_t    acq;        // 0x30: Admin CQ Base Address
    uint32_t    cmbloc;     // 0x38: Controller Memory Buffer Location
    uint32_t    cmbsz;      // 0x3C: Controller Memory Buffer Size
} NvmeBar;

// MMIO 读操作
static uint64_t nvme_mmio_read(void *opaque, hwaddr addr, unsigned size)
{
    NvmeCtrl *n = opaque;
    uint8_t *ptr = (uint8_t *)&n->bar;
    uint64_t val = 0;
    
    if (addr < sizeof(n->bar)) {
        // 读取控制寄存器
        memcpy(&val, ptr + addr, size);
        trace_nvme_mmio_read(addr, size, val);
    } else {
        // Doorbell 区域（只写，读返回 0）
        trace_nvme_mmio_doorbell_read(addr, size);
    }
    
    return val;
}

// MMIO 写操作
static void nvme_mmio_write(void *opaque, hwaddr addr,
                             uint64_t data, unsigned size)
{
    NvmeCtrl *n = opaque;
    
    trace_nvme_mmio_write(addr, size, data);
    
    // 处理特殊寄存器
    switch (addr) {
    case NVME_REG_INTMS:
        n->bar.intms |= data & 0xffffffff;
        nvme_update_msixcap_ts(pci_dev, n->bar.intms);
        break;
        
    case NVME_REG_INTMC:
        n->bar.intms &= ~(data & 0xffffffff);
        nvme_update_msixcap_ts(pci_dev, n->bar.intms);
        break;
        
    case NVME_REG_CC:
        nvme_write_bar(n, addr, data, size);
        if (NVME_CC_EN(data) && !NVME_CC_EN(n->bar.cc)) {
            // 启用控制器
            n->bar.cc = data;
            nvme_start_ctrl(n);
        } else if (!NVME_CC_EN(data) && NVME_CC_EN(n->bar.cc)) {
            // 禁用控制器
            nvme_clear_ctrl(n);
            n->bar.cc = 0;
        }
        break;
        
    default:
        if (addr >= 0x1000) {
            // Doorbell 寄存器
            nvme_process_db(n, addr, data);
        } else {
            nvme_write_bar(n, addr, data, size);
        }
    }
}
```

## 中断机制

### MSI-X 配置

```c
static int nvme_init_msix(NvmeCtrl *n)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    int ret;
    
    // MSI-X 表和 PBA 位于独立的 BAR
    ret = msix_init_exclusive_bar(pci_dev, n->params.msix_qsize,
                                   NVME_MSIX_BAR, NULL);
    if (ret < 0) {
        return ret;
    }
    
    // 初始化所有向量为未使用
    for (int i = 0; i < n->params.msix_qsize; i++) {
        msix_vector_use(pci_dev, i);
    }
    
    return 0;
}
```

### 中断注入

```c
static void nvme_irq_assert(NvmeCtrl *n, NvmeCQueue *cq)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    
    if (cq->irq_enabled) {
        // 检查中断是否被屏蔽
        if (n->bar.intms & (1 << cq->vector)) {
            return;
        }
        
        trace_nvme_irq_assert(cq->cqid, cq->vector);
        
        // 触发 MSI-X 中断
        if (msix_enabled(pci_dev)) {
            msix_notify(pci_dev, cq->vector);
        } else if (msi_enabled(pci_dev)) {
            msi_notify(pci_dev, cq->vector);
        } else {
            // Legacy INTx
            pci_irq_assert(pci_dev);
        }
    }
}

static void nvme_irq_deassert(NvmeCtrl *n, NvmeCQueue *cq)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    
    if (msix_enabled(pci_dev) || msi_enabled(pci_dev)) {
        // MSI/MSI-X 自动清除，无需操作
        return;
    }
    
    // Legacy INTx 需要手动清除
    pci_irq_deassert(pci_dev);
}
```

## DMA 操作

### DMA 读取

```c
static int nvme_dma_read(NvmeCtrl *n, uint8_t *ptr, uint32_t len,
                          NvmeCmd *cmd, NvmeRequest *req)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    uint64_t prp1 = le64_to_cpu(cmd->prp1);
    uint64_t prp2 = le64_to_cpu(cmd->prp2);
    QEMUSGList qsg;
    int ret;
    
    if (len == 0) {
        return 0;
    }
    
    trace_nvme_dma_read(prp1, len);
    
    // 使用 PRP (Physical Region Page) 构建 scatter-gather 列表
    pci_dma_sglist_init(&qsg, pci_dev, 1);
    
    if (len <= NVME_PAGE_SIZE) {
        // 单个 PRP 足够
        qemu_sglist_add(&qsg, prp1, len);
    } else {
        // 需要 PRP List
        ret = nvme_map_prp(&qsg, prp1, prp2, len, n);
        if (ret) {
            qemu_sglist_destroy(&qsg);
            return ret;
        }
    }
    
    // 执行 DMA 读取
    ret = pci_dma_read(pci_dev, qsg.sg[0].base, ptr, len);
    qemu_sglist_destroy(&qsg);
    
    return ret;
}
```

### DMA 写入

```c
static int nvme_dma_write(NvmeCtrl *n, const uint8_t *ptr, uint32_t len,
                           NvmeCmd *cmd, NvmeRequest *req)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    uint64_t prp1 = le64_to_cpu(cmd->prp1);
    uint64_t prp2 = le64_to_cpu(cmd->prp2);
    QEMUSGList qsg;
    int ret;
    
    if (len == 0) {
        return 0;
    }
    
    trace_nvme_dma_write(prp1, len);
    
    pci_dma_sglist_init(&qsg, pci_dev, 1);
    
    if (len <= NVME_PAGE_SIZE) {
        qemu_sglist_add(&qsg, prp1, len);
    } else {
        ret = nvme_map_prp(&qsg, prp1, prp2, len, n);
        if (ret) {
            qemu_sglist_destroy(&qsg);
            return ret;
        }
    }
    
    // 执行 DMA 写入
    ret = pci_dma_write(pci_dev, qsg.sg[0].base, ptr, len);
    qemu_sglist_destroy(&qsg);
    
    return ret;
}
```

### PRP 列表处理

```c
static int nvme_map_prp(QEMUSGList *qsg, uint64_t prp1, uint64_t prp2,
                         uint32_t len, NvmeCtrl *n)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    uint32_t trans_len;
    uint64_t prp_list[NVME_MAX_PRPS];
    int num_prps, i;
    
    // 第一个 PRP
    trans_len = MIN(len, NVME_PAGE_SIZE - (prp1 & NVME_PAGE_MASK));
    qemu_sglist_add(qsg, prp1, trans_len);
    len -= trans_len;
    
    if (len == 0) {
        return 0;
    }
    
    // 第二个 PRP 或 PRP List
    if (len <= NVME_PAGE_SIZE) {
        // 单个 PRP
        qemu_sglist_add(qsg, prp2, len);
    } else {
        // PRP List - 读取列表
        num_prps = (len + NVME_PAGE_SIZE - 1) / NVME_PAGE_SIZE;
        if (pci_dma_read(pci_dev, prp2, prp_list,
                         num_prps * sizeof(uint64_t))) {
            return -EINVAL;
        }
        
        // 添加所有 PRP
        for (i = 0; i < num_prps; i++) {
            uint64_t prp = le64_to_cpu(prp_list[i]);
            trans_len = MIN(len, NVME_PAGE_SIZE);
            qemu_sglist_add(qsg, prp, trans_len);
            len -= trans_len;
        }
    }
    
    return 0;
}
```

## 队列处理

### Doorbell 处理

```c
static void nvme_process_db(NvmeCtrl *n, hwaddr addr, uint64_t val)
{
    uint32_t qid;
    
    // 计算队列 ID
    if (addr < n->db_stride) {
        return;  // 无效地址
    }
    
    addr -= 0x1000;
    qid = addr / (2 * (4 << n->db_stride));
    
    if (qid > n->params.max_ioqpairs + 1) {
        return;  // 无效队列
    }
    
    // 判断是 SQ 还是 CQ
    if (addr & (4 << n->db_stride)) {
        // CQ Head Doorbell
        trace_nvme_mmio_doorbell_cq(qid, val);
        nvme_cq_update_head(n, qid, val);
    } else {
        // SQ Tail Doorbell
        trace_nvme_mmio_doorbell_sq(qid, val);
        nvme_sq_process(n, qid);
    }
}
```

### Submission Queue 处理

```c
static void nvme_sq_process(NvmeCtrl *n, uint16_t qid)
{
    NvmeSQueue *sq = &n->sq[qid];
    NvmeRequest *req;
    
    while (sq->head != sq->tail) {
        // 获取命令
        NvmeCmd *cmd = &sq->dma_buf[sq->head];
        
        // 分配请求结构
        req = nvme_alloc_req(sq);
        if (!req) {
            break;
        }
        
        // 拷贝命令
        memcpy(&req->cmd, cmd, sizeof(*cmd));
        req->cqe.cid = cmd->cid;
        
        trace_nvme_sq_cmd(qid, sq->head, cmd->opcode);
        
        // 分发命令
        if (qid == 0) {
            nvme_admin_cmd(n, req);
        } else {
            nvme_io_cmd(n, req);
        }
        
        // 更新 head
        sq->head = (sq->head + 1) % sq->size;
    }
}
```

### Completion Queue 处理

```c
static void nvme_post_cqe(NvmeCtrl *n, NvmeRequest *req)
{
    NvmeCQueue *cq = req->cq;
    NvmeSQueue *sq = req->sq;
    NvmeCqe *cqe = &cq->dma_buf[cq->tail];
    
    trace_nvme_post_cqe(cq->cqid, req->cqe.cid);
    
    // 填充完成条目
    req->cqe.status = cpu_to_le16((req->status << 1) | cq->phase);
    req->cqe.sq_head = cpu_to_le16(sq->head);
    req->cqe.sq_id = cpu_to_le16(sq->sqid);
    
    // 写入 CQ
    memcpy(cqe, &req->cqe, sizeof(*cqe));
    
    // 更新 tail
    cq->tail = (cq->tail + 1) % cq->size;
    if (cq->tail == 0) {
        cq->phase = !cq->phase;
    }
    
    // 触发中断
    nvme_irq_assert(n, cq);
    
    // 释放请求
    nvme_free_req(req);
}
```

## I/O 命令处理

### 读命令

```c
static void nvme_rw(NvmeCtrl *n, NvmeRequest *req)
{
    NvmeNamespace *ns = req->ns;
    NvmeRwCmd *rw = (NvmeRwCmd *)&req->cmd;
    uint64_t slba = le64_to_cpu(rw->slba);
    uint32_t nlb = le16_to_cpu(rw->nlb) + 1;
    uint64_t data_size = nlb << ns->lba_shift;
    uint64_t data_offset = slba << ns->lba_shift;
    
    trace_nvme_rw(req->cmd.opcode == NVME_CMD_WRITE ? "write" : "read",
                  nlb, data_size, slba);
    
    // 边界检查
    if (slba + nlb > ns->id_ns.nsze) {
        nvme_set_error(req, NVME_LBA_RANGE);
        return;
    }
    
    // 创建 QEMU block I/O
    if (req->cmd.opcode == NVME_CMD_WRITE) {
        // DMA 读取数据
        if (nvme_dma_read(n, req->buf, data_size, &req->cmd, req)) {
            nvme_set_error(req, NVME_INTERNAL_DEV_ERROR);
            return;
        }
        
        // 写入后端存储
        blk_aio_pwritev(ns->blk, data_offset, req->qsg.size,
                        &req->qsg, 0, nvme_rw_cb, req);
    } else {
        // 从后端存储读取
        blk_aio_preadv(ns->blk, data_offset, req->qsg.size,
                       &req->qsg, 0, nvme_rw_cb, req);
    }
}

static void nvme_rw_cb(void *opaque, int ret)
{
    NvmeRequest *req = opaque;
    NvmeCtrl *n = req->sq->ctrl;
    
    if (ret) {
        nvme_set_error(req, NVME_INTERNAL_DEV_ERROR);
    } else if (req->cmd.opcode == NVME_CMD_READ) {
        // DMA 写入数据到 Guest
        if (nvme_dma_write(n, req->buf, req->qsg.size,
                           &req->cmd, req)) {
            nvme_set_error(req, NVME_INTERNAL_DEV_ERROR);
        }
    }
    
    // 发送完成
    nvme_post_cqe(n, req);
}
```

## 调试和追踪

### Trace Events

```c
// trace-events 文件
nvme_mmio_read(uint64_t addr, unsigned size, uint64_t val) "addr 0x%"PRIx64" size %u val 0x%"PRIx64
nvme_mmio_write(uint64_t addr, unsigned size, uint64_t val) "addr 0x%"PRIx64" size %u val 0x%"PRIx64
nvme_mmio_doorbell_sq(uint16_t qid, uint16_t tail) "qid %u tail %u"
nvme_mmio_doorbell_cq(uint16_t qid, uint16_t head) "qid %u head %u"
nvme_irq_assert(uint16_t cqid, uint16_t vector) "cqid %u vector %u"
nvme_sq_cmd(uint16_t qid, uint16_t head, uint8_t opc) "qid %u head %u opc 0x%x"
nvme_post_cqe(uint16_t cqid, uint16_t cid) "cqid %u cid %u"
nvme_rw(const char *op, uint32_t nlb, uint64_t size, uint64_t slba) "%s nlb %u size 0x%"PRIx64" slba 0x%"PRIx64
```

### 启用追踪

```bash
# 启动 FEMU 时启用追踪
qemu-system-x86_64 \
    -trace "nvme_*" \
    -trace "pci_*" \
    ...

# 或使用 trace 文件
echo "nvme_mmio_*" > trace-events.txt
echo "nvme_irq_*" >> trace-events.txt
qemu-system-x86_64 -trace events=trace-events.txt ...
```

## FEMU 扩展

### SSD 时序模拟

```c
// hw/block/femu/ftl.c
static void femu_process_write(FemuCtrl *n, NvmeRequest *req)
{
    // 1. 计算 NAND 闪存延迟
    uint64_t nand_lat = calc_nand_write_latency(req);
    
    // 2. GC 开销
    if (need_garbage_collection(n)) {
        nand_lat += calc_gc_latency(n);
    }
    
    // 3. 延迟注入
    timer_mod(n->io_timer, qemu_clock_get_ns(QEMU_CLOCK_VIRTUAL) + nand_lat);
}
```

## 启动配置

```bash
# 基本 FEMU 配置
qemu-system-x86_64 \
    -enable-kvm \
    -cpu host \
    -smp 4 \
    -m 4G \
    -device nvme,id=nvme0,serial=femu001 \
    -drive file=nvme.img,if=none,id=nvm,format=raw \
    -device nvme-ns,drive=nvm,bus=nvme0,nsid=1 \
    -nographic

# 配置多个 namespace
-device nvme,id=nvme0,serial=femu001 \
-drive file=ns1.img,if=none,id=nvm1 \
-device nvme-ns,drive=nvm1,bus=nvme0,nsid=1 \
-drive file=ns2.img,if=none,id=nvm2 \
-device nvme-ns,drive=nvm2,bus=nvme0,nsid=2

# 配置 MSI-X 向量数量
-device nvme,id=nvme0,serial=femu001,msix_qsize=33
```

## 总结

FEMU 的 PCIe/NVMe 实现关键点：

1. **PCIe 设备模型**：使用 QEMU 的 PCIe 框架，实现标准 PCIe Endpoint
2. **BAR 映射**：通过 MemoryRegion 实现 MMIO 寄存器访问
3. **MSI-X 中断**：使用 QEMU 的 MSI-X API 注入中断到 Guest
4. **DMA 操作**：通过 PCI DMA API 访问 Guest 物理内存
5. **队列处理**：实现 NVMe 的 SQ/CQ Doorbell 机制
6. **异步 I/O**：使用 QEMU block layer 的异步接口

参见：
- [NVMe 实现](nvme.md)
- [PCIe 驱动开发](driver-guide.md)
- [MSI-X 中断](../interrupts/msix.md)
