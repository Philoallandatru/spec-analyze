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
6. **[IOMMU 硬件架构深潜：Intel VT-d / AMD-Vi / ARM SMMUv3](iommu-vtd-smmuv3-deep-dive.md)**
   - 三大平台发现表（`DMAR`/`IVRS`/`IORT`）、Context/Stream Table、多级页表走查与 `ATS`/`PRI`/`PASID` 硬件机制。
7. **[ACPI 6.5 与 UEFI 2.10：固件到操作系统的 PCIe/NVMe 交接](acpi-uefi-pcie-deep-dive.md)**
   - `_OSC` 原生热插拔/AER 权限谈判、`_HPP`/`_HPX`、`IORT` 以及 UEFI 启动期 NVMe 协议栈。
8. **[CXL 3.x 架构与协议复用](cxl-architecture-overview.md)**
   - 基于 PCIe 5.0/6.0 PHY 的 `CXL.io` / `CXL.cache` / `CXL.mem` 三协议复用、Type 1/2/3 设备与 HDM 内存池化。
9. **[UCIe 芯粒互连概览](ucie-die-to-die-overview.md)**
   - 先进封装内 Die-to-Die 互连如何复用 PCIe 6.0 / CXL 3.x 协议栈与 256B Flit。

---

## 推荐阅读路径

- **刚接手驱动开发**：`上电与枚举` → `Linux PCIe 与 NVMe 驱动模型` → `IOMMU 与 DMA`
- **做虚拟化 / 云原生存储**：`IOMMU 与 DMA` → `中断重映射、VFIO 与 SR-IOV` → `性能调优与排错`
- **日常性能调优 / 定位卡顿**：直接看 `主机侧性能调优与排错 SOP`，再按需回查对应章节。
