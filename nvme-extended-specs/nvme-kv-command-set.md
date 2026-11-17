# NVMe Key-Value（KV）命令集

在 RocksDB、对象存储与缓存引擎中，应用天然按 **`Key → Value`** 访问数据，却被迫经过文件系统和块层把 Key 翻译成固定 4KB LBA，再由 SSD FTL 维护第二层 `LBA → NAND 物理页` 映射，造成**双重映射开销与严重写放大（WAF）**。**NVMe KV Command Set（`CSI = 01h`）** 直接让 SSD 原生理解变长 Key-Value 对 `[NVMe KV CmdSet §1–3]`。

---

## 5 大核心 KV I/O 命令

| 命令 | Opcode | 数据方向 | 核心语义 |
| :--- | :--- | :--- | :--- |
| **`Store`** | `01h` | Host → Ctrl | 写入键值对；支持条件标志（如“仅当 Key 存在/不存在时才写”） |
| **`Retrieve`** | `02h` | Ctrl → Host | 根据 Key 读取对应变长 Value（支持按偏移/长度读取部分 Value） |
| **`List`** | `06h` | Ctrl → Host | 枚举命名空间内满足指定前缀（Prefix）的 Key 列表 |
| **`Delete`** | `10h` | 无数据 | 原子删除指定 Key 及其关联 Value，释放底层介质空间 |
| **`Exist`** | `14h` | 无数据 | 探测 Key 是否存在（无需传输 Value 载荷即可快速判断） |

---

## 与传统块（NVM）命名空间的本质差异

- **无固定扇区概念**：KV Namespace 不存在 `LBA`、也不需要 `Format NVM` 选 512B/4KB；Key 长度通常支持 **1–255 字节**（由 `Identify` 报告上限），Value 可达数 MB。
- **单次原子写入**：主机不再需要维护复杂的日志结构合并树（LSM-Tree）跨块对齐或预写日志（WAL）来防半写撕裂，由 SSD 内部 FTL 直接保证单条 `Store` 的原子性。

```bash
# 查看控制器是否支持 KV 命令集 (CSI=01h)
nvme id-ctrl /dev/nvme0 | grep -iE "ctratt|csupp"
```

---

## 关联与引申概念

- **命令集路由**：控制器如何在同一设备内同时暴露 NVM (`00h`) 与 KV (`01h`) 命名空间，详见 [命令集](../nvme/concepts/command-sets.md)。
- **NVM 块命令集**：传统基于 `SLBA`/`NLB` 的块访问对比，详见 [NVM 核心 I/O 命令详解](../nvme-nvm-cmdset/nvm-io-commands.md)。
