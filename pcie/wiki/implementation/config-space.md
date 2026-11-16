# PCIe 配置空间实现

> **一句话总结**：PCIe 配置空间是每个设备的"标准化身份证与遥控器面板"——BIOS 和操作系统在枚举阶段通过读写这片 4KB 寄存器空间，识别设备身份、分配内存窗口（BAR），并控制其中断与 DMA 权限。

---

## 配置空间结构总览

每个 PCIe Function 拥有 **4096 字节（4KB）** 的配置空间，分为前 256B 的 PCI 兼容区与后 3840B 的 PCIe 扩展区：

```
0x000 ┌─────────────────────────────────────────┐
      │ 标准配置头 (64B, Type 0 Endpoint)        │ ◄── Vendor/Device ID, Command/Status, BAR0–5
0x040 ├─────────────────────────────────────────┤
      │ 标准 Capability 链表 (PCI-PM, MSI-X...) │ ◄── 由 0x34 Capabilities Pointer 串联
0x100 ├─────────────────────────────────────────┤
      │ PCIe Extended Capability 链表           │ ◄── AER(0x0001), SR-IOV(0x0010), ATS...
0xFFF └─────────────────────────────────────────┘
```

### Type 0 标准配置头关键字段（`0x00–0x3F`）

| 偏移 | 字段 | 作用简述 |
| :--- | :--- | :--- |
| `0x00` | Vendor ID / Device ID | 厂商与设备型号标识（如 Intel `0x8086`），只读 |
| `0x04` | Command / Status | **Command** 控制 Memory Space、Bus Master (DMA)、INTx 屏蔽；**Status** 指示 Capability 链表存在与错误状态 |
| `0x08` | Revision ID / Class Code | 设备类别（如 `0x010802` 表示 NVMe 存储控制器） |
| `0x10–0x24` | BAR0 – BAR5 | 基地址寄存器（Base Address Register），映射设备 MMIO 或 I/O 窗口 |
| `0x34` | Capabilities Pointer | 指向标准 Capability 链表的起始偏移（如 `0x40`） |
| `0x3C` | Interrupt Line / Pin | Legacy INTx 中断引脚（INTA#–INTD#）与路由号 |

---

## 硬件层：写掩码与 BAR 大小探测

硬件设计（ASIC/FPGA）和虚拟化模拟（QEMU）并非把配置空间做成普通 RAM，而是为每个字节维护 **可写掩码（`wmask`）** 和 **写 1 清零掩码（`w1cmask`）**，以保证只读字段（如 Vendor ID）不可被篡改、状态位符合 W1C 语义。

### BAR 大小探测原理

操作系统枚举设备时，如何知道一个 BAR 需要多大的地址空间？标准做法是**"全写 1 再回读"**：

```
1. OS 向 BAR0 写入 0xFFFFFFFF
2. 设备硬件屏蔽低位（例如 16KB 窗口的低 14 位 [13:0] 硬编码只读为 0）
3. OS 回读 BAR0 得到 0xFFFFC000
4. 取反加 1：~(0xFFFFC000 & ~0xF) + 1 = 0x4000 (16 KB)
5. OS 分配一段 16KB 对齐的物理地址写回 BAR0
```

> **64-bit BAR**：若 BAR 低 4 位属性指示 64-bit 寻址（bit[2:1]=`10b`），则由相邻两个 32-bit BAR（如 BAR0 + BAR1）共同拼成 64-bit 基地址。

---

## 软件访问方法对比

访问配置空间需要先通过 **BDF**（`Bus:Device.Function`）唯一锁定目标设备。不同软件层级采用不同的访问机制：

| 访问方式 | 适用层级 | 寻址与原理 | 范围 |
| :--- | :--- | :--- | :--- |
| **Legacy I/O 端口** | x86 早期固件/裸机 | 写 `0xCF8`（地址）再读写 `0xCFC`（数据），非线程安全 | 仅前 `0x00–0xFF`（256B） |
| **ECAM（MMIO）** | 现代 OS / PCIe 根复合体 | 将所有设备的 4KB 配置空间平铺到一段物理内存，直接 `readl/writel` | 完整 `0x000–0xFFF`（4KB） |
| **Linux 内核 API** | 内核驱动 | `pci_read_config_byte/word/dword()`、`pci_write_config_*()`（内部封装锁与 ECAM） | 完整 4KB |
| **sysfs / libpci** | 用户态工具 | 读取 `/sys/bus/pci/devices/<BDF>/config` 或调用 `libpci` 库 | 受权限与内核保护限制 |

### ECAM 地址计算公式

**ECAM**（Enhanced Configuration Access Mechanism，增强配置访问机制）把 `Bus / Device / Function / Offset` 直接编码为内存偏移：

```
ECAM 物理地址 = ECAM_Base + (Bus << 20) + (Device << 15) + (Function << 12) + Offset
```
- 256 个 Bus × 32 个 Device × 8 个 Function × 4KB = **256MB** 连续物理地址空间。

---

## 虚拟化与安全隔离（QEMU / VFIO / SR-IOV / IOMMU）

1. **QEMU 设备模拟**：QEMU 在 `PCIDevice` 中维护 `config[]`、`wmask[]`、`w1cmask[]` 三个数组。Guest 写入配置空间时，QEMU 按掩码更新寄存器；若检测到修改了 `BAR0–5` 或 `PCI_COMMAND`，则自动触发 `pci_update_mappings()` 重新注册内存映射。
2. **SR-IOV（Single Root I/O Virtualization）**：允许一个物理设备（PF）虚拟出多个轻量级虚拟功能（VF）。PF 拥有完整配置空间及 SR-IOV Extended Capability（`0x0010`），而每个 VF 拥有精简配置空间并共享 PF 的 Vendor ID 与部分 BAR 属性。
3. **VFIO 与安全保护**：用户态驱动（如 SPDK/DPDK）或虚拟机直通通过 **VFIO**（Virtual Function I/O）的 `VFIO_PCI_CONFIG_REGION_INDEX` 读写配置空间，内核会拦截危险写操作；同时配合 **IOMMU** 对设备的 DMA 地址进行重映射与隔离，防止恶意设备篡改系统内存。

---

## 常用调试命令

```bash
# 查看设备完整解析信息（含 BAR、Capabilities、Link 状态）
lspci -vvv -s 01:00.0

# 十六进制导出完整 4096 字节扩展配置空间（需要 root）
lspci -xxxx -s 01:00.0

# 使用 setpci 读取 Command 寄存器并开启 Bus Master (bit 2) + Memory Space (bit 1)
setpci -s 01:00.0 COMMAND
setpci -s 01:00.0 COMMAND=0x0006

# 追踪内核配置空间读写事件
trace-cmd record -e pci:pci_read_config -e pci:pci_write_config
```

---

## 关联与引申概念

- **Bus Master（总线主控）**：`Command` 寄存器 bit 2，必须置 1 后设备才能主动发起 DMA 读写请求（如 NVMe 读写数据、MSI-X 写内存触发中断）。
- **ATS / PRI**：地址转换服务与页面请求接口，常在扩展配置空间中启用，配合 IOMMU 加速虚拟机直通设备的 DMA 地址翻译。
- **延伸阅读**：
  - [配置空间详解](../configuration/config-space.md)
  - [BAR 寄存器与地址映射](../configuration/bars.md)
  - [PCIe 设备枚举过程](../architecture/enumeration.md)
  - [SR-IOV 虚拟化](../advanced-features/sriov.md)
  - [PCIe 驱动开发指南](driver-guide.md)
