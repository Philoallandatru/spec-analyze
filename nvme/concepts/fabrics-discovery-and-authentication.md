# Fabrics 发现与认证（Fabrics Discovery and Authentication）

## 一句话说明

NVMe over Fabrics 把"找路"和"进门"分两步：发现服务（Discovery Service）告诉主机哪些 NVM 子系统与路径可用；之后主机连接控制器时按需建立传输安全通道并完成带内认证，两道门都通过后才有正常命令流量。

## 生活化类比

把发现与认证想成**酒店入住的两步**：

- **发现（Discovery）** = 查"哪些酒店有空房、怎么去"：问前台（Discovery Service）拿一份清单（Discovery Log Page），上面列了酒店地址、房间类型、入住要求。
- **传输安全通道** = 酒店门口的安全门：没它（明文）不能进客房。
- **带内认证** = 办入住：出示证件、签单（Authentication Send/Receive）；认证完才能用自己的房卡（正常命令）。

发现服务只管"指路"，不管"进门"；进门前必须看清两份要求：门口有没有安全门、办入住要不要认证。

## 工作流程

```text
  引导信息（实现特定：主机配置 / OS 提示 / 其他）
         |
         v
  [Discovery Service]
         |  返回子系统路径 + 安全需求
         v
  Connect 到控制器
         |
   +-----+-----+---- 是否需要安全通道？
   |               |
   |  建立通道     | 建立前不接受任何 Fabrics/Admin/I/O 命令
   |               |
   +-------+-------+
           |
     是否需要带内认证？
           |
   +-------+-------+   队列只接受 Authentication Send/Receive
   |  在 Connect   |   直到认证完成
   |  创建的队列   |
   +---------------+
           |
           v
     正常命令流量
```

简化说明：发现和认证可以**单独存在或同时存在**。一个 NVM 子系统可只要求安全通道、只要求带内认证，或两者都要求。

## 初学者案例

**场景：NVMe-oF 客户端拿到 Discovery 日志后连接失败 `AUTHREQ`**

1. 主机从 Discovery Service 拿到子系统 NQN + 传输地址 + `TRS`（Transport Requirements）字段。
2. `TRS` 标记需要安全通道 + 带内认证。
3. 主机 Connect 后立刻发 Identify 命令 → `AUTHREQ` 返回"需要认证"或"需要安全通道"。
4. 解决：
   - 若是认证要求：在该队列上**只**发 Authentication Send/Receive；先 Send 协商参数、Receive 读取控制器状态，循环直到成功。
   - 若是安全通道要求：在 Connect 之外用底层协议（IPsec、RDMA PS 等）建立通道。
5. 主机误把 Identify 当普通命令发出 → 控制器拒；这是**预期行为**。

> 错误码速记：Connect 失败用**连接特定状态码** + 参数位置信息（不是 `Invalid Field in Command`，也不会写 Error Information Log）。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 发现服务只指路 | Discovery Service 暴露 Discovery 控制器；不处理 I/O |
| 两道门可独立 | 安全通道 / 带内认证可只一个，也可都要求 |
| 引导信息实现特定 | 主机如何发现 Discovery Service 不在规范范围 |
| Discovery 控制器特性 | 必为消息 + 动态模型；只支持发现相关特性 |
| Discovery 控制器无 I/O | 无 I/O Queue、无 I/O 命令、无命名空间 |
| Discovery NQN 规则 | 唯一 Discovery Service NQN 必须接受该 NQN + 知名 NQN；否则只接受知名 NQN |
| Authentication Send | 传安全协议命令或参数到控制器 |
| Authentication Receive | 取回协议定义的状态/数据（关联于一次或多次 Send） |
| 安全协议 | 由 SPC-5 选定；NVMe 提供 FCTYPE/SGL/协议选择器/传输与分配长度 |
| 保留协议值 | 视为 `Invalid Parameter` |
| 认证数据时效 | Authentication Receive 数据在通信丢失和 CLR 后**临时**丢失 |
| 队列灵活性 | Authentication Send/Receive 可在 Admin 或 I/O Queue 发 |
| 方向位 | Send = 主机→控制器；Receive = 控制器→主机 |
| Connect 成功 | `AUTHREQ` 报告是否需认证 / 认证 + 安全通道 |
| Connect 失败 | 用连接特定状态码 + 参数位置信息（不写 Error Information Log） |
| Discovery Log Page | LID `70h`，**仅在 Discovery 控制器**；持久 |
| Host Discovery Log | LID `71h`，仅 Discovery 控制器；记录注册过发现信息的主机 |
| AVE Discovery Log | LID `72h`，认证验证实体 |
| Pull Model DDC Request | LID `73h`，DDC 向 CDC 拉日志 |
| 能力协商 | `EXTDLPE→EXTDLPE→EXTEND`、`PLEO→PLEO→PORTLCL`（仅 DDC）、`ALLSUBE→ALLSUBE→ALLSUB` |
| 快照协议 | 分段读时重读 Generation Counter；变则丢弃重读 |
| 索引偏移 | 索引 0 = 头部；1 = 第一条 |
| 主机级条目 | Host 提交 = Host 扩展条目；DDC 提交 = NVM 子系统基本/扩展条目 |
| 头部 NQN | UUID 形式的 NQN 标识提交方 |
| 键选择 | 键可基于传输地址（主机记录必须）或 Port ID（子系统记录允许） |
| 任务基数 | Register = 1..N；De-register = 1..N；Update = 恰好 2 |
| 容量不足 | Register 整体失败 |
| 字段冲突 | 报 `Invalid Discovery Information` |
| 主机扩展条目 | 至少含 1 个 Host Identifier 扩展属性 |
| 子系统扩展条目 | 可无 Host Identifier 扩展属性 |
| Extended 条目 | 1024 字节前缀 + `TEL` / `NUMEXAT` 框架的可变属性列表 |
| TRADDR 首字节为 0 | 选择连接的远端 IP；该条目注册/注销每条命令限 1 条 |
| 扩展属性长度 | 必非 0、必 4 的倍数；未用字节为 0 |
| Host Identifier 属性 | 主机条目必含（16 字节）；子系统条目禁止 |
| Admin label | ASCII 或 UTF-8 可选，4–256 字节 |
| Vendor specific | 长度由类型决定 |
| 主机条目路由字段 | 子系统类型 / Port ID / Controller ID / Admin SQ 大小清零忽略 |
| 共同字段 | transport、NQN、地址、总长度、属性列表字段两种条目都用 |
| Host Discovery | 可返回请求者子集；`ALLHOSTES` 声明 + `ALLHOSTE` 请求 = 全量；`ALLHOST` 标志表示范围 |
| Host Discovery 条目 | 必含至少一个 Host Identifier 扩展属性；`NCC` 报告主机是否已连 CDC；非 CDC 该位为 0 忽略 |
| AVE Discovery | AVE NQN + 0 或多个 20 字节 IPv4/IPv6 TCP 传输记录 |
| Pull Model DDC | 包含 `ORI`、总长、GAZ/AAZ/RAZ/Discovery Log Page Request |
| 持久发现连接 | 非零 Keep Alive Timer；必须支持 Keep Alive、AER、Discovery 变更通知；Discovery 永不实现 Disconnect |
| 控制器 ID 哨兵 | `FFF0h–FFFCh` = 保留 / Connect 非法；`FFFDh` = 跨子系统分散；`FFFEh` = 任意静态；`FFFFh` = 任意动态 |
| 静态分配记住 ID | `FFFEh` 拿到后主机应记住真实 Controller ID 用于重连 |
| 变更通知 | `Notice (2h)` + `LID 70h` + `information F0h` = Discovery 变更；`LID 71h` + `F1h` = Host Discovery 变更 |
| 通知对象 | 只发给**请求过对应异步事件类型**的实体 |
| SDLP 限制 | 仅 Discovery (`70h`)、Host Discovery (`71h`)、AVE Discovery (`72h`) 允许；其他 LID 不带数据报 `Not Allowed` |
| LPUR | Pull-model DDC 可在完成里置 1 请求重发；注册**仅持续到下次 SDLP** |
| 关联边界 | 一个控制器同一时刻只与一个主机关联；多主机经同一端口到达不同控制器 ≠ 多主机共享关联 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Discovery Service vs Discovery 控制器 | Service 是"找路系统"；控制器是 Service 的接口端点 |
| 引导信息 vs Discovery Log | 引导信息（实现特定）告诉你"去哪里问"；Discovery Log 告诉你"有什么可访问" |
| 安全通道 vs 带内认证 | 安全通道在传输层（底层协议）；带内认证在 NVMe 命令层 |
| EXTDLPE vs PLEOS vs ALLSUBES | 三个**能力位**，分别对应扩展 / 端口本地 / 全量子系统支持 |
| EXTDLPE vs PLEO vs ALLSUBE | 上行是"能力位"，下行是"请求位" |
| EXTEND vs PORTLCL vs ALLSUB | 上行是"能力位"，下行是"返回页"标志 |
| Register vs De-register vs Update | Register 1..N；De-register 1..N；Update 恰好 2 |
| 主机条目 vs 子系统条目 | 主机条目用 Host 扩展；子系统条目用 NVM 子系统条目 |
| 头部 NQN vs 提交条目 NQN | 头部 NQN 标识提交方；条目 NQN 标识被操作实体 |
| 传输地址键 vs Port ID 键 | 主机记录**必须**用传输地址；子系统记录可用任一 |
| FFFEh vs FFFFh vs FFFDh | FFFEh = 任意静态；FFFFh = 任意动态；FFFDh = 跨子系统分散 |
| 子系统清单 vs 主机清单 | 70h 列子系统；71h 列主机 |
| Pull Model vs Push Model | Pull = DDC 主动从 CDC 拉（73h）；Push = CDC 主动注册到 DDC |
| 通知 vs 轮询 | 通知通过 AER；轮询通过 Get Log Page |
| LPUR vs Keep Alive | LPUR 是"请求重发该日志"；Keep Alive 是"控制器还活着" |
| 控制器 ID FFFEh 取得后 | 主机应**记住**真实 ID 用于重连，不是每次 FFFEh |
| Discovery 控制器 vs I/O 控制器 | Discovery 不挂载命名空间；I/O 才能挂载 |

## 进阶细节

- **Discovery Service 行为**（规范 4.2.1.1 / 3.1.3.3 / 5.1.13）：
  - 引导信息：实现特定（主机配置 / OS 属性 / 其他）
  - 列举可访问子系统、多路径、静态配置的控制器
  - 可支持持久 Discovery 控制器连接 + 异步通知
- **Discovery 控制器特性**（规范 3.1.2 / 3.1.3.3）：
  - 必为消息（Message-Based）+ 动态模型
  - 仅支持发现相关特性
  - 无 I/O Queues / 无 I/O 命令 / 无命名空间
- **Discovery NQN 规则**（规范 3.1.3.3 / 5.2.13）：
  - 唯一 Discovery Service NQN ⇒ 必须接受该 NQN + 知名 NQN
  - 否则只接受知名 NQN
- **认证门控**（规范 4.1.3 / 6.3）：
  - Connect 后若需认证：队列**只接受** Authentication Send/Receive 直到成功
  - AUTHREQ 报告：是否需认证 / 认证 + 安全通道
- **Authentication Send/Receive**（规范 6.x / 5.2.13）：
  - Send = 主机→控制器，传输安全协议命令/参数
  - Receive = 控制器→主机，取回协议定义状态/数据
  - FCTYPE/SGL/协议选择器/传输与分配长度由 NVMe 提供
  - 协议语义由 SPC-5 安全协议定义
  - 保留协议值 → `Invalid Parameter`
  - Receive 数据在通信丢失和 CLR 后**临时**
  - 两者可在 Admin 或 I/O Queue 发
  - 方向位：Send vs Receive 区分
- **Discovery 日志**（规范 5.2.12）：
  - LID `70h`：子系统路径 / referrals
  - LID `71h`：已注册主机
  - LID `72h`：认证验证实体（AVE）
  - LID `73h`：Pull-model DDC 请求
- **条目类型**（规范 5.2.12.1.18 / 4.2.1.1）：
  - `SUBTYPE=01h` = referral 到另一 Discovery Service
  - `SUBTYPE=02h` = NVM 子系统（控制器可能暴露命名空间）
  - `SUBTYPE=03h` = 当前 Discovery Service 的另一访问路径
  - `DUPRETINFO=1` = 当前服务条目返回重复信息，可让一条路径代表整组
- **能力协商三件套**（规范 5.2.12.1.18）：
  - `EXTDLPE` 能力位 → `EXTDLPE` 请求位 → `EXTEND` 返回标志
  - `PLEOS` 能力位（仅 DDC） → `PLEO` 请求位 → `PORTLCL` 返回标志
  - `ALLSUBES` 能力位 → `ALLSUBE` 请求位 → `ALLSUB` 返回标志
- **快照协议**（规范 5.2.12.1.18）：单命令读原子；多命令读需重读 Generation Counter；变则丢弃重读。
- **索引偏移**（规范 5.2.12）：支持时，索引 0 = 头部，1 = 第一条。
- **条目固定 1,024 字节 + 扩展属性**（规范 5.2.12.1.18 / 5.2.12.1.19）：
  - 固定 1,024 字节：传输/地址族、子系统类型、安全要求、Port ID、Controller ID、最大 Admin SQ 大小、标志、服务 ID、子系统 NQN、传输地址、传输特定地址
  - 扩展属性：`TEL` + `NUMEXAT` 框架的可变属性列表
- **Host Identifier 属性规则**（规范 5.2.12.1.19）：
  - 主机条目**必含**，16 字节
  - 子系统条目**禁止**
- **Admin label 规则**（规范 5.2.12.1.19）：
  - ASCII 或 UTF-8，可选
  - 长度 4–256 字节
- **路由字段**（规范 5.2.12.1.19）：主机条目中子系统类型 / Port ID / Controller ID / Admin SQ 大小**清零忽略**；共同字段（transport、NQN、地址、总长、属性列表）两种条目都用。
- **任务基数**（规范 5.2.13.x）：
  - Register = 1..N
  - De-register = 1..N
  - Update = 恰好 2（首条识别旧键，第二条原子替换）
- **错误码**（规范 5.2.13.x）：
  - 容量不足 → Register 整体失败
  - 字段冲突 → `Invalid Discovery Information`
  - 保留协议值 → `Invalid Parameter`
- **DDC 提交**（规范 5.2.13.x）：DDC 提交 = NVM 子系统基本或扩展条目；**DDC 单独**可把条目标记为 port-local。
- **头部 NQN**（规范 5.2.13.x）：UUID 形式的 NQN 标识提交方。
- **键选择**（规范 5.2.13.x）：键可基于传输地址（主机记录必须）或 Port ID（子系统记录允许）。
- **Extended Discovery Log Page Entry**（规范 5.2.12.1.19）：1024 字节前缀 + `TEL` + `NUMEXAT` 框架的可变属性列表。
- **TRADDR 首字节为 0**（规范 5.2.12.1.19）：选连接的远端 IP；该条目注册/注销每条命令限 1 条。
- **AVE Discovery 条目**：AVE NQN + 0 或多个 20 字节 IPv4/IPv6 TCP 传输记录。
- **Pull Model DDC Request**（规范 5.2.13.x）：`ORI` + 总长 + GAZ/AAZ/RAZ/Discovery Log Page Request。
- **Host Discovery**（规范 5.2.12.x）：
  - 可返回请求者子集
  - `ALLHOSTES` 声明 + `ALLHOSTE` 请求 = 全量
  - `ALLHOST` 标志表示结果范围
  - 必含至少一个 Host Identifier 扩展属性
  - `NCC` 报告主机是否已连 CDC；非 CDC 该位为 0 忽略
- **持久发现连接**（规范 3.1.3.3 / 5.1.13）：
  - 非零 Keep Alive Timer 表示请求
  - 必支持 Keep Alive、AER、Discovery Log 变更通知
  - Discovery 控制器**永不支持** Disconnect
- **控制器 ID 哨兵**（规范 6.3 / 5.2.6）：
  - `FFF0h–FFFCh` = 保留；Connect 非法
  - `FFFDh` = 跨子系统分散命名空间；Connect 非法
  - `FFFEh` = 任意可用静态
  - `FFFFh` = 任意可用动态
- **静态分配的 ID 复用**（规范 6.3）：`FFFEh` 拿到后主机应记住真实 Controller ID 用于重连。
- **变更通知**（规范 5.1.13）：
  - `Notice (2h)` + `LID 70h` + `information F0h` = Discovery 变更
  - `Notice (2h)` + `LID 71h` + `information F1h` = Host Discovery 变更
  - 只发给**请求过对应异步事件类型**的实体
- **SDLP 限制**（规范 5.2.13.x）：
  - 仅 `70h` / `71h` / `72h` 允许
  - 其他 LID 不带数据 + 报 `Not Allowed`
- **LPUR 一次性**（规范 5.2.13.x）：Pull-model DDC 在完成里置 1 = 请求重发；注册**仅持续到下次 SDLP**。
- **关联 vs 多主机**（规范 4.2.1.1）：一个控制器同一时刻只与一个主机关联；多主机经同一端口到达不同控制器 ≠ 多主机共享关联。

## 规范依据

- [Discovery Service 行为，PDF 第 45 页](../_source/pages/page-045.md)
- [认证门控，PDF 第 46 页](../_source/pages/page-046.md)
- [Discovery 控制器模型与 NQN 规则，PDF 第 58、62 页](../_source/pages/page-058.md)
- [Discovery 条目、持久连接、路径选择，PDF 第 63 页](../_source/pages/page-063.md)
- [Discovery 日志能力与过滤协商，PDF 第 306-308 页](../_source/pages/page-306.md)

## 相关阅读

- [transport-models.md](transport-models.md) - 属于消息类 Fabrics 传输
- [association-and-command-lifecycle.md](association-and-command-lifecycle.md) - 认证通过后才能建立关联
- [fabric-zoning-model.md](fabric-zoning-model.md) - CDC 集中化访问控制机制
- [command-sets.md](command-sets.md) - Fabrics 命令集负责建联
