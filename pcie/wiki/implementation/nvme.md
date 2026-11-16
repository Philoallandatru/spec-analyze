# NVMe over PCIe 实现

> **一句话总结**：NVMe（Non-Volatile Memory Express）是专为 PCIe 打造的高速存储协议——它用"多车道无锁环形队列 + Doorbell 门铃 + MSI-X 直连中断"取代了传统 SATA/AHCI 的单队列串行机制，充分榨干 PCIe 的全双工带宽。

---

## NVMe 多队列架构与 PCIe 配置

NVMe 控制器在 PCIe 配置空间中声明 Class Code 为 `0x010802`（NVM Express），必须支持 **Power Management** 与 **MSI-X** Capability。其核心并发能力源于 **成对的环形队列**：

```
  CPU Core 0          CPU Core 1          ...         CPU Core N
      │                   │                               │
      ▼                   ▼                               ▼
  [Admin SQ/CQ 0]     [I/O SQ/CQ 1]       ...         [I/O SQ/CQ N]
      │                   │                               │
      └───────────────────┴───────────┬───────────────────┘
                                      ▼
                            NVMe PCIe Controller
                                      │
                                      ▼
                       各自绑定独立 MSI-X 中断向量
```

- **Admin Queue（Queue 0）**：全局唯一的一对 SQ/CQ，用于控制器识别（Identify）、创建/删除 I/O 队列、固件更新等管理命令。
- **I/O Queues（Queue 1–65535）**：每个 CPU 核心可独占一对 **SQ（Submission Queue，提交队列，64B/条）** 与 **CQ（Completion Queue，完成队列，16B/条）**，彻底消除多核锁竞争。

---

## BAR0 核心寄存器与 Doorbell 布局

NVMe 设备将全部控制寄存器与门铃（Doorbell）映射到 **64-bit BAR0**（通常 8KB–16KB）：

| BAR0 偏移 | 寄存器名称 | 位宽 | 核心作用 |
| :--- | :--- | :--- | :--- |
| `0x0000` | `CAP` (Controller Capabilities) | 64-bit | 报告最大队列深度（MQES）、**Doorbell 步长（DSTRD）**、超时时间等 |
| `0x0008` | `VS` (Version) | 32-bit | NVMe 规范版本号（如 `0x00010400` 表示 1.4） |
| `0x000C / 0x0010` | `INTMS / INTMC` | 32-bit | 中断屏蔽置位 / 清零寄存器 |
| `0x0014` | `CC` (Controller Configuration) | 32-bit | 配置页大小（MPS）、队列条目大小（IOSQES/IOCQES）及使能位 `EN` |
| `0x001C` | `CSTS` (Controller Status) | 32-bit | 控制器就绪状态 `RDY`、致命错误 `CFS` |
| `0x0024` | `AQA` (Admin Queue Attributes) | 32-bit | 设置 Admin SQ 与 Admin CQ 的队列深度 |
| `0x0028 / 0x0030` | `ASQ / ACQ` | 64-bit | Admin SQ 与 Admin CQ 在主机内存中的 **DMA 基地址** |
| `0x1000+` | **Doorbell 寄存器区** | 32-bit/个 | 按 `DSTRD` 步长排列每对队列的 `SQ Tail Doorbell` 与 `CQ Head Doorbell` |

---

## 控制器初始化流程

Linux `nvme-pci` 驱动接管设备后，按以下顺序完成控制器握手：

1. **PCIe 基础使能**：`pci_enable_device_mem()` → `pci_request_mem_regions()` → `dma_set_mask_and_coherent(64bit)` → `pci_ioremap_bar(pdev, 0)` → `pci_set_master()`。
2. **中断分配**：先为 Admin Queue 分配 1 个 MSI-X 向量，待识别控制器支持的队列数后再扩展分配 I/O MSI-X 向量。
3. **复位控制器**：读取 `CAP` 获取 `DSTRD` 与最大队列深度；若 `CC.EN=1`，先写 `CC.EN=0` 并轮询等待 `CSTS.RDY` 变为 `0`。
4. **建立 Admin Queue**：用 `dma_alloc_coherent()` 分配物理连续且缓存一致的 Admin SQ/CQ 内存，将大小写入 `AQA`，物理地址写入 `ASQ/ACQ`。
5. **使能控制器**：配置 `CC`（页大小 `MPS`、SQ 条目 64B `IOSQES=6`、CQ 条目 16B `IOCQES=4`、`CC.EN=1`），轮询等待 `CSTS.RDY == 1`。
6. **识别与建队列**：通过 Admin Queue 发送 `Identify Controller / Namespace`，随后发送 `Create I/O CQ` 与 `Create I/O SQ`（注意：**必须先建 CQ 再建 SQ**）。

---

## 核心 I/O 路径（Submit → DMA → Complete）

NVMe 的精妙之处在于**驱动与控制器通过共享内存环形缓冲区 + Doorbell 单向写完成通信**，全程无需耗时的 MMIO 读取：

```
  [主机内存]                                        [NVMe 控制器]
  1. 驱动写入 64B 命令到 SQ[sq_tail]
     (含 PRP/SGL 数据缓冲区物理地址)
  2. 写 BAR0 SQ Tail Doorbell ───────────────────► 3. 收到门铃，DMA 读取 SQ 命令
                                                   4. DMA 读写主机数据页 (PRP/SGL)
                                                   5. 写 16B CQE 到 CQ[cq_tail]
                                                      (写入当前 Phase Tag 标志位)
  7. 收到中断，轮询 CQ[cq_head]    ◄─────────────── 6. 触发对应 MSI-X 中断
     直到 Phase Tag 不匹配为止
  8. 写 BAR0 CQ Head Doorbell ───────────────────► 9. 回收 CQ 槽位
```

> **Phase Tag（相位标志）**：CQE 中 `status` 字段的最低位（bit 0）。初始时驱动期望 `phase = 1`，控制器写入新 CQE 时置该位为 `1`；当 `cq_head` 绕回队列头部 `0` 时，驱动与控制器同时翻转 `phase` 为 `0`。借此机制，驱动无需读取任何 MMIO 寄存器即可无锁判断哪些 CQE 是新完成的。

---

## 高性能调优机制

- **CPU 亲和与多队列（`blk-mq`）**：通过 `blk_mq_pci_map_queues()` 将每个 CPU 核心的软件请求队列直接绑定到专属的硬件 SQ/CQ 与 MSI-X 向量，实现跨核零锁扩展。
- **轮询模式（IOPoll）**：在超低延迟场景（如 Optane / 高端 NVMe SSD）下，为特定队列关闭中断，由应用通过 `io_uring` IOPoll 直接死循环扫描 CQ 的 Phase Tag，消除中断上下文切换延迟。
- **批量提交（Batching）**：驱动连续向 SQ 写入多条命令后，只写一次 `SQ Tail Doorbell` 通知硬件，大幅减少 PCIe MMIO 写事务开销。

---

## 常用调试与压测命令

```bash
# 列出 NVMe 设备及控制器/命名空间详情
nvme list
nvme id-ctrl /dev/nvme0

# 查看当前块设备启用的硬件队列数量
cat /sys/block/nvme0n1/queue/nr_hw_queues

# 使用 fio 进行 4K 随机读压测（Direct I/O，队列深度 32，4 并发）
fio --name=randread --ioengine=libaio --iodepth=32 \
    --rw=randread --bs=4k --direct=1 --size=1G --numjobs=4 --filename=/dev/nvme0n1
```

---

## 关联与引申概念

- **SGL（Scatter-Gather List）**：除 PRP 外，NVMe 还支持 SGL 来描述任意长度与对齐方式的非连续内存段，常用于企业级 SSD 与 NVMe-oF（NVMe over Fabrics）。
- **NVMe-oF**：将 NVMe 队列模型扩展到网络传输层（RDMA、TCP、FC），实现跨网络的低延迟共享存储访问。
- **延伸阅读**：
  - [PCIe 驱动开发指南](driver-guide.md)
  - [FEMU 代码概览](femu-overview.md)
  - [PCIe 中断实现](interrupts.md)
  - [PCIe 配置空间实现](config-space.md)
  - [MSI-X 协议详解](../interrupts/msix.md)
