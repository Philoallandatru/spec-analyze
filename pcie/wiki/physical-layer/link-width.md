# 链路宽度 (Link Width)

**链路宽度（Link Width）** 指的是两个 PCIe 设备之间并行工作的 **Lane（通道）** 数量。如果把单个 Lane 比作一条双向两车道公路（去向和回向各一对差分线），链路宽度就是把多条 Lane 并排捆绑成一条多车道高速路——车道越多，总带宽就成倍增加。

---

## 标准链路宽度与带宽

PCIe 规范支持 **x1、x2、x4、x8、x16**（以及 PCIe 6.0 服务器级的 **x32**，少数场景也存在 x12）。每个 Lane 包含 `TX+/TX-` 和 `RX+/RX-` 共 4 根信号线：

```text
x4 链路物理连接示例（共 16 根信号线）：
设备 A                             设备 B
+-------+                         +-------+
| Lane0 |<--- TX+/TX-, RX+/RX- -->| Lane0 |
| Lane1 |<--- TX+/TX-, RX+/RX- -->| Lane1 |
| Lane2 |<--- TX+/TX-, RX+/RX- -->| Lane2 |
| Lane3 |<--- TX+/TX-, RX+/RX- -->| Lane3 |
+-------+                         +-------+
```

| 链路宽度 | 信号线总数 | Gen 3 单向带宽 | Gen 4 单向带宽 | Gen 5 单向带宽 | 常见设备类型 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **x1** | 4 线 | ~985 MB/s | ~1.97 GB/s | ~3.94 GB/s | 声卡、Wi-Fi 网卡、低速采集卡 |
| **x2** | 8 线 | ~1.97 GB/s | ~3.94 GB/s | ~7.88 GB/s | 入门级 M.2 SSD、部分主板扩展 |
| **x4** | 16 线 | ~3.94 GB/s | ~7.88 GB/s | ~15.75 GB/s | 主流 NVMe SSD、万兆/25G 网卡 |
| **x8** | 32 线 | ~7.88 GB/s | ~15.75 GB/s | ~31.51 GB/s | 100G/200G 网卡、RAID 卡、HBA |
| **x16** | 64 线 | ~15.75 GB/s | ~31.51 GB/s | ~63.01 GB/s | 独立显卡 (GPU)、AI 加速卡 |
| **x32** | 128 线 | ~31.51 GB/s | ~63.01 GB/s | ~126.03 GB/s | 服务器/高密度互连（罕见） |

---

## 宽度协商与降级机制

在 LTSSM 的 **`Configuration`** 阶段，双方通过交换 TS1/TS2 序列自动协商链路宽度：

1. **取交集原则**：以两端共同支持的最大合法宽度为准（如 x16 显卡插在电气只接了 x8 线的插槽上，最终协商为 x8）。
2. **自动降级（Degradation）**：若部分高编号 Lane 因接触不良或信号衰减无法锁定，链路会自动降级到可用的最宽合法配置（如 `x16 → x8 → x4 → x2 → x1`），保证系统仍能启动而非彻底失联。
3. **宽度升级（Upconfigure）**：若设备在 `LnkCap` 中声明支持 Upconfigure，可在空闲或重训练后尝试从窄宽度恢复到更宽配置。

---

## Lane 反转与极性反转（PCB 布线容错）

为了简化多层 PCB 的走线、避免信号线在板子上交叉绕远路，PCIe 物理层允许硬件在训练阶段自动纠正两种常见的物理反接：

### 1. Lane 反转（Lane Reversal）

发送端的 `Lane 0..N-1` 允许按倒序连接到接收端的 `Lane N-1..0`。硬件在 `Configuration` 阶段发现逻辑 Lane 号倒序时，会自动在内部重新映射：

```text
正常连接：                 Lane 反转连接（硬件自动识别并纠正）：
设备 A       设备 B        设备 A       设备 B
Lane 0 ───▶ Lane 0        Lane 0 ───▶ Lane 3
Lane 1 ───▶ Lane 1        Lane 1 ───▶ Lane 2
Lane 2 ───▶ Lane 2        Lane 2 ───▶ Lane 1
Lane 3 ───▶ Lane 3        Lane 3 ───▶ Lane 0
```

### 2. 极性反转（Polarity Inversion）

如果某条 Lane 的差分对 `P/N`（`+` 与 `-`）在 PCB 上接反，接收端会在 `Polling` 阶段检测到 TS1/TS2 标识字节按位取反，并在接收前端自动翻转极性，完全无需软件干预。

---

## 配置空间相关寄存器

链路宽度能力与协商结果记录在 PCIe Capability 结构（基址记为 `CAP_EXP`）中：

| 寄存器 | 偏移 | 关键字段 | 含义与编码 |
| :--- | :--- | :--- | :--- |
| **Link Capabilities (`LnkCap`)** | `CAP_EXP + 0x0C` | **MLW** (Bits 9:4) | 最大支持宽度：`0x01`=x1, `0x02`=x2, `0x04`=x4, `0x08`=x8, `0x10`=x16, `0x20`=x32 |
| **Link Status (`LnkSta`)** | `CAP_EXP + 0x12` | **NLW** (Bits 9:4) | 当前实际协商宽度（编码同 MLW） |
| **Link Control (`LnkCtl`)** | `CAP_EXP + 0x10` | **Retrain Link** (Bit 5) | 置 `1` 可触发链路重新训练并重新协商宽度 |

---

## Linux 观测与排查命令

```bash
# 1. 使用 lspci 对比最大能力与当前协商宽度
lspci -vvv -s 01:00.0 | grep -E "LnkCap:|LnkSta:"
# 若 LnkCap 为 Width x16，但 LnkSta 为 Width x8 (downgraded)，说明发生了宽度降级

# 2. 通过 sysfs 直接读取当前宽度与最大宽度
cat /sys/bus/pci/devices/0000:01:00.0/max_link_width
cat /sys/bus/pci/devices/0000:01:00.0/current_link_width
```

**常见宽度降级排查**：
- **插槽物理/电气限制**：主板物理 x16 插槽很多在电气上仅连接 x8 或 x4（或与 M.2 插槽共享通道）。
- **金手指接触不良**：灰尘或未插紧会导致高编号 Lane 失锁，清洁后重插即可恢复。

---

## 关联与引申概念

- **LTSSM `Configuration` 状态**：具体执行宽度协商、Lane 编号与去偏斜的状态机阶段，详见 [LTSSM 状态机](ltssm.md)。
- **链路训练（Link Training）**：TS1/TS2 中如何交换 Lane Number 与检测极性反转，详见 [链路训练过程](link-training.md)。
- **Lane 条带化（Striping）与去偏斜（Deskew）**：多 Lane 并行传输时如何拆分数据包并消除走线长度差，详见 [物理层概述](overview.md)。
- **Bifurcation（端口分叉）**：主板或交换芯片将一个 x16 端口拆分为 `x8+x8` 或 `x4+x4+x4+x4` 连接多个设备的拓扑技术。
