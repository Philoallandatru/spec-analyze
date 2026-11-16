# PCIe 中断机制概述

**中断是 PCIe 设备在任务完成或异常发生时主动通知 CPU 的信号机制**，避免 CPU 浪费算力去不断轮询设备状态。PCIe 支持三代中断机制：**Legacy INTx**（虚拟引脚消息）、**MSI**（消息信号中断）和 **MSI-X**（扩展消息信号中断）。

> **直观类比**：
> - **INTx** 像整栋宿舍楼共用**1 个公共门铃**：铃响后，宿管（OS）必须逐个房间敲门问“是不是你点的外卖？”（共享中断轮询）。
> - **MSI** 像给每个房间装了**专属门铃**（最多 32 个），但所有门铃必须走同一条总线路（共享地址，向量必须连续）。
> - **MSI-X** 像每个员工工位都有**独立分机号**（最多 2048 个），可以直接把 NVMe 的每个 I/O 队列精准绑定到不同 CPU 核心，互不干扰。

---

## 1. 三种中断机制对比

| 特性 | Legacy INTx | MSI | MSI-X |
|------|-------------|-----|-------|
| **引入标准** | PCI 1.0 | PCI 2.2 | PCI 3.0 |
| **底层传输方式** | `Assert/Deassert_INTx` Message TLP | **Memory Write TLP**（Posted 写） | **Memory Write TLP**（Posted 写） |
| **最大向量数** | 4 条逻辑线（INTA#–INTD#） | **1–32**（必须为 2 的幂） | **最多 2048**（任意数量） |
| **是否共享** | **是**（多设备常共用同一 IRQ） | **否**（独占） | **否**（独占） |
| **独立地址/数据** | 不适用 | 共享同一地址，仅 Data 低位递增 | **每个向量完全独立**（可绑定不同 CPU） |
| **配置位置** | 配置空间 `0x3C–0x3D` | 配置空间 MSI Capability (`0x05`) | **BAR 内存空间**（由 Capability `0x11` 指向） |
| **典型延迟** | 较高（~5–20 μs，含共享轮询） | 低（~1–3 μs） | **极低（~1–2 μs，多核并行）** |

---

## 2. 系统中的中断传递路径

```
PCIe 设备产生事件
   │
   ├── [INTx]     ──▶ Assert_INTx Message TLP ──▶ Root Complex ──▶ IOAPIC ──▶ Local APIC ──▶ CPU
   │
   └── [MSI/MSI-X]──▶ Posted Memory Write TLP ──▶ Root Complex ──────────────▶ Local APIC ──▶ CPU
                      (写入 0xFEExxxxx 特殊地址)      (直接按地址与数据路由，无需 IOAPIC)
```

---

## 3. 核心机制速览

### 3.1 Legacy INTx（传统兼容）

PCIe 物理层已取消了传统 PCI 的 4 根铜线，改为通过 **Message TLP**（`Assert_INTA~D` / `Deassert_INTA~D`）模拟电平触发中断。由于多设备共享同一 IRQ，内核必须逐个调用共享链表上的中断处理函数去读取设备状态寄存器确认“是不是自己触发的”，开销大且无法支持多队列并行。详见 [Legacy INTx 详解](intx.md)。

### 3.2 MSI（Message Signaled Interrupts）

MSI 彻底抛弃中断线，直接让设备向特定物理地址（x86 上为 `0xFEExxxxx`，即 Local APIC 地址窗口）发送一笔 **32 位 Memory Write TLP** 来触发中断。因为写入的是 Posted（非阻塞）内存写事务，且无需共享轮询，延迟大幅下降。缺点是最多只支持 32 个连续向量，且所有向量只能发往同一个目标地址（CPU）。详见 [MSI 详解](msi.md)。

### 3.3 MSI-X（Extended MSI）

现代高性能设备（NVMe SSD、100G 网卡、GPU）的绝对主流。它将中断向量表（**MSI-X Table**）和挂起位数组（**PBA**）从狭窄的配置空间搬到了设备的 **BAR 内存空间**中，支持最多 **2048 个完全独立的中断向量**。驱动可以为每个 I/O 队列分配专属向量并绑定到对应的 CPU 核心（**中断亲和性**），实现无锁、NUMA 友好的极致吞吐。详见 [MSI-X 详解](msix.md)。

---

## 4. 选型与优先级原则

操作系统与驱动在初始化时遵循固定优先级：**MSI-X > MSI > INTx**（三者互斥，启用 MSI/MSI-X 时应设置配置空间 Command 寄存器的 Bit 10 禁用 INTx）：

- **优先 MSI-X**：所有多队列、高 IOPS/高带宽或需要 CPU 亲和性调优的现代设备。
- **退回 MSI**：硬件不支持 MSI-X，但需要独占中断且向量数 $\le 32$ 的简单设备。
- **保底 INTx**：仅用于兼容老旧系统、早期引导阶段或极低频控制设备。

---

## 5. 实用 CLI 观察命令

```bash
# 1. 查看设备当前启用的中断机制（Enable+ 表示已启用）
lspci -vvv -s 01:00.0 | grep -E "Interrupt:|MSI|MSI-X"
# 示例:
#   Interrupt: pin A routed to IRQ 11
#   Capabilities: [50] MSI: Enable- Count=1/1 Maskable- 64bit+
#   Capabilities: [70] MSI-X: Enable+ Count=16 Masked-

# 2. 查看内核中各队列中断在多核 CPU 上的实时分布
cat /proc/interrupts | grep nvme
```

---

## 关联与引申概念

| 概念 | 简要说明 | 相关文档 |
|------|----------|----------|
| **Legacy INTx** | 通过 Message TLP 模拟传统电平中断，存在 IRQ 共享与轮询开销。 | [Legacy INTx 详解](intx.md) |
| **MSI** | 利用 Posted Memory Write TLP 触发中断，最多 32 个连续向量。 | [MSI 详解](msi.md) |
| **MSI-X** | 将 Table/PBA 放入 BAR 空间，支持 2048 个独立向量与每向量独立掩码。 | [MSI-X 详解](msix.md) |
| **APIC / IOAPIC / 中断重映射** | 决定 INTx 如何经过 IOAPIC、MSI 如何直达 Local APIC，以及 IOMMU（VT-d）如何在虚拟化下安全重映射中断。 | [中断路由详解](routing.md) |
| **Capability Linked List** | MSI (`0x05`) 与 MSI-X (`0x11`) 均作为标准 Capability 挂在配置空间链表中。 | [能力结构详解](../configuration/capabilities.md) |
