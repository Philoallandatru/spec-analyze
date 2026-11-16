# PCIe 扩展能力结构（Extended Capabilities）

**Extended Capabilities（扩展能力结构）是位于 PCIe 配置空间 `0x100–0xFFF` 区域（共 3840 字节）的高级特性链表**。它用 16 位能力 ID 和 12 位偏移指针取代了传统 8 位链表，用来承载错误报告（AER）、硬件虚拟化（SR-IOV）、安全隔离（ACS/IDE）等现代 PCIe 核心特性。

> **直观类比**：如果传统 `0x40–0xFF` 的能力链表是空间紧凑的“一楼标准柜台”，很快就会被各种新功能塞满；那么 PCIe 把配置空间扩到 4 KB 后，在 `0x100–0xFFF` 开辟的“二楼扩展大厅”就是 Extended Capabilities——空间宽敞了 20 倍，能容纳 65536 种新能力。

---

## 1. 访问前提：ECAM / MMCFG

传统 PCI 的 I/O 端口方式（`0xCF8/0xCFC`）只有 8 位寄存器偏移字段，**完全看不到 `0x100` 及以上的区域**。因此，Extended Capabilities **必须通过 ECAM（Enhanced Configuration Access Mechanism，也称 MMCFG）** 访问：

```
物理地址 = ECAM_Base + (Bus << 20) + (Device << 15) + (Function << 12) + Offset
```

> 若在系统中运行 `lspci` 看不到 `[100]` 及以后的扩展能力，通常是因为缺少权限（需 `sudo`）、系统未启用 MMCFG，或设备本身是传统 PCI 设备。

---

## 2. Extended Capability 链表格式

链表**固定从偏移 `0x100` 开始**（不再需要 Header 中的指针），每个扩展能力以一个 **4 字节（1 DWORD）头部** 开头：

```
31                    20 19      16 15                         0
┌───────────────────────┬──────────┬────────────────────────────┐
│  Next Cap Offset      │ Version  │   Extended Capability ID   │  DW 0 (+0x00)
│     (12 bits)         │ (4 bits) │         (16 bits)          │
└───────────────────────┴──────────┴────────────────────────────┘
│               Capability-Specific Registers ...               │  DW 1+
└───────────────────────────────────────────────────────────────┘
```

- **Extended Capability ID（Bits 15:0）**：16 位能力编号（`0x0000` 为 Null/空能力）。
- **Capability Version（Bits 19:16）**：版本号（通常为 `1` 或 `2`）。
- **Next Capability Offset（Bits 31:20）**：下一个扩展能力的**绝对字节偏移**（4 字节对齐，即低 2 位恒为 0）；为 **`0x000`** 时表示链表结束。

```
[0x100] AER (ID=0x0001)  ──Next=0x140──▶  [0x140] DSN (ID=0x0003)  ──Next=0x200──▶  [0x200] SR-IOV (ID=0x0010)  ──Next=0x000 (结束)
```

---

## 3. 核心 Extended Capabilities 分类详解

### 3.1 错误管理与可靠性

| ID | 名称 | 核心作用与关键概念 |
|----|------|-------------------|
| `0x0001` | **AER (Advanced Error Reporting)** | **高级错误报告**：将错误细分为可纠正（Bad TLP/DLLP、Replay 超时）和不可纠正（Completion Timeout、Malformed TLP、Poisoned TLP），并通过 Mask/Severity 寄存器决定上报为 Fatal 还是 Non-Fatal，同时用 4 个 DWORD 的 Header Log 记录出错 TLP 现场。 |
| `0x001D` | **DPC (Downstream Port Containment)** | **下游端口遏制**：当下游设备触发致命或不可纠正错误时，Root Port 或 Switch 下游端口立即硬件切断该链路（禁止收发 TLP），防止错误数据扩散污染主机内存（热恢复后再解除）。 |

### 3.2 虚拟化与地址路由

| ID | 名称 | 核心作用与关键概念 |
|----|------|-------------------|
| `0x0010` | **SR-IOV (Single Root I/O Virtualization)** | **单根 I/O 虚拟化**：允许一块物理网卡或 NVMe SSD（PF，物理功能）在硬件层面虚拟出数十至数百个独立的虚拟设备（VF，虚拟功能），配合 IOMMU 直接直通给不同虚拟机，绕过 Hypervisor 转发开销。 |
| `0x000D` | **ACS (Access Control Services)** | **访问控制服务**：在 Switch/Root Port 上控制 P2P（点对点）流量路由，强制将设备间互访重定向到 Root Complex 经过 **IOMMU** 检查，是虚拟机设备直通与安全隔离的基石。 |
| `0x000E` | **ARI (Alternative Routing-ID Interpretation)** | **备选路由 ID**：传统 PCIe 每个 Device 最多 8 个 Function（3 位）；ARI 将 5 位 Device 号和 3 位 Function 号合并为 8 位 Function 号，使单设备最多支持 **256 个 Function**（常与 SR-IOV 搭配）。 |
| `0x000F` | **ATS (Address Translation Services)** | **地址翻译服务**：允许 PCIe 设备自带地址缓存（ATC），提前向主机 IOMMU 查询虚拟地址（IOVA）到物理地址的映射，大幅降低 DMA 翻译延迟。 |
| `0x001A` | **PASID (Process Address Space ID)** | **进程地址空间 ID**：在 TLP 前缀中携带 20 位进程 ID，让设备能直接向用户态进程的虚拟地址发起 DMA（支持共享虚拟内存 SVM）。 |

### 3.3 性能、QoS 与时间同步

| ID | 名称 | 核心作用与关键概念 |
|----|------|-------------------|
| `0x0002` | **VC (Virtual Channel)** | **虚拟通道**：在一条物理链路上划分多个独立流控的逻辑通道（VC0–VC7），为实时音频/控制报文与大流量数据提供隔离与 QoS 保障。 |
| `0x0013` | **Resizable BAR** | **可调整 BAR**：让设备向系统声明多种可选 BAR 容量（1 MB ~ 8 TB），OS 在枚举时按可用地址空间动态放大 BAR（如显卡全显存直通）。 |
| `0x0016` | **TPH (TLP Processing Hints)** | **TLP 处理提示**：设备在 DMA 写 TLP 中附带提示，告诉 CPU 缓存子系统“这份数据马上要被哪个核读取”，优化缓存预取（如 DDIO）。 |
| `0x0017` | **LTR (Latency Tolerance Reporting)** | **延迟容忍度报告**：设备主动向 Root Complex 汇报自己能容忍的最大唤醒延迟，帮助系统协同决定何时进入深度省电。 |
| `0x001F` | **PTM (Precision Time Measurement)** | **精确时间测量**：通过硬件时间戳报文补偿链路传输延迟，实现 Root Complex 与端点设备之间**纳秒级**时钟同步。 |

### 3.4 电源、安全与设备标识

| ID | 名称 | 核心作用与关键概念 |
|----|------|-------------------|
| `0x0003` | **Device Serial Number (DSN)** | **设备序列号**：提供全球唯一的 64 位硬件序列号（8 字节），用于设备指纹识别与多路径区分。 |
| `0x0004` | **Power Budgeting** | **功率预算**：向系统固件报告设备在不同功耗状态下的精确瓦数需求。 |
| `0x001E` | **L1 PM Substates (L1SS)** | **L1 电源管理子状态**：在 ASPM L1 基础上进一步细化出 **L1.1**（关闭参考时钟，Clock Gating）和 **L1.2**（切断部分电源，Power Gating），显著降低笔记本/服务器待机功耗。 |
| `0x0023` | **DOE (Data Object Exchange)** | **数据对象交换**：提供配置空间邮箱（Mailbox），用于在设备与主机之间传输加密认证协议报文，如 **CMA/SPDM**（设备身份与固件完整性验证）。 |
| `0x002E` | **IDE (Integrity and Data Encryption)** | **完整性与数据加密**：PCIe 6.0/5.0 引入的链路层硬件加密机制，对总线上的 TLP 进行实时加密和防篡改校验，防止物理旁路窃听（配合 DOE/SPDM 协商密钥）。 |

---

## 4. 实用 CLI 调试命令

```bash
# 查看设备所有 Extended Capabilities（偏移 >= 0x100，需 root 权限）
sudo lspci -vvv -s 01:00.0 | grep -A 4 "Capabilities: \[[1-9a-f][0-9a-f][0-9a-f]"
```

典型输出示例：
```text
Capabilities: [100 v1] Advanced Error Reporting
    UESta:  DLP- SDES- TLP- FCP- CmpltTO- ...
    CESta:  RxErr- BadTLP- BadDLLP- ...
Capabilities: [140 v1] Device Serial Number 00-00-00-00-12-34-56-78
Capabilities: [1c0 v1] Latency Tolerance Reporting
Capabilities: [300 v1] L1 PM Substates
```

> **常见排障**：
> - **SR-IOV 创建 VF 失败**：检查 BIOS 是否开启 `VT-d`/`AMD-Vi`（IOMMU）和 `Above 4G Decoding`（为 VF BAR 提供足够的 64-bit MMIO 空间）。
> - **AER 不上报错误**：检查 AER Capability 的 Uncorrectable/Correctable Error Mask 寄存器，以及标准 PCIe Capability (`0x10`) 中 Device Control 的错误报告使能位。

---

## 关联与引申概念

| 概念 | 简要说明 | 相关文档 |
|------|----------|----------|
| **ECAM / MMCFG** | 通过内存映射访问 4 KB 配置空间的机制，是发现和配置所有 Extended Capabilities 的前提。 | [配置空间详解](config-space.md) |
| **传统 Capabilities** | 位于 `0x40–0xFF` 的 8-bit ID 链表，包含电源管理、MSI/MSI-X、PCIe 核心能力。 | [能力结构详解](capabilities.md) |
| **BAR 与 Resizable BAR** | 扩展能力 `0x0013` 允许动态协商 Type 0 头部中的 BAR 大小。 | [BAR 详解](bars.md) |
| **IOMMU / VT-d 与中断重映射** | 与 ACS、ATS、SR-IOV 紧密配合，完成虚拟化场景下的 DMA 地址隔离与中断安全路由。 | [中断路由](../interrupts/routing.md) |
