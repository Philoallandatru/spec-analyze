# NVM Set 与耐久度组（NVM Sets and Endurance Groups）

**NVM Set** 是从子系统中划出的逻辑（或物理）独立容量池，命名空间从中分配创建；**耐久度组（Endurance Group，EG）** 则是寿命与磨损管理的边界——一个 Endurance Group 可以只包含单个 NVM Set（按集算寿命），也可以跨多个 NVM Set 统一协调寿命。[PDF pp. 99-102](../_source/pages/page-099.md)

> **类比**：把企业级 SSD 想成一座**带分温区的冷库**——**Endurance Group** 是“温区”（同温区内统一做温度/寿命均衡），**NVM Set** 是温区里的“货架排”（按排切分容量池），而**命名空间**则是货架上的“箱子”（继承所在货架排的最优写入大小等属性）。

## 层级包含关系与标识符

```text
Domain (域)
  ├── Endurance Group Y (寿命跨 Set A/B 统一管理)
  │     ├── NVM Set A
  │     │     ├── Namespace A1 (整体位于 Set A 内，继承 Set A 属性)
  │     │     └── 未分配容量 (Unallocated NVM)
  │     └── NVM Set B
  │           └── Namespace B1
  └── Endurance Group Z (寿命仅在 Set C 内管理)
        └── NVM Set C
              └── Namespace C1
```

| 核心规则 | 具体要求 |
|---|---|
| **包含与继承** | 每个已格式化命名空间完全属于且仅属于 1 个 NVM Set（继承其最优写入大小等属性）；每个 NVM Set 属于 1 个 EG；每个 EG 属于 1 个 Domain。[PDF pp. 99-102](../_source/pages/page-099.md) |
| **标识符规格** | `NVM Set ID` 和 `Endurance Group ID` 均为 **16 位**、子系统内唯一；`0h` 为保留/无效值，在需要时填 `0h` 会返回 `Invalid Field in Command`。[PDF p. 101](../_source/pages/page-101.md) |
| **退化路径** | 若设备不支持 NVM Set，所有 `NVMSETID` 字段清零，寿命管理直接按 EG 进行；若也不支持 EG，则退化至 Domain 级。[PDF pp. 100-102](../_source/pages/page-100.md) |
| **感知控制路径** | Identify（含 NVM Set List）、Capacity Management、Namespace Management、部分 Feature（如 Read Recovery）、Connect、Create I/O SQ 及日志页均支持携带/暴露 `NVMSETID`。[PDF pp. 100-102](../_source/pages/page-100.md) |

## 耐久度监控与事件聚合

```text
Set Features (Feature 18h, 按 EG 配置告警掩码)
        │
        ▼ (若启用时条件已为真，立即触发)
Endurance Group 告警事件发生
        ├──→ 写入 Endurance Group Event Aggregate 日志 (按 EG ID 升序)
        └──→ 可选：向主机发送异步事件通知 (Asynchronous Event)
                │
                ▼
主机以 RAE=0 读取该 EG 的 Endurance Group Information 日志 (512B) ──→ 确认并移除聚合条目
```

- **Endurance Group Event Configuration（Feature `18h`）**：按 EG 选择哪些非保留的 Critical Warning（关键告警）位会进入聚合日志；`ENDGID=0` 用于关闭该次操作的告警掩码。[PDF p. 410](../_source/pages/page-410.md)
- **Endurance Group Information 日志（512 字节）**：通过 `Get Log Page` 的 16 位 `ENDGID` 选择，覆盖该组内已分配给 NVM Set 的容量及未分配容量：[PDF pp. 246-248](../_source/pages/page-246.md)
  - **健康与告警**：剩余备用空间（Spare）、阈值、已用寿命百分比，以及组级只读、可靠性降级、备用空间低于阈值三类告警（若所有 EG 均触发同一告警，SMART/Health 也会同步上报子系统级告警）。
  - **写入放大估算**：`Host Data Units Written` 不含控制器内部写，而 `Media Units Written` 包含垃圾回收（GC）等内部写，两者对比可看出组级**写放大（Write Amplification）**。读写量与耐久度均以“向上取整的十亿字节”为单位，`0` 表示可选估计值或计数器未上报。[PDF pp. 247-248](../_source/pages/page-247.md)
- **Endurance Group Event Aggregate 日志**：按升序列出有挂起事件的 EG ID；主机以 `RAE=0` 成功读取对应 EG 的 Information 日志即视为确认并移除该条目，超出列表部分返回全 `0`。[PDF p. 275](../_source/pages/page-275.md)

## 关联与引申概念

- **Write Amplification（写放大，WA）**：闪存介质实际写入量与主机下发写入量之比（由于垃圾回收、磨损均衡等后台搬移导致 $WA \ge 1$）。
- **Optimal Write Size（最优写入大小）**：NVM Set 暴露给命名空间的建议写入对齐/粒度，按此大小写入可减少跨单元读写与写放大。
- **相关文档**：
  - [storage-resource-hierarchy.md](storage-resource-hierarchy.md) — 存储资源总体层次
  - [nvm-capacity-model.md](nvm-capacity-model.md) — 容量在 EG 与 NVM Set 间的分配模型
  - [flexible-data-placement-configurations.md](flexible-data-placement-configurations.md) — FDP（灵活数据放置）配置与隔离
  - [common-controller-features.md](common-controller-features.md) — Feature 配置与 Read Recovery

## 规范依据

- [NVM Set 包含关系与命令集绑定，PDF pp. 99-100](../_source/pages/page-099.md)
- [NVM Set 属性/要求与 Endurance Group 模型，PDF p. 101](../_source/pages/page-101.md)
- [Endurance Group 要求与事件配置，PDF p. 102](../_source/pages/page-102.md)
- [Endurance Group Information 日志字段与写放大，PDF pp. 246-248](../_source/pages/page-246.md)
- [Endurance Group 事件聚合与确认，PDF p. 275](../_source/pages/page-275.md)
- [Endurance Group Event Configuration (18h)，PDF p. 410](../_source/pages/page-410.md)
