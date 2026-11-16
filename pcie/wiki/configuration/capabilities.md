# PCIe 能力结构（Capabilities）

**Capability（能力结构）是 PCIe 配置空间在 `0x40–0xFF` 区域采用单向链表组织的“功能插件机制”**。设备通过它将自己支持的可选特性（如电源管理、MSI/MSI-X 中断、PCIe 链路控制）一个个串起来，供操作系统按需发现和启用。

> **直观类比**：配置空间前 64 字节（Header）就像一张固定格式的“基础登记表”，但网卡需要多队列、显卡需要电源调速、老设备不用高级中断——如果把所有字段都写死在表里会极度浪费空间。Capability 链表就像“附页目录”：登记表第 `0x34` 栏写着第一张附页在哪，每张附页开头写明“我是谁（Cap ID）”和“下一张附页在哪（Next Pointer）”，直到 `0x00` 结束。

---

## 1. Capability 链表机制

### 1.1 链表结构与遍历流程

1. 先检查 **Status 寄存器（`0x06`）的 Bit 4（Capabilities List）**：为 `1` 表示设备支持能力链表。
2. 读取 **Capabilities Pointer（`0x34`）**：获得第一个 Capability 的字节偏移（必须 4 字节对齐，至少为 `0x40`）。
3. 在该偏移处读取前 2 字节：
   - **Byte 0（Capability ID）**：标识能力类型（如 `0x01` 代表电源管理）。
   - **Byte 1（Next Pointer）**：下一个 Capability 的偏移地址；若为 **`0x00`**，表示链表结束。

```
Header (0x34): Cap Ptr = 0x40
   │
   ▼
[0x40] Cap ID = 0x01 (Power Mgmt)  │ Next = 0x50
   │
   ▼
[0x50] Cap ID = 0x05 (MSI)         │ Next = 0x70
   │
   ▼
[0x70] Cap ID = 0x10 (PCIe Cap)    │ Next = 0xB0
   │
   ▼
[0xB0] Cap ID = 0x11 (MSI-X)       │ Next = 0x00 (链表结束)
```

> **提示**：传统 PCI Capabilities 位于 `0x40–0xFF`（用 8 位指针寻址）；而 PCIe 独有的高级特性则放在 `0x100–0xFFF` 的 **Extended Capabilities** 链表中（详见 [扩展能力详解](extended-capabilities.md)）。

---

## 2. 常见 PCI Capability ID 速查

| Cap ID | 名称 | 典型长度 | 核心作用 |
|--------|------|----------|----------|
| `0x01` | **Power Management (PM)** | 8 B | 控制设备 D0–D3 功耗状态及远程唤醒（PME#） |
| `0x03` | **VPD (Vital Product Data)** | 8 B | 读取板卡序列号、型号、资产标签等只读信息 |
| `0x05` | **MSI** | 10–24 B | 消息信号中断（最多 32 个向量，2 的幂） |
| `0x09` | **Vendor Specific** | 可变 | 厂商自定义调试或专有特性寄存器 |
| `0x10` | **PCI Express Capability** | 36–60 B | **PCIe 核心能力**（设备类型、链路速率/宽度、ASPM、MPS） |
| `0x11` | **MSI-X** | 12 B | 扩展消息信号中断（最多 2048 个独立向量，Table 位于 BAR） |
| `0x14` | **Enhanced Allocation** | 可变 | 固定资源分配机制（替代传统可变 BAR，多见于集成端点） |

---

## 3. 核心 Capability 结构详解

### 3.1 Power Management Capability（`0x01`，8 字节）

```
+0x00: [Cap ID = 0x01 (8b)] [Next Ptr (8b)] [PMC (16b): 电源管理能力]
+0x04: [PMCSR (16b): 控制与状态]            [Data / Bridge Ext (16b)]
```

- **PMC（`+0x02`）**：声明设备支持的低功耗状态（D1/D2）以及在哪些状态下能发出唤醒事件（PME）。
- **PMCSR（`+0x04`）**：**Bits 1:0（Power State）** 用于切换设备电源状态：
  - `00b` = **D0**（全速工作）
  - `01b / 10b` = **D1 / D2**（浅度省电，PCIe 设备较少实现）
  - `11b` = **D3hot**（深度省电，配置空间仍可访问；若断电则进入 **D3cold**，恢复需完整重新枚举）。

### 3.2 MSI Capability（`0x05`，10–24 字节）

用内存写 TLP 替代物理中断线，支持 1 ~ 32 个向量，结构随是否支持 **64 位地址** 和 **Per-Vector Masking** 变化：

```
+0x00: [Cap ID = 0x05] [Next Ptr] [Message Control (16b)]
+0x04: [Message Address Lower (32b)]
+0x08: [Message Address Upper (32b)]     ← 仅 64-bit 时存在
+0x0C: [Message Data (16b)] [Rsvd (16b)] ← 32-bit 时位于 +0x08
+0x10: [Mask Bits (32b)]                 ← 仅支持 Per-Vector Masking 时存在
+0x14: [Pending Bits (32b)]              ← 仅支持 Per-Vector Masking 时存在
```

- **Message Control（`+0x02`）**：`Bit 0` 为 MSI Enable；`Bits 3:1 (MMC)` 为硬件支持的向量数（`2^MMC`）；`Bits 6:4 (MME)` 为 OS 实际分配的向量数。详见 [MSI 详解](../interrupts/msi.md)。

### 3.3 MSI-X Capability（`0x11`，12 字节）

现代高性能设备（NVMe、高速网卡）的首选，配置空间只放指针，真正的中断表在 BAR 空间中：

```
+0x00: [Cap ID = 0x11] [Next Ptr] [Message Control (16b)]
+0x04: [Table Offset (Bits 31:3)] [Table BIR (Bits 2:0)]
+0x08: [PBA Offset (Bits 31:3)]   [PBA BIR (Bits 2:0)]
```

- **Message Control（`+0x02`）**：`Bits 10:0` 为 `Table Size - 1`（`0~2047` 对应 `1~2048` 个向量）；`Bit 14` 为 Function Mask（全局屏蔽）；`Bit 15` 为 MSI-X Enable。
- **BIR（BAR Indicator Register）**：`0–5` 指明 MSI-X Table 和 PBA（挂起位数组）存放在哪个 BAR 中。详见 [MSI-X 详解](../interrupts/msix.md)。

### 3.4 PCI Express Capability（`0x10`，核心能力）

几乎所有 PCIe 设备都必须实现该能力，集中管理设备属性、链路协商和插槽状态：

| 偏移 | 寄存器组 | 核心信息与作用 |
|------|----------|----------------|
| `+0x02` | **PCIe Capabilities** | 识别 **Device/Port Type**：Endpoint(`0x0`)、Root Port(`0x4`)、Upstream/Downstream Switch Port(`0x5`/`0x6`)。 |
| `+0x04–0x0B` | **Device Cap / Ctl / Sta** | 协商 **Max Payload Size (MPS)**（单包最大载荷，如 256B/512B）、错误报告开关、FLR（功能级复位）支持。 |
| `+0x0C–0x13` | **Link Cap / Ctl / Sta** | 查看硬件最大速率/位宽与当前实际协商结果（如 `Speed 8GT/s, Width x4`）、配置 **ASPM**（主动状态电源管理 L0s/L1）。 |
| `+0x14–0x1B` | **Slot Cap / Ctl / Sta** | 仅 Root Port / 下游交换端口有效，管理热插拔按钮、指示灯和电源开关。 |

---

## 4. 能力优先级与互斥规则

- **中断互斥**：**MSI-X > MSI > INTx**。同一时间只能启用其中一种；启用 MSI 或 MSI-X 时，通常需将 Command 寄存器的 Bit 10（Interrupt Disable）置 1 以关闭 INTx。
- **性能与功耗权衡**：开启 ASPM（L0s/L1）可降低空闲功耗，但从低功耗恢复到 L0 状态会引入微秒级唤醒延迟，对超低延迟存储/网络设备常需关闭 ASPM。

---

## 5. 实用 CLI 调试示例

```bash
# 查看设备所有标准 Capabilities 的解析结果
lspci -vvv -s 01:00.0 | grep -A 3 "Capabilities: \["
```

典型输出解读：
```text
Capabilities: [40] Power Management version 3
    Status: D0 NoSoftRst+ PME-Enable-
Capabilities: [50] MSI: Enable- Count=1/32 Maskable- 64bit+
Capabilities: [70] Express Endpoint, MSI 00
    LnkCap: Port #0, Speed 8GT/s, Width x4
    LnkSta: Speed 8GT/s, Width x4
Capabilities: [b0] MSI-X: Enable+ Count=8 Masked-
    Vector table: BAR=0 offset=00003000
```

---

## 关联与引申概念

| 概念 | 简要说明 | 相关文档 |
|------|----------|----------|
| **Extended Capabilities** | 位于 `0x100–0xFFF` 的 16-bit ID 链表，承载 AER、SR-IOV、ACS、PTM、IDE 等 PCIe 高级扩展能力。 | [扩展能力详解](extended-capabilities.md) |
| **ECAM / MMCFG** | 允许 CPU 通过内存映射直接访问 4 KB 配置空间，是读取 `0x100+` 扩展能力的必要前提。 | [配置空间详解](config-space.md) |
| **INTx vs MSI vs MSI-X** | 三代 PCIe 中断机制，其中 MSI 和 MSI-X 通过 Capability 链表声明。 | [中断机制概述](../interrupts/overview.md) |
| **ASPM（主动状态电源管理）** | 在链路空闲时自动将 PCIe 物理层切入 L0s/L1 低功耗状态，在 PCIe Capability (`0x10`) 中配置。 | [扩展能力详解](extended-capabilities.md) |
