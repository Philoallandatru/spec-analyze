# 上电、UEFI/ACPI 与 PCIe 枚举

**上电枚举（Enumeration）** 是计算机通电后，由主板固件（BIOS/UEFI）充当“户籍登记员”，给所有 PCIe 设备分配总线号和内存门牌号（BAR），再通过 **ACPI 表** 把这份地图移交给操作系统的过程。没有这一步，操作系统根本不知道插槽里插了什么设备，也无法通过读写寄存器与 NVMe SSD 对话。

---

## 从通电到操作系统接管的完整时间线

```text
1. 电源与时钟就绪 (Power Good & 100 MHz REFCLK 稳定，>= 100ms)
       │
       ▼
2. 释放 PERST# (主板拉高复位信号) ──▶ 设备 PHY 开始 LTSSM 链路训练进入 L0
       │
       ▼
3. UEFI/BIOS 深度优先枚举 (DFS)
   • 通过 ECAM 读取所有 BDF 的 Vendor ID（非 0xFFFF 即存在设备）
   • 为 Bridge/Switch 分配 Primary / Secondary / Subordinate 总线号
   • 向每个设备的 BAR 写全 1 探测大小，在物理地址空间中切分并写入基址
       │
       ▼
4. UEFI 加载 NVMe 驱动读取启动分区，同时构建 ACPI 系统描述表
       │
       ▼
5. ExitBootServices() 移交控制权 ──▶ Linux/Windows 内核读取 ACPI 表接管设备
```

---

## 关键 ACPI 表：固件给操作系统的“交接清单”

**ACPI（高级配置与电源接口）** 是固件向操作系统描述硬件拓扑、电源与安全边界的标准表格集合。与 PCIe/NVMe 最相关的 4 张表如下：

| ACPI 表 | 全称 | 核心作用 |
| :--- | :--- | :--- |
| **`MCFG`** | Memory Mapped Configuration | 告诉 OS **ECAM（扩展配置空间）** 的物理内存基址与总线号范围，OS 据此用普通内存读写访问 4KB 配置空间。 |
| **`_PRT`** | PCI Routing Table | 描述传统 **INTx 中断引脚**（INTA#–INTD#）在主板上连接到 IOAPIC 的哪一根具体中断线。 |
| **`DMAR` / `IVRS`** | DMA Remapping / I/O Virt | Intel (`DMAR`) 与 AMD (`IVRS`) 专用，告知 OS 系统里有几个 **IOMMU** 硬件引擎、各自管辖哪些 PCIe BDF。 |
| **`EINJ`** | Error Injection | 允许在受控环境下通过固件接口向特定 PCIe/内存地址注入硬件错误，用于验证 OS 的 AER 恢复链路。 |

> **注**：在 ARM 服务器中，除 ACPI 外，嵌入式或定制平台也常用 **设备树（Device Tree, `.dts`）** 描述 PCIe Root Complex 的 ECAM 地址与中断映射。

---

## 操作系统接管时的资源重分配

Linux 内核启动后，通常直接信任固件分配好的资源，但遇到以下情况时内核会主动重新分配（`pci=realloc`）：
- **热插拔（Hot-Plug）**：开机时插槽为空，运行时新插入的 NVMe 盘需要动态申请总线号与 BAR 窗口，因此固件常会预先为支持热插拔的 Root Port 留出富余的 Bus 和 Memory 窗口（Padding）。
- **Resizable BAR（RBAR）**：显卡或大内存设备在 OS 阶段申请扩大 BAR 空间时，内核需要重新调整上游 Bridge 的地址窗口。

```bash
# 查看内核启动后看到的 PCIe 资源分配（BAR、Bus 窗口、Bridge 范围）
cat /proc/iomem | grep -iE "PCI|nvme"
dmesg | grep -iE "ACPI: MCFG|pci 0000:" | head -n 15
```

---

## 关联与引申概念

- **设备枚举与 BDF 编址**：DFS 遍历算法与 Type 0/1 配置头细节，详见 [PCIe 设备枚举](../pcie/wiki/architecture/enumeration.md)。
- **BAR 地址探测**：写全 `1` 回读探测空间大小的原理，详见 [BARs 详解](../pcie/wiki/configuration/bars.md)。
- **PERST# 复位时序**：上电稳定性与电气边带信号规范，详见 [PCIe 电气规范](../pcie/wiki/physical-layer/electrical-spec.md)。
- **控制器初始化**：OS 拿到 BAR0 地址后如何按 NVMe 协议启动控制器，详见 [控制器初始化](../nvme/concepts/controller-initialization.md)。
