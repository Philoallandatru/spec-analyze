# Fabric 分区模型（Fabric Zoning Model）

## 一句话说明

Fabric Zoning 是 NVMe-oF（NVMe over Fabrics）下由 CDC（Centralized Discovery Controller，中央化发现控制器）维护的**集中式访问控制模型**：通过在 CDC 的 Zoning Database 中定义"哪些主机和哪些子系统能互相通信"，对 Discovery Log 的可见性与（可选的）网络层流量进行统一管控。

## 生活化类比

把 CDC 想象成**一栋写字楼的门禁中心**：

- **门禁系统数据库**（Zoning Database）记录哪家公司（Host）能上几楼（Subsystem）
- **Zone** = 门禁卡里的一条授权规则："B 公司的员工可以进入 5 楼机房"
- **ZoneGroup** = 门禁系统的某一组配置（来自某物业系统 Originator）
- **ZoneDBConfig vs ZoneDBActive** = "草稿配置"vs"已生效配置"——只生效的才会被门禁执行
- **Push Model** = 物业主动把规则推给门禁（使用统一的 ZoneGroup Key）
- **Pull Model** = 门禁按需向物业拉取（使用门禁自选的事务 ID）

> 关键：分区是 CDC 集中下发的；分区对象是 NVM 子系统端口（Subsystem Port），不是控制器。

## 工作流程

```text
+----------------------------- CDC -----------------------------+
|  Zoning Database                                              |
|  +------------------------+   +---------------------------+   |
|  | ZoneDBConfig (候选)    |   | ZoneDBActive (已生效)     |   |
|  |  - ZoneGroup 1..m      |   |   - ZoneGroup 1..n       |   |
|  |  - ZoneAlias 1..p      |   |     - Zone 1..k          |   |
|  +------------------------+   |       - Zone Member 1..x  |   |
|                                |       - Zone Property 1..y|   |
|                                +---------------------------+   |
+----------------------------------------------------------------+
         ^                          ^
         | Push (FZR/FZS, Key)      | Pull (AEN/RequestLog, TID)
         |                          |
+---------+----------+    +---------+----------+
|  Originator DDC    |    |     Pull DDC        |
| (Push 模式)        |    |  (Pull 模式)        |
+--------------------+    +--------------------+
```

**核心操作（CRUD）**：

| 操作 | 含义 | 方向 |
|------|------|------|
| GAZ | Get Active ZoneGroup | CDC → DDC |
| AAZ | Activate Active ZoneGroup（原子替换）| DDC → CDC |
| RAZ | Remove Active ZoneGroup | DDC → CDC |

## 初学者案例

**场景：多租户云，需要把租户 A 和租户 B 的流量隔离开。**

1. 管理员在 CDC 上创建两个 ZoneGroup：tenant-A、tenant-B。
2. tenant-A 内放：
   - Zone Member：Host NQN `nqn.tenantA.client` + Subsystem Port 列表
   - Zone Property：`ZPVAL=1h` 启用 Fabric Enforced（网络层逐包过滤）
3. tenant-B 同理，包含 tenantA **不出现**的 Host NQN。
4. 激活两个 ZoneGroup（AAZ）。
5. 主机发起 Discovery Log 请求：CDC 过滤掉 tenant-A 不允许看到的 NVM 子系统端口。
6. 即便主机试图直接 Connect 那些端口，网络层（开启 Fabric Enforced 时）也会丢弃包。

> 关键收获：分区在 CDC 集中执行，主机无法绕过；ZoneDBConfig 是草稿，AAZ 才是真正生效的提交点。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 只有 Active 生效 | ZoneDBActive 中的 ZoneGroup 才被强制执行 |
| ZoneMember 通信矩阵 | 仅允许 Host ↔ NVM Subsystem；禁止 Host↔Host、Subsystem↔Subsystem |
| 通信矩阵不可收窄 | Type `1h` 仅含 NQN+Role，不能再按 IP 收窄访问控制 |
| ZoneAlias 不能递归 | ZoneAlias 可包含其他类型成员，但不可再嵌套 ZoneAlias |
| Compact Key 不复用 | CDC 为每个 ZoneGroup 维护唯一 Compact Key 与 Generation Number |
| Generation 范围 | `1h..FFFFFFFFh`，跳过 `0h`；到最大值回绕到 `1h` |
| Domain 内一致性 | 同一 Domain 内所有控制器必须披露相同支持、选择相同模式 |
| Push vs Pull 权限 | Originator 只能管理自己创建的 ZoneGroup；其他 DDC 无权 |
| Push 用 Key / Pull 用 TID | Push 操作携带 CDC 返回的 Compact Key；Pull 操作携带 DDC 选定的 Transaction ID |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| ZoneDBConfig vs ZoneDBActive | 前者只是配置候选；后者是真正被强制执行的 |
| Zone vs ZoneGroup | Zone 是访问控制单元（成员+属性）；ZoneGroup 是 Zone 的容器 |
| ZoneAlias vs Zone Member | ZoneAlias 是命名引用，激活时展开；Zone Member 是直接的实体标识 |
| Push Model vs Pull Model | Push 走 FZR/FZS + Key；Pull 走 AEN/Request Log + TID |
| Zone Property vs Zone Member | Property 控制行为（如 Fabric Enforced）；Member 标识参与者 |
| Fabric Zoning vs Allowed Host List | Zoning 在 CDC 集中控制；Allowed Host List 在各 Subsystem 本地控制 |

## 进阶细节

- **ZoneGroup 数据布局**（规范 Figures 682-683）：
  - ZoneGroup 固定前缀 256 字节（Originator NQN 224B + Name 30B + Zone Count 2B）
  - Zone 固定前缀 128 字节（Name 124B + Member Count 2B + Property Count 2B）
  - Zone Members 固定 256 字节/个；Zone Properties 可变长
- **Zone Member 类型**（规范 7.10.3）：
  - `1h` = NQN + Role
  - `EFh` = ZoneAlias Name + Role（激活时展开）
  - 其他类型定义于规范 Figure 686
- **Zone Property**（规范 7.10.4）：TLV 格式；`1h` = Fabric Enforced（`ZPVAL=1h` 启用网络层逐包过滤）
- **AAZ 原子性**（规范 7.10.5）：Push 模式下若 30 秒内未收齐数据，CDC 丢弃片段、释放锁，`ZoneDBActive` 不变
- **Generation 校验**（规范 7.10.6）：GAZ Push 模式下，传输期间必须保持 Generation 不变；DDC 收齐前不得处理
- **Pull Model GAZ 完整性**（规范 7.10.7）：若传输期间 ZoneGroup 改变，以零长度片段和 `ZoneGroup Changed` 状态中止
- **权限验证**（规范 7.10.8）：Push Model 验证 `HOSTNQN`；Pull Model 验证 `SUBNQN`
- **Pull Model 状态码**（规范 Figure 715）：
  - `0h` 成功
  - `1h` 处理中
  - `2h` Zoning 数据结构不存在
  - `3h` 被锁定
  - `4h` ZoneGroup Originator 无效
  - `5h` 传输期间改变
- **变更通知**（规范 7.10.9）：仅发送给"已明确请求该类事件"的实体
- **ZoneDBConfig 不存在的别名**：如果 Zone Member 引用了不存在的 ZoneAlias，AAZ 时 CDC 应报告错误
- **Discovery Log 过滤**（规范 7.10.10）：CDC 过滤后的 Discovery Log 只包含主机在当前 ZoneGroup 中被授权看到的 NVM Subsystem Port

## 规范依据

- [Centralized Discovery 与 Zoning 过滤目的，PDF 第 632 页](../_source/pages/page-632.md)
- [ZoneDBActive/Config 抽象与 Kickstart，PDF 第 637 页](../_source/pages/page-637.md)
- [ZoneGroup Identity/Generation 与 Zone 布局，PDF 第 639 页](../_source/pages/page-639.md)
- [Zone Member 类型与 NQN-only 格式，PDF 第 640 页](../_source/pages/page-640.md)
- [ZoneAlias 解析与 Property 类型，PDF 第 645 页](../_source/pages/page-645.md)
- [Push-model 与 Pull-model GAZ/AAZ/RAZ，PDF 第 647 页](../_source/pages/page-647.md)
- [Pull-model 操作状态码与变更通知，PDF 第 656 页](../_source/pages/page-656.md)

## 相关阅读

- [fabrics-discovery-and-authentication.md](fabrics-discovery-and-authentication.md) - 分区依赖发现服务流程
- [fabric-zoning-data-transfer.md](fabric-zoning-data-transfer.md) - 分区数据操作的三组命令
- [association-and-command-lifecycle.md](association-and-command-lifecycle.md) - 控制器与主机关联建链
- [transport-models.md](transport-models.md) - 基于 Fabrics 消息传输
