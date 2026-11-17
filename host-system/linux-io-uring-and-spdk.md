# 高性能 I/O 路径对比：`blk-mq` vs `io_uring` Passthru vs SPDK

理解了 NVMe 队列对与 PCIe DMA 后，高性能存储开发最常面临的架构抉择是：**走内核标准块层（`blk-mq`）、`io_uring` 直通（`IORING_OP_URING_CMD`），还是彻底旁路内核的 SPDK 用户态轮询驱动？**

---

## 三种 I/O 栈全景对比

| 架构路径 | 控制权归属 | 上下文切换 / 中断开销 | 典型单核 IOPS 能力 | 核心适用场景 |
| :--- | :--- | :--- | :--- | :--- |
| **传统 `blk-mq` + `libaio`** | 内核块层（VFS/PageCache/调度器） | 每次系统调用 + MSI-X 中断 | 10–30 万 IOPS | 通用文件系统（ext4/XFS）、常规数据库 |
| **`io_uring` + `NVMe Passthru`** | 内核管理设备，用户态共享环形队列 | 极低（支持 `SQPOLL` 零系统调用 + `IOPOLL` 轮询） | 80–150 万 IOPS | 现代异步数据库、不想放弃内核安全隔离的极致存储引擎 |
| **SPDK（`vfio-pci` 用户态）** | **用户态完全独占整块 NVMe 盘** | **零系统调用、零中断、100% 绑核轮询** | **200–400 万+ IOPS** | 超高性能分布式存储后端（Ceph Crimson/NVMf Target）、高频交易 |

---

## `io_uring` Passthru（`/dev/ng0n1`）为什么能媲美 SPDK？

过去即使使用 `io_uring`，普通读写仍要穿过 Linux 块层的 `bio` 分配、合并与安全检查。Linux 5.19+ 引入了 **`/dev/ngXnY`（NVMe Generic 字符设备）** 与 **`IORING_OP_URING_CMD`**：
- 应用程序直接在 `io_uring` 的 **SQE（128 字节大条目模式）** 中填入原生的 64 字节 NVMe 命令（如 ZNS `Zone Append`、FDP `Write` 或普通 `Read/Write`）。
- 内核 `nvme-pci` 驱动几乎**零转换**直接把该命令下发到硬件 SQ，同时保留了内核态 IOMMU 与多进程共享同一块盘的权限管控。

```text
  blk-mq 路径     : App ─(syscall)─▶ VFS/Block Layer (bio分配/调度) ─▶ nvme-pci ─▶ 硬件 SQ
  io_uring 路径   : App ─(共享内存环)─▶ nvme-pci (直通 /dev/ng0n1) ───────────────▶ 硬件 SQ
  SPDK 路径       : App (用户态轮询线程 + DPDK大页内存) ──(vfio-pci直写BAR0门铃)──▶ 硬件 SQ
```

```bash
# 查看内核暴露的 NVMe 块设备 (/dev/nvme0n1) 与通用直通字符设备 (/dev/ng0n1)
ls -l /dev/nvme* /dev/ng*
```

---

## 关联与引申概念

- **驱动核心架构**：`nvme-pci` 如何把 `blk-mq` 映射到硬件队列对，详见 [Linux PCIe 与 NVMe 驱动模型](linux-pci-nvme-driver.md)。
- **SPDK 隔离基石**：SPDK 如何借助 `vfio-pci` 与 IOMMU 实现用户态安全 DMA，详见 [中断重映射、VFIO 与 SR-IOV 虚拟化](interrupt-remapping-and-vfio.md)。
