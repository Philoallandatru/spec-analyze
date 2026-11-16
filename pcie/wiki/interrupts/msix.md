# MSI-X 中断详解

**MSI-X（Extended Message Signaled Interrupts）是 PCIe 性能最强、最灵活的中断机制**。它解决了传统 MSI 最多 32 个向量且所有向量只能发往同一 CPU 的瓶颈，支持多达 **2048 个完全独立的中断向量**，是 NVMe SSD、高速网卡、GPU 等多队列设备的标准配置。

> **直观类比**：传统 MSI 把所有中断信息都挤在狭小的配置空间（256 字节）里，最多只能写 32 条；**MSI-X** 干脆在配置空间里只留两张“门牌指引”（Table 和 PBA 的偏移），把真正的中断通讯录（**MSI-X Table**）搬到了宽敞的 **BAR 内存空间**里——不仅能存 2048 个独立联系人（不同 CPU 核心），还能随时通过内存写操作单独静音（Mask）某一个联系人。

---

## 1. MSI-X 整体架构

```
┌──────────────────────────────────────────────┐
│ 配置空间：MSI-X Capability (ID = 0x11, 12B)  │
│  • Control: Enable, Function Mask, Size=N-1  │
│  • Table Offset + BIR (指向哪个 BAR)          │
│  • PBA Offset   + BIR (指向哪个 BAR)          │
└───────────────────┬──────────────────────────┘
                    │  通过 BAR 映射到主机内存空间 (MMIO)
                    ▼
┌──────────────────────────────────────────────┐
│ BAR N（设备内存空间）                         │
│  ┌────────────────────────────────────────┐  │
│  │ 设备控制寄存器（如 NVMe Doorbell）      │  │
│  ├────────────────────────────────────────┤  │
│  │ MSI-X Table（每条 16B × N 个向量）      │ ◀── OS 为每个向量写独立 Addr/Data/Mask
│  │  Entry 0: Addr=0xFEE00000, Data=0x40   │  │
│  │  Entry 1: Addr=0xFEE01000, Data=0x41   │  │
│  ├────────────────────────────────────────┤  │
│  │ PBA (Pending Bit Array，每向量 1 bit)  │ ◀── 中断被屏蔽时硬件在此置位记账
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
```

---

## 2. MSI-X Capability 结构（ID = `0x11`，12 字节）

| 偏移 | 字段 | 关键位域说明 |
|------|------|--------------|
| `+0x00` | **Cap ID / Next Ptr** | Byte 0 = `0x11`；Byte 1 = 下一个 Capability 偏移。 |
| `+0x02` | **Message Control (16b)** | **Bits 10:0**：`Table Size - 1`（`0~2047` 表示 `1~2048` 个向量）；<br>**Bit 14**：`Function Mask`（`1`=全局屏蔽所有向量）；<br>**Bit 15**：`MSI-X Enable`（`1`=启用 MSI-X）。 |
| `+0x04` | **Table Offset / BIR (32b)** | **Bits 2:0 (BIR)**：指示 Table 位于 BAR 0–5 中的哪一个；<br>**Bits 31:3**：Table 在该 BAR 内的偏移（8 字节对齐，低 3 位清零即偏移）。 |
| `+0x08` | **PBA Offset / BIR (32b)** | **Bits 2:0 (BIR)**：指示 PBA 位于哪个 BAR；<br>**Bits 31:3**：PBA 在该 BAR 内的偏移（8 字节对齐）。 |

> **注意**：MSI-X Table 和 PBA 所在的 BAR 区域（至少包含它们的页）应映射为 **Non-Prefetchable**，且驱动不应将其与普通寄存器混用或做写合并。

---

## 3. MSI-X Table 与 PBA 内存结构

### 3.1 MSI-X Table Entry（每项 16 字节）

每个中断向量在 MSI-X Table 中独占 16 字节（4 个 32 位寄存器）：

```
Entry i (偏移 = Table_Base + i * 16):
+0x00 ┌─────────────────────────────────────────┬───┐
      │       Message Address Lower [31:2]      │0 0│  目标 CPU APIC 地址低 32 位
+0x04 ├─────────────────────────────────────────┴───┤
      │       Message Address Upper [63:32]         │  目标地址高 32 位（x86 通常为 0）
+0x08 ├─────────────────────────────────────────────┤
      │       Message Data (32b，低 16 位有效)       │  中断向量号与投递模式
+0x0C ├─────────────────────────────────────────┬───┤
      │               Reserved                  │ M │  Vector Control: Bit 0 = Mask (1=屏蔽)
      └─────────────────────────────────────────┴───┘
```

### 3.2 PBA（Pending Bit Array，挂起位数组）

PBA 是一个只读位图数组（按 64 位 QWORD 组织，共 `ceil(N / 64) * 8` 字节），第 `i` 位对应第 `i` 个向量：
- 当某向量的 `Vector Control[Mask] = 1`（或全局 `Function Mask = 1`）时，如果设备内部触发了该中断，硬件**不发送 TLP**，而是将 PBA 中对应位设为 `1`。
- 当 OS 解除屏蔽（将 Mask 位清 `0`）时，硬件检测到 PBA 对应位为 `1`，立即补发一次 Memory Write TLP 并清零该 PBA 位。

---

## 4. 中断配置与发送流程

1. **发现与映射**：OS 读取 MSI-X Capability 获取向量数 `N` 与 `BIR/Offset`，将对应 BAR 映射为内核虚拟地址。
2. **逐条填表**：OS 为每个队列分配中断向量号与目标 CPU（计算各自的 `0xFEExxxxx` 地址和 Data），写入 `Entry 0 ~ Entry N-1`，并清零 `Vector Control[Mask]`。
3. **开启总开关**：置位 `Message Control[MSI-X Enable]`，并关闭 Command[Bit 10] 的 INTx。
4. **硬件触发**：设备完成任务（如 NVMe 队列 3 完成 I/O）→ 读取 `Entry 3` → 构造 **Posted Memory Write TLP**（写往 `Entry 3` 的 `Msg Addr`，数据为 `Msg Data`）→ Root Complex 直投目标 CPU 的 Local APIC。

---

## 5. 为什么 MSI-X 是多队列与 NUMA 优化的基石？

- **一对一绑定（Zero Contention）**：4 核或 64 核服务器上，NVMe SSD 可为每个 CPU 核心分配专属的 Submission/Completion 队列和独立的 MSI-X 向量。I/O 完成中断直接打到提交该 I/O 的同一个 CPU 核心上，完全消除跨核锁竞争与缓存行弹跳（Cache Line Bouncing）。
- **NUMA 亲和性**：多路服务器中，可将挂在 Socket 0 上的网卡/NVMe 队列的 MSI-X 地址全部指向 Socket 0 的 CPU，避免跨节点 UPI/xGMI 总线延迟。
- **动态调优**：运行时修改某个向量的目标 CPU，只需更新对应 Table Entry 的 `Msg Addr` 即可（先 Mask 再改再 Unmask），无需像 MSI 那样重新分配整块连续向量。

---

## 6. 实用 CLI 调试命令

```bash
# 1. 查看 MSI-X 启用状态、向量总数及 Table/PBA 所在 BAR 与偏移
lspci -vvv -s 01:00.0 | grep -A 3 "MSI-X:"
# 示例: Capabilities: [70] MSI-X: Enable+ Count=33 Masked-
#         Vector table: BAR=0 offset=00002000
#         PBA: BAR=0 offset=00003000

# 2. 查看各 MSI-X 向量（队列）在不同 CPU 核心上的触发统计
cat /proc/interrupts | grep nvme
```

---

## 关联与引申概念

| 概念 | 简要说明 | 相关文档 |
|------|----------|----------|
| **MSI** | MSI-X 的前代，最多 32 个连续向量且共用同一目标地址。 | [MSI 详解](msi.md) |
| **BAR (基址寄存器)** | MSI-X Table 和 PBA 均存放在设备 BAR 映射的内存空间中，由 BIR 指明位置。 | [BAR 详解](../configuration/bars.md) |
| **中断亲和性与重映射** | 通过修改 MSI-X Table 的 Destination ID 绑定 CPU 核心；虚拟化直通时由 IOMMU 中断重映射表（IRTE）接管。 | [中断路由详解](routing.md) |
| **PCIe 中断概述** | 横向对比 INTx、MSI、MSI-X 的性能与选型建议。 | [中断机制概述](overview.md) |
