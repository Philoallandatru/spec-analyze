# 命名空间写保护（Namespace Write Protection）

**命名空间写保护（Feature ID `84h`）** 允许主机将单个命名空间设置为不同严格程度的只读状态，防止数据被意外或恶意修改。该特性独立于常规的 Feature 保存（Saveability）机制，并在子系统内跨控制器强制执行。[PDF pp. 425-426, 553-556](../_source/pages/page-425.md)

> **类比**：把四种状态想成文件的四种封存方式——**无保护**是桌面敞开的文件；**写保护**是加了密码锁的文件夹（可随时解锁）；**直到掉电前保护**是“必须整栋楼断电才能打开的临时保险柜”；**永久保护**则是“刻在石板上”（永远无法改回可写）。

## 状态转换与持久化特性

```text
                    Set Feature (写保护)
 [无写保护 (0)] <──────────────────────────> [写保护 (1)]
     ▲    │
     │    ├── Set Feature (直到掉电前) ──→ [直到掉电前保护 (2)] ── 仅 Power Cycle 恢复
     │    │
     │    └── Set Feature (永久) ────────→ [永久写保护 (3)]     (不可逆终态)
     │
     └─────────────── 仅系统掉电重启 (Power Cycle) ───────────────┘
```

| 保护级别 | Feature 值 | 可否用 Set Feature 解除 | 跨掉电保持 | 跨控制器重置保持 | 多域（Multi-Domain）支持 |
|---|---|---|---|---|---|
| **无写保护（No Write Protect）** | `0` | —（初始状态） | 是 | 是 | 允许 |
| **写保护（Write Protect）** | `1` | **是**（可恢复为 `0`） | 是 | 是 | 允许 |
| **直到掉电前写保护（Until Power Cycle）** | `2` | **否**（返回 `Feature Not Changeable`） | **否**（掉电后恢复 `0`） | 是 | **禁止** |
| **永久写保护（Permanent Write Protect）** | `3` | **否**（返回 `Feature Not Changeable`） | 是 | 是 | 允许 |

## 核心约束与数据一致性屏障

1. **落盘屏障（Flush Barrier）**：在进入“直到掉电前写保护”或“永久写保护”之前，控制器**必须将该命名空间所有易失性写入数据和元数据刷到非易失性介质**，确保保护生效后不会有悬空数据丢失。[PDF p. 426](../_source/pages/page-426.md)
2. **全子系统强制执行**：只要子系统中**任意一个控制器**支持写保护能力，连接到该命名空间的**每个控制器**都必须强制执行其保护状态。[PDF pp. 553-555](../_source/pages/page-553.md)
3. **RPMB 联动与多域禁用**：进入“直到掉电前”和“永久”状态可能受 **RPMB（Replay Protected Memory Block）** 写保护控制限制；多域子系统中禁止使用“直到掉电前”模式。[PDF pp. 426, 554](../_source/pages/page-426.md)
4. **与 SMART 隔离**：通过 Feature `84h` 设置的写保护**不会**触发 SMART 中的 “All Media Read-Only” 警告。[PDF pp. 553-555](../_source/pages/page-553.md)

## 写保护状态下的命令执行边界

| 命令类别 | 典型命令 | 写保护下的行为 |
|---|---|---|
| **只读与校验** | Read, Compare, Verify, Device Self-test | **允许执行**。[PDF p. 555](../_source/pages/page-555.md) |
| **信息查询与管理** | Identify, Get Log Page, Attach/Detach, Reservation | **允许执行**（不修改介质用户数据）。[PDF p. 555](../_source/pages/page-555.md) |
| **缓存刷新** | Flush | **成功执行但无实际效果**（进入保护前已完成落盘屏障）。[PDF p. 555](../_source/pages/page-555.md) |
| **条件性命令** | Directive Receive | 名义上允许，但**不能**分配 Streams 资源等会修改受保护命名空间的操作。[PDF p. 555](../_source/pages/page-555.md) |
| **直接或间接写/擦除** | Write, Write Zeroes, Format NVM, Sanitize | **被拒绝**，返回 `Namespace Is Write Protected`（即使命令不带 NSID 如 Sanitize，只要会修改受保护命名空间即被拒）。[PDF pp. 555-556](../_source/pages/page-555.md) |

## 关联与引申概念

- **RPMB（Replay Protected Memory Block，重放保护内存块）**：基于 HMAC-SHA256 密钥认证的安全存储区域，常用于存放防篡改策略或固件状态，可联动控制写保护权限。
- **Feature Saveability（特性保存性）**：NVMe 中通过 `Set Features` 的 `SV` 位将配置跨重启保存的机制；写保护 Feature 本身标记为 Non-Saveable，其持久性由保护级别自身语义决定。
- **相关文档**：
  - [namespace.md](namespace.md) — 命名空间基础模型
  - [common-controller-features.md](common-controller-features.md) — Set/Get Features 控制平面
  - [format-nvm-lifecycle.md](format-nvm-lifecycle.md) 与 [sanitize-operation-lifecycle.md](sanitize-operation-lifecycle.md) — 会被写保护阻止的格式化与擦除操作

## 规范依据

- [Feature 状态与变更限制，PDF p. 425](../_source/pages/page-425.md)
- [Multi-Domain 限制与缓存提交屏障，PDF p. 426](../_source/pages/page-426.md)
- [状态定义与 Figure 622 转换图，PDF pp. 553-554](../_source/pages/page-553.md)
- [子系统范围强制执行与命令交互，PDF pp. 555-556](../_source/pages/page-555.md)
