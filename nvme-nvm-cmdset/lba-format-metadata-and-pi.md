# LBA 格式、元数据（DIF/DIX）与端到端数据保护（PI）

现代企业级存储不仅要求“读写快”，更要求杜绝**静默数据损坏（Silent Data Corruption）**——例如内存比特翻转、PCIe 线缆干扰或软件写偏（Misdirected Write）导致数据悄无声息地写错位置。NVMe 通过 **LBA Format（`FLBAS`）** 与 **端到端数据保护（End-to-End Data Protection, PI）** 解决这一问题 `[NVM CmdSet §4.2]`。

---

## 逻辑块格式（LBA Format）与 DIF vs DIX

格式化命名空间（`Format NVM`）时，控制器在 `Identify Namespace.LBAF0–LBAFn` 中列出支持的扇区组合，核心由 **LBA Data Size（`LBADS`，如 512B 或 4KB）** 和 **Metadata Size（`MS`，如 0 / 8 / 16 / 64B）** 组成。

当 `MS > 0` 时，元数据有两种物理存放布局（由 `FLBAS` Bit `4` 决定）：

```text
1. Extended LBA (DIF 模式，Metadata 与 User Data 交错拼接在同一个缓冲区)
   ┌──────────────────────┬──────────┬──────────────────────┬──────────┐
   │  LBA 0 Data (512/4K) │ Meta (8B)│  LBA 1 Data (512/4K) │ Meta (8B)│
   └──────────────────────┴──────────┴──────────────────────┴──────────┘
   * 由 SQE.DPTR (PRP/SGL) 一次性指向，适合硬件 HBA/SSD 内部校验。

2. Separate Metadata Buffer (DIX 模式，Metadata 独立存放在 MPTR 指向的缓冲区)
   SQE.DPTR ──▶ [ LBA 0 Data | LBA 1 Data | ... ]
   SQE.MPTR ──▶ [ Meta 0 (8B)| Meta 1 (8B)| ... ]
   * 适合 OS 内核/文件系统/数据库在内存中独立生成校验信息，实现“应用层到盘片”全链路保护。
```

> **4KB Native（`4Kn`）vs `512e`**：现代 SSD 物理页与 FTL 映射粒度均为 4KB（或更大）。使用 4KB LBA 格式可避免 512B 非对齐写导致的**读-改-写（Read-Modify-Write）**惩罚。

---

## 8 字节 Protection Information（PI）结构与 Type 1/2/3

若开启端到端数据保护（PI），元数据末尾（或前部，视 `DPS` 而定）包含 **8 字节 PI**，拆分为三个字段 `[NVM CmdSet §5.2]`：

| 字段 | 长度 | 含义与防呆目标 |
| :--- | :--- | :--- |
| **`Guard`** | **16-bit (2B)** | 基于用户数据计算的 **T10 CRC-16** 校验和，防止数据内容本身在传输/存储中发生比特翻转 |
| **`Application Tag`** | **16-bit (2B)** | 应用/文件系统自定义标签（如 `0xFFFF` 常作为屏蔽魔数，表示跳过校验） |
| **`Reference Tag`** | **32-bit (4B)** | 通常存放目标 **LBA 的低 32 位**（写入时由主机填入，每块递增），防止**写偏（Misdirected Write）** |

在格式化命名空间时，通过 `DPS`（Data Protection Settings）选择保护类型：

| 保护类型 | `Guard` 检查 | `RefTag` 检查规则 | 典型适用场景 |
| :--- | :--- | :--- | :--- |
| **Type 0** | 禁用 | 禁用 | 普通消费级/无 PI 需求的云盘（`MS=0` 时必为 Type 0） |
| **Type 1** | 启用 | 必须等于当前块 **LBA 低 32 位**（`ILBRT` 校验） | **企业级最常用**，严格绑定物理 LBA 位置，严防写偏 |
| **Type 2** | 启用 | 校验 `CDW14` 指定的初始参考标签并逐块递增 | 兼容传统 SCSI 逻辑映射场景（较少用） |
| **Type 3** | 启用 | **不校验 RefTag**（应用可复用这 4 字节） | LBA 会被虚拟化重映射、物理 LBA 与逻辑地址不一致的系统 |

---

## `PRINFO`（`CDW12[29:26]`）指令位域

主机每条 Read/Write 命令可通过 `PRINFO` 精细控制控制器如何处理 PI：
- **`PRACT`（Bit 3）**：Protection Information Action。置 `1` 时，若 `MS=8` 则由控制器**自动生成/剥离 PI**（主机侧无需计算 CRC，只传纯数据）；置 `0` 时由主机自行携带 PI 实现全链路保护。
- **`PRCHK`（Bits `[2:0]`）**：依次对应开启 `Guard`（Bit 2）、`AppTag`（Bit 1）、`RefTag`（Bit 0）校验。若校验失败，控制器返回 **`End-to-end Guard/Application/Reference Tag Check Error`**。

```bash
# 查看命名空间支持的 LBA Format (lbaf) 与当前生效的格式 (flbas)
nvme id-ns /dev/nvme0n1 -H | grep -E "flbas|dps|lbaf"
# 将命名空间格式化为 4KB LBA + 8B 元数据 (具体 lbaf 索引需先查询 id-ns)
# nvme format /dev/nvme0n1 --lbaf=1 --pi=1
```

---

## 关联与引申概念

- **公共命令格式**：`MPTR`（Metadata Pointer）在 64 字节 SQE 中的偏移（Bytes `[23:16]`），详见 [公共命令格式](../nvme/concepts/common-command-format.md)。
- **媒体错误与静默损坏**：PI 校验失败时的状态码与错误日志记录，详见 [完成队列条目与状态](../nvme/concepts/completion-queue-entry-and-status.md)。
- **Linux `blk-integrity`**：内核块层如何把 DIX/DIF 暴露给文件系统（如 XFS/ext4），详见 [Linux PCIe 与 NVMe 驱动模型](../host-system/linux-pci-nvme-driver.md)。
