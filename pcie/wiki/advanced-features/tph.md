# PCIe TLP 处理提示（TPH - TLP Processing Hints）

> **一句话总结**：TPH 允许 PCIe 设备在发起 DMA 读写时，在数据包头部附带一个"投递标签（Steering Tag）"，提示 CPU/Root Complex 把数据直接塞进即将处理它的那个 CPU 核心缓存（Direct Cache Injection）中，而不是先落到慢速主内存。

如果把普通 DMA 写入比作快递员把包裹送到大楼一楼仓库（DRAM），CPU 核心处理数据时还得亲自下楼去取（Cache Miss，耗时 ~70–100 ns）；那么 **TPH** 就像在快递盒上贴一张写着"3 楼 5 号工位"的引导标签（Steering Tag），大楼智能分拣系统（Root Complex / DDIO）直接把包裹送到 5 号工位的桌面上（CPU L2/L3 缓存），CPU 伸手就能拿到（Cache Hit，耗时 ~4–14 ns）。

---

## 为什么需要 TPH？

在高速网卡（40G/100G/400G）和低延迟 NVMe SSD 场景中，**CPU 缓存未命中（Cache Miss）** 和 **跨 NUMA 节点内存访问** 往往比 PCIe 链路本身更拖慢系统吞吐：

```text
【无 TPH（传统 DMA 写入）】
设备 DMA 写入 ──▶ 数据落入 DRAM ──▶ 触发中断唤醒 CPU ──▶ CPU 读取数据发生 Cache Miss
                                                            └──▶ 从 DRAM 加载 (~70-100 ns)

【启用 TPH（Direct Cache Injection，如 Intel DDIO / AMD SDCI）】
设备 DMA 写入（携带 Steering Tag = 目标 CPU）
  ──▶ Root Complex 解析 ST ──▶ 直接注入目标 CPU 的 L3/L2 缓存（~14 ns 命中）
  ──▶ 触发中断唤醒该 CPU ──▶ CPU 立即从缓存读取处理，零 DRAM 往返！
```

---

## TLP 中的关键字段：`TH`、`PH` 与 `Steering Tag`

TPH 通过在 Memory Request TLP（如 Memory Write / Memory Read）中携带提示信息来实现，不改变数据本身的正确性（即使接收方不支持 TPH，也可以安全忽略提示，回退到普通内存访问）：

| 字段 | 位置 | 作用 |
| :--- | :--- | :--- |
| **`TH`（TLP Header Hint）** | TLP Header DW0 | 1 位标志位。`TH = 1` 表示该 TLP 携带 TPH 处理提示 |
| **`PH[1:0]`（Processing Hint）** | 3DW/4DW Header 的 `Address[1:0]`（因地址 4 字节对齐而空出） | 2 位抽象提示，描述数据的局部性/访问模式（如双向频繁读写、高频复用、优先驻留缓存等） |
| **`ST`（Steering Tag）** | TLP Header 的 `Tag` / `Byte Enable` 字段（或 Extended TPH Prefix） | **核心引导标签**（通常 8 位，Extended TPH 可达 16 位），由平台软件下发，用于索引具体的目标 CPU 核心、L2/L3 缓存切片或 NUMA 节点 |

---

## TPH Extended Capability（ID `0x0017`）与三种工作模式

TPH 能力位于端点设备的 **TPH Requester Extended Capability（Capability ID `0x0017`）** 中：

| 偏移 | 寄存器 | 关键字段与作用 |
| :--- | :--- | :--- |
| `0x04` | **TPH Requester Capability** | 声明设备支持的三种 ST 模式、是否支持 Extended TPH、以及 **ST Table Location & Size**（ST 表位于本 Capability 内还是 MSI-X 表内） |
| `0x08` | **TPH Requester Control** | `ST Mode Select`（选择工作模式）；`TPH Requester Enable`（禁用 / 仅系统内存请求启用 / 全部启用） |
| `0x0C+` | **ST Table**（可选） | 映射表：将设备内部的逻辑队列索引转换为平台真实的 `Steering Tag` 值 |

### 三种 Steering Tag 模式对比

| 模式 | 原理 | 适用场景 |
| :--- | :--- | :--- |
| **1. No ST Mode** | `TH = 1`，但不指定具体 `Steering Tag`，仅告知平台这是热点数据 | 设备不知道哪个 CPU 核心会处理数据，由 Root Complex 采用通用缓存注入策略 |
| **2. Interrupt Vector Mode** | 直接使用设备队列绑定的 **MSI/MSI-X 中断向量号** 作为索引查找或映射 `ST` | **最常用**：多队列网卡/NVMe，每个队列固定绑定一个 MSI-X 向量与一个 CPU 核心，数据与中断天然同核 |
| **3. Device Specific Mode** | 设备使用自定义逻辑索引，通过 **ST Table** 查表转换成平台 `Steering Tag` | 复杂流量分类场景，或需要比 MSI-X 向量数更多的独立 Steering Tag |

---

## 平台实现：Intel DDIO 与 AMD SDCI

TPH 是 PCIe 规范层的标准协议，而 CPU 厂商在 Root Complex 与末级缓存（LLC）侧提供了对应的硬件落地技术：

- **Intel DDIO（Data Direct I/O）**：允许 PCIe 设备（即使不带显式 ST，或结合 TPH）直接将 DMA 数据写入处理器 **LLC（L3 缓存）** 的预留区域（默认约占 LLC 路数的 10%–20%，防止冲刷掉正常 CPU 热数据）。
- **AMD SDCI（Smart Data Cache Injection）**：结合 PCIe TPH Steering Tag，可将数据直接注入目标 CCX/核心的 **L2 或 L3 缓存**，对多 CCD/NUMA 架构延迟优化尤为显著。

---

## 常用检查命令与避坑指南

```bash
# 查看设备是否支持 TPH Requester 能力及模式
lspci -vvv -s 01:00.0 | grep -A 6 "Transaction Processing Hints"

# 配合 CPU 亲和性绑定，避免 ST 指向的 CPU 与实际处理中断的 CPU 不一致
echo 4 > /proc/irq/125/smp_affinity_list
```

**常见反效果（踩坑点）**：
1. **ST 指向错误的核心**：如果 ST 把数据注入了 CPU 0 的缓存，但操作系统调度 CPU 4 去处理该中断，不仅浪费了 CPU 0 的缓存空间，CPU 4 还会发生跨核缓存同步（Cache-to-Cache Transfer），性能反而下降。务必保证 **MSI-X 中断亲和性**、**NUMA 内存分配** 与 **Steering Tag** 三者一致。
2. **缓存污染（Cache Thrashing）**：对于只做纯转发（如路由旁路）或不经过 CPU 读取的流式大数据，开启 TPH 反而会挤占宝贵的 L3 缓存空间。

---

## 关联与引申概念

- **MSI-X 中断与 CPU 亲和性**（[../interrupts/msix.md](../interrupts/msix.md)）：TPH 的 Interrupt Vector Mode 直接依赖 MSI-X 向量与目标 CPU 核心的绑定关系。
- **TLP 格式与路由**（[../transaction-layer/tlp-format.md](../transaction-layer/tlp-format.md)）：`TH` 位和 `PH[1:0]`/`ST` 直接复用或扩展了标准 Memory Request TLP 的头部字段。
- **Cache Coherency（缓存一致性）与 No Snoop**：与普通 DMA 依赖缓存侦听（Snoop）保证一致性不同，TPH 在保证硬件一致性的前提下，主动将"置无效（Invalidate）"优化为"直接注入（Cache Injection）"。
