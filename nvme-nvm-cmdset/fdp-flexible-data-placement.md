# FDP（Flexible Data Placement）实战指南：零改造降写放大

传统 SSD 写放大（WAF）高的根本原因，在于主机把不同生命周期的数据（如数据库 WAL 日志、SSTable 冷数据、临时文件）混着写进同一个 NAND 擦除块（Erase Block）；当短命数据失效时，GC（垃圾回收）被迫把同块里的长命数据反复搬走。

**ZNS** 虽然消除了 WAF，但强迫应用改造成“严格顺序写 + 自己管理 Reset Zone”，改造成本极高。**FDP（Flexible Data Placement，NVMe TP4146）** 走了一条更务实的路：**保留传统随机读写块接口（完全兼容现有 ext4/XFS/数据库），主机只需给每次 Write 打上一个“标签（Placement Handle）”**，SSD 内部就自动把不同标签的数据放进彼此隔离的物理 **Reclaim Unit（回收单元）** 中 `[Base §8.1.9 / NVM CmdSet]`。

---

## ZNS vs Streams vs FDP 核心对比

| 维度 | 传统块 SSD（无提示） | NVMe Streams（早期，已过时） | ZNS（Zoned Namespaces） | **FDP（Flexible Data Placement）** |
| :--- | :--- | :--- | :--- | :--- |
| **主机写入方式** | 任意随机写 | 任意随机写 | **强制严格顺序写**（改造成本极高） | **任意随机写**（零生态改造成本） |
| **WAF 写放大** | 高（通常 2.0–4.0+） | 中（资源有限且跨厂商不统一） | 极低（~1.0） | **极低（接近 1.0–1.1）** |
| **隔离粒度** | 无（全混在一起 GC） | 逻辑 Stream ID | Zone（必须整 Zone 擦除） | **Reclaim Unit Handle（RUH）** 映射到物理擦除单元 |

---

## FDP 工作流程与关键机制

1. **开启 FDP**：主机在 Endurance Group 级别通过 `Set Features (FID 1Dh)` 启用 FDP，并从 `Get Log Page` 读取可用的 **Placement Handle（`PH`）** 列表。
2. **带标签写入**：主机发 `Write` 命令时，在 `CDW12.DTYPE = 02h`（Data Placement）并在 **`CDW13.DSPEC`** 中填入目标 **`Placement Identifier（PID = RGIF + PH）`**。
3. **物理隔离落盘**：SSD FTL 把相同 `PH` 的写入路由到同一个物理 **Reclaim Unit（RU）**，互不干扰；即使某类临时数据集中被 `Deallocate (TRIM)`，整个 RU 也能瞬间整块回收，无需搬迁冷数据。

```bash
# 查看 Endurance Group 支持的 FDP 配置与 Reclaim Unit Handle 状态
nvme fdp configs /dev/nvme0 -e 1
nvme fdp status /dev/nvme0n1
```

---

## 关联与引申概念

- **配置与事件详情**：FDP 配置目录与 `PID` 拆分规则，详见 [灵活数据放置配置](../nvme/concepts/flexible-data-placement-configurations.md)。
- **物理资源层级**：Reclaim Group 与 Reclaim Unit 的底层定义，详见 [存储资源层级](../nvme/concepts/storage-resource-hierarchy.md)。
- **ZNS 对比**：强顺序写分区模型，详见 [ZNS（Zoned Namespaces）命令集与 Zone 状态机](zns-command-set.md)。
