# FEMU 代码概览

> **一句话总结**：FEMU（Full-system Emulated NVMe）是基于 QEMU 的全系统 NVMe SSD 模拟器——它在虚拟机内部完整复现了 PCIe Endpoint、NVMe 控制器、Doorbell 队列与内部闪存（FTL/NAND）延迟模型，让开发者无需真实硬件固件即可开展 SSD 与存储栈实验。

---

## 系统架构与源码结构

FEMU 在虚拟化链路中充当 **Guest OS 驱动** 与 **Host 存储后端** 之间的协议转换器：

```
  ┌──────────────────────────────────┐
  │  Guest OS (Linux NVMe Driver)    │  ◄── 发起 MMIO 读写、配置队列、等待 MSI-X
  └───────────────┬──────────────────┘
                  │ PCIe TLP (模拟)
                  ▼
  ┌──────────────────────────────────┐
  │  QEMU / FEMU (hw/block/nvme.c)   │  ◄── PCIe 配置空间、BAR0 MMIO、PRP DMA、Doorbell
  │  └─ FTL 扩展 (hw/block/femu/)    │  ◄── NAND 读写延迟建模、垃圾回收 (GC) 时序注入
  └───────────────┬──────────────────┘
                  │ 异步块 I/O (blk_aio_preadv/pwritev)
                  ▼
  ┌──────────────────────────────────┐
  │  Host Storage (镜像文件 / 内存)   │
  └──────────────────────────────────┘
```

### 核心目录结构

| 路径 | 职责说明 |
| :--- | :--- |
| `hw/block/nvme.c` & `nvme.h` | NVMe PCIe 控制器核心实现（BAR0、Doorbell、Admin/IO 命令、DMA、MSI-X） |
| `hw/block/nvme-ns.c` | Namespace（命名空间）管理与块设备后端绑定 |
| `hw/block/femu/ftl.c` | FTL（Flash Translation Layer，闪存转换层）与延迟计算 |
| `hw/block/femu/bbssd.c` | BlackBox SSD（黑盒 SSD）模拟模式 |
| `hw/block/femu/oc12.c` | Open-Channel 1.2 协议支持 |
| `include/block/nvme.h` | NVMe 规范寄存器与命令结构体定义 |

---

## PCIe 设备模型与 BAR0 寄存器模拟

FEMU 利用 QEMU 的 **QOM（QEMU Object Model）** 将 NVMe 控制器注册为标准 PCIe Endpoint（`TYPE_PCI_DEVICE` + `INTERFACE_PCIE_DEVICE`），在 `nvme_realize()` 中完成硬件初始化：

1. **配置空间与能力链表**：设置 Vendor ID（如 Intel `0x8086`）、Device ID（`0x5845`）、Class Code（`0x010802`，NVM Express），并依次挂载 **PCIe Capability**、**Power Management Capability** 与 **MSI-X Capability**（`msix_init_exclusive_bar` 分配独立 BAR 存放 MSI-X Table 与 PBA）。
2. **BAR0 MMIO 映射**：通过 `memory_region_init_io()` 与 `pci_register_bar()` 注册 64-bit BAR0，将 Guest 的寄存器读写路由到 `nvme_mmio_read/write()`：

| MMIO 偏移范围 | Guest 操作 | FEMU 内部响应 |
| :--- | :--- | :--- |
| `0x0000–0x003F` | 读控制寄存器（`CAP`, `VS`, `CSTS` 等） | 直接返回 `n->bar` 结构体对应字段 |
| `0x000C / 0x0010` | 写 `INTMS` / `INTMC` | 更新中断掩码位并同步刷新 MSI-X 掩码状态 |
| `0x0014` (`CC`) | 写 Controller Configuration | 检测 `CC.EN` 0→1 时调用 `nvme_start_ctrl()`；1→0 时清理控制器 |
| `>= 0x1000` | 写 Doorbell 寄存器（只写，读返回 0） | 调用 `nvme_process_db()` 触发 SQ 命令拉取或更新 CQ Head |

---

## 核心数据路径：Doorbell、PRP DMA 与异步 I/O

当 Guest 提交一条读写命令时，FEMU 内部按以下 5 步完成闭环处理：

```
1. Doorbell 触发: Guest 写 0x1000+ 偏移 -> nvme_process_db() 解析 qid 与 SQ/CQ 类型
       │
       ▼
2. 拉取命令: nvme_sq_process() 通过 pci_dma_read 从 Guest SQ 拷贝 64B NvmeCmd
       │
       ▼
3. 解析 PRP: nvme_map_prp() 解析 cmd->prp1/prp2 (及 PRP List) 构建 QEMUSGList
       │
       ▼
4. 后端执行 & 延迟注入:
   ├─ 标准路径: blk_aio_preadv / pwritev 异步读写 Host 镜像
   └─ FEMU FTL: 计算 NAND 页编程/读取与 GC 开销，通过 timer_mod() 延迟触发完成
       │
       ▼
5. 完成与中断: nvme_post_cqe() 写回 16B CQE、翻转 Phase Tag -> msix_notify() 注入中断
```

> **PRP（Physical Region Page）**：NVMe 用于描述非连续物理内存页的机制。单页以内直接使用 `prp1`；跨两页使用 `prp1 + prp2`；超过两页时 `prp2` 指向一张存放后续物理页地址的 **PRP List**。

---

## 常用启动参数与 Trace 调试

通过 QEMU 命令行即可挂载 FEMU NVMe 设备并开启内核级事件追踪（无需重新编译即可观察 MMIO 与中断）：

```bash
# 启动带 NVMe 设备及双 Namespace 的虚拟机，并开启 nvme_* trace 事件
qemu-system-x86_64 \
    -enable-kvm -cpu host -smp 4 -m 4G -nographic \
    -device nvme,id=nvme0,serial=femu001,msix_qsize=33 \
    -drive file=nvme.img,if=none,id=nvm1,format=raw \
    -device nvme-ns,drive=nvm1,bus=nvme0,nsid=1 \
    -trace "nvme_mmio_*" -trace "nvme_irq_*"
```

常用 trace 事件包括 `nvme_mmio_doorbell_sq`、`nvme_sq_cmd`、`nvme_rw`、`nvme_post_cqe` 与 `nvme_irq_assert`，可用于精确追踪单条 I/O 的生命周期。

---

## 关联与引申概念

- **FTL（Flash Translation Layer）**：SSD 内部将逻辑块地址（LBA）映射到物理闪存页（PPA）的固件层，负责磨损均衡、坏块管理与垃圾回收（GC），是 FEMU 延迟模拟的核心。
- **QEMU MemoryRegion**：QEMU 模拟设备 MMIO/PIO 的抽象对象，负责截获 Guest 的物理地址访问并回调设备的 `read`/`write` 函数。
- **延伸阅读**：
  - [NVMe over PCIe 实现](nvme.md)
  - [PCIe 驱动开发指南](driver-guide.md)
  - [PCIe 中断实现](interrupts.md)
  - [PCIe 配置空间实现](config-space.md)
  - [MSI-X 协议详解](../interrupts/msix.md)
