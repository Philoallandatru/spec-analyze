# NVMe 计算存储（Computational Storage）

在 AI 推理预处理、列式数据库扫描（如 Parquet 过滤）与全盘加密/压缩场景中，往往需要读取 TB 级原始数据，但真正有用的结果只有几 MB。如果全部通过 PCIe 总线搬回主机 CPU 再过滤，PCIe 带宽和主机内存带宽会瞬间成为瓶颈。**NVMe Computational Storage（TP4091 / CS Command Set）** 将计算逻辑直接下沉到 SSD 控制器内部执行 `[NVMe CS §1–3]`。

可以把计算存储想成**在仓库里直接分拣打包，只把选中的货发上卡车**：不再把整座仓库的货全拉回总部挑完再扔掉。

---

## 核心架构抽象：SLM 与 Program

1. **计算命名空间（Compute Namespace）**：不同于存数据的 NVM Namespace，它专门用于承载**计算程序（Program）**与**子系统本地内存（Subsystem Local Memory, SLM）**。
2. **数据搬运（`Memory Copy`）**：控制器内部直接把存储命名空间（NVM NS）的数据读入高速 **SLM**（全程不经过主机 PCIe 总线）。
3. **程序执行（`Execute`）**：SSD 内嵌的计算引擎（如 **eBPF 虚拟机**或专用硬件加速核，支持固定功能或可下载程序）直接对 SLM 中的数据执行过滤、正则匹配、解压或向量检索，最后仅把极小的结果集返回主机。

```text
  传统模式：SSD NAND ──(100 GB 原始数据经 PCIe)──▶ Host DRAM ──▶ CPU 过滤出 10 MB
  计算存储：SSD NAND ──(100 GB 内部直读)──▶ SSD SLM ──▶ SSD eBPF/硬件核 ──(仅 10 MB 结果经 PCIe)──▶ Host
```

---

## 工程定位与安全边界

- **程序沙箱**：可下载程序（Downloadable Program）通常采用 **eBPF** 字节码并经签名校验，确保不会越界访问其他租户的命名空间内存。
- **典型落地场景**：数据库谓词下推（Predicate Pushdown）、透明压缩/纠删码卸载、以及大规模日志/特征向量预筛。

---

## 关联与引申概念

- **控制器内存窗口**：主机可直接映射访问的 CMB/PMR 与 SLM 的区别，详见 [控制器内存窗口](../nvme/concepts/controller-memory-windows.md)。
- **命令集扩展**：多命令集协同模型，详见 [命令集](../nvme/concepts/command-sets.md)。
