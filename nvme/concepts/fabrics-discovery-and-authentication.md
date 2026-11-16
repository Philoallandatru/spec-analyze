# Fabrics 发现与认证（Fabrics Discovery and Authentication）

NVMe over Fabrics（NVMe-oF）将“**找路（Discovery）**”与“**进门授权（Authentication）**”分成两个独立阶段：发现服务（Discovery Service）暴露 Discovery 控制器告诉主机有哪些子系统路径与安全要求；主机随后发起 `Connect`，按需通过“传输安全通道”和“带内认证”两道门控后，才能开始正常命令流量。[PDF 第 45-46 页](../_source/pages/page-045.md)

## 心智模型：酒店查房与办入住

```text
引导信息（实现特定：主机配置 / OS 属性等）
       |
       v
[Discovery Service] ── 返回 Discovery Log (子系统路径 + 安全要求)
       |
       v
Connect 到目标控制器
       |
       +--> 需要 Fabric 安全通道？ ── 是 ──> 先建通道（建立前不收任何命令）
       |
       +--> 需要 NVMe 带内认证？  ── 是 ──> 仅接受 Authentication Send/Receive
       |
       v
正常 Fabrics / Admin / I/O 命令流量
```

> 一个 NVM 子系统可以只要求安全通道、只要求带内认证，或两者都要求。[PDF 第 46 页](../_source/pages/page-046.md)

## Discovery 控制器与安全门控

- **Discovery 控制器特征**：始终采用**基于消息（Message-Based）+ 动态控制器模型**，只实现发现相关特性——**没有 I/O 队列、没有 I/O 命令、没有命名空间、也不支持 Disconnect**。若子系统宣告了唯一 Discovery Service NQN，则 Connect 时必须同时接受该 NQN 与知名（Well-Known）Discovery NQN；否则只接受知名 NQN。[PDF 第 58, 62-63 页](../_source/pages/page-058.md)
- **持久发现连接与通知**：主机在 Connect 中传入非零 `Keep Alive Timer` 即可建立持久发现连接，控制器必须支持 Keep Alive、AER 和 Discovery 变更通知（`Notice 2h`：`LID 70h + info F0h` 为 Discovery 变更，`LID 71h + info F1h` 为 Host Discovery 变更，仅发给请求过对应事件的主机）。[PDF 第 63-65 页](../_source/pages/page-063.md)
- **带内认证（`Authentication Send / Receive`）**：遵循 SPC-5 安全协议（保留协议值报 `Invalid Parameter`），可通过命令“方向位”区分收发，且**可在 Admin 或 I/O 队列上发送**；Receive 数据在通信丢失或 CLR 后不持久保留。Connect 成功时通过 `AUTHREQ` 报告是否需认证或安全通道；Connect 失败则使用 Connect 专有状态码，不写 Error Information Log。[PDF 第 471-473, 477-478 页](../_source/pages/page-471.md)

## Discovery 日志家族与快照协议

| 日志页 `LID` | 名称与用途 | 关键特性与边界 |
|---|---|---|
| **`70h`** | **Discovery Log Page** | 仅 Discovery 控制器支持且持久；含子系统路径（`SUBTYPE=02h`）、其它发现服务引用（`01h`，深度超 8 层不强制支持）及当前服务其它路径（`03h`）；`DUPRETINFO=1` 表示同组路径返回重复信息，可任选其一代表整组监听变更 |
| **`71h`** | **Host Discovery Log** | 列出已注册的主机；必含至少 1 个 16 字节 Host Identifier 扩展属性；`NCC` 位指示主机是否已连 CDC |
| **`72h`** | **AVE Discovery Log** | 列出认证验证实体（AVE）NQN 及其 IPv4/IPv6 TCP 传输记录 |
| **`73h`** | **Pull Model DDC Request** | 携带 `ORI`、总长及 GAZ/AAZ/RAZ/日志请求 |

- **能力协商三件套**（`能力位 -> 请求位 -> 返回页标志`）：`EXTDLPES -> EXTDLPE -> EXTEND`（扩展条目）、`PLEOS -> PLEO -> PORTLCL`（仅 DDC 支持的端口本地记录）、`ALLSUBES -> ALLSUBE -> ALLSUB`（全量非过滤子系统，Host Discovery 对应 `ALLHOSTES -> ALLHOSTE -> ALLHOST`）。[PDF 第 306-308, 312-315 页](../_source/pages/page-306.md)
- **一致性快照协议**：单条命令读取为原子操作；分段读取多条时，主机应在读完 Header 与条目后**重读 Generation Counter**，若发生变化则丢弃重读（控制器亦可直接返回 `Discover Restart`）。支持索引偏移时，索引 `0` 为 Header，`1` 起为条目。[PDF 第 307 页](../_source/pages/page-307.md)
- **Entry 布局与边界**：固定 Entry 为 **1,024 字节**（含传输/地址族、子系统类型、安全需求、Port ID、Controller ID、最大 Admin SQ 大小、NQN 与传输地址等）；扩展 Entry 在 1,024 字节后追加 `TEL` 与 `NUMEXAT` 框定的可变属性列表（属性长度非零且为 4 的倍数）。[PDF 第 308-311 页](../_source/pages/page-308.md)

## Discovery 信息管理与哨兵 ID

### 1. 注册、注销与更新（DIM）

主机和 CDC 提交“主机扩展 Entry”（**必含** 16 字节 Host Identifier，路由字段清零忽略）；DDC 提交“NVM 子系统基本/扩展 Entry”（**禁止**含 Host Identifier，且仅 DDC 可标记为 port-local）。可选的 Admin Label（ASCII 或 UTF-8）长度限 `4–256` 字节；`TRADDR` 首字节为 `0` 表示使用本连接远端 IP（每条命令限 1 条）。[PDF 第 449-456 页](../_source/pages/page-449.md)

- **Register**：`1..N` 条（键匹配则替换，新键则新增；容量不足整条失败）。
- **De-register**：`1..N` 条（移除匹配键）。
- **Update**：**恰好 2 条**（第 1 条识别旧键，第 2 条原子替换）。

### 2. 拉模型日志下发（`SDLP`）

CDC 可通过 `Send Discovery Log Page (SDLP)` 向拉模型 DDC 下发 `70h`/`71h`/`72h` 日志（其它 LID 报 `Not Allowed`）。DDC 在完成项中置位 `LPUR = 1` 可注册“日志变化时重发”，该注册**仅持续到下一次 SDLP 发送为止**（一次性）。[PDF 第 469-470 页](../_source/pages/page-469.md)

### 3. Controller ID 哨兵值

| `CNTLID` 值 | 含义与规则 |
|---|---|
| `FFF0h`–`FFFCh` | 保留；在 Connect 中非法 |
| `FFFDh` | 分散命名空间远端参与方哨兵；在 Connect 中非法 |
| `FFFEh` | 分配任意可用**静态**控制器；主机拿到真实 ID 后应记住用于后续重连 |
| `FFFFh` | 分配任意可用**动态**控制器 |

> **关联排他性**：Discovery 仅识别路径，不改变“一个控制器同一时刻只关联一台主机”的原则；多主机通过同一端口连接的是不同的控制器实例。[PDF 第 44-45 页](../_source/pages/page-044.md)

## 关联与引申概念

- **传输模型**：Discovery 与认证运行在基于消息的 Fabrics 传输之上，见 [传输模型](transport-models.md)。
- **关联与命令生命周期**：认证通过后主机与控制器建立正式关联，见 [关联与命令生命周期](association-and-command-lifecycle.md)。
- **Fabric 分区模型**：CDC 如何基于 ZoneGroup 过滤 Discovery Log，见 [Fabric 分区模型](fabric-zoning-model.md)。
- **命令集**：Fabrics 命令集（Connect、Property Get/Set、Authentication Send/Receive），见 [命令集](command-sets.md)。

## 规范依据

- [Discovery Service 行为与认证门控，PDF 第 44-46 页](../_source/pages/page-044.md)
- [Discovery 控制器模型、哨兵 ID 与变更通知，PDF 第 58, 62-65 页](../_source/pages/page-058.md)
- [Discovery 日志能力、固定/扩展 Entry 与 Host/AVE/DDC 日志，PDF 第 306-317 页](../_source/pages/page-306.md)
- [Discovery 信息管理（Register/De-register/Update）与扩展属性，PDF 第 449-456 页](../_source/pages/page-449.md)
- [Send Discovery Log Page（SDLP）边界与 LPUR，PDF 第 469-470 页](../_source/pages/page-469.md)
- [Authentication Send/Receive 与 Connect AUTHREQ，PDF 第 471-473, 477-478 页](../_source/pages/page-471.md)
