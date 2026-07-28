# 命名空间管理生命周期（Namespace Management Lifecycle）

## 一句话说明

命名空间管理把生命周期拆成**两个独立维度**：子系统级的分配/回收（Create/Delete）和控制器级的挂接管理（Attach/Detach）；Attach/Detach 状态会持久化跨所有 Reset 事件，Delete 会自动 Detach 所有控制器并回收资源。

## 生活化类比

把命名空间想成**酒店的可出租房型**：

- **Create** = 物业在系统中"上线一种房型"（比如"豪华大床房"），但还没决定分配给哪几家分店。
- **Attach** = 把这种房型**配置**到某家分店（控制器）的售卖系统中。
- **Detach** = 把房型从某家分店**下架**（其他分店仍可卖）。
- **Delete** = 物业直接**下线**这种房型——所有分店自动下架，资源回收。

关键点：Attach/Detach 是**持久化**的，即使分店断电重启（控制器级复位），也记得这家分店卖不卖这种房。

## 工作流程

```text
  [未分配的 NSID]
       |
       | Namespace Management: Create
       v
  [已分配但未挂接的命名空间]
       |         ^
  Attach|         | Detach
  (持久化)         | (持久化)
       v         |
  [在选定的 I/O 控制器上 active]
       |
       | Namespace Management: Delete
       v
  [从所有控制器解挂 + 资源回收]
```

简化说明：Create 不自动 Attach；Delete 自动 Detach 所有控制器并回收。Attach/Detach 状态穿越所有 Reset 事件保持；I/O 命令在管理操作期间继续执行（非阻塞设计）。

## 初学者案例

**场景：批量创建 10 个命名空间并挂到控制器 1**

1. 主机发 10 次 Namespace Management (Create) 命令：
   - 每条 NSID=`0`（让控制器选）
   - 指定 Command Set Identifier、容量、格式
   - 完成时 CQE 报告新分配的 NSID
2. 然后发 1 次 Namespace Attachment 命令：
   - Controller List 4 KiB 缓冲区，填入控制器 1 的 ID
   - 一次挂载 10 个命名空间（按 NSID 列表）
3. 验证：主机 `nvme list-ns /dev/nvme0` 应看到这 10 个 NSID。

排错思路：
- Create 返回 `Namespace Is Private` → 私有命名空间已挂到其他位置
- Attach 返回 `Namespace Attachment Limit Exceeded` → 超过 `MAXDNA`（Domain 总挂接数）或 `MAXCNA`（单控制器挂接数）
- Attach 返回 `Controller List Invalid` → 列表里包含管理控制器或格式错误

> 速记：**先 Create 再 Attach；Delete 会隐式 Detach**。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 两层管理 | 子系统级（Create/Delete）+ 控制器级（Attach/Detach） |
| Create 不自动 Attach | 创建后命名空间存在但不挂任何控制器 |
| Delete 自动 Detach | 删除时自动从所有控制器解挂，再回收资源 |
| 持久化 | Attach/Detach 状态穿越所有 Reset 事件（控制器级与子系统级） |
| 持久化抗虚拟化 | 把从控制器 Offline 也不影响挂接状态 |
| 控制器列表大小 | Attach/Detach 用 4 KiB Controller List 作命令数据 |
| 挂接目标 | 必须是支持且已启用该 I/O 命令集的 I/O 控制器 |
| 必选关系 | 支持 Namespace Management ⇒ 必支持 Namespace Attachment |
| 禁止位置 | 导出的 NVM 子系统中**禁止**支持 Namespace Management |
| 非阻塞 | 管理操作进行期间 I/O 命令继续执行 |
| Create NSID | 必须设为 `0`（由控制器选并返回） |
| Create 参数 | Command Set Identifier、512 字节参数区、可选 Vendor Data |
| Create 返回 | 完成时 CQE 报告新分配的 NSID |
| Delete NSID | 特定 NSID = 删一个；`FFFFFFFFh` = 删所有（即使不存在也成功） |
| Delete 事件 | 每次成功 Delete 必生成 allocated-namespace 变更报告 |
| Domain 上限 | Domain 总挂接数 ≤ `MAXDNA` |
| 控制器上限 | 单控制器挂接数 ≤ `MAXCNA` |
| ANA 检查 | Attach 触发不允许的 ANA 条件 → `ANA Attach Failed` |
| 首错停止 | 处理 Controller List 遇首错即停；Error Information Log 记失败条目字节偏移 |
| 失败原因 | Create 失败：不支持格式 / 容量不足 / NSID 耗尽 / 精简不支持 / ANA 组无效 / 命令集不支持 |
| 容量不足信息 | 错误信息日志报告还差多少未分配字节 |
| 最佳实践 | Delete 前先手动 Detach，避免不必要的事件通知 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Create vs Attach | Create 在子系统层"圈地"；Attach 在控制器层"装门" |
| Attach vs Detach | Attach 挂上；Detach 取下（命名空间仍存在） |
| Delete vs Detach | Delete 删命名空间（含自动 Detach）；Detach 只解挂 |
| Create vs Allocate | Create 是命名空间管理的命令；Allocate 是更通用的"分配"概念 |
| 持久化 vs 运行时 | Attach/Detach 状态持久化；I/O 队列、内存窗口等不持久化 |
| 私有 vs 共享 Attach | 私有挂到 1 个控制器；共享可挂多控制器（受命令集规则约束） |
| MAXDNA vs MAXCNA | MAXDNA = Domain 总挂接数；MAXCNA = 单控制器挂接数 |
| 错误日志 vs 状态码 | 状态码给出错类型；错误日志记失败 Controller List 条目的字节偏移 |
| 非阻塞 vs 阻塞 | 命名空间管理是**非阻塞**的；Format NVM、Sanitize 等是**阻塞**的 |
| 导出的 NVM 子系统 | 跨子系统暴露的命名空间管理中**禁止**；本子系统允许 |

## 进阶细节

- **两个独立管理层次的边界**（规范 5.2.1 / 5.2.22）：
  - 子系统级：Create / Delete 影响整个子系统
  - 控制器级：Attach / Detach 影响单个控制器
- **Attach 持久化**（规范 5.2.22 / Figure 338）：
  - 穿越所有 Reset 事件（控制器级与子系统级）
  - 不受虚拟化管理操作影响（即使从控制器 Offline，挂接状态保持）
  - Controller List 大小 = 4 KiB
- **Attach 操作的错误码**（规范 5.2.22）：
  - `Namespace Attachment Limit Exceeded`：超过 `MAXDNA` 或 `MAXCNA`
  - `I/O Command Set Not Supported` / `Not Enabled`：命令集不受支持或未启用
  - `Namespace Is Private`：私有命名空间已挂到其他位置
  - `Controller List Invalid`：包含管理控制器或格式错误
  - `ANA Attach Failed`：挂接会触发不允许的 ANA 条件
- **首错停止**（规范 5.2.22）：处理 Controller List 遇首错即停；Error Information Log 记录失败条目的字节偏移；列表中较早条目可能已成功。
- **能力要求**（规范 5.2.21 / 5.2.22）：
  - 支持 Namespace Management ⇒ 必支持 Namespace Attachment
  - 导出的 NVM 子系统中**禁止**支持 Namespace Management
  - I/O 命令在管理操作进行期间继续执行（非阻塞）
- **Create 参数**（规范 5.2.21 / Figure 342）：
  - NSID 必须设为 `0`（控制器选并返回）
  - Command Set Identifier
  - 512 字节参数区（命令集特定）
  - 可选 Vendor Data
  - 完成时 CQE 报告新分配的 NSID
- **Delete 行为**（规范 5.2.21）：
  - 具体 NSID = 删指定一个
  - `FFFFFFFFh` = 删所有（即使不存在也成功）
  - 副作用：自动从所有控制器解挂
  - 事件：若仍挂接则生成 attached-namespace 变更事件；每次成功必生成 allocated-namespace 变更报告
- **Create 失败原因**（规范 5.2.21 / Figure 343）：
  - 不支持的格式（参数超能力）
  - 容量不足（错误日志报告还差多少字节）
  - NSID 耗尽
  - 不支持精简配置
  - 无效的 ANA Group
  - 不支持的 I/O 命令集
- **挂接目标要求**（规范 5.2.22）：
  - 必须是 I/O 控制器
  - 必须支持该命名空间的 I/O 命令集
  - 必须已启用该 I/O 命令集（通过配置或命令集配置文件）
- **多域场景**：Attach 不受多域限制；命名空间写保护的"直到掉电前"模式在多域子系统中**禁止**（参见 namespace-write-protection.md）。

## 规范依据

- [Namespace Attachment 持久化与命令封装，PDF 第 383 页](../_source/pages/page-383.md)
- [Attachment 限制、命令集检查与首错行为，PDF 第 384 页](../_source/pages/page-384.md)
- [Attachment 状态码与 Namespace Management 生命周期，PDF 第 385 页](../_source/pages/page-385.md)
- [Create 数据结构与失败信息，PDF 第 386 页](../_source/pages/page-386.md)
- [Create 返回的 NSID 完成值，PDF 第 387 页](../_source/pages/page-387.md)

## 相关阅读

- [namespace.md](namespace.md) - 被管理的命名空间对象
- [namespace-access-models.md](namespace-access-models.md) - Attach 影响访问模型选择
- [namespace-identifiers.md](namespace-identifiers.md) - 标识随命名空间生命周期变化
- [format-nvm-lifecycle.md](format-nvm-lifecycle.md) - 类似的资源级生命周期
