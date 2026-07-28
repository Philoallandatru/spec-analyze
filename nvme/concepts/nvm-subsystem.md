# NVM 子系统（NVM Subsystem）

## 一句话说明

NVM 子系统是 NVMe 定义的"一个完整存储设备"边界：它把控制器、端口、命名空间和底层存储资源打包在一起，主机只通过子系统内部的控制器访问数据，不直接触达介质。

## 生活化类比

把 NVM 子系统想成一个**自助快递柜集群**：

- **整个集群** = NVM 子系统（一个边界、一个所有权）
- **柜机/取件屏** = 控制器（用户交互的入口）
- **柜格** = 命名空间（每一格都是一块可寻址的存储空间）
- **柜子里的物理仓位** = 底层存储资源（NVM Set / Endurance Group / Domain）
- **集群门口的网络/电源** = 端口（Port）

快递员（主机）只能在柜机（控制器）上扫码下单、放件、取件，永远不能绕过柜机去翻仓库里的货位——这层"只能通过控制器访问"就是 NVM 子系统要守住的边界。

## 工作流程

```text
                +------------------- NVM 子系统 -------------------+
                |                                                 |
  主机 A ---->  |  控制器 A  ---+                                  |
                |               |                                  |
  主机 B ---->  |  控制器 B  --+--> 命名空间 NS1 / NS2 ...         |
                |              |        |                          |
                |              |        v                          |
                |              |   NVM Set ── Reclaim Group        |
                |              |        |                          |
                |              |        v                          |
                |              |   Endurance Group ── Domain       |
                |              |                                  |
                |  Port ────── (PCIe / RDMA / TCP ...)            |
                +-------------------------------------------------+

  数据流： 主机 → 控制器 → 命名空间 → NVM Set → Reclaim Unit → 物理介质
  权限： 主机永远跨不过"控制器"这一层
```

简化说明：上图刻意省略了多端口、共享命名空间、ANA 等复杂场景，仅展示"边界 + 控制器 + 命名空间 + 底层组织"四层关系。

## 初学者案例

**场景：为什么我插了一块新 SSD，系统却找不到任何盘？**

1. 你买了一块 2TB NVMe SSD，物理插上主板，BIOS 能识别。
2. 操作系统启动后用 `nvme list` 查看：设备在，但**没有命名空间**（Namespace 列表为空）。
3. 这是因为：SSD 在出厂时 NVM 子系统存在，但**底层存储资源还没被分配成命名空间**——快递柜装好了，但柜格没划线。
4. 管理员用 `nvme create-ns` 把 NVM Set 的一部分容量划成 Namespace 1。
5. 主机再 `nvme attach-ns /dev/nvme0 -n 1` 把 NS1 挂到控制器 1。
6. 此时操作系统才能看到 `/dev/nvme0n1`，并在其上建分区/文件系统。

> 故障速查：若"控制器在但读不到盘"，90% 是命名空间没分配或没挂载，不是 NVM 子系统本身坏了。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 子系统是所有权边界 | 一个 NVM 子系统内的所有资源由同一管理实体拥有，可被控制器集中管理 |
| 主机只能通过控制器访问 | 主机永远不直接寻址介质，必须经控制器提交命令 |
| 资源包含关系 | Domain ⊂ Subsystem；Endurance Group ⊂ Domain；NVM Set ⊂ Endurance Group；Namespace ⊂ NVM Set |
| 命名空间绑定命令集 | 一个命名空间创建时就绑定唯一一个 I/O 命令集，整个生命周期不变 |
| 控制器可支持多命令集 | 一个控制器可同时启用 NVM / Key Value / Zoned 等多个 I/O 命令集 |
| 命名空间可共享 | 同一命名空间可挂到子系统内多个控制器（共享语义受各命令集规则约束） |
| 端口是子系统出入口 | Port 隶属于子系统，决定传输方式（PCIe / RDMA / TCP 等） |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| NVM 子系统 vs NVM Set | 子系统是"整台设备"的边界；NVM Set 是子系统内的容量分配单元 |
| NVM 子系统 vs 控制器 | 控制器是子系统对外的"服务员"；子系统是"店铺+仓库" |
| Domain vs Endurance Group | Domain 是故障/上电隔离边界；Endurance Group 是磨损均衡和寿命管理边界 |
| Namespace vs NVM Set | Namespace 是主机可见的逻辑卷；NVM Set 是底层容量池 |
| Port vs Controller | Port 是物理/逻辑传输出入口；Controller 是处理命令的处理单元 |

## 进阶细节

- **存储实体清单**（规范 2.3.1）：NVM Subsystem、Domain、Endurance Group、Reclaim Group、Reclaim Unit、NVM Set、Namespace、Media Unit，缺一不可用于完整建模。
- **包含关系的方向性**：
  - 每个 Domain 必须**正好属于一个** NVM Subsystem。
  - 每个 Endurance Group 必须**正好属于一个** Domain。
  - 每个 NVM Set **正好属于一个** Endurance Group；每个 Namespace **正好属于一个** NVM Set。
  - Reclaim Group ⊂ Endurance Group；Reclaim Unit ⊂ Reclaim Group。
  - Media Unit ⊂ Endurance Group。
- **I/O 命令集绑定**（规范 2.3.2）：Namespace 与 I/O 命令集的关联在创建时确定，整个生命周期固定；只有"该控制器支持且已启用该命令集"时，命名空间才能挂载。
- **示例配置**（规范 Figure 16）：最简形态 = 1 Subsystem、1 Port、1 Domain、1 Controller、1 Endurance Group、1 NVM Set、1 Namespace（即单盘消费级 SSD 的常见形态）。
- **边界意义**：子系统的存在使"故障域"、"管理域"、"可见域"在物理上是同一组设备，避免跨边界产生不明确的资源共享。

## 规范依据

- [存储实体定义，PDF 第 46 页](../_source/pages/page-046.md)
- [NVM 存储模型与包含关系，PDF 第 46 页](../_source/pages/page-046.md)
- [I/O 命令集与命名空间绑定，PDF 第 52 页](../_source/pages/page-052.md)
- [NVM 子系统示例图 Figure 16，PDF 第 52 页](../_source/pages/page-052.md)

## 相关阅读

- [controller.md](controller.md) - 子系统的对外接口实体
- [namespace.md](namespace.md) - 命名空间挂在子系统内
- [transport-models.md](transport-models.md) - 端口决定子系统传输方式
- [command-sets.md](command-sets.md) - 命名空间创建时绑定命令集
