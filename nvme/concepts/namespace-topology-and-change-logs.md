# 命名空间拓扑与变更日志（Namespace Topology & Change Logs）

NVMe 通过四类日志页向主机暴露命名空间在子系统间与子系统内的拓扑关系及变更历史：**分散式参与者列表（`17h`）**、**可达性组日志（`1Ah`）**、**可达性关联日志（`1Bh`）**以及**已分配命名空间变更列表（`1Ch`）**。[PDF pp. 288-293](../_source/pages/page-288.md)

> **类比**：把命名空间想成**连锁公寓里的一间房**——`17h` 告诉你“哪些城市的分店有这间房的副本”；`1Ah` 是“同楼层房间清单”（共享相同的通道与可达性属性）；`1Bh` 说明“楼层之间能不能快速搬东西（Fast Copy）”；`1Ch` 则是物业的“最近一次全楼房间增删改盘点表”。

## 拓扑与变更日志全景

```text
分散式命名空间视图 (17h):
  Dispersed Namespace (指定 NSID) ──→ 参与的 NVM 子系统 NQN 列表 (每条 256B，索引 0 为本子系统)

控制器可见拓扑 (1Ah / 1Bh):
  Reachability Association (1Bh，按 RGID 升序)  ── 定义组间操作特性 (如是否支持 Fast Copy)
    └── Reachability Group (1Ah，按 NSID 升序)    ── 定义共享相同可达性属性的命名空间集合

变更追踪 (1Ch):
  正常: [ NSID 1 | NSID 2 | ... | NSID N ]  (≤ 1024)
  溢出: [ FFFFFFFFh (溢出哨兵) | 0 | ... | 0 ]  ──→ 主机必须全量重新枚举
```

| 日志类型 | Log ID | 成员与排序规则 | 核心限制与一致性协议 |
|---|---|---|---|
| **分散式参与者列表** | `17h` | 每条 256 字节存储一个子系统 `SUBNQN`；索引 0 固定为处理命令的控制器所属子系统 | 不接受 `NSID=0h` 或 `FFFFFFFFh`；目标必须为分散式命名空间；参与者增删时 Generation Counter 递增。[PDF pp. 288-289](../_source/pages/page-288.md) |
| **可达性组日志** | `1Ah` | 仅包含挂接到当前控制器的命名空间所属的组；组内 NSID 按升序排列 | `RGO=1` 省略 NSID 列表；分段读取后须重读头部核对全局变更计数。[PDF pp. 290-291](../_source/pages/page-290.md) |
| **可达性关联日志** | `1Bh` | 仅包含满足条件的可达性组所参与的关联；组 ID（RGID）按升序排列 | `RAO=1` 省略 RGID 列表；使用与 `1Ah` 相同的头部校验协议。[PDF pp. 291-293](../_source/pages/page-291.md) |
| **已分配命名空间变更列表** | `1Ch` | 最多记录 1024 个自上次读取以来发生变更的 NSID | 覆盖 Identify 变化、Attach、Detach、Create、Delete；超 1024 项时首项置 `FFFFFFFFh` 其余填 0。[PDF p. 293](../_source/pages/page-293.md) |

## 关键协议与字段语义

1. **快照一致性与索引偏移**：
   - 当日志支持索引偏移（Index Offset）时，索引 `0` 对应日志头部，索引 `1, 2, 3...` 依次对应后续变长描述符。[PDF pp. 290-292](../_source/pages/page-290.md)
   - 分段读取 `1Ah` / `1Bh` 后，主机必须**重读日志头部**并比对全局变更计数；仅当计数仍匹配时才接受该快照，否则重试。[PDF pp. 290-292](../_source/pages/page-290.md)
2. **两类变更计数器**：[PDF pp. 290-293](../_source/pages/page-290.md)
   - **全局变更计数**：控制器级复位（CLR）后从 `0` 开始，达到上限后回绕至 `0`。
   - **描述符局部计数（可选）**：从 `1` 开始，回绕至 `1`；若值为 `0` 表示控制器不支持局部计数，主机需直接比对描述符内容。
3. **可达性关联特征值（`1Bh`）**：[PDF p. 293](../_source/pages/page-293.md)
   - `01h`：基本可达性（可互访，但不保证性能特征）。
   - `02h`：成员组之间**不支持**快速复制（Fast Copy）。
   - `03h`：成员组之间**支持**快速复制（Fast Copy）。
4. **区分 `1Ch` 与 `05h`**：`1Ch`（Changed Allocated Namespace List）报告**子系统级**已分配命名空间的变更（含 Create/Delete）；`05h`（Changed Attached Namespace List）仅报告**控制器级**已挂接命名空间的变更。[PDF p. 293](../_source/pages/page-293.md)

## 关联与引申概念

- **NQN / SUBNQN（NVMe Qualified Name）**：用于在全球唯一标识 NVM 主机或 NVM 子系统的字符串（类似 iSCSI IQN）。远程参与者的发现通常由 NVMe-oF Discovery 服务完成，不在基础规范范围内。
- **Fast Copy（快速复制）**：允许子系统内部或可达性关联组之间在设备端直接拷贝数据，无需主机经由内存来回搬运。
- **相关文档**：
  - [namespace-access-models.md](namespace-access-models.md) — 访问模型与分散式命名空间
  - [namespace-management-lifecycle.md](namespace-management-lifecycle.md) — Create/Delete/Attach/Detach 如何触发变更日志
  - [namespace-identifiers.md](namespace-identifiers.md) — NSID 与 `05h` 变更通知日志
  - [log-page-retrieval.md](log-page-retrieval.md) — Get Log Page 通用读取与分段机制

## 规范依据

- [分散式参与者日志校验规则与头部，PDF p. 288](../_source/pages/page-288.md)
- [分散式参与者列表的数据格式，PDF p. 289](../_source/pages/page-289.md)
- [可达性组快照协议（1Ah），PDF pp. 290-291](../_source/pages/page-290.md)
- [可达性关联快照协议（1Bh），PDF pp. 291-293](../_source/pages/page-291.md)
- [已分配命名空间变更列表（1Ch），PDF p. 293](../_source/pages/page-293.md)
