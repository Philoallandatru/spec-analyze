# 主机驱动与系统架构（Host Driver & System）

本目录站在**主机操作系统、平台固件与虚拟化层**的视角，串联 [PCIe 规范](../pcie/wiki/README.md) 与 [NVMe 规范](../nvme/CONCEPT_INDEX.md)，解释一块 NVMe SSD 从上电枚举、驱动接管、DMA 内存访问到高并发 I/O 与虚拟机直通的完整系统链路。

---

## 文档导航

1. **[上电、UEFI/ACPI 与 PCIe 枚举](boot-and-enumeration.md)**
   - 主板上电复位时序（`PERST#`）、UEFI 分配 BAR 地址空间、ACPI 表（`MCFG`/`_PRT`/`DMAR`）如何把硬件拓扑移交给操作系统。
2. **[Linux PCIe 与 NVMe 驱动模型](linux-pci-nvme-driver.md)**
   - `pci_driver` 生命周期、`nvme-pci` 驱动初始化、以及 Linux `blk-mq` 多队列块层如何与 NVMe SQ/CQ、CPU/NUMA 一一绑定。
3. **[IOMMU、DMA 重映射与地址翻译](iommu-and-dma.md)**
   - 物理地址/虚拟地址/IOVA 的区别、一致性与流式 DMA、IOMMU 硬件隔离，以及 `ATS`、`PRI`、`PASID` 加速机制。
4. **[中断重映射、VFIO 与 SR-IOV 虚拟化](interrupt-remapping-and-vfio.md)**
   - 中断如何穿透 Hypervisor 直达虚拟机（IRTE / Posted Interrupts）、VFIO 安全直通与用户态驱动（SPDK）原理。
5. **[主机侧性能调优与排错 SOP](host-debug-and-tuning.md)**
   - 真实工程中的高频瓶颈与排查套路：MPS/MRRS、IOMMU 开销、中断亲和性、链路重置与 `nvme-cli` / `lspci` 工具链。

---

## 推荐阅读路径

- **刚接手驱动开发**：`上电与枚举` → `Linux PCIe 与 NVMe 驱动模型` → `IOMMU 与 DMA`
- **做虚拟化 / 云原生存储**：`IOMMU 与 DMA` → `中断重映射、VFIO 与 SR-IOV` → `性能调优与排错`
- **日常性能调优 / 定位卡顿**：直接看 `主机侧性能调优与排错 SOP`，再按需回查对应章节。
