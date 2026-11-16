# 命名空间（Namespace）

**命名空间（Namespace）** 是 NVMe 中主机可寻址的逻辑存储卷：从 NVM 存储资源中划出一段容量并格式化为**逻辑块（Logical Block）**，挂载到一个或多个 I/O 控制器后，主机即可通过该控制器对其进行读写。[PDF pp. 33, 46-57](../_source/pages/page-033.md)

> **类比**：把整块 SSD 想成一栋写字楼，**命名空间**就是楼里划出来的独立办公区；**分配（Allocation）**是把地划出来，**挂载（Attachment）**是给租户（控制器）发门禁卡，**可访问（Visibility）**则是从主机到这扇门的路（PCIe 或网络）是否畅通。

## 三个独立维度

理解命名空间的核心在于区分**分配、挂载、可访问**这三个互不依赖的维度（理论上共有 $2^3 = 8$ 种状态组合）：[PDF pp. 52-57](../_source/pages/page-052.md)

```text
  NVM 物理存储资源
        │
        │ 1. 分配 (Namespace Management: Create) —— 子系统级“圈地”
        ▼
  [已分配但未挂载 (allocated, inactive)]
        │
        │ 2. 挂载 (Namespace Attachment: Attach) —— 控制器级“发门禁卡”
        ▼
  控制器 A (active) ──┐
                      ├── 同一命名空间可挂载到一个（私有）或多个（共享）I/O 控制器
  控制器 B (active) ──┘
        │
        │ 3. 路径连通 (PCIe / NVMe-oF Fabric) —— 主机侧“可见性”
        ▼
  主机读写逻辑块 (LBA 0 .. N-1)
```

- **常见排错场景**：若 `lspci` 能看到 NVMe 控制器但 `nvme list` 看不到设备，通常是命名空间**尚未分配**（`nn=0`）或**已分配但未挂载**到当前控制器（此时发 I/O 命令会返回 `Invalid Field in Command`）。

## 核心规则与概念对比

1. **命令集与控制器约束**：
   - 命名空间与 **I/O 命令集（I/O Command Set）** 的绑定在创建时确定，整个生命周期不变。[PDF p. 46](../_source/pages/page-046.md)
   - 命名空间**只能**挂载到支持并已启用对应 I/O 命令集的 **I/O 控制器**（Administrative 控制器不能挂载命名空间）。
   - 与提交队列（SQ）无固定绑定：任一 I/O SQ 都可访问该控制器上所有已挂载的命名空间。
2. **标识与一致性**：
   - `NSID` 是控制器相对的临时数字句柄（有效范围 `1h ~ NN`，`0h` 无效，`FFFFFFFFh` 为广播值）；跨重启或跨路径追踪同一命名空间应使用持久标识符（**UUID > NGUID > EUI64**）。
   - 共享命名空间在所有已挂载控制器上必须呈现相同的 `NSID` 和 `Identify Namespace` 数据；并发操作的原子性仅以“接收命令的控制器”为单位，跨控制器排序由主机负责。[PDF pp. 54-56](../_source/pages/page-054.md)
3. **写保护能力（Feature ID `84h`）**：支持无保护、写保护（可解除）、直到掉电前保护（仅掉电解除）、永久保护（不可逆）四级；进入后两级前控制器必须先将易失性缓存与元数据刷盘。[PDF pp. 425-426](../_source/pages/page-425.md)

### 新手常见概念辨析

| 概念对比 | 实际区别 |
|---|---|
| **命名空间 vs 传统分区** | 命名空间是 NVMe 设备层原语，支持动态创建删除、多控制器共享；分区（Partition）和卷（Volume）是操作系统/文件系统在命名空间之上划分的逻辑结构。 |
| **命名空间 vs 逻辑块（LBA）** | 命名空间是整段逻辑卷；逻辑块（如 512B 或 4KB）是最小读写寻址单位，每个块有唯一编号 LBA（Logical Block Address）。 |
| **命名空间 vs NVM Set** | NVM Set 是底层的物理容量/性能隔离池；命名空间是从 NVM Set 中切片创建的逻辑卷。 |
| **私有 vs 共享 vs 分散** | 私有 = 同一时刻最多 1 个控制器挂载；共享 = 同子系统内多控制器并发挂载；分散 = 跨多个参与子系统并发访问。 |

## 关联与引申概念

- **LBA（Logical Block Address，逻辑块地址）**：命名空间内每个逻辑块的线性编号（从 0 开始），是主机读写 I/O 命令的寻址单位。
- **元数据（Metadata）**：附着在逻辑块上的额外信息（如端到端数据保护标签 PI/DIF、校验码），用于检测静默数据损坏。
- **相关文档**：
  - [nvm-subsystem.md](nvm-subsystem.md) — 命名空间所属的子系统边界
  - [namespace-identifiers.md](namespace-identifiers.md) — NSID 与持久身份标识
  - [namespace-access-models.md](namespace-access-models.md) — 私有、共享、分散与多路径/ANA
  - [namespace-management-lifecycle.md](namespace-management-lifecycle.md) — 创建、删除、挂载与解挂
  - [namespace-write-protection.md](namespace-write-protection.md) — 四级写保护状态机

## 规范依据

- [命名空间定义，PDF p. 33](../_source/pages/page-033.md)
- [存储模型、共享命名空间与多路径 I/O，PDF pp. 46-57](../_source/pages/page-046.md)
- [命名空间写保护 Feature 84h，PDF pp. 425-426](../_source/pages/page-425.md)
