# PCIe 中断路由与亲和性

**PCIe 中断路由决定了设备发出的中断信号如何穿越总线拓扑、硬件中断控制器（IOAPIC / Local APIC）以及虚拟化隔离层（IOMMU），最终精准投递到目标 CPU 核心**。

> **直观类比**：如果中断是一封加急信件，**INTx** 就像寄往“片区总站”（IOAPIC）再由站长查表（`_PRT`）转交；**MSI/MSI-X** 就像在信封上直接写明了收件人门牌号（目标 CPU 的 Local APIC ID），直达本人；而开启**中断重映射（Interrupt Remapping）**则相当于在大楼入口加装了智能门禁（IOMMU），既能防止伪造信件攻击，又能把寄给虚拟机的信件直接转进对应的虚拟房间（vCPU）。

---

## 1. 中断路由整体架构

```
PCIe 设备
  ├── [INTx]      ──▶ Assert_INTx Msg TLP ──▶ Root Complex ──▶ IOAPIC (查 RTE 表) ──┐
  │                                                                                  ▼
  └── [MSI/MSI-X] ──▶ Posted MemWr TLP ──────▶ Root Complex ──▶ [IOMMU 重映射?] ──▶ Local APIC ──▶ CPU 核心
                      (0xFEExxxxx)                              (可选，VT-d/AMD-Vi)
```

- **IOAPIC（I/O 高级可编程中断控制器）**：负责接收传统物理/模拟中断线（INTx），通过内部的**重定向表项（RTE）**将其转换为中断向量并转发给指定 CPU 的 Local APIC。
- **Local APIC（本地 APIC）**：每个 CPU 核心内置的中断接收单元。MSI/MSI-X 写往 `0xFEExxxxx` 的内存写请求会被 Root Complex 直接路由到目标核心的 Local APIC，绕过 IOAPIC。

---

## 2. INTx 路由：ACPI `_PRT` 与 IOAPIC

由于 INTx 消息里只包含设备地址（BDF）和引脚编号（INTA#–INTD#），系统需要通过两级映射找到目标 CPU：

1. **ACPI `_PRT`（PCI Routing Table）**：主板固件（BIOS/UEFI）在 ACPI 表中声明“某条总线上的哪个设备、哪根引脚对应系统中的哪个全局中断号（GSI / IRQ）”。
2. **IOAPIC RTE（Redirection Table Entry）**：OS 内核读取 `_PRT` 后配置 IOAPIC 的对应表项，指定该 IRQ 的触发方式（PCIe INTx 必须为 **Level-Triggered 电平触发、Active-Low 低电平有效**）、分配的中断向量号以及目标 CPU 的 APIC ID。

---

## 3. MSI / MSI-X 路由：Local APIC 直接寻址

MSI 和 MSI-X 不经过 `_PRT` 和 IOAPIC，路由信息直接编码在设备的配置空间（MSI）或 BAR 内存表（MSI-X）中：

- **Message Address（x86/x64）**：`0xFEE00000 | (Destination_APIC_ID << 12)`，其中的 `Destination ID` 直接指明接收该中断的 CPU 核心。
- **Message Data**：低 8 位存放 CPU 中断向量号（`0x20–0xFF`），投递模式通常为 `Fixed`（`000b`），触发模式为 `Edge`（边沿触发）。

---

## 4. 中断重映射（Interrupt Remapping）与虚拟化

在现代服务器与虚拟化环境中，设备直接向 `0xFEExxxxx` 写任意地址会带来安全与隔离隐患。开启 **IOMMU（Intel VT-d / AMD-Vi）** 的中断重映射后，硬件会在 Root Complex 处拦截所有 MSI/MSI-X 写请求：

```
物理/直通设备 ──(Remapped MSI: Handle 索引)──▶ IOMMU 查 IRTE 表 ──(真实 Vector + 目标 pCPU)──▶ Local APIC
```

- **安全隔离**：设备不再能自行指定目标 CPU 和向量号，只能携带一个受控的 **Handle（索引）**；IOMMU 根据内核维护的 **IRTE（Interrupt Remapping Table Entry）** 翻译出真正的物理目标，防止恶意设备或故障固件向系统注入伪造中断。
- **虚拟化直通与 Posted Interrupts**：当 PCIe 设备直通（VFIO）给虚拟机时，结合硬件 **Posted Interrupts** 技术，IOMMU 可将设备中断直接写入虚拟机的内存描述符并注入正在运行的 vCPU，无需触发昂贵的 VM-Exit 陷入 Hypervisor 转发。

---

## 5. 中断亲和性与多队列负载均衡

合理规划中断路由对高吞吐存储和网络设备至关重要：

- **CPU 亲和性（IRQ Affinity）**：对于支持 MSI-X 的设备，OS 可将每个队列的 MSI-X Table Entry 指向不同的 CPU 核心（例如 NVMe `Queue i → CPU i`），使 I/O 提交与完成中断落在同一个核心上，最大化 L1/L2 缓存命中率。
- **NUMA 感知**：在多路服务器上，应优先将 PCIe 设备的中断绑定到该设备物理直连的 **NUMA 节点** 内的 CPU 核心上，避免跨插槽通信延迟。
- **网卡 RSS（Receive Side Scaling）**：网卡硬件根据数据包的五元组哈希值将不同流分散到多个接收队列，再配合每个队列独立的 MSI-X 中断路由到不同 CPU，实现多核线性扩展。
- **自动均衡（`irqbalance`）**：通用服务器常运行 `irqbalance` 守护进程动态调整各 IRQ 的亲和性；但在超低延迟或高性能存储调优场景下，通常会关闭 `irqbalance` 并手动静态绑核。

---

## 6. 实用 CLI 调试命令

```bash
# 1. 查看各中断号的触发次数、控制器类型（IO-APIC vs IR-PCI-MSI）及 CPU 分布
cat /proc/interrupts | grep -E "CPU0|nvme|eth"

# 2. 查看并手动修改某个 IRQ 的 CPU 亲和性（示例：将 IRQ 130 绑定到 CPU 2）
cat /proc/irq/130/smp_affinity_list
echo 2 | sudo tee /proc/irq/130/smp_affinity_list

# 3. 实时监控中断分布是否均衡
watch -n 1 'cat /proc/interrupts | grep nvme'
```

> **提示**：在 `/proc/interrupts` 中看到 `IR-PCI-MSI` 前缀，说明系统已启用 **Interrupt Remapping（IR）**；若看到 `IO-APIC` 且带 `fasteoi`，则说明该设备正在使用共享的 Legacy INTx。

---

## 关联与引申概念

| 概念 | 简要说明 | 相关文档 |
|------|----------|----------|
| **PCIe 中断概述** | 总结 INTx、MSI、MSI-X 三代中断机制的原理差异与选型策略。 | [中断机制概述](overview.md) |
| **Legacy INTx** | 依赖 ACPI `_PRT` 与 IOAPIC 路由的共享电平模拟中断。 | [Legacy INTx 详解](intx.md) |
| **MSI / MSI-X** | 通过 `0xFEExxxxx` Posted 写事务直接路由到 Local APIC。 | [MSI 详解](msi.md) · [MSI-X 详解](msix.md) |
| **ACS / SR-IOV（IOMMU 隔离）** | 虚拟化直通时与中断重映射配合使用，确保 DMA 与 P2P 流量的安全隔离。 | [扩展能力详解](../configuration/extended-capabilities.md) |
