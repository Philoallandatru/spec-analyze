# PCIe 配置空间（Configuration Space）

**PCIe 配置空间是每个 PCIe 设备的“身份证 + 控制面板”**：系统上电时，BIOS/UEFI 和操作系统通过读写这段标准化的寄存器区域，来识别设备身份（厂商、型号、类别）、探测并分配内存资源（BAR），以及开启 DMA、中断和高级特性。

> **直观类比**：就像插上一块新主板或 USB 设备前，系统需要先看“铭牌”才知道它是显卡还是 NVMe SSD，再给它分配门牌号（地址）和通行证（Bus Master 权限）。配置空间就是这块标准化的“铭牌与总开关”。

---

## 1. 传统 256B 与 PCIe 4KB 空间

为了兼容早期 PCI 软件，PCIe 保留了前 256 字节的布局，并将总大小扩展到 **4096 字节（4 KB）**：

```
0x000 ┌─────────────────────────────────────────┐
      │ 标准头部（Header，Type 0 或 Type 1）     │  64 B (0x00 - 0x3F)
0x040 ├─────────────────────────────────────────┤
      │ 传统 PCI 能力链表（Capabilities）        │  192 B (0x40 - 0xFF)
0x100 ├─────────────────────────────────────────┤  ← 传统 IO 端口方式只能看到这里
      │ PCIe 扩展能力链表（Extended Capabilities）│  3840 B (0x100 - 0xFFF)
0xFFF └─────────────────────────────────────────┘  ← 需通过 ECAM/MMCFG 访问
```

- **0x000–0x0FF（256 B）**：传统 PCI 兼容区，包含设备基本标识、BAR 和传统能力（如电源管理、MSI、MSI-X、PCIe Capability）。
- **0x100–0xFFF（3840 B）**：PCIe 扩展配置空间，存放 AER（高级错误报告）、SR-IOV（虚拟化）、ACS 等扩展能力。

---

## 2. Type 0 配置空间头部概览（0x00–0x3F）

普通端点设备（Endpoint，如 NVMe SSD、网卡、显卡）使用 **Type 0 Header**；而 Root Port、Switch 等桥接设备使用 **Type 1 Header**（详见 [头部类型详解](header-types.md)）。

```
偏移    +0x0        +0x1        +0x2        +0x3
      ┌───────────────────────┬───────────────────────┐
0x00  │      Vendor ID        │      Device ID        │  设备身份
      ├───────────────────────┼───────────────────────┤
0x04  │       Command         │        Status         │  总开关与状态
      ├───────────┬───────────┴───────────────────────┤
0x08  │  Rev ID   │          Class Code (24b)         │  版本与设备分类
      ├───────────┼───────────┬───────────┬───────────┤
0x0C  │Cache Line │ Lat Timer │Header Type│   BIST    │  杂项（含多功能标志）
      ├───────────┴───────────┴───────────┴───────────┤
0x10  │                    BAR 0                      │
0x14  │                    BAR 1                      │  基址寄存器
0x18  │                    BAR 2                      │  （映射设备 MMIO/IO）
0x1C  │                    BAR 3                      │
0x20  │                    BAR 4                      │
0x24  │                    BAR 5                      │
      ├───────────────────────────────────────────────┤
0x28  │              CardBus CIS Pointer              │
      ├───────────────────────┬───────────────────────┤
0x2C  │  Subsystem Vendor ID  │  Subsystem Device ID  │  板卡厂商标识
      ├───────────────────────┴───────────────────────┤
0x30  │           Expansion ROM Base Address          │
      ├───────────┬───────────────────────────────────┤
0x34  │  Cap Ptr  │             Reserved              │  能力链表起点
      ├───────────┴───────────────────────────────────┤
0x38  │                   Reserved                    │
      ├───────────┬───────────┬───────────┬───────────┤
0x3C  │ Int Line  │  Int Pin  │  Min_Gnt  │  Max_Lat  │  传统中断引脚
      └───────────┴───────────┴───────────┴───────────┘
```

---

## 3. 核心字段详解

### 3.1 设备识别（0x00–0x0B）

| 字段 | 偏移/位宽 | 作用与常见示例 |
|------|-----------|----------------|
| **Vendor ID** | `0x00` / 16b | 由 PCI-SIG 分配的芯片厂商 ID。如 `0x8086`（Intel）、`0x144D`（Samsung）、`0x10DE`（NVIDIA）。全 `0xFFFF` 表示槽位空置。 |
| **Device ID** | `0x02` / 16b | 厂商自定义的设备型号 ID，驱动通过 `Vendor ID + Device ID` 匹配硬件。 |
| **Class Code** | `0x09` / 24b | 由 Base Class / Sub-Class / Prog IF 组成，用于通用驱动识别。例如 `0x010802` 代表 **NVMe 控制器**，`0x020000` 代表以太网卡。 |
| **Header Type** | `0x0E` / 8b | Bit 7 = 1 表示多功能设备（Multi-Function）；Bits 6:0 为 `0x00` 表示 Type 0（端点），`0x01` 表示 Type 1（桥）。 |

### 3.2 Command 与 Status 寄存器（0x04–0x07）

**Command（`0x04`，16-bit）** 是设备的“总闸门”，未开启对应位时设备会忽略请求或无法发起操作：

| 位 | 名称 | 作用说明 |
|----|------|----------|
| Bit 0 | **I/O Space Enable** | 允许设备响应 I/O BAR 访问（现代 PCIe 设备通常不用）。 |
| Bit 1 | **Memory Space Enable** | **必须置 1**，CPU 才能读写该设备 BAR 映射的寄存器（MMIO）。 |
| Bit 2 | **Bus Master Enable** | **必须置 1**，设备才能主动发起 **DMA**（直接内存访问）读写主机内存或发送 MSI/MSI-X 中断。 |
| Bit 10 | **Interrupt Disable** | 置 1 时禁用传统 INTx 中断（启用 MSI/MSI-X 后通常会关闭此位）。 |

**Status（`0x06`，16-bit）** 用于报告错误与能力标志，其中 **Bit 4（Capabilities List）** 若为 1，表示偏移 `0x34` 处的 Capabilities Pointer 有效。

### 3.3 BAR 与能力链表指针

- **BAR0–BAR5（`0x10`–`0x24`）**：告诉系统“设备需要多大一块内存空间来放控制寄存器或缓冲区”，系统分配物理地址后回写到此寄存器（详见 [BAR 详解](bars.md)）。
- **Capabilities Pointer（`0x34`）**：指向 `0x40`–`0xFF` 区域中第一个能力结构的偏移，构成单向链表（详见 [能力结构详解](capabilities.md)）。
- **Extended Capabilities（`0x100` 起）**：从固定偏移 `0x100` 开始的增强型链表（详见 [扩展能力详解](extended-capabilities.md)）。

---

## 4. 配置空间如何被访问：传统 IO 端口 vs ECAM/MMCFG

CPU 访问配置空间有两种机制：

| 访问机制 | 原理 | 可访问范围 | 说明 |
|----------|------|------------|------|
| **Configuration Mechanism #1** | 通过 x86 I/O 端口 `0xCF8`（地址）和 `0xCFC`（数据）间接读写 | 仅前 **256 字节** | 传统 PCI 遗留方式，需要两条指令且有锁竞争，无法访问 `0x100+` 扩展空间。 |
| **ECAM / MMCFG** | 将所有 PCIe 设备的 4 KB 配置空间平铺映射到一段连续物理内存 | 完整 **4096 字节** | PCIe 标准方式（ACPI `MCFG` 表告知基址），CPU 用普通内存读写指令即可访问。 |

**ECAM 物理地址计算公式**（每台设备独占 4 KB = `2^12` 字节）：

```
物理地址 = ECAM_Base + (Bus << 20) + (Device << 15) + (Function << 12) + Offset
```

> 例如：`ECAM_Base = 0xE0000000`，访问 `01:00.0`（Bus 1, Dev 0, Func 0）偏移 `0x100` 的 AER 能力，物理地址为 `0xE0000000 + (1 << 20) + 0x100 = 0xE0100100`。

---

## 5. 系统启动时的配置空间初始化流程

```
[1] 上电/热插拔 ──▶ BIOS/OS 枚举总线（读 Vendor ID != 0xFFFF）
        │
[2] 探测与分配  ──▶ 写全 1 读回探测 BAR 大小 ──▶ 分配不冲突的物理地址写回 BAR
        │
[3] 驱动加载    ──▶ 匹配 Vendor/Device ID 或 Class Code
        │
[4] 启用与映射  ──▶ 置位 Command[Memory Space] & Command[Bus Master]
        │           ──▶ ioremap BAR 地址，配置 MSI-X 中断 ──▶ 设备就绪
```

---

## 6. 实用 CLI 调试命令（`lspci` / `setpci` / `sysfs`）

```bash
# 1. 查看设备详细配置摘要（含 BAR、Capabilities、Link 状态）
lspci -vvv -s 01:00.0

# 2. 十六进制转储完整 4096 字节配置空间（需 root）
lspci -xxxx -s 01:00.0

# 3. 通过 sysfs 直接读取配置空间二进制内容
hexdump -C /sys/bus/pci/devices/0000:01:00.0/config | head -n 10

# 4. 使用 setpci 读写特定寄存器（示例：读 Command 寄存器、读 BAR0）
setpci -s 01:00.0 COMMAND
setpci -s 01:00.0 BASE_ADDRESS_0
```

> **常见排障**：若设备无法访问 MMIO 寄存器，用 `lspci -vvv` 检查 `Flags` 是否缺少 `bus master` 或显示 `Memory at ... [disabled]`，这通常意味着驱动尚未置位 Command 寄存器的 Bit 1/Bit 2。

---

## 关联与引申概念

| 概念 | 简要说明 | 相关文档 |
|------|----------|----------|
| **Type 0 vs Type 1 Header** | Type 0 用于终端设备（含 6 个 BAR）；Type 1 用于桥接器/Switch（含总线号与地址转发窗口）。 | [头部类型详解](header-types.md) |
| **BAR / MMIO / DMA** | BAR 将设备内部寄存器映射为 CPU 可读写的 **MMIO**（内存映射 I/O）地址；**DMA** 允许设备绕过 CPU 直接读写主机内存。 | [BAR 详解](bars.md) |
| **Capability Linked List** | 位于 `0x40–0xFF` 的单向链表，声明电源管理、MSI、MSI-X、PCIe 链路等基础能力。 | [能力结构详解](capabilities.md) |
| **Extended Capability** | 位于 `0x100–0xFFF` 的 4 KB 扩展链表，只能通过 ECAM 访问，包含 AER、SR-IOV、ACS、IDE 等特性。 | [扩展能力详解](extended-capabilities.md) |
| **BDF（Bus/Device/Function）** | PCIe 设备的逻辑坐标，用于 ECAM 寻址及 TLP 报文路由中的 Requester ID / Completer ID。 | [头部类型详解](header-types.md) |
| **PCIe 中断机制** | INTx、MSI、MSI-X 均依赖配置空间中的 Pin 寄存器或 Capability 结构进行启用与路由。 | [中断机制概述](../interrupts/overview.md) |
