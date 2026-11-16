# 中断重映射、VFIO 与 SR-IOV 虚拟化

在云数据中心里，一台物理服务器上跑着几十个虚拟机（VM）或容器。**SR-IOV** 负责把一块物理 NVMe SSD 或网卡“切分”成多个虚拟设备，**VFIO** 负责把这些设备安全地“直通”给虚拟机或用户态进程，而 **中断重映射（Interrupt Remapping）** 则确保设备发出的 MSI-X 中断能精准、高速地直达目标虚拟机内部，而不会被 Hypervisor 反复拦截拖垮性能。

---

## 中断重映射（Interrupt Remapping）与 Posted Interrupts

传统 MSI/MSI-X 通过向 `0xFEExxxxx` 内存地址写一个 32 位数据来触发 CPU 中断。但在虚拟化环境下存在两个问题：
1. **安全性**：若不加校验，恶意直通设备可以伪造任意 `0xFEExxxxx` 写请求，向宿主机注入非法中断。
2. **性能**：每次虚拟机里的设备发中断，如果都要先退回给宿主机 Hypervisor（**VM-Exit**）再转发给 vCPU，高 IOPS 下 CPU 会被上下文切换吃光。

```text
普通虚拟中断：设备 MSI-X ──▶ IOMMU (查 IRTE 表) ──▶ 触发宿主机 VM-Exit ──▶ 注入虚拟机 vCPU (慢)
Posted Interrupts：设备 MSI-X ──▶ IOMMU (查 IRTE 表) ──▶ 直接写入 vCPU 的 PIR 内存页 ──▶ 若 vCPU 正在运行则零 VM-Exit 通知！
```

- **IRTE（Interrupt Remapping Table Entry）**：IOMMU 在接收到 MSI-X 写事务后，校验设备来源（BDF），再通过 IRTE 表把中断安全路由到目标物理 CPU 或虚拟机 vCPU。

---

## VFIO：安全的用户态与虚拟机直通框架

**VFIO（Virtual Function I/O）** 是 Linux 内核提供的标准设备直通框架，它必须依赖 **IOMMU** 才能工作：

1. **IOMMU Group（最小隔离粒度）**：硬件上如果多个设备共享同一组地址翻译或能互相发起不受控的 P2P 访问（未开启 ACS），它们会被划入同一个 **IOMMU Group**。安全规则要求：**同一个 Group 内的所有设备必须一起直通，不能拆开分给不同虚拟机**。
2. **两大核心场景**：
   - **QEMU/KVM 虚拟机直通**：把物理 NVMe 盘或 SR-IOV VF 直接交给 Guest OS，虚拟机原生加载 `nvme` 驱动，性能接近裸金属。
   - **用户态驱动（如 SPDK / DPDK）**：用户态进程通过 VFIO 拿到设备 BAR0 的映射权限和 IOVA 内存池，完全绕过内核 `blk-mq` 与中断处理，实现微秒级存储/网络访问。

```bash
# 将设备从内核 nvme 驱动解绑并绑定到 vfio-pci（标准直通准备流程）
echo "0000:01:00.0" > /sys/bus/pci/drivers/nvme/unbind
echo "vfio-pci" > /sys/bus/pci/devices/0000:01:00.0/driver_override
echo "0000:01:00.0" > /sys/bus/pci/drivers_probe
```

---

## SR-IOV 在 NVMe 中的落地与挑战

- **PF（Physical Function）与 VF（Virtual Function）**：PF 是全功能物理控制器，负责创建、分配虚拟化资源（队列数 VQ 与中断向量 VI）；VF 是轻量级虚拟控制器，每个 VF 拥有独立的 PCIe BDF、BAR0 和 Admin/I/O 队列对，直接透传给不同租户。
- **NVMe 虚拟化资源管理**：在 NVMe 2.1 中，主机通过 `Virtualization Management` 命令从主控制器的共享资源池中，动态把 VQ/VI 分配给各个从控制器（Secondary Controller）并置为 Online。
- **常见落地痛点**：
  - 当单设备 VF 数量超过 8 个时，上游 Root Port 与设备必须同时开启 **ARI**（否则 BDF 只能表示 8 个 Function）。
  - 必须在 Switch 和 Root Port 上开启 **ACS**，否则同一槽位下的多个 VF 可能互发 P2P 流量绕过 IOMMU，被迫缩进同一个 IOMMU Group 而无法独立分配。

---

## 关联与引申概念

- **SR-IOV / ARI / ACS 规范细节**：详见 [SR-IOV 详解](../pcie/wiki/advanced-features/sriov.md)、[ARI](../pcie/wiki/advanced-features/ari.md) 与 [ACS](../pcie/wiki/advanced-features/acs.md)。
- **NVMe 虚拟化资源池**：主/从控制器的 VQ/VI 分配状态机，详见 [控制器虚拟化资源](../nvme/concepts/controller-virtualization-resources.md)。
- **中断路由全链路**：INTx / MSI / MSI-X 在系统总线上的传递路径，详见 [中断路由机制](../pcie/wiki/interrupts/routing.md)。
- **IOMMU 基础**：IOVA 翻译与页表失效开销，详见 [IOMMU、DMA 重映射与地址翻译](iommu-and-dma.md)。
