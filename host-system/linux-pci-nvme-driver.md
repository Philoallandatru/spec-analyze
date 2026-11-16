# Linux PCIe 与 NVMe 驱动模型

在 Linux 内核中，**PCIe 子系统（PCI Core）**负责总线发现与资源供给，**NVMe 驱动（`nvme-pci`）**负责把控制器接入块设备层，而 **多队列块层（`blk-mq`）**则负责把来自不同 CPU 核心的读写请求无锁、并行地塞进 NVMe 的提交队列（SQ）。如果把这套架构比作大型物流中心，**PCI Core** 是修路通电的基建部门，**`nvme-pci`** 是站台调度员，**`blk-mq`** 则是让每条车道互不堵车的智能分流系统。

---

## `pci_driver` 初始化标准流水线

当内核发现设备的 `Vendor ID / Device ID` 或 `Class Code`（NVMe 为 `0x010802`）与驱动匹配时，会触发 `nvme_probe()`，核心步骤严格遵循以下顺序：

```text
1. pci_enable_device_mem()   ──▶ 唤醒设备至 D0 状态，使能 BAR 内存解码
2. pci_set_master()          ──▶ 置位 Command[2] (Bus Master)，允许 SSD 发起 DMA
3. dma_set_mask_and_coherent(64) ──▶ 声明支持 64 位 DMA 寻址，避免 32 位回退拷贝
4. pci_ioremap_bar(BAR0)     ──▶ 将 NVMe 控制器寄存器 (CAP/CC/Doorbell) 映射到内核虚拟地址
5. 初始化 Admin Queue        ──▶ 分配 ASQ/ACQ 物理内存，写 CC.EN=1 等待 CSTS.RDY=1
6. Identify & 协商队列数     ──▶ 读取能力 (MDTS 等)，Set Features (07h) 申请 I/O 队列
7. pci_alloc_irq_vectors()   ──▶ 申请 MSI-X 向量，与每个 CPU 核心及 I/O CQ 绑定
8. 创建 I/O SQ/CQ 并注册 blk-mq ──▶ 暴露 /dev/nvme0n1 块设备节点
```

---

## `blk-mq` 与 NVMe 队列对的映射模型

早期的 Linux 块层只有一个全局请求锁，在多核 CPU 上会成为严重瓶颈。**`blk-mq`（Multi-Queue Block IO Queueing Mechanism）** 采用“两级队列”直接对齐 NVMe 的多队列硬件设计：

```text
  CPU 0 ──▶ [软件队列 ctx 0] ─┐
  CPU 1 ──▶ [软件队列 ctx 1] ─┴─▶ [硬件派发队列 hctx 0] ──▶ NVMe SQ 1 / CQ 1 (IRQ 向量 1)
  CPU 2 ──▶ [软件队列 ctx 2] ─┐
  CPU 3 ──▶ [软件队列 ctx 3] ─┴─▶ [硬件派发队列 hctx 1] ──▶ NVMe SQ 2 / CQ 2 (IRQ 向量 2)
```

- **无锁并发**：理想情况下每个 CPU 核心独占一组 `hctx` 与一对 `SQ/CQ`（且中断亲和性固定到该核心），提交和完成路径完全**跨核无锁**，并保持 **NUMA 本地内存亲和性**。
- **三类特殊队列（`blk-mq` 队列集合）**：现代 `nvme-pci` 会把控制器分配的队列划分为三池：
  1. **Default Queues**：常规读写队列（触发 MSI-X 中断）。
  2. **Read Queues**：独立读队列（防止大量写请求阻塞低延迟读请求）。
  3. **Poll Queues**：**轮询队列（无中断）**，配合 `io_uring` `IORING_SETUP_IOPOLL` 或 `hipri`，在极低延迟场景下由 CPU 主动轮询 CQ Phase Tag，省去中断上下文切换开销。

---

## 用户态直通与 `io_uring` 演进

- **内核路径（`blk-mq`）**：通用性强，支持文件系统（ext4/xfs）与 Page Cache，但存在系统调用与块层调度开销。
- **`io_uring` Passthrough（`/dev/ng0n1`）**：通过 `IORING_OP_URING_CMD` 直接把 NVMe 命令送入驱动队列，绕过传统块层解析，适合高性能数据库和 ZNS/KV 场景。
- **SPDK（用户态驱动）**：借助 **VFIO / UIO** 将 BAR0 直接 `mmap` 到用户态进程，完全绕过内核，以 100% CPU 轮询换取极致 IOPS 与微秒级尾延迟。

```bash
# 查看 NVMe 设备的驱动绑定、中断数量与 blk-mq 队列映射
ls -la /sys/block/nvme0n1/mq/
grep nvme0q /proc/interrupts
```

---

## 关联与引申概念

- **驱动开发细节**：PCIe 驱动注册、托管资源 `devm_*` 与电源管理回调，详见 [Linux PCIe 驱动开发指南](../pcie/wiki/implementation/driver-guide.md)。
- **NVMe 队列与门铃**：SQ/CQ 环形缓冲与 Phase Tag 无锁判断原理，详见 [队列对（Queue Pair）](../nvme/concepts/queue-pair.md) 与 [NVMe over PCIe 实现](../pcie/wiki/implementation/nvme.md)。
- **DMA 与 IOMMU**：`dma_set_mask` 和 `nvme_map_prp` 背后的地址翻译，详见 [IOMMU、DMA 重映射与地址翻译](iommu-and-dma.md)。
- **VFIO 用户态直通**：SPDK 如何安全地在用户态直接敲 Doorbell，详见 [中断重映射、VFIO 与 SR-IOV 虚拟化](interrupt-remapping-and-vfio.md)。
