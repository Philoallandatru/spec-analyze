# PCIe 地址空间管理

**一句话总结**：PCIe 世界共有**内存（Memory）、配置（Configuration）、I/O 与消息（Message）**四大地址空间；系统通过配置设备上的 **BAR（基地址寄存器）** 和中间 **Bridge 的地址窗口**，让 CPU 能像访问内存一样读写外设寄存器（MMIO），也让外设能直接读写系统内存（DMA）。

> **直观类比**：把计算机的物理地址空间想象成一栋写字楼的门牌号体系——1 到 10 楼（DRAM）住着系统内存，11 楼以上（MMIO 区域）被划给了各个 PCIe 设备作为“对外接待前台”。**BAR** 就是分配给每个设备前台的门牌号范围，而 **Switch/Bridge 的地址窗口**则是楼层指示牌，确保快递员（TLP 数据包）不会走错楼层。

---

## 四大地址空间概览

| 地址空间 | 作用与特点 | 典型访问方式 |
|---------|-----------|-------------|
| **Memory Space（内存空间）** | 最核心的地址空间（32/64 位）。用于映射设备寄存器（**MMIO**，内存映射 I/O）、显存以及系统内存（**DMA**）。 | CPU 指针读写 / 设备 Memory R/W TLP |
| **Configuration Space（配置空间）** | 每个 Function 拥有独立的 4KB（前 256B 为传统 PCI 兼容头）配置空间，用于存放 ID、BAR 和能力链表。 | **ECAM**（内存映射）或传统 `0xCF8/0xCFC` 端口 |
| **I/O Space（I/O 空间）** | 仅 x86 架构兼容保留的 64KB 独立地址空间（`IN/OUT` 指令），不支持预取，现代 PCIe 设备已弃用。 | I/O Read / Write TLP |
| **Message Space（消息空间）** | 无独立地址，复用 Memory Write 语义传递控制信息。 | **MSI/MSI-X** 中断、电源管理、错误报告 Message TLP |

---

## 典型系统地址布局与 Bridge 转发窗口

在系统物理地址映射中，固件（BIOS/UEFI）会在 4GB 以下保留一段 MMIO 区域（以及 4GB 以上的 64 位高地址空间）分配给 PCIe 设备，避免与系统 DRAM 冲突：

```
系统物理地址空间
┌──────────────────────────────────────┐ 0xFFFFFFFF_FFFFFFFF
│  64-bit PCIe MMIO (GPU 显存 / 大 BAR)│
├──────────────────────────────────────┤ 4GB (0x1_0000_0000)
│  32-bit PCIe MMIO (NVMe/网卡寄存器)   │ ← 例如 0x80000000 ~ 0xBFFFFFFF
├──────────────────────────────────────┤
│  系统保留 (APIC / HPET / ECAM 区域)  │
├──────────────────────────────────────┤
│  系统主存 (DRAM)                     │
└──────────────────────────────────────┘ 0x0
```

为了让 TLP 沿着树形拓扑精准到达目标设备，每个 **Bridge（Root Port 或 Switch 端口）** 都配置有地址窗口（`Memory Base` 到 `Memory Limit`，粒度 1MB；以及支持 64 位地址的 `Prefetchable Memory` 窗口）：
- 只有目标地址落在该 Bridge 窗口范围内的下行 Memory TLP，才会被转发到下游链路，从而实现硬件级快速路由。

---

## BAR（基地址寄存器）工作机制

每个标准 Endpoint（Type 0 配置头）最多拥有 **6 个 32 位 BAR 槽位**（偏移 `0x10`–`0x24`）。

### 1. BAR 编码格式与类型
- **Bit 0**：`0` 表示 Memory Space，`1` 表示 I/O Space。
- **Bits [2:1]（Memory BAR 类型）**：`00b` = 32 位 BAR（占用 1 个槽位）；`10b` = **64 位 BAR**（占用相邻 2 个槽位，`BAR[n]` 存低 32 位，`BAR[n+1]` 存高 32 位，因此一个设备最多有 3 个 64 位 BAR）。
- **Bit 3（Prefetchable，可预取）**：置 `1` 表示读取该区域无副作用（如 GPU 显存），CPU/Bridge 可合并读写并预取缓存；设备控制寄存器（如 NVMe Doorbell）**必须设为 Non-Prefetchable（`0`）**，防止硬件误合并或乱序丢操作。

### 2. 系统如何探测 BAR 大小？（经典“写全 1”技巧）
设备出厂时并不知道系统会给它分配哪个物理地址，但硬件内部已经固定了所需空间的大小（必须是 2 的幂，例如 16KB）。固件通过以下 4 步探测大小：
1. 备份当前 BAR 原始值。
2. 向该 BAR 写入全 `1`（`0xFFFFFFFF`）。硬件会将低位表示“区域大小掩码”的比特强制保持为 `0`（不可写）。
3. 读回 BAR 值，屏蔽低 4 位属性位后取反加 1（`~mask + 1`），即得到精确的字节大小。例如读回 `0xFFFFC000`，取反加 1 为 `0x4000`（**16 KB**）。
4. 恢复 BAR 原始值，随后由系统为其分配对齐到自身大小的基地址（如 16KB 空间必须对齐到 16KB 边界）。

---

## DMA 地址视角与 IOMMU 隔离

当设备主动读写系统内存（**DMA**）时，它发出的地址视角取决于系统是否启用了 **IOMMU**（如 Intel VT-d、AMD-Vi、ARM SMMU）：

```
【无 IOMMU】 设备发出物理地址 (PA) ──────────────────────▶ 系统内存 (DRAM)
            (风险：恶意或故障设备可读写任意内存)

【有 IOMMU】 设备发出 IOVA (I/O 虚拟地址)
                │
                ▼
         [ IOMMU 页表翻译 ] (按设备 Requester ID 查表隔离)
                │
                ▼
            真实物理地址 (PA) ───────────────────────────▶ 系统内存 (DRAM)
```

在 Linux 驱动中，DMA 缓冲区通常分为两类：
- **一致性 DMA（Coherent DMA）**：CPU 与设备可随时并发读写且无需手动刷新缓存，常用于 **NVMe 提交/完成队列（SQ/CQ）**、描述符环等小块控制结构。
- **流式 DMA（Streaming DMA）**：针对单次大块数据传输（如读写 128KB 磁盘数据块），传输前后由内核维护缓存一致性（Cache Sync），性能更高。

---

## Linux 下查看地址分配

```bash
# 查看设备分配到的 BAR 物理地址与大小
lspci -vv -s 01:00.0 | grep "Region"
# 输出示例：Region 0: Memory at f8000000 (64-bit, non-prefetchable) [size=16K]

# 查看系统全局物理内存映射 (/proc/iomem)
sudo cat /proc/iomem | grep -i nvme
```

---

## 关联与引申概念

- **[设备枚举](enumeration.md)**：了解系统启动时如何按顺序扫描总线、探测所有 BAR 并最终写入基地址与 Bridge 窗口。
- **[数据包类型](packet-types.md)**：理解 CPU 访问 BAR 与设备发起 DMA 分别对应 `Memory Read/Write TLP` 的哪些字段。
- **[拓扑结构](../introduction/topology.md)**：回顾 Bridge（Root Port 与 Switch Port）在拓扑树中的层级关系。
