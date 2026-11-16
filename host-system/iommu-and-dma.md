# IOMMU、DMA 重映射与地址翻译

**DMA（直接内存访问）** 让 NVMe SSD 无需 CPU 搬运就能直接读写系统内存；而 **IOMMU（I/O 内存管理单元，如 Intel VT-d、AMD-Vi、ARM SMMU）** 则是插在 PCIe 设备与物理内存之间的“专用翻译官与安检门”。它把设备发出的 **IOVA（I/O 虚拟地址）** 翻译成真实的 **HPA（主机物理地址）**，既防止恶意或故障设备越界踩踏内核内存，又让设备能安全地直通给虚拟机使用。

---

## 为什么需要 IOMMU？四类地址一眼分清

| 地址类型 | 缩写 | 谁在使用 | 含义说明 |
| :--- | :--- | :--- | :--- |
| **主机虚拟地址** | HVA | CPU 应用程序/内核 | 经过 CPU MMU 页表映射的连续虚拟地址。 |
| **主机物理地址** | HPA | 物理内存条 (DRAM) | 真实的物理内存地址；无 IOMMU 时设备必须直接用它。 |
| **客户机物理地址** | GPA | 虚拟机内部 OS | 虚拟机以为自己看到的“物理地址”，实际由 Hypervisor 映射为 HPA。 |
| **I/O 虚拟地址** | IOVA | PCIe 设备 (DMA) | 开启 IOMMU 后，设备在 TLP 中携带的地址，经 IOMMU 页表译为 HPA。 |

```text
无 IOMMU：NVMe SSD ──(TLP 带 HPA)──────────────────────────────▶ 物理内存 (无隔离，可越界)
有 IOMMU：NVMe SSD ──(TLP 带 IOVA)──▶ [ IOMMU 页表 + IOTLB ] ──▶ 物理内存 (仅限授权页)
```

---

## 内核驱动中的两类 DMA 映射

在 Linux 驱动中，不同生命周期的内存采用不同的 DMA 映射 API：

| 映射方式 | 典型函数 | 适用场景与特点 |
| :--- | :--- | :--- |
| **一致性 DMA (Coherent)** | `dma_alloc_coherent()` | **长期存在、CPU 与设备高频双向可见**的控制结构，如 NVMe 的 **SQ、CQ 环形队列内存**与小块元数据。分配时自动保证 Cache 一致性（无需每次手动刷 Cache）。 |
| **流式 DMA (Streaming)** | `dma_map_single()` / `dma_map_sg()` | **单次 I/O 临时使用**的用户数据缓冲区（如 4KB–1MB 读写页），拼装成 NVMe 的 **PRP List 或 SGL**。传输前/后由内核按方向（`TO_DEVICE` / `FROM_DEVICE`）同步 Cache 并建立/销毁 IOVA 映射。 |

> **SWIOTLB（软件回弹缓冲，Bounce Buffer）**：如果设备只支持 32 位寻址但数据位于 4GB 以上的物理内存，内核会被迫在低端内存分配一块临时缓冲区并执行一次 CPU 内存拷贝（`memcpy`），导致性能暴跌。

---

## 性能瓶颈与硬件加速：IOTLB、ATS、PRI、PASID

每次 DMA 都要查多级 IOMMU 页表会拖慢高 IOPS 性能，因此芯片和规范引入了以下加速机制：

1. **IOTLB（I/O TLB）**：IOMMU 内部缓存最近翻译过的 `IOVA → HPA` 映射，命中时几乎零开销；但在高并发随机小包下频繁 `dma_map/unmap` 会触发 **IOTLB 失效广播（Invalidation）** 造成明显性能损耗（这也是高性能服务器常配 `iommu=pt` 透传模式的原因）。
2. **ATS（Address Translation Services，地址翻译服务）**：允许 PCIe 设备（如高端网卡、GPU/AI 卡）在**设备本地自建 ATC 缓存**，提前向 IOMMU 查询并缓存翻译好的物理地址，后续发出的 TLP 带上 `AT=Translated` 标记直接绕过 IOMMU 查表。
3. **PRI（Page Request Interface，页请求接口）**：配合 ATS 使用。当设备访问尚未分配物理页的虚拟地址时，不必触发 DMA 错误，而是发 PRI 请求通知 OS **按需缺页调入（Page Fault）**。
4. **PASID（Process Address Space ID，进程地址空间 ID）**：在 TLP 前缀中携带 20-bit 进程编号，让同一块 PCIe 设备能同时安全地直接访问多个用户态进程的虚拟地址空间（**SVM，共享虚拟内存**）。

```bash
# 查看当前系统 IOMMU 工作模式（如是否启用透传 iommu=pt）及设备所属的 IOMMU Group
dmesg | grep -iE "DMAR|IOMMU|AMD-Vi"
ls -la /sys/bus/pci/devices/0000:01:00.0/iommu_group/devices/
```

---

## 关联与引申概念

- **配置空间使能**：设备发起任何 DMA 前必须开启 `Command[2] (Bus Master Enable)`，详见 [PCIe 配置空间](../pcie/wiki/configuration/config-space.md)。
- **NVMe 数据指针**：流式 DMA 生成的分散物理页如何通过 PRP/SGL 交给控制器，详见 [数据指针布局](../nvme/concepts/data-pointer-layouts.md)。
- **ACS 访问控制**：防止点对点（P2P）TLP 绕过 IOMMU 检查的关键开关，详见 [ACS 访问控制服务](../pcie/wiki/advanced-features/acs.md)。
- **虚拟机直通**：如何利用 IOMMU Group 把设备安全交给虚拟机或 SPDK，详见 [中断重映射、VFIO 与 SR-IOV 虚拟化](interrupt-remapping-and-vfio.md)。
- **硬件架构深潜**：Intel VT-d / AMD-Vi / ARM SMMUv3 页表与 `IORT` 细节，详见 [IOMMU 硬件架构深潜](iommu-vtd-smmuv3-deep-dive.md)。
