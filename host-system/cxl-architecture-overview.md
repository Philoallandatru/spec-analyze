# CXL 3.x 架构与协议复用（基于 PCIe 5.0/6.0 PHY）

**CXL（Compute Express Link）** 是构建在标准 **PCIe 物理层（CXL 2.0 基于 PCIe 5.0 32 GT/s，CXL 3.x 基于 PCIe 6.0 64 GT/s PAM4）** 之上的高速缓存一致性互连标准。它打破了过去 PCIe 只能做非一致性 DMA I/O 的局限，让 CPU、GPU 加速器与大容量内存池能够共享同一片物理地址空间 `[CXL 3.x §1–2]`。

可以把 CXL 与 PCIe 的关系想成**同一条高铁轨道（Flex Bus）上跑的两种列车**：
- **PCIe** = 普通货运专列（只管把大块数据从 A 搬到 B，各自管各自的内存，不保证缓存一致性）。
- **CXL** = 多车厢联运快车（共享同一条物理轨道，但同时拉着设备枚举、缓存同步、内存直接读写三种业务）。

---

## Flex Bus 物理复用与三大子协议

上电链路训练（LTSSM）阶段，CPU Root Port 与设备通过 **Flex Bus** 协商：若两端都支持 CXL，链路切入 CXL 模式，在同一个 **256B Flit**（CXL 3.x / PCIe 6.0 固定流控单元，含前向纠错 FEC 与可选 **Latency-Optimized 256B Flit**）中多路复用三大协议 `[CXL 3.x §3–4]`：

| 协议 | 语义性质 | 核心作用 |
| :--- | :--- | :--- |
| **`CXL.io`** | **非一致性**（等效标准 PCIe） | 设备发现、PCIe 配置空间枚举、BAR 分配、MMIO、DMA 与中断（**所有 CXL 设备必选**） |
| **`CXL.cache`** | **缓存一致性**（Device → Host） | 允许加速器/智能网卡以 64B 缓存行粒度直接缓存并读写主机 CPU 内存，硬件自动维护一致性 |
| **`CXL.mem`** | **内存语义**（Host → Device） | 允许主机 CPU 用普通 `Load`/`Store` 指令直接读写设备上的 DRAM/持久内存，就像访问本地内存条 |

---

## Type 1 / Type 2 / Type 3 三类设备

根据所实现的协议组合，CXL 将设备划分为三类 `[CXL 3.x §2.3]`：

```text
  Type 1 (CXL.io + CXL.cache)           : 智能网卡 (SmartNIC) / 无本地内存加速器
  Type 2 (CXL.io + CXL.cache + CXL.mem) : GPU / AI 加速器 (自带 HBM，与 CPU 双向缓存一致)
  Type 3 (CXL.io + CXL.mem)             : CXL 内存扩展卡 / 内存池 (常采用 EDSFF E3.S 2T 形态)
```

---

## HDM 解码器与 CXL 3.x 内存池化（Memory Pooling）

1. **HDM（Host-managed Device Memory）Decoder**：
   - CPU 访问物理地址时，Host Bridge 与 CXL Switch 内的 **HDM Decoder** 寄存器会比对地址窗口，将落入 CXL 区间的 `Load`/`Store` 内存请求路由到对应的 `CXL.mem` 链路，并支持多设备 **交织（Interleaving）** 提升带宽。
2. **CXL 3.x 核心演进**：
   - **Switch 级交换与 Fabric 扩展**：支持多级 CXL Switch、Peer-to-Peer 直接通信与 **Dynamic Capacity Device（DCD）**，允许多个主机按需动态租借/归还同一个 **Type 3 内存池**里的容量块，解决单机内存墙（Memory Wall）与“内存孤岛”浪费问题。

```bash
# 在 Linux 中查看已枚举的 CXL 设备、内存区域与 HDM 解码器拓扑 (需安装 cxl-cli)
cxl list -M -D -R
lspci -vvv | grep -i "CXL"
```

---

## 关联与引申概念

- **底层共享物理层**：CXL 2.0/3.x 复用的 PCIe Gen5/Gen6 PAM4 与 256B Flit 机制，详见 [PCS、PMA 与 PIPE 接口](../pcie/wiki/physical-layer/pcs-pma-pipe.md)。
- **机械载体**：CXL Type 3 内存扩展模块常用的前部热插拔形态，详见 [EDSFF 架构与连接器](../pcie-form-factors/edsff-e1-e3-architecture.md)。
- **异构内存调度**：OS 如何通过 ACPI `HMAT` 识别 CXL 内存的 NUMA 延迟，详见 [ACPI 6.5 与 UEFI 2.10 PCIe/NVMe 交互机制](acpi-uefi-pcie-deep-dive.md)。
