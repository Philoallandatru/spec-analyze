# 控制器虚拟化资源（Controller Virtualization Resources）

控制器虚拟化资源是**主控制器（Primary Controller）**统一管理的资源池，用于在自身与最多 **127 个从控制器（Secondary Controller）**（如 PCIe SR-IOV VF）之间动态调配**虚拟队列（VQ, Virtual Queue）**和**虚拟中断（VI, Virtual Interrupt）**容量。[PDF pp. 366-368](../_source/pages/page-366.md)

可以把它想成**公寓楼的总配电箱**：主控制器是物业，拥有总容量池（VQ/VI 灵活资源）和物业自用电表（私有资源，租户不可用）；从控制器是租户，物业必须先让某户搬出（Offline）、调好电表配额，再让其入住（Online）。

## 资源池模型与类型

```text
主控制器资源池（通过 Identify CNS=14h 查询 Primary Controller Capabilities）
├── VQ 灵活资源总量 (VQFRT)  ──> 已分配给从控制器 (VQRFA) + 主控制器保留 (VQRFAP) + 未分配池
├── VQ 私有资源总量 (VQPRT)  ──> 仅主控制器专用，不可分配给从控制器
├── VI 灵活资源总量 (VIFRT)  ──> 已分配给从控制器 (VIRFA) + 主控制器保留 (VIRFAP) + 未分配池
└── VI 私有资源总量 (VIPRT)  ──> 仅主控制器专用，不可分配给从控制器
```

[PDF pp. 366-367](../_source/pages/page-366.md)

| 资源类型 | 资源单位与标识符 | 独立性与继承约束 |
|---|---|---|
| **VQ（虚拟队列）** | **1 个 SQ + 1 个 CQ 配对**；VQ ID = Queue ID | VQ 与 VI 相互独立；不支持的类型全部视为私有资源 |
| **VI（虚拟中断）** | **1 个中断向量**（NVMe 2.1 仅支持 MSI-X）；VI ID = 中断向量号 | **能力继承**：主控制器支持的资源类型（`CRT.VQRS`/`VIRS`），所有从控制器**必须**同样支持 [PDF pp. 624-625](../_source/pages/page-624.md) |

- **上限与粒度**：每个从控制器受单控制器灵活资源上限（`VQFRSM`/`VIFRSM`）约束，并有首选分配粒度（`VQGRAN`/`VIGRAN`）以减少碎片浪费。
- **主控制器自身调整**：主控制器的灵活资源分配可通过虚拟化管理命令修改，并在下一次适用的 Controller Level Reset 后生效。[PDF p. 367](../_source/pages/page-367.md)

## 从控制器列表与生命周期状态机

通过 Identify `CNS=15h` 可从指定 `CNTID` 起查询最多 **127 个**关联从控制器（包含 Offline 状态和被禁用的 SR-IOV VF），返回其 Controller ID、主控制器 ID、在线/离线状态、SR-IOV VF 编号（非 VF 类型报 0）以及当前分配的 VQ/VI 数量。[PDF pp. 367-368](../_source/pages/page-367.md)

```text
[从控制器：Online 在线]
       |  Virtualization Management: Offline
       v
[从控制器：Offline 离线]（所有灵活 VQ/VI 资源被移除）
       |-- Virtualization Management: Assign VQ 资源（指定 CNTLID, NR）
       |-- Virtualization Management: Assign VI 资源（指定 CNTLID, NR）
       |  Virtualization Management: Online（要求配置完成且主控制器已使能）
       v
[从控制器：Online 在线]
```

[PDF pp. 446-447](../_source/pages/page-446.md)

## 虚拟化管理命令与排错

仅具备 Virtualization Enhancements 能力的主控制器可执行该命令，核心参数包括 `Resource Type`（VQ/VI）、目标 `CNTLID` 和请求数量 `NR`：[PDF pp. 446-448](../_source/pages/page-446.md)

- **完成结果（`NRM`）**：对主控制器调整或从控制器分配操作，CQE DW0 返回实际被修改的资源数 `NRM`（根据实现，可能小于、等于或大于请求的 `NR`）。
- **门控与常见报错**：
  - 资源分配要求目标从控制器处于 **Offline** 状态；Online 要求配置完成且主控制器已使能。
  - 请求不存在、属于私有资源或超出剩余可用灵活池，会返回 `Invalid Resource Identifier`（若超单从控制器上限 `VQFRSM`/`VIFRSM` 或总量耗尽也会被拒）。
  - 排错顺序：**先查总量（`VQFRT`/`VIFRT`）剩余 → 再查单从控制器上限（`VQFRSM`/`VIFRSM`）→ 最后确认未误请求私有资源**。

## 概念辨析

| 容易混淆 | 实际区别 |
|---|---|
| VQ 灵活资源 vs I/O 队列 | VQ 是允许创建的队列**配额上限**；I/O 队列是实际创建的 SQ/CQ 实例 |
| `NR` vs `NRM` | `NR` 是主机请求的分配数量；`NRM` 是 CQE 返回的实际修改数量 |
| `CRT.VQRS` vs `VQFRT` | `CRT.VQRS=1` 表示支持 VQ 资源；`VQFRT` 表示灵活资源池的具体总量 |

## 关联与引申概念

- **控制器模型（[Controller](controller.md)）**：定义主控制器（Primary）与从控制器（Secondary）的角色维度。[PDF pp. 34-35](../_source/pages/page-034.md)
- **Identify 命令模型（[Identify Command Model](identify-command-model.md)）**：提供 `CNS=14h`（主控制器能力）与 `CNS=15h`（从控制器列表）查询封装。[PDF pp. 317-320](../_source/pages/page-317.md)
- **队列对（[Queue Pair](queue-pair.md)）**：每个 VQ 资源对应的底层 SQ/CQ 队列对机制。
- **SR-IOV（Single Root I/O Virtualization）**：PCIe 硬件虚拟化标准，将单个物理功能（PF，对应主控制器）虚拟出多个虚拟功能（VF，对应从控制器）直接分配给虚拟机使用。

## 规范依据

- [Primary Controller Capabilities 结构（Figure 330），PDF 第 366 页](../_source/pages/page-366.md)
- [VQ/VI 资源池约束与从控制器列表（Figure 331），PDF 第 367-368 页](../_source/pages/page-367.md)
- [虚拟化管理命令与门控（5.2.6），PDF 第 446-448 页](../_source/pages/page-446.md)
- [VQ/VI 资源定义与独立性，PDF 第 624-625 页](../_source/pages/page-624.md)
