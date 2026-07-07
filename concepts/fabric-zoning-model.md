# Fabric 分区模型（Fabric Zoning Model）

## 概述

Fabric Zoning（Fabric 分区）是由 CDC（Centralized Discovery Controller，中央化发现控制器）维护的集中式访问控制模型。它的主要作用是控制主机（Host）能够发现哪些 NVM 子系统端口（NVM Subsystem Ports）。

**工作机制：**
- 过滤 Discovery Log（发现日志）的内容
- 发送相应的变更通知（Change Notification）
- 可选择性地在网络层施加访问限制

[PDF pp. 632, 637](../_source/pages/page-637.md)

---

## 核心概念与数据模型

### 整体架构

CDC 的分区数据库（Zoning Database）包含两个主要部分：

```text
CDC Zoning DB（CDC 分区数据库）
├── ZoneDBConfig（配置候选库）
│   ├── ZoneGroup 1..m（分区组 1 到 m）
│   └── ZoneAlias 1..p（分区别名 1 到 p）
└── ZoneDBActive（当前生效库）
    └── ZoneGroup 1..n（分区组 1 到 n）
        └── Zone 1..k（分区 1 到 k，访问控制单元）
            ├── Zone Member 1..x（分区成员 1 到 x）
            └── Zone Property 1..y（分区属性 1 到 y）
```

**重要区分：**
- **ZoneDBConfig**：保存配置好但尚未激活的分区组和分区别名
- **ZoneDBActive**：保存当前正在强制执行的分区组
- 只有位于 `ZoneDBActive` 中的 ZoneGroups 才会实际生效

该架构基于规范中的 Figures 679-683 绘制。[PDF pp. 637-639](../_source/pages/page-637.md)

---

## 关键对象与约束规则

### 对象职责

| 对象 | 作用 | 核心约束 |
|------|------|----------|
| **ZoneDBConfig** | 保存已配置的分区组与分区别名 | 配置不等于生效，需要激活才能进入 ZoneDBActive |
| **Zone** | 访问控制的基本单元 | 同一 Zone 内的成员按规则可以互相通信 |
| **ZoneGroup** | Zone 的集合，由 Originator 创建 | CDC 为每个 ZoneGroup 维护唯一的紧凑密钥（Compact Key） |

### ZoneGroup 管理机制

CDC 为每个 ZoneGroup 维护两个关键标识：

1. **Compact Key（紧凑密钥）**：用于 FZR/FZS 命令，不应复用
2. **Generation Number（代次号）**：每次更新时递增
   - 范围：`1h` 到 `FFFFFFFFh`
   - 从 `FFFFFFFFh` 递增时回绕到 `1h`（跳过 `0h`）

[PDF p. 639](../_source/pages/page-639.md)

### Zone Member（分区成员）

**基本特性：**
- 固定大小：256 字节
- 采用 TLV（Type-Length-Value）格式
- `Role` 字段只能表示 Host 或 NVM Subsystem

**通信规则：**

| 允许的通信 | 禁止的通信 |
|------------|------------|
| Host ↔ NVM Subsystem | Host ↔ Host |
|  | NVM Subsystem ↔ NVM Subsystem |

**成员类型与匹配维度：**

| 类型值 | 参与匹配的身份维度 | 典型粒度 | 说明 |
|--------|-------------------|----------|------|
| `1h` | NQN + Role | 实体的全部接口、协议与端口 | 仅使用 NQN 标识，不能再按 IP 地址收窄 |
| `EFh` | ZoneAlias Name + Role | 激活时展开为实体集合 | 引用预定义的分区别名 |

**重要限制：**  
成员类型未包含的维度不得用于该 Zone 的分区强制执行。例如，Type `1h` 只包含 NQN 和 Role，因此不能再按 IP 地址进行访问控制。

[PDF p. 640](../_source/pages/page-640.md)

---

## 数据结构布局

### ZoneGroup 结构

```text
ZoneGroup 固定前缀（256 字节）
├── Originator NQN: 224 字节
├── Name: 30 字节
├── Zone Count: 2 字节
└── 可变长度 Zone 列表（Zone 1 到 Zone n）

Zone 固定前缀（128 字节）
├── Name: 124 字节
├── Member Count: 2 字节
├── Property Count: 2 字节
├── 固定大小 Zone Members（每个 256 字节）
└── 可变大小 Zone Properties
```

**字段说明：**

| 字段 | 大小 | 说明 |
|------|------|------|
| **Originator NQN** | 224 字节 | 创建该 ZoneGroup 的 CDC 或 DDC 的 NQN |
| **ZoneGroup Name** | 30 字节 | Null-terminated ASCII 字符串 |
| **Zone Name** | 124 字节 | Zone 的名称 |
| **Zone Members** | 每个 256 字节 | 从 byte 128 开始，固定大小 |
| **Zone Properties** | 可变 | 紧随全部成员，按各自长度串接 |

该布局基于 Figures 682-683 整理，用于显示固定前缀与可变尾部的分界。精确的偏移量与长度公式以规范原文为准。[PDF pp. 638-639](../_source/pages/page-638.md)

### ZoneAlias 解析机制

```text
配置阶段（ZoneDBConfig）
    Zone Member (ZMTYPE=EFh) ──引用──> ZoneAlias Name
                                           │
                                           │ 激活 ZoneGroup
                                           │
                                           ↓
生效阶段（ZoneDBActive）
    Active Zone Member <────解析为──── 具体的 NVMe 实体
```

**解析规则：**
1. 在 `ZoneDBConfig` 中，Zone Member 可以引用 ZoneAlias
2. 当 ZoneGroup 激活并进入 `ZoneDBActive` 时，ZoneAlias 成员会解析为其引用的具体 NVMe 实体
3. ZoneAlias 自身可包含任意非别名成员类型（参考 Figure 684）
4. **禁止递归嵌套**：ZoneAlias 不能包含其他 ZoneAlias

[PDF pp. 640, 645-646](../_source/pages/page-640.md)

### Zone Properties（分区属性）

Zone Properties 也采用 TLV 格式：

| 属性类型 | 值 | 含义 |
|----------|---|------|
| `1h` (Fabric Enforced Zone) | `ZPVAL=1h` | 请求在网络层逐包强制执行分区规则 |
|  | 其他值 | 被忽略 |
| 未知类型 | - | 可以忽略 |

[PDF pp. 645-646](../_source/pages/page-645.md)

---

## 操作模型

### 访问控制与权限

**ZoneGroup 的 CRUD 操作：**
- 位于 CDC 的 Zoning Database 中
- 管理接口访问期间可以锁定 ZoneGroup，阻止其他接口并发访问

**权限规则：**

| 访问者 | 权限 |
|--------|------|
| CDC 的管理接口 | 可访问所有 ZoneGroup |
| Originator DDC | 只能访问自己创建的 ZoneGroup |
| Push-model DDC | 只能管理以自身 NQN 为 Originator 的 ZoneGroup |
| Pull-model DDC | 只能管理以自身 NQN 为 Originator 的 ZoneGroup |

**禁用状态：**  
当 CDC 未启用 Fabric Zoning 时：
- Push-model Fabric Zoning 命令返回 `Requested Function Disabled` 状态
- Pull-model DDC 的相关异步请求被忽略

[PDF pp. 646-647](../_source/pages/page-646.md)

### 操作流程对比：Push vs Pull

#### Push Model（推送模型）

在 Push Model 中，DDC 主动发起操作：

```text
GAZ（Get Active ZoneGroup）
  DDC ──FZL lookup {originator, name}──> CDC
  CDC ──返回 Compact ZoneGroup Key────> DDC
  DDC ──FZR(key) 分片请求──────────> CDC
  CDC ──返回 ZoneGroup 数据片段────> DDC

AAZ（Activate Active ZoneGroup）
  DDC ──FZL lookup──> CDC ──返回 key──> DDC
  DDC ──FZS(key, data) 分片发送──────> CDC

RAZ（Remove Active ZoneGroup）
  DDC ──FZL(identity)──────────────> CDC
```

#### Pull Model（拉取模型）

在 Pull Model 中，DDC 通过异步请求机制协作：

```text
GAZ（Get Active ZoneGroup）
  DDC ──上报 AEN/Log Request──> CDC
  CDC ──FZS(T_ID, data) 分片发送──> DDC

AAZ（Activate Active ZoneGroup）
  DDC ──上报 AEN/Log Request──> CDC
  CDC ──FZR(T_ID) 拉取数据──> DDC
  DDC ──返回数据───────────> CDC
  CDC ──FZS(status) 返回结果──> DDC

RAZ（Remove Active ZoneGroup）
  DDC ──上报 AEN/Log Request──> CDC
  CDC ──FZS(T_ID, status) 返回结果──> DDC
```

**关键区别：**
- **Push Model**：使用 CDC 返回的 ZoneGroup Key
- **Pull Model**：使用 DDC 选定的 Transaction ID 作为后续 FZR/FZS 的 Zoning Data Key

该流程合并了规范中的 Figures 697、700、703、705、708 与 712。[PDF pp. 647-654](../_source/pages/page-647.md)

---

## 核心操作详解

### GAZ（Get Active ZoneGroup）：获取生效分区组

**操作方向：** CDC → DDC

#### Push Model 完整性保证

- 分片传输期间必须保持 Generation Number 不变
- DDC 在收齐所有片段前不得处理数据

#### Pull Model 完整性保证

- 若发送期间 ZoneGroup 发生改变，以零长度片段和 `ZoneGroup Changed` 状态中止
- DDC 在收齐所有片段前不得处理数据

**实现要点：**  
Push Model 通过 Generation Number 检查保证完整性；Pull Model 使用 Operation Status 显式报告进度状态（处理中、成功、未找到、已锁定、Originator 无效或已改变）。

[PDF pp. 648, 651-652](../_source/pages/page-648.md)

---

### AAZ（Activate Active ZoneGroup）：激活分区组

**操作方向：** DDC → CDC

**原子性保证：** AAZ 是带锁的原子替换操作

#### Push Model 流程

1. **Lookup 阶段**：若目标不存在，可先创建并锁定一个空 ZoneGroup
2. **传输阶段**：DDC 分片发送完整的 ZoneGroup 定义
3. **提交阶段**：
   - 只有完整数据成功到达，CDC 才更新并递增 Generation Number
   - 若从建锁起 30 秒内未收齐，CDC 丢弃片段、释放锁，`ZoneDBActive` 保持不变

#### Pull Model 流程

1. **锁定阶段**：CDC 先锁定目标 ZoneGroup
2. **拉取阶段**：CDC 使用 FZR 从 DDC 拉取完整定义
3. **提交阶段**：CDC 通过 FZS 返回结果
4. **完整性要求**：完整定义到达前不得更新

[PDF pp. 649-650, 653-654](../_source/pages/page-649.md)

---

### RAZ（Remove Active ZoneGroup）：移除生效分区组

**操作方向：** 删除 CDC 中的 Active ZoneGroup

#### 删除前提条件

| 条件 | 说明 |
|------|------|
| **存在性** | 目标 ZoneGroup 必须存在 |
| **锁定状态** | ZoneGroup 不能处于锁定状态 |
| **权限验证** | Originator 授权检查必须通过 |

**验证失败处理：**  
任一条件不满足时，保持原配置不变并报告对应状态。

#### Pull Model 具体流程

1. DDC 在 Request Log 中提交删除请求（携带 Transaction ID）
2. CDC 读取 Request Log 并验证条件
3. CDC 将 Transaction ID 复制到 FZS 的 Zoning Data Key
4. CDC 在单次 FZS 中返回删除结果

[PDF pp. 655-656](../_source/pages/page-655.md)

---

## 权限验证机制

访问检查绑定于连接方向：

| 操作模型 | 验证方法 | 失败报告方式 |
|----------|----------|--------------|
| **Push Model** | 使用 DDC→CDC Connect 的 `HOSTNQN` 验证 ZoneGroup Originator | Command Status 返回 `ZoneGroup Originator Invalid` |
| **Pull Model** | 使用 CDC→DDC Connect 的 `SUBNQN` 验证 ZoneGroup Originator | Pull-Operation Status 返回 `ZoneGroup Originator Invalid` |

[PDF pp. 648, 650-652, 654](../_source/pages/page-648.md)

---

## Pull Model 操作状态码

| 状态码 | 含义 | 说明 |
|--------|------|------|
| `0h` | 成功 | 操作已成功完成 |
| `1h` | 处理中 | 操作正在进行 |
| `2h` | Zoning 数据结构不存在 | 目标 ZoneGroup 或 Zone 未找到 |
| `3h` | Zoning 数据结构被锁定 | 其他操作正在访问该对象 |
| `4h` | ZoneGroup Originator 无效 | 权限验证失败 |
| `5h` | ZoneGroup 在传输期间改变 | GAZ 操作期间 ZoneGroup 被修改 |
| 其他 | 保留 | - |

该状态表基于规范 Figure 715 整理。[PDF p. 656](../_source/pages/page-656.md)

---

## 变更通知机制

当 Zone 成员关系发生变化时，CDC 会发送 Discovery Log Page Change AEN（异步事件通知）：

| 变更类型 | 通知对象 | 前提条件 |
|----------|----------|----------|
| 增加或移除 **Subsystem Member** | 同一 Zone 内的所有 Hosts | Host 已请求此类事件 |
| 增加或移除 **Host Member** | 同一 Zone 内的所有 Subsystems | Subsystem 已请求此类事件 |
| 增加或移除 **整个 Zone** | 该 Zone 的所有成员 | 成员已请求此类事件 |

**注意：** 通知只发送给已明确请求该类事件的实体。

[PDF p. 657](../_source/pages/page-657.md)

---

## 相关概念

### Fabric Zoning Data Transfer

[Fabric Zoning Data Transfer](fabric-zoning-data-transfer.md) 定义了 DDC 如何使用 Lookup Key 和 Fragments（片段）读写本文档描述的 Zoning 数据结构。

**职责分工：**
- **Fabric Zoning Data Transfer**：定义传输机制和命令格式
- **Fabric Zoning Model**（本文档）：定义数据结构的含义与生效边界

[PDF pp. 457-460, 637-639](../_source/pages/page-637.md)

### 与 Allowed Host List 的区别

Fabric Zoning 与 Exported NVM Subsystem 的 Allowed Host List 是两个独立的访问控制机制：

| 特性 | Fabric Zoning | Allowed Host List |
|------|---------------|-------------------|
| **控制位置** | CDC 集中控制 | 各 NVM Subsystem 本地控制 |
| **作用范围** | 过滤 Discovery Log | 控制 Subsystem 连接 |
| **粒度** | Zone 级别 | Subsystem 级别 |

[PDF pp. 31, 632](../_source/pages/page-632.md)

---

## 规范引用

- [Centralized Discovery 与 Zoning 过滤目的，PDF p. 632](../_source/pages/page-632.md)
- [Kickstart 时序与 Zoning DB 抽象，PDF p. 637](../_source/pages/page-637.md)
- [ZoneDBActive、ZoneDBConfig 与 ZoneGroup 前缀，PDF p. 638](../_source/pages/page-638.md)
- [ZoneGroup Identity/Generation 与 Zone 布局，PDF p. 639](../_source/pages/page-639.md)
- [Zone Member 类型、通信矩阵与 NQN-only 格式，PDF p. 640](../_source/pages/page-640.md)
- [IP/NQN/PortID 成员匹配格式，PDF pp. 641-644](../_source/pages/page-641.md)
- [ZoneAlias Member 与 Property 类型，PDF pp. 645-646](../_source/pages/page-645.md)
- [Zoning 操作门控与 GAZ 起始流程，PDF p. 647](../_source/pages/page-647.md)
- [Push-model GAZ、AAZ 与 RAZ，PDF pp. 647-650](../_source/pages/page-647.md)
- [Pull-model GAZ、AAZ 与 RAZ，PDF pp. 651-654](../_source/pages/page-651.md)
- [Pull-model RAZ 完成与统一状态值，PDF pp. 655-656](../_source/pages/page-655.md)
- [Zoning 变更通知扇出，PDF p. 657](../_source/pages/page-657.md)
