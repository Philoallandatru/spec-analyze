# IOMMU 硬件架构深潜：Intel VT-d / AMD-Vi / ARM SMMUv3

在 [IOMMU、DMA 重映射与地址翻译](iommu-and-dma.md) 与 [中断重映射、VFIO 与 SR-IOV 虚拟化](interrupt-remapping-and-vfio.md) 中，我们理解了 IOVA 与设备直通的基本概念。本篇进一步拆解三大主流服务器平台（**Intel VT-d**、**AMD-Vi**、**ARM SMMUv3**）的官方架构规范，对比它们的发现表、页表索引与中断重映射实现 `[VT-d §3–5 / SMMUv3 §3–5]`。

---

## 三大平台 IOMMU 架构对照表

| 架构维度 | Intel VT-d | AMD-Vi (IOMMU) | ARM SMMUv3 |
| :--- | :--- | :--- | :--- |
| **ACPI 发现表** | **`DMAR`**（DMA Remapping） | **`IVRS`**（I/O Virtualization Reporting） | **`IORT`**（I/O Remapping Table，关联 ITS） |
| **设备索引表** | **Root Table → Context Table**（按 Bus/Dev/Func 索引） | **Device Table**（按 16-bit DeviceID 线性索引） | **Stream Table**（线性或两级，按 **StreamID** 索引） |
| **进程上下文** | **PASID Table**（Scalable Mode，支持共享虚拟地址 SVA） | **DTE + PASID** | **CD（Context Descriptor）**（绑定不同 PASID/SubstreamID） |
| **IOVA 页表结构** | 4-level / 5-level（与 x86 EPT/Paging 兼容） | 4-level / 6-level（与 AMD-V NPT 兼容） | ARMv8 AArch64 Stage 1 + Stage 2 两阶段页表 |
| **中断重映射** | **IRTE**（Interrupt Remapping Table Entry）+ **Posted Interrupts** | **IRTE** + **AVIC / GALog** | **GICv3 ITS（Interrupt Translation Service）** |

---

## DMA 地址翻译硬件走查（以 VT-d Scalable Mode 与 SMMUv3 为例）

当 NVMe SSD 携带 `BDF`（PCIe Requester ID）和可选 `PASID` 发起一笔 DMA 读写请求（IOVA）时，IOMMU 硬件按如下路径查找物理内存页：

```text
  PCIe TLP (Requester ID = BDF, 可选 PASID, 地址 = IOVA)
         │
         ▼
  [ IOTLB 缓存命中？ ] ──(是)──▶ 直接返回 HPA (Host Physical Address)
         │ (否: TLB Miss)
         ▼
  ┌── Intel VT-d: Root Table[Bus] ──▶ Context Table[DevFn] ──▶ PASID Table[PASID]
  │
  └── ARM SMMUv3: Stream Table[StreamID] (STE) ──▶ Context Descriptor[PASID] (CD)
         │
         ▼
  遍历 4/5 级 IOVA 页表 (Stage 1: GVA→GPA / Stage 2: GPA→HPA) ──▶ 填入 IOTLB
```

- **双阶段翻译（Nested / Two-Stage Translation）**：在虚拟化嵌套或用户态直通（如 vIOMMU）场景下，Stage 1 负责 Guest 虚拟地址 → Guest 物理地址（由 Guest OS 管理），Stage 2 负责 Guest 物理地址 → Host 物理地址（由 Hypervisor 管理），硬件一次走查即可完成安全隔离。
- **IOTLB 失效广播**：驱动 `dma_unmap_*` 时，OS 向 IOMMU 命令队列（如 VT-d Invalidation Queue / SMMUv3 CMDQ）下发失效命令；若设备启用了 **ATS（Address Translation Services）**，IOMMU 还会自动向 SSD 发送 PCIe `Invalidate Request` 消息刷新设备本地 **ATC**。

---

## 为什么 `ATS` + `PRI` + `PASID` 是下一代存储/加速卡基石？

- **`PASID`（Process Address Space ID, 20-bit）**：在 PCIe TLP Prefix 中携带进程号，让同一颗 NVMe/CXL 控制器可以同时安全地直接 DMA 访问多个不同用户态进程的内存空间。
- **`ATS`（Address Translation Services）**：允许设备把翻译结果缓存在卡内 ATC，大幅减少高并发随机小 I/O 时的 IOMMU 页表遍历延迟。
- **`PRI`（Page Request Interface）**：当设备访问的页面尚未分配物理内存（缺页）时，设备可向主机发起 PRI 缺页请求，使主机无需预先 `pin` 住全部内存即可实现 **SVM/SVA（共享虚拟地址）**。

```bash
# 查看当前系统启用的 IOMMU 类型与设备所属的 IOMMU Group
dmesg | grep -iE "DMAR:|AMD-Vi:|smmu-v3" | head -n 10
ls -la /sys/kernel/iommu_groups/*/devices/
```

---

## 关联与引申概念

- **基础概念与内核参数**：`iommu=pt` 性能权衡与一致性/流式 DMA API，详见 [IOMMU、DMA 重映射与地址翻译](iommu-and-dma.md)。
- **虚拟化与直通**：VFIO 隔离组与 Posted Interrupts 原理，详见 [中断重映射、VFIO 与 SR-IOV 虚拟化](interrupt-remapping-and-vfio.md)。
- **ACPI 发现表**：`DMAR`/`IVRS`/`IORT` 在开机阶段如何由固件构建，详见 [ACPI 6.5 与 UEFI 2.10 PCIe/NVMe 交互机制](acpi-uefi-pcie-deep-dive.md)。
