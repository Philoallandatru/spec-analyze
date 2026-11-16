# NVM 子系统（NVM Subsystem）

**NVM 子系统（NVM Subsystem）** 是 NVMe 定义的“完整存储设备”管理与所有权边界，内部包含**控制器（Controller）**、**端口（Port）**、**命名空间（Namespace）**以及底层存储资源。主机只能通过子系统内的控制器访问数据，绝不能绕过控制器直接寻址物理介质。[PDF pp. 46-52](../_source/pages/page-046.md)

> **类比**：把 NVM 子系统想成一座**自助快递柜集群**——整个集群是一台设备的边界；**端口**是连网与供电接口；**控制器**是对外交互的操作屏；**命名空间**是逻辑柜格；底层存储组织则是背后的物理仓位。快递员（主机）只能通过操作屏（控制器）存取件，不能直接翻仓库。

## 子系统边界与存储层次

```text
                +------------------- NVM 子系统 -------------------+
                |                                                  |
  主机 A <====> |  控制器 A ──┐                                    |
                |             ├──→ 命名空间 (NS1, NS2 ...)         |
  主机 B <====> |  控制器 B ──┘         │                          |
                |                       ▼                          |
                |                  NVM Set ⊂ Endurance Group       |
                |                                ⊂ Domain          |
                |  端口 (PCIe / RDMA / TCP)                        |
                +--------------------------------------------------+
```

| 组成实体 | 核心职责与包含关系 |
|---|---|
| **控制器（Controller）** | 主机与子系统之间的命令处理接口；一个子系统可包含多个控制器（I/O、Administrative 或 Discovery 控制器）。[PDF pp. 52-58](../_source/pages/page-052.md) |
| **端口（Port）** | 隶属于子系统的物理或逻辑传输入口（如 PCIe 端口或 NVMe-oF 网络端口）。 |
| **命名空间（Namespace）** | 主机可见的逻辑块卷；创建时绑定唯一一个 **I/O 命令集**（如 NVM、ZNS、KV）且生命周期内不变，只能挂载到支持并启用该命令集的 I/O 控制器。[PDF p. 52](../_source/pages/page-052.md) |
| **底层存储组织** | **Domain**（故障/电源隔离边界）⊃ **Endurance Group**（寿命管理边界）⊃ **NVM Set**（容量分配池）/ **Reclaim Group → Reclaim Unit**（数据放置单元）⊃ **Media Unit**（物理介质单元）。[PDF p. 46](../_source/pages/page-046.md) |

- **最简形态（Figure 16）**：1 个子系统、1 个端口、1 个 Domain、1 个控制器、1 个 Endurance Group、1 个 NVM Set、1 个命名空间（典型消费级单盘 SSD）。[PDF p. 52](../_source/pages/page-052.md)
- **常见排错**：若操作系统能识别控制器（`lspci` 可见）但 `nvme list` 看不到盘，通常是因为子系统内尚未创建（`create-ns`）或未挂载（`attach-ns`）命名空间。

## 关联与引申概念

- **SUBNQN（Subsystem NVMe Qualified Name）**：全局唯一标识一个 NVM 子系统的字符串，用于 NVMe-oF 发现和跨子系统识别。
- **I/O Command Set（I/O 命令集）**：定义命名空间上可执行的数据读写语义（如传统块存储 NVM、分区命名空间 ZNS、键值存储 KV）。
- **相关文档**：
  - [controller.md](controller.md) — 子系统的对外命令处理实体
  - [namespace.md](namespace.md) — 命名空间与分配/挂载/可见性
  - [storage-resource-hierarchy.md](storage-resource-hierarchy.md) — 底层存储实体的完整包含关系
  - [transport-models.md](transport-models.md) — 端口与传输层模型

## 规范依据

- [存储实体定义与包含关系，PDF pp. 46-52](../_source/pages/page-046.md)
- [控制器与命名空间共享架构，PDF pp. 52-57](../_source/pages/page-052.md)
- [控制器架构概览，PDF p. 58](../_source/pages/page-058.md)
