# NVMe NVM 命令集与 ZNS 规范（NVM & ZNS Command Set）

本模块基于 **NVMe NVM Command Set Specification** 与 **NVMe Zoned Namespaces (ZNS) Command Set Specification**，拆解日常块存储与分区存储研发中最核心的 **I/O 命令布局、LBA 扇区格式与端到端数据保护（PI）、以及 ZNS 顺序写状态机**。

> **20/80 裁剪原则**：聚焦读写/释放/拷贝命令字段、512B/4K + DIF/DIX 格式转换、以及 ZNS Zone 生命周期与 `Zone Append`，省略生僻的融合对比与历史保留字段。

---

## 文档导航

1. **[NVM 核心 I/O 命令详解（Read / Write / DSM / Copy）](nvm-io-commands.md)**
   - `Read`/`Write`/`Flush`/`Write Zeroes`/`Dataset Management (TRIM)`/`Copy` 的 Opcode、`CDW10–12` 关键字段（`SLBA`、`NLB` 0-based、`FUA`、`LR`、`AD`）。
2. **[LBA 格式、元数据（DIF/DIX）与端到端数据保护（PI）](lba-format-metadata-and-pi.md)**
   - 512B vs 4KB 扇区（`FLBAS`）、Extended LBA（DIF）vs Separate Metadata（DIX）、8 字节 PI（Guard/AppTag/RefTag）与 Type 1/2/3 检查规则。
3. **[ZNS（Zoned Namespaces）命令集与 Zone 状态机](zns-command-set.md)**
   - Zone 7 种状态转换、`Zone Append (7Dh)` 解决并发写入的原理、`Zone Management Send/Receive (79h/7Ah)` 与 `MAR`/`MOR` 资源限制。
4. **[FDP（Flexible Data Placement）实战指南：零改造降写放大](fdp-flexible-data-placement.md)**
   - 对比 ZNS/Streams，保留随机写接口的同时用 `Placement Handle` 将冷热数据物理隔离进独立 Reclaim Unit。

---

## 关联与引申概念

- **命令路由与队列**：Base 规范如何按 `CSI` 选择不同的 I/O 命令集，详见 [命令集](../nvme/concepts/command-sets.md)。
- **公共命令骨架**：所有 I/O 命令共用的 64 字节 SQE 前 10 个 DWORD，详见 [公共命令格式](../nvme/concepts/common-command-format.md)。
- **命名空间抽象**：LBA 地址空间的分配与多路径挂载，详见 [命名空间](../nvme/concepts/namespace.md)。
