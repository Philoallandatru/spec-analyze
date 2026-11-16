# 资源与管理清单日志（Resource and Management Inventory Logs）

资源与管理清单日志是 NVMe 中一组**只读、无状态协调成本**的清单型日志，用于在特定资源边界上报告"设备有什么物理资源、由谁在带外管理"。它们不是命令历史，主要包括耐久度组（Endurance Group）粒度的**旋转介质信息日志（LID `16h`）**和 NVM 子系统粒度的**管理地址列表日志（LID `18h`）**。[PDF pp. 287-289](../_source/pages/page-287.md)

可以把它们想成**服务器机箱上的资产铭牌**：
- **LID `16h`（旋转介质信息）** = 机械硬盘的物理铭牌：标明有几根磁头臂（Actuator）、转速（RPM）多少、电机起停成功/失败各多少次。
- **LID `18h`（管理地址列表）** = 机箱背面的运维二维码：列出带外管理端点（如 BMC 的 Redfish HTTPS 地址、Fabric 交换机的 SNMP URI），看一眼就能找到管理入口。

---

## 心智模型与清单对比

```text
  Endurance Group（由 Command Dword 11 中的 ENDGID 选定）
  └── LID 16h 旋转介质信息：Actuator 数量、PS0 转速、Spinup/Load 成功与失败计数

  NVM Subsystem（子系统全局，无需选择器）
  └── LID 18h 管理地址列表：0~8 个 512 字节定长描述符（MAT + NUL 结尾 UTF-8 URI）
```

| 日志名称与 LID | 作用范围 / 选择符 | 关键契约与不变量 |
|----------------|-------------------|------------------|
| **Rotational Media Information (`16h`)** | 通过 `ENDGID` 选定 **Endurance Group** | 跨电源循环和重置保留（终身统计）；报告 Actuator 数、PS0 转速、32 位**饱和**事件计数。无旋转介质 EG 的子系统不应支持此日志 [PDF pp. 287-288](../_source/pages/page-287.md) |
| **Management Address List (`18h`)** | **NVM 子系统**全局 | 包含 0 到 8 个 512 字节描述符（`MAD0`–`MAD7`）；从 `MAD0` 扫描至 `MAD7` 或首个 `MAT=FFh` 终止符停止 [PDF p. 289](../_source/pages/page-289.md) |

---

## 核心字段与规则详解

### 1. 旋转介质信息日志（LID `16h`，Figure 269）

- **字段结构**：`01:00`=`ENDGID`；`03:02`=`NUMA`（Actuator 数量）；`05:04`=`NRS`（Power State 0 下的标称转速 RPM：`0000h`=未报告，`0001h` 与 `FFFFh` 保留）；`11:08` 起为 `SPINC`（Spinup 成功次数）及 Load 成功计数、Spinup/Load 失败计数。
- **Spinup vs Load**：**Spinup** 指控制器从非操作态进入操作态的转换；**Load** 指单个 Actuator（机械臂）自身的对应状态转换。
- **饱和计数语义**：所有 32 位计数器递增至 `FFFFFFFFh` 后**饱和停留**，不会回绕到 `0`；成功与失败次数分开计数。

### 2. 管理地址列表日志（LID `18h`，Figures 271-272）

每个 512 字节描述符包含 `Byte 00`=`MAT`（Management Address Type）、`03:01`=保留、`511:04`=`MADRS`（以 NUL 结尾的 UTF-8 RFC 3986 URI）：

| `MAT` 编码 | 含义 |
|------------|------|
| `0h` | 保留 `MADRS` 地址字段（该槽位无有效 URI） |
| `1h` | 子系统管理代理（Subsystem Management Agent，如板载 BMC） |
| `2h` | Fabric 接口管理器（Fabric Interface Manager，如交换机管理代理） |
| `FFh` | 终止符（表示后续所有描述符均无效/保留） |

**常见命令行用法**：
```bash
# 读取子系统管理地址列表（LID 18h，8 × 512 = 4096 字节）
nvme get-log /dev/nvme0n1 --log-id=0x18 --log-len=4096
# 读取 Endurance Group 2 的旋转介质信息（LID 16h）
nvme get-log /dev/nvme0n1 --log-id=0x16 --log-len=512 --endgid=2
```

---

## 关联与引申概念

- **带外管理（Out-of-Band Management）**：管理地址列表提供的 URI 属于带外通道（不经过主机 NVMe I/O 队列或 Association），通常配合 **NVMe-MI（Management Interface）** 或 Redfish/SNMP 使用。
- **Endurance Group（耐久度组）**：LID `16h` 补充了耐久度组在旋转介质物理层面的健康状态，而寿命消耗和容量统计则由 Endurance Group Information 日志（LID `09h`）报告。
- **Dispersed Namespace Participating NVM Subsystems Log（LID `19h`）**：姊妹清单日志，但列出的是参与分布式命名空间的 NQN 列表，而非管理 URI。
- **相关文档**：
  - [日志页获取](log-page-retrieval.md) — `Get Log Page` 通用机制与 `ENDGID` 选择器
  - [NVM 集与耐久度组](nvm-sets-and-endurance-groups.md) — Endurance Group 资源模型
  - [分布式命名空间生命周期](dispersed-namespace-lifecycle.md) — LID `19h` 姊妹清单日志
  - [控制器虚拟化资源](controller-virtualization-resources.md) — 资源管理视角

## 规范依据

- [旋转介质信息日志选择器与字段定义（Figure 269），PDF pp. 287-288](../_source/pages/page-287.md)
- [管理地址列表与描述符（Figures 271-272），PDF p. 289](../_source/pages/page-289.md)
- [Log Page Retrieval 通用机制，PDF p. 223](../_source/pages/page-223.md)
