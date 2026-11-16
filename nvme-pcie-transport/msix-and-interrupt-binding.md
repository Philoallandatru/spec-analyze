# MSI-X 中断向量绑定与屏蔽机制

PCIe 传统引脚中断（INTx）需要多设备共享一条中断线，触发后 OS 必须逐个轮询设备状态寄存器，在百万级 IOPS 下根本无法扩展。NVMe over PCIe 全面围绕 **MSI-X（Message Signaled Interrupts-Extension）** 设计，允许为每个 CPU 核心绑定的完成队列（CQ）分配独立的中断向量 `[PCIe Transport §3.2]`。

---

## MSI-X 向量与 CQ 的绑定模型

在 PCIe 配置空间中，MSI-X Capability 指向设备 BAR 空间（常在 `BAR0` 高偏移或 `BAR2`/`BAR4`）中的 **MSI-X Table** 与 **PBA（Pending Bit Array）**。每个表项包含一个主机内存地址（如 x86 APIC 地址 `0xFEE0_xxxx`）和一条消息数据（中断向量号）。

```text
  CPU Core 0  ◀─── MSI-X Vector 0 ◀─── Admin CQ (CQ 0, IV=0)
  CPU Core 1  ◀─── MSI-X Vector 1 ◀─── I/O CQ 1   (IV=1)  ◀── SQ 1
  CPU Core 2  ◀─── MSI-X Vector 2 ◀─── I/O CQ 2   (IV=2)  ◀── SQ 2
```

- **创建 CQ 时绑定**：主机发送 Admin 命令 `Create I/O Completion Queue`（Opcode `05h`）时，在 `CDW11` 中填入 `IV`（Interrupt Vector，0-based）并置位 `IEN=1`（Interrupt Enabled） `[Base §5.1.6]`。
- **单核亲和性**：Linux `nvme-pci` 默认为每个在线 CPU 分配一个 I/O 队列对并绑定专属 MSI-X 向量，使提交（SQ）与完成中断（CQ）在同一 CPU 核心闭环，消除跨核缓存行弹跳。
- **数量不足时的回退**：若控制器提供的 MSI-X 向量数少于逻辑 CPU 数，多个 CQ 可共享同一个 `IV`，此时 OS 需在中断处理函数中遍历共享该向量的所有 CQ。

---

## `INTMS` / `INTMC` 寄存器 vs MSI-X 屏蔽

BAR0 中的 `INTMS`（`0Ch`）与 `INTMC`（`10h`）用于屏蔽/解除屏蔽中断向量，但在不同中断模式下有**严格禁区** `[Base §3.1.3–3.1.4]`：

| 中断模式 | 如何屏蔽单个向量 | `INTMS` / `INTMC` 行为 |
| :--- | :--- | :--- |
| **Pin-based (INTx) / 单消息 MSI** | 写 `INTMS` 置位对应 bit 屏蔽；写 `INTMC` 置位对应 bit 解除 | 正常工作（仅 Bit 0 用于 INTx） |
| **多消息 MSI / MSI-X（主流）** | 写 PCIe **MSI-X Table 条目中的 Vector Control Mask bit** | **禁止使用！** 在 MSI-X 模式下读写 `INTMS`/`INTMC` 结果**未定义** |

> **排错提示**：若在虚拟化或自研驱动中看到启用 MSI-X 后中断莫名丢失，首先检查是否误写了 `INTMS` 或忘记清除 MSI-X Table Entry 的 Mask 位。

---

## 中断合并（Interrupt Coalescing, FID `08h` / `09h`）

高频小包 I/O 时，每完成 1 条命令就发 1 次 MSI-X 会把 CPU 打满中断风暴。NVMe 允许通过 `Set Features` 配置全局或单向量的中断合并 `[Base §5.1.25.1.5]`：
- **Aggregation Time（`TIME`）**：从第一个未通知的 CQE 写入开始计时（单位 `100µs`），超时即触发中断。
- **Aggregation Threshold（`THR`）**：累积完成条数达到阈值（0-based）即触发中断。
- **Admin CQ 例外**：`CQ 0`（Admin 队列）**永远不参与中断合并**，任何 Admin 命令完成都会立即触发中断。
- **现代替代方案**：在极高 IOPS 场景下，Linux 驱动更常用 **NAPI 风格的轮询（`io_poll`）** 或 **Hybrid Polling** 配合 `Phase Tag` 翻转检测，直接关闭对应 CQ 的 `IEN` 以消除中断开销。

```bash
# 查看 NVMe 设备的 MSI-X 向量数量、Table/PBA 所在 BAR 以及 Linux 中断分配
lspci -vvv -s 0000:01:00.0 | grep -A 5 "MSI-X"
grep nvme /proc/interrupts
```

---

## 关联与引申概念

- **PCIe MSI-X 底层机制**：MSI-X Capability 结构、Table/PBA 与 TLP Memory Write 原理，详见 [MSI-X 详解](../pcie/wiki/interrupts/msix.md)。
- **虚拟化中断投递**：Hypervisor 如何用 IRTE 与 Posted Interrupts 让 MSI-X 直达虚拟机，详见 [中断重映射、VFIO 与 SR-IOV 虚拟化](../host-system/interrupt-remapping-and-vfio.md)。
- **驱动多队列绑定**：Linux `blk-mq` 与 MSI-X 向量映射细节，详见 [Linux PCIe 与 NVMe 驱动模型](../host-system/linux-pci-nvme-driver.md)。
