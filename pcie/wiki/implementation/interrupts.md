# PCIe 中断实现

> **一句话总结**：PCIe 中断经历了从"共享大铜锣（Legacy INTx）"到"点对点发短信（MSI / MSI-X）"的演进——现代设备通过向特定内存地址发送 Memory Write TLP 来直接唤醒目标 CPU 核心，彻底消除了共享中断检查与引脚冲突。

---

## 三种中断机制核心对比

| 特性 | Legacy INTx | MSI | MSI-X |
| :--- | :--- | :--- | :--- |
| **传输方式** | 4 条虚拟中断线（INTA#–INTD#） | Memory Write TLP（Message） | Memory Write TLP（Message） |
| **Capability ID** | 无（配置头 `0x3C` Pin/Line） | `0x05` | `0x11` |
| **最大向量数** | 4（设备间常共享） | 32（必须为 2 的幂） | **2048**（任意数量） |
| **向量独立性** | 共享同一 IRQ，需轮询状态 | 共用单一地址，仅 Data 低位递增 | **每个向量独立地址与数据**（可绑不同 CPU） |
| **单向量屏蔽** | 不支持 | 可选（Per-Vector Masking） | 必需（Table 内置 Mask 位 + PBA） |
| **适用场景** | 老旧设备 / 后备兼容 | 简单设备（Gen1+ 必需） | **高性能多队列设备首选**（NVMe、高速网卡） |

---

## 1. Legacy INTx（物理引脚与虚拟线消息）

早期 PCI 使用 4 根低电平有效物理引脚 `INTA#–INTD#` 连接到中断控制器（APIC）；PCIe 取消了物理中断线，改由 **Assert_INTx / Deassert_INTx** TLP 消息在链路上模拟引脚电平变化。

- **实现要点**：由于多个设备共享同一 IRQ，驱动注册时必须传入 `IRQF_SHARED`；中断处理函数被唤醒后，必须先读取设备 BAR 中的中断状态寄存器——若不是本设备触发则返回 `IRQ_NONE`，若是则写 1 清除状态并返回 `IRQ_HANDLED`。

---

## 2. MSI（Message Signaled Interrupts）

MSI 抛弃了电平信号，改为让设备主动发起一次 **Memory Write TLP** 向系统 APIC 写入指定数值，从而触发 CPU 中断。

- **Capability 结构（ID `0x05`）**：包含 `Message Control`（使能位、MMC 最大支持数、MME 实际分配数、64-bit 地址支持）、`Message Address`（32/64-bit）与 `Message Data`。
- **x86 架构寻址格式**：
  - `Message Address` 高位固定为 `0xFEE`，Bits `[19:12]` 指定目标 **APIC ID**（即目标 CPU）。
  - `Message Data` Bits `[7:0]` 指定 **Interrupt Vector**（0–255），多向量时硬件自动修改低位递增。

---

## 3. MSI-X（现代高性能首选）

MSI-X 将中断表从狭窄的配置空间搬到了设备的 **BAR 空间**中，由两张表组成：
1. **MSI-X Table**（由 Capability 中的 `Table Offset + BIR` 定位）：每个向量占 **16 字节**（`Addr Lo`、`Addr Hi`、`Data`、`Vector Control` Mask 位），允许每个队列独立指向不同的 CPU 核心。
2. **PBA（Pending Bit Array）**：记录向量被屏蔽期间是否发生过挂起中断，解除屏蔽后硬件可立即补发。

### Linux 现代驱动配置（4.10+ 标准 API）

现代内核不再手动拼接 `msix_entry`，统一使用 `pci_alloc_irq_vectors()` 自动分配向量并绑定 CPU 亲和性：

```c
int nvec = pci_alloc_irq_vectors(pdev, 1, max_vecs,
                                 PCI_IRQ_MSIX | PCI_IRQ_AFFINITY);
for (int i = 0; i < nvec; i++) {
    int irq = pci_irq_vector(pdev, i);
    request_irq(irq, my_msix_handler, 0, DRV_NAME, per_vec_data[i]);
}
```

> 传入 `PCI_IRQ_AFFINITY` 后，内核会自动将各个中断向量均匀分散到不同 CPU 核心，避免跨核缓存失效。

---

## 4. QEMU / FEMU 中的中断注入原理

在 QEMU/FEMU 中，设备通过 `msix_init_exclusive_bar()` 创建独立的 MSI-X Table 与 PBA 内存区域；当需要触发中断时（如 NVMe 写完 CQE），调用 `msix_notify(pci_dev, vector)`：
1. 检查该向量是否在 Table 中被 Mask；若被屏蔽，则在 **PBA** 中置位挂起标志并返回。
2. 若未屏蔽，则从 MSI-X Table 读取对应条目的 `msg.address` 与 `msg.data`。
3. 调用 `pci_dma_write(dev, msg.address, &msg.data, sizeof(msg.data))` 向 Guest 内存总线发送一次写事务，由虚拟 APIC 接收并注入对应 vCPU 中断。

---

## 中断亲和性与性能优化

在高吞吐场景下（如百万级 IOPS 的 NVMe SSD 或 100G 网卡），频繁的硬中断会耗尽 CPU 资源，工程上常用三种优化手段：
- **中断亲和性（IRQ Affinity）**：将特定队列的 MSI-X 向量绑定到处理该队列的同一颗 CPU 核心，提升 L1/L2 缓存命中率。
- **中断调节（Interrupt Coalescing）**：在设备寄存器中设置时间阈值（如 `usecs`）或完成帧数阈值（`frames`），攒够一批请求再发一次中断。
- **NAPI / IOPoll 轮询**：高负载时在首个中断到来后暂时关闭中断，改由内核线程或驱动主动轮询完成队列（如网络 NAPI、块设备 `io_uring` IOPoll），负载降低后再恢复中断。

---

## 调试命令

```bash
# 查看各 CPU 核心上 NVMe 设备的中断计数分布
cat /proc/interrupts | grep nvme

# 查看设备 MSI-X 向量数量、Table/PBA 所在 BAR 及使能状态
lspci -vv -s 01:00.0 | grep -A 10 "MSI-X"

# 查看或修改指定 IRQ 号的 CPU 亲和性（例如绑定到 CPU 0-3）
cat /proc/irq/125/smp_affinity_list
echo 0-3 > /proc/irq/125/smp_affinity_list
```

---

## 关联与引申概念

- **APIC（Advanced Programmable Interrupt Controller）**：x86 平台的中断控制器，负责接收来自 PCIe Root Complex 的 MSI/MSI-X 写事务并投递给目标 CPU 核心。
- **PBA（Pending Bit Array）**：MSI-X 规范中的位图数组，确保向量在被临时屏蔽期间产生的中断信号不会丢失。
- **延伸阅读**：
  - [PCIe 中断机制概述](../interrupts/overview.md)
  - [MSI 协议详解](../interrupts/msi.md)
  - [MSI-X 协议详解](../interrupts/msix.md)
  - [Legacy INTx 详解](../interrupts/intx.md)
  - [PCIe 驱动开发指南](driver-guide.md)
