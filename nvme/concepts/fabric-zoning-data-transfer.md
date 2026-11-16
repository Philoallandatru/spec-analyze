# Fabric Zoning 数据传输（Fabric Zoning Data Transfer）

Fabric Zoning 数据传输提供三组 Admin 命令——**Lookup（`FZL`）**、**Receive（`FZR`）** 和 **Send（`FZS`）**——让直连发现控制器（DDC）与集中式发现控制器（CDC）之间能够定位并按 4 字节（dword）片段来回搬运 Zoning 数据结构。[PDF 第 457-460 页](../_source/pages/page-457.md)

## 心智模型：凭提货单分片收发货物

- **FZL（Lookup，查表拿钥匙）**：DDC 把操作查找数据发给 CDC，CDC 查表后在 CQE Dword 0 返回一个 **Zoning Data Key（`ZDK`）**（相当于提货单号）。
- **FZR（Receive，CDC → DDC）**：接收方按偏移（offset）和长度（`NUMD`）分片拉取数据，CDC 在最后一个片段的 **CQE Dword 0** 中置位 **`LF = 1`**（Last Fragment）。
- **FZS（Send，DDC → CDC）**：发送方按偏移和长度分片推送数据，由发送方自己在最后一个片段的 **命令 Dword 12（CDW12）** 中置位 **`LF = 1`**。

```text
DDC 操作数据 ──FZL──> CDC Zoning 数据库
                            |
                            v
                  Zoning Data Key (ZDK)
                            |
        +-------------------+-------------------+
        |                                       |
        v                                       v
 FZR (CDC -> DDC)                        FZS (DDC -> CDC)
 offset + NUMD; CQE.Dword0.LF            offset + NUMD + CDW12.LF
        |                                       |
        +-----> [片段循环至 LF = 1 结束] <------+
```

## 上下文选择与核心规则

| 规则维度 | 规范要求 |
|---|---|
| **`ZDKC` 位（上下文区分）** | 区分使用 `ZDK`（业务键）还是 `Transaction ID`（Pull 模式会话 ID）。**CDC 只接受 `ZDK` 上下文，DDC 只接受 `Transaction ID` 上下文**；用反会报 `Invalid Field in Command` |
| **`NUMD`（dword 计数）** | **必须非零**（为 `0` 直接拒绝） |
| **偏移（Offset）对齐** | 偏移必须 dword 对齐；若低位非零，控制器可拒绝（不拒绝则按 0 处理）。**FZR 若偏移超过目标结构总大小，必须拒绝** |
| **`LF` 标志位置** | **FZR 的 `LF` 在 CQE Dword 0**（由响应方告知结束）；**FZS 的 `LF` 在命令 Dword 12**（由发送方告知结束） |
| **共享错误场景** | 当结构被锁定、结构缺失、Fabric Zoning 被禁用或 DDC 无权访问该 ZoneGroup 时，FZL/FZR/FZS 会返回对应的命令特定错误 |

## 关联与引申概念

- **Fabric Zoning 模型**：定义了 `ZoneDBConfig`、`ZoneDBActive`、`ZoneGroup` 等被本命令搬运的数据结构语义及 Push/Pull 模式，见 [Fabric 分区模型](fabric-zoning-model.md)。
- **Fabrics 发现与认证**：主机与 DDC/CDC 建立连接及发现服务的流程，见 [Fabrics 发现与认证](fabrics-discovery-and-authentication.md)。
- **Allowed Host List**：子系统本地的访问控制清单，与网络级的 Fabric Zoning 属于不同层级，见 [导出资源与底层资源](exported-and-underlying-resources.md)。[PDF 第 31 页](../_source/pages/page-031.md)
- **命令集 & 生命周期**：Admin 命令集的提交与完成机制，见 [命令集](command-sets.md) 与 [关联与命令生命周期](association-and-command-lifecycle.md)。

## 规范依据

- [Lookup 与返回 Zoning Data Key，PDF 第 457 页](../_source/pages/page-457.md)
- [Receive 字段与完成状态，PDF 第 458 页](../_source/pages/page-458.md)
- [Receive 终止与 Send 字段，PDF 第 459 页](../_source/pages/page-459.md)
- [Send 完成状态，PDF 第 460 页](../_source/pages/page-460.md)
- [Fabric Zoning 与 Allowed Host List 的边界，PDF 第 31 页](../_source/pages/page-031.md)
