# Legacy INTx 中断详解

**Legacy INTx 是 PCIe 为兼容传统 PCI 软件而保留的中断机制**。由于 PCIe 物理层已经取消了传统 PCI 的 4 根物理中断线（`INTA#–INTD#`），PCIe 改用**成对的 Message TLP（`Assert_INTx` 与 `Deassert_INTx`）** 在总线上模拟“拉低电平”与“释放电平”的行为。

> **直观类比**：传统 PCI 就像用一根真实电线连着警铃，设备一直按住开关（低电平有效），直到 CPU 来处理完才松手；PCIe 拆掉了这根电线，改成给设备发“两封电报”——开始中断时发一封 `Assert_INTA`，驱动清掉设备状态后，设备再发一封 `Deassert_INTA`。

---

## 1. INTx 虚拟信号机制（Message TLP）

PCIe 设备通过发送 **3DW 无数据载荷的 Message TLP** 模拟电平变化，由沿途的 Switch 逐级汇总并最终交给 Root Complex 转换为中断控制器（IOAPIC）的电平信号：

| Message Code | 含义 | 模拟动作 |
|--------------|------|----------|
| `0x20` / `0x21` / `0x22` / `0x23` | `Assert_INTA` / `B` / `C` / `D` | 触发对应逻辑中断线（拉低电平） |
| `0x24` / `0x25` / `0x26` / `0x27` | `Deassert_INTA` / `B` / `C` / `D` | 撤销对应逻辑中断线（释放电平） |

```
[PCIe 设备] ──(Assert_INTA Message TLP)──▶ [Switch 汇总] ──▶ [Root Complex] ──▶ [IOAPIC] ──▶ CPU
   (状态置位)                                                                      (IRQ 16 拉低)
                                                                                       │
[PCIe 设备] ◀──(驱动 MMIO 写清中断状态)──────────────────────────────────────────────────┘
   │
   └──────────(Deassert_INTA Message TLP)──▶ [Root Complex] ──▶ [IOAPIC 释放 IRQ 16]
```

> **注意**：`Assert` 和 `Deassert` 必须成对出现；如果驱动没有清除设备内部的中断状态，设备就不会发 `Deassert_INTx`，从而引发**中断风暴（Interrupt Storm）**，导致 CPU 陷入死循环。

---

## 2. 配置空间相关字段

INTx 的配置位于 Type 0/1 公共头部的 `0x04` 和 `0x3C–0x3D`：

- **Interrupt Pin（偏移 `0x3D`，只读）**：声明设备使用哪条逻辑引脚：
  - `0x00`：不使用 INTx（例如仅支持 MSI/MSI-X 的纯 PCIe 设备）
  - `0x01` ~ `0x04`：分别对应 `INTA#`、`INTB#`、`INTC#`、`INTD#`
- **Interrupt Line（偏移 `0x3C`，读写）**：由 BIOS/OS 写入分配好的系统 IRQ 号（仅对传统 PIC 模式有意义，现代 ACPI/IOAPIC 系统主要依赖 `_PRT` 路由表）。
- **Command[Bit 10] — Interrupt Disable**：置 `1` 时**强制禁止**设备发送任何 `Assert_INTx` 消息（切换为 MSI 或 MSI-X 时，驱动必须置位此标志以防重复触发）。

### 多功能设备的引脚错开（Pin Swizzling）

- 单功能设备通常固定使用 `INTA#`（`0x01`）。
- 多功能设备建议按 `Func 0 → INTA#`、`Func 1 → INTB#`、`Func 2 → INTC#`、`Func 3 → INTD#` 错开分配；经过 PCIe Switch 上行时，Switch 还会按端口号做标准轮转（Swizzling），以减少不同设备挤在同一条中断线上的概率。

---

## 3. 中断共享与软件处理要点

由于全系统只有 4 条逻辑线（INTA–INTD），多个 PCIe 设备不可避免地会映射到同一个系统 IRQ（例如设备 A、B、C 共用 IRQ 16）：

1. **必须使用共享标志（`IRQF_SHARED`）**：Linux 驱动注册 INTx 处理程序时必须传入 `IRQF_SHARED`，否则只有一个设备能注册成功。
2. **先读状态、再认领**：当 IRQ 16 触发时，内核会依次调用链上的所有处理函数：
   - 若设备状态寄存器表明不是本设备触发的，必须立即返回 **`IRQ_NONE`**；
   - 若确认是本设备触发的，需先通过 MMIO 写清设备中断状态（触发设备发 `Deassert`），再返回 **`IRQ_HANDLED`**。
3. **常见故障**：若所有处理函数都返回 `IRQ_NONE` 达到阈值，内核会报错 `irq XX: nobody cared` 并强制屏蔽该 IRQ；若驱动误返回 `IRQ_HANDLED` 却未清状态，则会导致中断风暴或吞掉同线其他设备的中断。

---

## 4. 为什么现代设备弃用 INTx？

- **延迟高（~5–20 μs）**：需要经过 Message TLP → RC → IOAPIC → LAPIC 多级转换，且每次都要遍历共享设备链并执行耗时的 MMIO 状态读取。
- **无法多核扩展**：最多 4 条线且通常绑定到单个目标，无法像 MSI-X 那样为 NVMe/网卡的每个 I/O 队列分配独立向量与 CPU 核心。
- **与 MSI/MSI-X 互斥**：规范要求设备在启用 MSI 或 MSI-X 后停止生成 INTx 消息。

---

## 5. 实用 CLI 调试命令

```bash
# 1. 查看设备使用的 Interrupt Pin 与系统路由后的 IRQ 号
lspci -vvv -s 01:00.0 | grep -E "Interrupt:|Control:"
# 示例: Interrupt: pin A routed to IRQ 16
#       Control: ... DisINTx+  (DisINTx+ 表示已关闭 INTx)

# 2. 查看某个 IRQ 是否被多个设备共享，以及触发次数
grep " 16:" /proc/interrupts
```

---

## 关联与引申概念

| 概念 | 简要说明 | 相关文档 |
|------|----------|----------|
| **PCIe 中断概述** | 对比 INTx、MSI、MSI-X 三代机制的延迟、数量与适用场景。 | [中断机制概述](overview.md) |
| **MSI / MSI-X** | 用 Posted Memory Write TLP 取代 INTx 消息，彻底消除共享轮询。 | [MSI 详解](msi.md) · [MSI-X 详解](msix.md) |
| **ACPI `_PRT` 与 IOAPIC** | 决定设备的 `INTA#–INTD#` 最终映射到哪个系统 IRQ 号及哪个 CPU 处理。 | [中断路由详解](routing.md) |
| **Command[Interrupt Disable]** | 配置空间偏移 `0x04` 的 Bit 10，用于软件屏蔽或切换中断模式时关闭 INTx。 | [配置空间详解](../configuration/config-space.md) |
