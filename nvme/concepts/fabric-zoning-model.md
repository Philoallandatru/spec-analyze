# Fabric 分区模型（Fabric Zoning Model）

Fabric Zoning 是 NVMe over Fabrics（NVMe-oF）下由 **CDC（Centralized Discovery Controller，集中式发现控制器）** 维护的集中式访问控制模型。它通过过滤 Discovery Log（发现日志）的可见性、发送变更通知以及可选的网络层逐包过滤，精确控制哪些主机（Host）能够发现并访问哪些 NVM 子系统端口。[PDF 第 632, 637 页](../_source/pages/page-637.md)

## 心智模型：写字楼门禁中心

- **Zoning Database = 门禁数据库**，分为两本账：
  - **`ZoneDBConfig`（草稿/候选库）**：存放已配置但尚未激活的 `ZoneGroup` 和 `ZoneAlias`。
  - **`ZoneDBActive`（生效库）**：存放当前正在强制执行的 `ZoneGroup`——**只有进入 `ZoneDBActive` 的规则才会真正生效**。
- **`Zone`（分区）**：最小访问控制单元，包含成员列表（`Zone Member`）和属性（`Zone Property`）。
- **`ZoneGroup`（分区组）**：由某个 Originator 创建的一组 `Zone`。

```text
CDC Zoning Database
 ├── ZoneDBConfig（候选配置）
 │    ├── ZoneGroup 1..m
 │    └── ZoneAlias 1..p
 └── ZoneDBActive（当前生效）
      └── ZoneGroup 1..n
           └── Zone 1..k
                ├── Zone Member 1..x   (固定 256 字节/个)
                └── Zone Property 1..y (可变长 TLV)
```

## 核心对象与结构规则

### 1. ZoneGroup 与 Zone 布局

- **标识与代次**：CDC 为每个 `ZoneGroup` 维护一个不复用的 **Compact Key**（用于 FZR/FZS 传输）和一个 **Generation Number**（范围 `1h`–`FFFFFFFFh`，递增回绕时跳过 `0h`）。[PDF 第 639 页](../_source/pages/page-639.md)
- **固定前缀 + 可变尾部**：`ZoneGroup` 固定前缀 256 字节（Originator NQN 224B + Name 30B + Zone Count 2B）；每个 `Zone` 固定前缀 128 字节（Name 124B + Member Count 2B + Property Count 2B），后接固定 256 字节的 `Zone Member` 数组和可变长的 `Zone Property` 列表。[PDF 第 638-639 页](../_source/pages/page-638.md)

### 2. Zone Member、通信矩阵与 ZoneAlias

- **通信矩阵**：同一 `Zone` 内**仅允许 `Host ↔ NVM Subsystem` 通信**；禁止 `Host ↔ Host` 或 `Subsystem ↔ Subsystem`。[PDF 第 640 页](../_source/pages/page-640.md)
- **成员类型（TLV 格式，256 字节）**：
  - `1h`（NQN + Role）：仅按 NQN 与角色匹配，**不得**再用成员类型未包含的维度（如 IP 地址）收窄访问控制。
  - `EFh`（ZoneAlias Name + Role）：在 `ZoneDBConfig` 中引用预定义的 `ZoneAlias`，激活进入 `ZoneDBActive` 时展开为具体实体。**`ZoneAlias` 禁止递归嵌套**（不能包含另一个 `ZoneAlias`）。[PDF 第 640, 645-646 页](../_source/pages/page-640.md)
- **Zone Property**：`1h`（Fabric Enforced Zone）在 `ZPVAL=1h` 时请求网络层逐包强制执行分区规则；未知属性类型可忽略。[PDF 第 645-646 页](../_source/pages/page-645.md)

## 操作模型：Push vs Pull 与三类操作

Zoning 操作围绕三个原子动作展开：**GAZ**（Get Active ZoneGroup，获取）、**AAZ**（Activate Active ZoneGroup，激活/原子替换）和 **RAZ**（Remove Active ZoneGroup，移除）。同一 Domain 内所有控制器必须披露相同的支持模式。[PDF 第 647-656 页](../_source/pages/page-647.md)

| 对比维度 | Push Model（推送模型） | Pull Model（拉取模型） |
|---|---|---|
| **发起与通道** | DDC 主动通过 `FZL` / `FZR` / `FZS` 命令推送或拉取 | DDC 通过 AEN/Request Log 提交请求，CDC 异步拉取或下发 |
| **上下文凭证** | 使用 CDC 返回的 **Compact Key** | 使用 DDC 选定的 **Transaction ID** |
| **Originator 鉴权** | 验证 DDC→CDC 连接的 **`HOSTNQN`** | 验证 CDC→DDC 连接的 **`SUBNQN`** |
| **AAZ 超时与完整性** | 建锁后若 **30 秒**内未收齐完整定义，CDC 丢弃片段并释放锁 | 完整定义到达前不得更新 `ZoneDBActive` |
| **状态/错误反馈** | 命令状态码（如 `Requested Function Disabled`、`ZoneGroup Originator Invalid`） | Pull Operation Status（`0h` 成功、`1h` 处理中、`2h` 不存在、`3h` 已锁定、`4h` Originator 无效、`5h` 传输期间改变） |

- **权限边界**：CDC 管理接口可访问所有 `ZoneGroup`；DDC 只能管理以自身 NQN 为 Originator 的 `ZoneGroup`。
- **变更通知扇出**：当 Zone 成员关系变化时，CDC 仅向**已明确请求该类事件**的同 Zone 成员（Host 或 Subsystem）发送 Discovery Log Page Change AEN。[PDF 第 657 页](../_source/pages/page-657.md)

## 关联与引申概念

- **Fabric Zoning 数据传输**：定义 `FZL` / `FZR` / `FZS` 如何分片搬运本模型的数据结构，见 [Fabric Zoning 数据传输](fabric-zoning-data-transfer.md)。
- **Fabrics 发现与认证**：CDC 如何过滤 Discovery Log 并下发变更通知，见 [Fabrics 发现与认证](fabrics-discovery-and-authentication.md)。
- **Allowed Host List**：各导出子系统本地的访问控制列表，与 CDC 集中式的 Fabric Zoning 相互独立，见 [导出资源与底层资源](exported-and-underlying-resources.md)。[PDF 第 31 页](../_source/pages/page-031.md)
- **传输模型 & 关联生命周期**：NVMe-oF 消息传输与建链过程，见 [传输模型](transport-models.md) 与 [关联与命令生命周期](association-and-command-lifecycle.md)。

## 规范依据

- [Centralized Discovery 与 Zoning 过滤目的，PDF 第 632 页](../_source/pages/page-632.md)
- [Zoning DB 抽象、ZoneGroup 与 Zone 布局，PDF 第 637-639 页](../_source/pages/page-637.md)
- [Zone Member 通信矩阵与类型格式，PDF 第 640-644 页](../_source/pages/page-640.md)
- [ZoneAlias 解析与 Zone Property，PDF 第 645-646 页](../_source/pages/page-645.md)
- [Push 与 Pull 模型 GAZ/AAZ/RAZ 流程，PDF 第 647-654 页](../_source/pages/page-647.md)
- [Pull 模型状态码（Figure 715）与变更通知，PDF 第 655-657 页](../_source/pages/page-655.md)
