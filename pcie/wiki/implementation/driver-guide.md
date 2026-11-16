# PCIe 驱动开发指南

> **一句话总结**：PCIe 驱动程序是操作系统与硬件之间的"翻译官"——它在设备插入时接管配置空间与 BAR 寄存器，搭建好 DMA 内存通道与 MSI-X 中断路由，并在运行期响应读写、休眠与硬件错误恢复。

---

## Linux PCIe 驱动分层与生命周期

Linux 将总线枚举、资源分配与具体设备逻辑解耦，驱动开发者只需实现 `struct pci_driver` 并注册设备 ID 表（`MODULE_DEVICE_TABLE`）：

```
  应用程序 / 块设备 / 网络子系统
                 │
                 ▼
  ┌─────────────────────────────┐
  │  PCIe 设备驱动 (你的代码)    │  ◄── probe / remove / irq_handler / err_handler
  └─────────────────────────────┘
                 │
  ┌─────────────────────────────┐
  │  Linux PCI Core 子系统       │  ◄── 配置空间读写、BAR 资源管理、MSI-X 分配
  └─────────────────────────────┘
                 │
                 ▼
         Root Complex / 硬件设备
```

| `pci_driver` 核心成员 | 作用 |
| :--- | :--- |
| `id_table` | 声明驱动支持的 `Vendor ID / Device ID`（或 Class Code）列表 |
| `probe(pdev, id)` | 内核匹配到设备后调用，完成资源使能、映射、中断与上层注册 |
| `remove(pdev)` | 设备热拔出或驱动卸载时调用，按 `probe` 逆序释放全部资源 |
| `driver.pm` (`dev_pm_ops`) | 系统休眠（Suspend/Resume）与运行时节能（Runtime PM）回调 |
| `err_handler` | AER 错误恢复回调（`error_detected`、`slot_reset`、`resume`） |

---

## Probe 初始化标准流程（8 步走）

一个健壮的 PCIe 驱动在 `probe()` 中应严格遵循以下顺序初始化（若某一步失败，按逆序回滚释放）：

```
1. pci_enable_device(pdev)           ──► 唤醒设备电源、启用 IO/Memory 解码
2. pci_request_regions(pdev, name)   ──► 独占声明 BAR 资源，防止驱动冲突
3. dma_set_mask_and_coherent(64bit)  ──► 协商设备 DMA 寻址能力（优先 64-bit）
4. pci_ioremap_bar(pdev, 0)          ──► 将 BAR 物理地址映射为内核虚拟指针 (MMIO)
5. pci_set_master(pdev)              ──► 开启 Command.BusMaster，允许设备发起 DMA
6. pci_alloc_irq_vectors(...)        ──► 分配 MSI-X（或回退 MSI/INTx）并 request_irq
7. 初始化硬件寄存器 & 队列             ──► 重置控制器、下发队列基地址
8. 注册到上层子系统                   ──► 如 blk_mq / netdev / char_dev
```

> **最佳实践**：优先使用 `devm_*` 资源托管 API（如 `pcim_enable_device`、`devm_ioremap_resource`、`devm_kzalloc`），设备卸载或 `probe` 失败时内核会自动回收内存与映射，大幅减少泄漏风险。

---

## 关键子系统实现要点

### 1. BAR 空间与 MMIO 访问

**BAR**（Base Address Register）将设备内部寄存器映射到 CPU 物理地址空间（**MMIO**，Memory-Mapped I/O）。驱动通过 `pci_ioremap_bar()` 获取虚拟地址后，必须使用 `ioread32()` / `iowrite32()`（或 `readl`/`writel`）读写，切勿直接解引用裸指针，以确保内存屏障与字节序安全。

```c
void __iomem *bar0 = pci_ioremap_bar(pdev, 0);
u32 status = ioread32(bar0 + REG_STATUS);
iowrite32(CTRL_ENABLE, bar0 + REG_CTRL);
```

### 2. DMA 内存管理

设备通过 **DMA**（Direct Memory Access）直接读写系统内存，无需 CPU 搬运。Linux 将 DMA 映射分为两类：

| 类型 | 核心 API | 适用场景 | 缓存一致性处理 |
| :--- | :--- | :--- | :--- |
| **一致性 DMA** | `dma_alloc_coherent` / `dma_free_coherent` | 长期共享的控制结构（如 NVMe SQ/CQ 环形队列、描述符表） | 分配时即保证 CPU 与设备视图一致，无需每次手动刷缓存 |
| **流式 DMA** | `dma_map_single` / `dma_map_sg`（Scatter-Gather） | 单次 I/O 数据缓冲区（如块设备读写页） | 每次传输前 map、完成后 unmap，支持非连续物理页（SG 列表） |

### 3. 中断与电源管理

- **中断分配**：现代驱动统一使用 `pci_alloc_irq_vectors(pdev, 1, max_vecs, PCI_IRQ_MSIX | PCI_IRQ_AFFINITY)` 自动申请 MSI-X 向量并绑定 CPU 亲和性，再通过 `pci_irq_vector(pdev, i)` 获取 Linux IRQ 号并调用 `request_irq()`。若硬件不支持 MSI-X，可加入 `PCI_IRQ_MSI | PCI_IRQ_LEGACY` 作为后备。
- **电源管理**：在 `suspend` 中停止业务队列、释放中断、调用 `pci_save_state()` 保存配置空间并将设备切至低功耗状态（如 `D3hot`）；在 `resume` 中切回 `D0`、`pci_restore_state()`、重新使能设备与 Bus Master 并恢复中断。

### 4. 错误恢复（AER）

通过实现 `struct pci_error_handlers` 的三个回调对接内核 AER 框架：`error_detected()` 负责急停业务并返回是否需要复位；`slot_reset()` 在总线复位后重建硬件状态；`resume()` 通知上层恢复正常 I/O。

---

## 调试与排错工具

```bash
# 查看设备 BAR 分配、驱动绑定与 MSI-X 向量详情
lspci -vvv -s 01:00.0

# 查看设备 BAR 物理地址范围与标志
cat /sys/bus/pci/devices/0000:01:00.0/resource

# 动态移除设备并触发总线重新扫描（热插拔/驱动调试常用）
echo 1 > /sys/bus/pci/devices/0000:01:00.0/remove
echo 1 > /sys/bus/pci/rescan

# 开启指定驱动文件的动态调试日志 (dev_dbg)
echo 'file my_pci_drv.c +p' > /sys/kernel/debug/dynamic_debug/control
```

---

## 关联与引申概念

- **IOMMU / DMA Remapping**：在开启 IOMMU（如 Intel VT-d / AMD-Vi）的系统中，`dma_alloc_coherent` 与 `dma_map_sg` 返回的 `dma_addr_t` 是 **IOVA**（I/O 虚拟地址）而非真实物理地址，硬件需经 IOMMU 翻译后才能访问内存，从而防止越权 DMA 攻击。
- **blk-mq（Multi-Queue Block Layer）**：Linux 块设备多队列框架，可将每个 CPU 核心的软件队列直接映射到 NVMe 的一个硬件 SQ/CQ 与 MSI-X 向量，消除全局锁竞争。
- **延伸阅读**：
  - [PCIe 配置空间实现](config-space.md)
  - [PCIe 中断实现](interrupts.md)
  - [AER 错误处理实现](aer.md)
  - [NVMe over PCIe 实现](nvme.md)
  - [PCIe 电源状态](../power-management/power-states.md)
