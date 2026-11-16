# SR-IOV 虚拟化详解（Single Root I/O Virtualization）

> **一句话总结**：SR-IOV 允许一张物理 PCIe 设备（如网卡或 NVMe SSD）在硬件内部自我分裂出几十上百个独立的"虚拟功能（VF）"，让每个虚拟机（VM）像独占物理卡一样直接读写硬件，彻底绕过 Hypervisor 软件模拟瓶颈。

如果把传统虚拟化比作"合租公寓只有一个前台（Hypervisor）代收所有人的快递"，每个包裹都要经过前台拆包转交，效率极低；SR-IOV 则是把这栋物理大楼直接隔成几十间带独立门牌和独立电梯的办公室（VF），物业经理（PF）只在入驻时分配好房间，之后每个租户（VM）直接刷卡进门收发快递，速度和独享整栋楼几乎没有区别。

---

## 为什么需要 SR-IOV？

```text
【传统软件模拟 I/O】                    【SR-IOV 硬件直通】
  VM 1      VM 2      VM 3              VM 1      VM 2      VM 3
   │         │         │                 │         │         │
   └────┬────┴────┬────┘                VF 1      VF 2      VF 3  ← 独立 BAR / 队列 / MSI-X
        ▼         ▼                      └────┬────┴────┬────┘
   Hypervisor (Virtio/模拟)                   ▼         ▼
        │                              Physical Function (PF) ← 仅负责管理控制面
        ▼                                     │
    物理 PCIe 设备                         物理 PCIe 设备
(CPU 开销高、上下文切换频繁)          (数据面零 Hypervisor 介入、接近原生性能)
```

---

## 核心概念：PF vs VF

| 对比维度 | PF（Physical Function，物理功能） | VF（Virtual Function，虚拟功能） |
| :--- | :--- | :--- |
| **定位** | 设备主控功能，完整 PCIe 功能 | 轻量级 PCIe 功能，专注高速数据面 |
| **控制权限** | 拥有完整配置空间与 SR-IOV 能力，负责创建、销毁、重置 VF，分配资源配额 | 简化配置空间（无完整电源/链路控制），不能创建其他 VF |
| **归属方** | 宿主机（Host OS / Hypervisor） | 直接通过 VFIO 透传给虚拟机（Guest VM）或容器 |
| **独立硬件资源** | 完整 BAR、MSI-X、DMA 引擎 | 拥有**独立的 BAR 窗口、独立 MSI-X 中断与独立 Requester ID（BDF）** |

---

## SR-IOV 架构与核心寄存器

SR-IOV 通过位于 PF 扩展配置空间的 **SR-IOV Extended Capability（Capability ID `0x0010`）** 实现：

| 偏移 | 字段 | 作用 |
| :--- | :--- | :--- |
| `0x04` | `SR-IOV Capabilities` | 声明是否支持 VF Migration 及 ARI 等能力 |
| `0x08` | `SR-IOV Control` | **Bit 0 `VF Enable`**：启停 VF；**Bit 3 `VF MSE`**：使能 VF 内存空间；**Bit 4 `ARI Capable Hierarchy`**：启用 ARI 寻址模式 |
| `0x0C–0x10` | `Initial VFs` / `Total VFs` / `Num VFs` | 分别为硬件最大 VF 数、当前可用总数、以及软件实际写入启用的 VF 数量 |
| `0x14–0x16` | `First VF Offset` / `VF Stride` | 用于计算每个 VF 的唯一 BDF（Routing ID） |
| `0x24–0x38` | `VF BAR0–BAR5` | 定义**单个 VF** 的 BAR 大小与类型；启用 $N$ 个 VF 时，系统为这 $N$ 个 VF 连续分配 $N$ 份独立的 MMIO 空间 |

### VF 的 BDF 编号计算公式

$$\text{VF}[i]\ \text{BDF} = \text{PF BDF} + \text{First VF Offset} + (i \times \text{VF Stride})$$

- **未开启 ARI**：传统每设备最多 8 个 Function，若 PF 为 `01:00.0`、`Offset = 1`、`Stride = 1`，则 `VF[0] = 01:00.1`，`VF[6] = 01:00.7`，`VF[7]` 会进位到 `01:01.0`（且最多 7 个 VF 时受限于非 ARI 枚举）。
- **开启 ARI**：结合 [ARI（备选路由 ID）](ari.md)，单设备可扩展至 **255 个 VF**（Function 1–255），`lspci` 仍按 `Bus:Dev.Func` 格式进位显示。

---

## 完整的虚拟化隔离栈：SR-IOV + ARI + ACS + IOMMU/VFIO

在生产云环境中，SR-IOV 从来不是孤立工作的，它依赖一整套 PCIe/系统硬件协同：

```text
VM 1 (Guest Driver) ──DMA (GPA)──▶ VF 1 (01:00.1)
                                     │ (携带独立 Requester ID)
                                     ▼
                              Switch / Root Port
                              [ACS 启用：禁止 VF1 与 VF2 私下 P2P，强制上行]
                                     │
                                     ▼
                              Root Complex (IOMMU / VT-d / AMD-Vi)
                              [按 Requester ID 查表：GPA → HPA 地址转换 + 隔离校验]
                                     │
                                     ▼
                              Host 物理内存 (仅限 VM 1 所属内存页)
```

1. **ARI**（[ari.md](ari.md)）：突破单设备 8 个 Function 上限，支持单卡切出数十至上百个 VF。
2. **ACS**（[acs.md](acs.md)）：保证每个 VF 处于独立的 **IOMMU Group**，防止同卡不同 VF 之间通过内部或 Switch P2P 越权互访。
3. **IOMMU + VFIO**：IOMMU 根据 TLP 携带的 VF Requester ID 执行 DMA 地址重映射与中断重映射；Linux `vfio-pci` 驱动将 VF 安全暴露给 QEMU/KVM。

---

## Linux 常用操作命令

```bash
# 1. 查看 PF 支持的最大 VF 数与当前启用数
cat /sys/bus/pci/devices/0000:01:00.0/sriov_totalvfs
cat /sys/bus/pci/devices/0000:01:00.0/sriov_numvfs

# 2. 启用 8 个 VF（关闭时写入 0）
echo 8 > /sys/bus/pci/devices/0000:01:00.0/sriov_numvfs

# 3. 查看生成的 VF 及 SR-IOV 能力详情
lspci -vvv -s 01:00.0 | grep -A 6 "Single Root I/O Virtualization"

# 4. 将 VF (01:00.1) 解绑原驱动并绑定到 vfio-pci 以直通给虚拟机
echo 0000:01:00.1 > /sys/bus/pci/devices/0000:01:00.1/driver/unbind
echo vfio-pci > /sys/bus/pci/devices/0000:01:00.1/driver_override
echo 0000:01:00.1 > /sys/bus/pci/drivers/vfio-pci/bind
```

---

## 局限性与演进方向

- **热迁移（Live Migration）难度高**：由于 VF 直接由硬件管理状态，传统 SR-IOV 虚拟机难以像纯软件 Virtio 那样直接冻结内存迁移（需依赖硬件 Dirty Page Tracking / VFIO Migration 协议或 Virtio-net + SR-IOV Failover 方案）。
- **细粒度动态切分受限**：VF 数量和资源通常在启动时静态划分；新一代 **SIOV（Scalable IOV）** 结合 **PASID（进程地址空间 ID）** 允许按工作队列（WQ）而非整颗 PCIe Function 进行更轻量的虚拟化切分。

---

## 关联与引申概念

- **ARI（Alternative Routing-ID Interpretation）**（[ari.md](ari.md)）：SR-IOV 突破 7 个 VF 上限的必备搭档。
- **ACS（Access Control Services）**（[acs.md](acs.md)）：确保每个 VF 获得独立 IOMMU Group、防止 VF 间绕过 IOMMU 互访的安全基石。
- **ATS / PRI / PASID**：Address Translation Services（设备侧地址转换缓存）、Page Request Interface（设备缺页请求）与 Process Address Space ID（进程地址空间标识），支持虚拟机和容器更高效地共享虚拟内存（SVA）与扩展到 SIOV。
- **BARs 与配置空间**（[../configuration/bars.md](../configuration/bars.md)、[../configuration/config-space.md](../configuration/config-space.md)）：PF 中的 `VF BAR0–BAR5` 决定了所有 VF 的 MMIO 空间映射布局。
- **MSI-X 中断**（[../interrupts/msix.md](../interrupts/msix.md)）：每个 VF 拥有独立的 MSI-X 表，使中断可以直接注入目标 VM 的 vCPU（Posted Interrupts）。
