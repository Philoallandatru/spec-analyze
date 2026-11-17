# NVMe 扩展命令集与启动规范（KV / 计算存储 / Boot NBFT）

除最常用的块存储（NVM）与分区存储（ZNS）外，NVMe 2.x 规范族还定义了面向**非结构化对象、近数据计算与无盘网络启动**的关键扩展规范 `[NVMe 2.x Family]`。

> **20/80 裁剪原则**：聚焦 3 个最具代表性的扩展方向——**Key-Value 命令集**、**Computational Storage（计算存储）** 与 **NVMe Boot (`NBFT`)** 的核心机制与工程价值。

---

## 文档导航

1. **[NVMe Key-Value（KV）命令集](nvme-kv-command-set.md)**
   - `Store (01h)` / `Retrieve (02h)` / `List (06h)` / `Delete (10h)` / `Exist (14h)`，用变长键直接存取，消除主机侧 LBA 转换与写放大。
2. **[NVMe 计算存储（Computational Storage）](nvme-computational-storage.md)**
   - **Program** / **Subsystem Local Memory (SLM)** 与近数据计算（如过滤/解压/聚合），避免海量冷数据挤爆 PCIe 带宽。
3. **[NVMe Boot 规范与 ACPI `NBFT` 表](nvme-boot-and-nbft.md)**
   - 无本地盘服务器如何通过 UEFI + NVMe/TCP 启动，并用 **`NBFT`**（类似 `iBFT`）把网络拓扑与认证信息无损移交操作系统。

---

## 关联与引申概念

- **规范族全景**：各 I/O 命令集与 Boot 规范在 NVMe 家族中的协作定位，详见 [规范族与适用范围](../nvme/concepts/specification-family-and-scope.md)。
- **NVM 与 ZNS 命令集**：传统块存储与顺序写分区规范，详见 [NVMe NVM 命令集与 ZNS](../nvme-nvm-cmdset/README.md)。
- **NVMe/TCP 传输**：`NBFT` 最常承载的底层无盘启动网络，详见 [NVMe/TCP 传输规范](../nvme-of-transports/nvme-tcp-transport.md)。
