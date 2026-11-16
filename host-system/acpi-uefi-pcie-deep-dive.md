# ACPI 6.5 与 UEFI 2.10：固件到操作系统的 PCIe/NVMe 交接

[上电、UEFI/ACPI 与 PCIe 枚举](boot-and-enumeration.md) 介绍了系统通电后枚举的宏观流程。本篇深入 **ACPI 6.5** 与 **UEFI 2.10** 规范，拆解固件（BIOS/UEFI）与操作系统（OSPM）之间围绕 **PCIe 控制权移交（`_OSC`）、热插拔参数（`_HPP`/`_HPX`）、ARM `IORT` 以及 UEFI 启动期 NVMe 协议栈** 的核心契约 `[ACPI 6.5 §6.2, §6.5 / UEFI 2.10 §13–14]`。

---

## ACPI `_OSC`：固件与 OS 的 PCIe 控制权谈判

现代服务器支持 **PCIe 原生热插拔（`pciehp`）**、**AER（高级错误报告）** 与 **PME（电源管理事件）**。但在上电初期，这些功能可能暂时由主板固件（SMM 模式）接管。Linux/Windows 启动时必须执行 Root Bridge 下的 **`_OSC`（Operating System Capabilities）** 控制方法，与固件谈判权限移交 `[ACPI 6.5 §6.2.11]`：

```text
  OS (Linux 内核)                                  平台固件 (ACPI AML / BIOS)
        │                                                    │
        ├── 调用 _OSC (请求控制: Native Hotplug, AER, PME) ─▶│
        │                                                    ├─ 检查 BIOS 设置 (是否允许 OS 接管)
        │◀─ 返回 _OSC 授权掩码 (Granted Control Bits) ───────┤
        │                                                    │
  * 若授权成功：Linux 启用 pciehp / aerdrv 原生驱动接管
  * 若固件拒绝：OS 禁止动对应寄存器，继续由固件 SMI/APEI 处理
```

> **常见工程坑**：很多服务器热插拔失灵或 `dmesg` 报 `AER: enabled with IRQ` 却收不到错误日志，往往不是驱动 Bug，而是 BIOS 里把热插拔/AER 设成了 `Firmware First`（固件优先），导致 `_OSC` 拒绝向 OS 移交控制权。

---

## 核心 ACPI 对象与表（补充）

| ACPI 表 / 对象 | 规范定位 | 核心作用 |
| :--- | :--- | :--- |
| **`_OSC`** | Root Bridge AML 方法 | 协商 OS 是否获得 Native Hotplug、PME、AER、LTR、DPC 等 PCIe 特性控制权 |
| **`_HPP` / `_HPX`** | 设备/槽位 AML 对象 | 为刚热插入的 PCIe 设备提供默认 **Cache Line Size、Latency Timer、AER 掩码与 SERR#/PERR#** 初始参数 |
| **`HMAT`** | 异构内存属性表 | 报告不同内存域（包括 **CXL 扩展内存**、PMEM）的延迟与带宽矩阵，供 OS NUMA 调度 |
| **`IORT`** | ARM 组件专用表 | 描述 ARM64 服务器中 PCIe Root Complex → **SMMUv3**（StreamID 映射）→ **GICv3 ITS**（DeviceID 映射）拓扑 |
| **`NHLT` / `EINJ`** | RAS / 错注表 | `EINJ` 允许 OS 通过固件 APEI 接口向指定 PCIe BDF 注入硬件错误，验证 AER/DPC 恢复流程 |

---

## UEFI 2.10 启动期 NVMe 驱动协议栈

在操作系统内核尚未加载的 UEFI DXE（Driver Execution Environment）阶段，BIOS 已经能从 NVMe SSD 读取 EFI 分区里的 `grubx64.efi` 或 Windows Boot Manager，靠的是 UEFI 标准化的两层 Protocol `[UEFI 2.10 §13–14]`：

1. **`EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL` / `EFI_PCI_IO_PROTOCOL`**：抽象 Root Bridge 的 ECAM 配置空间读写、BAR 空间 MMIO 访问与 DMA 内存映射（`Map`/`Unmap`）。
2. **`EFI_NVM_EXPRESS_PASS_THRU_PROTOCOL`**：UEFI 内置 `NvmExpressDxe` 驱动基于 PCI I/O 建立 Admin/I/O 队列，向外暴露标准的 PassThru 接口，使上层 `BlockIo`、磁盘分区驱动与启动管理器无需了解具体硬件即可读写 NVMe 命名空间。

```bash
# 在 Linux 中查看 BIOS 通过 _OSC 授予内核的 PCIe 控制权与 ACPI 表列表
dmesg | grep -iE "_OSC|pciehp|AER"
ls /sys/firmware/acpi/tables/
```

---

## 关联与引申概念

- **上电枚举全景**：`MCFG`/`_PRT`/`DMAR` 的基础作用，详见 [上电、UEFI/ACPI 与 PCIe 枚举](boot-and-enumeration.md)。
- **ARM SMMUv3 拓扑**：`IORT` 如何为 SMMUv3 分配 StreamID，详见 [IOMMU 硬件架构深潜：Intel VT-d / AMD-Vi / ARM SMMUv3](iommu-vtd-smmuv3-deep-dive.md)。
- **热插拔与错误恢复**：OS 拿到 `_OSC` 控制权后如何处理 AER 与 DPC，详见 [PCIe 高级错误报告（AER）](../pcie/wiki/error-handling/aer.md)。
