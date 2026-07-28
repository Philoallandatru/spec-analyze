# 导出资源与底层资源（Exported and Underlying NVM Resources）

## 一句话说明

NVMe 把"实际提供存储的物理/虚拟资源（**底层资源**）"和"专门给远程主机看的逻辑视图（**导出资源**）"严格分开：底层是事实，导出是门面；门面的生命周期独立维护，删除门面不会动到底层数据。

## 生活化类比

把 NVMe-oF 存储集群想成**国际快递的国内仓 + 海外门店**：

- **国内仓** = 底层 NVM 子系统（货真在这里）
- **海外门店** = 导出 NVM 子系统（给海外客户看的"我们的铺面"）
- **门店里的货架编号** = 导出命名空间（和国内某个货架上的货一一对应，但叫"门店代号"）
- **门店的收件窗口** = 导出端口（地址 + 传输类型）
- **门店的 VIP 客户名单** = 允许主机列表（决定谁能下单）

海外门店可以重新装修（变更访问模式、清空配置、关店），但**国内仓的货架和货不会被搬走**——这就是"导出"与"底层"分离的设计意图。

## 工作流程

```text
底层 NVM 子系统（Underlying Subsystem）
 ├─ 底层命名空间  ──关联 (associate)──>  导出命名空间 ──┐
 └─ 底层端口      ── 端口列表       ──>  导出端口     ──┤
                                                       ▼
                                          导出 NVM 子系统
                                          ├─ 控制器 [0..*]
                                          ├─ 导出命名空间 [0..*]
                                          ├─ 导出端口 [0..*]
                                          └─ 允许主机列表 [可选]
```

简化说明：上层是"给远程主机看的视图"，下层是"真实存数据的实体"；两者通过"关联/端口列表"桥接，但管理动作彼此独立。

## 初学者案例

**场景：管理员给租户 B 分配一台新远程盘（导出门店），但 B 报"看不到命名空间"。**

1. 管理员先在底层子系统上 `Create Exported NVM Subsystem`（规范 5.3.2 / PDF 第 448 页），选访问模式（Unrestricted / Restricted）。
2. 新建出的导出子系统：允许主机列表为空、未关联任何命名空间、未创建任何导出端口——B 自然看不到盘。
3. 管理员依次执行：
   - `Associate Namespace`（规范 5.3.x / PDF 第 460 页）→ 把底层命名空间绑到该导出子系统；
   - `Create Exported Port`（规范 5.3.x / PDF 第 467 页）→ 给导出子系统挂一个对外传输地址；
   - `Grant Access`（若是 Restricted 模式）→ 把 B 的 Host Identifier / Host NQN 加入允许列表。
4. B 端 NVMe-oF initiator 重新发现 → 命名空间出现，可挂载使用。
5. 故障速查：步骤缺一就会"看不到"；最常见遗漏是 Restricted 模式下忘记 Grant Access，或忘记 Create Exported Port。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 概念分离 | 底层 = 实际资源；导出 = 远程访问视图，两者生命周期独立 |
| 导出资源三类 | 导出 NVM 子系统、导出命名空间、导出端口（规范 1.5.35） |
| 导出命名空间定义 | "导出 NVM 子系统内的命名空间"（规范 1.5.34） |
| 目录两条 | 基于消息传递的控制器通过 `CNS=1Dh` 列底层命名空间；`CNS=1Eh` 列底层端口 |
| 条目内容 | 底层 NS 列表 = Subsystem NQN + NSID + Controller ID；底层端口列表 = Underlying Port ID + 传输地址/类型/地址族 |
| 版本计数 | 两个 Generation Counter：底层子系统重置时清零；目录变更时 +1（按 1 模 2³²递增） |
| 创建时初始化 | 新建导出子系统初始为空：允许主机列表空、无导出命名空间 |
| 命令执行位置 | `Manage Exported Namespace` / `Manage Exported NVM Subsystem` 都不能在导出子系统上执行 |
| 删除前置 | Clear Exported Config 前必须断开所有主机连接，否则报 Command Sequence Error |
| 关联 vs 附加 | 关联（Association）建"导出 NS ↔ 底层 NS"映射；附加（Attachment）建"NS ↔ 控制器"挂载 |
| 解除关联前置 | 导出 NS 必须先从所有控制器 Detach，才能 Disassociate |
| 访问控制范围 | Allowed Host List 是子系统层逻辑控制；Fabric Zoning 是网络/基础设施层控制 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 底层资源 vs 导出资源 | 底层是"事实存数据的实体"；导出是"给远程主机看的视图" |
| 关联 vs 附加 | 关联建"导出 NS 与底层 NS 的对应"；附加建"NS 挂在哪个控制器上" |
| Clear Exported Config vs 删除底层 | 前者只清导出层；后者才真正动数据 |
| Allowed Host List vs Fabric Zoning | 前者是子系统层逻辑访问控制；后者是基础设施层网络隔离 |
| `CNS=1Dh` vs `CNS=1Eh` | 前者列底层命名空间；后者列底层端口 |
| Unrestricted vs Restricted 模式 | 前者任意主机可连；后者只允许列表内主机，切换时强制断开不在列表内的已连接主机 |
| 撤销访问 vs 删除导出端口 | 撤销是取消允许；删除是关闭端口并终止主机/控制器关联（含未定义行为风险） |

## 进阶细节

- **规范 1.5.34 / 1.5.35 术语**（PDF 第 30 页）：Exported Namespace = 导出 NVM 子系统内的命名空间；Exported NVM Resources 包含 (a) Exported NVM Subsystem、(b) Exported Namespace、(c) Exported Port。
- **目录 Identify 字段**（规范 Identify CNS / PDF 第 368-370 页）：
  - `CNS=1Dh` 底层命名空间列表：每条 = Subsystem NQN + NSID + Controller ID（子系统内唯一）；
  - `CNS=1Eh` 底层端口列表：每条 = Underlying Port ID + 传输地址 / 子类型 / 类型 / 地址族 / 关联要求（具体格式由各 NVMe 传输规范定义）。
- **Generation Counter 行为**（规范 Identify / PDF 第 368 页）：底层 NVM 子系统重置时清零；目录内容变化时 +1；饱和后回绕到 0。主机可借此发现"目录是否被并发修改"。
- **Create Exported NVM Subsystem**（规范 5.3.2 / PDF 第 448-449 页）：
  - 必选访问模式 Unrestricted / Restricted；
  - 初始：允许主机列表空、无导出命名空间；
  - 命令成功返回新分配的 `SUBNQN`；
  - **不能在导出 NVM 子系统上执行**。
- **Clear Exported NVM Resource Configuration**（规范 5.3.1 / PDF 第 448 页）：
  - 删除全部导出资源配置信息；
  - 不影响底层命名空间 / 底层 NVM 子系统；
  - 前置：所有主机已从所有导出 NVM 子系统断开，否则 `Command Sequence Error`；
  - 同样不能在导出 NVM 子系统上执行。
- **Associate Namespace**（规范 5.3.x / PDF 第 460-461 页）：
  - 通过目标导出子系统 NQN + 底层三元组（Subsystem NQN / Controller ID / Active NSID）定位；
  - 前置：底层 NS 已分配、已附加到指定控制器；
  - 效果：分配新导出 NS；与底层 NS 访问同一份用户数据；
  - 不会自动附加到控制器——需另做 Namespace Attachment；
  - 触发 Namespace Attribute Changed 异步事件。
- **Disassociate Namespace**（规范 5.3.x / PDF 第 461-462 页）：
  - 前置：导出 NS 已从所有控制器 Detach；
  - 效果：移除关联、删除 ENSID；
  - 触发 Namespace Attribute Changed 异步事件。
- **删除依赖检查**（规范 5.3.x / PDF 第 462-463 页）：存在活跃控制器连接 / 已关联导出 NS / 已创建导出端口时，删除导出子系统报 `Command Sequence Error`。
- **Change Access Mode**（规范 5.3.x / PDF 第 463-466 页）：
  - 切到 Restricted：立即断开不在允许列表的已连接主机；
  - 切到 Unrestricted：允许任意主机连接。
- **Grant Access**（规范 5.3.x / PDF 第 464-466 页）：在非空主机列表 + 导出子系统列表上操作；每个目标子系统与一个 Underlying Port ID 绑定；主机条目含 128 位 Host Identifier + `HOSTNQN`；子系统条目含 `SUBNQN` + Underlying Port ID；失败条目可在 Error Information Log 中按字节偏移定位。
- **Revoke Access**（规范 5.3.x / PDF 第 465-466 页）：移除主机/子系统组合并**立即断开**受影响的已连接主机。
- **Create Exported Port**（规范 5.3.x / PDF 第 467-468 页）：绑定（底层端口、传输服务标识符）；端口 ID 可主机指定或让控制器生成；返回所选 ID。
- **Delete Exported Port**（规范 5.3.x / PDF 第 468-469 页）：参数 = `SUBNQN` + Exported Port ID；建议端口未在使用时删除；删除使用中端口会终止通过该端口的所有主机/控制器关联，**未完成命令与底层 Fabric 资源行为未定义**——务必先断连。
- **Fabric Zoning vs Allowed Host List**（规范 1.5 概念 / PDF 第 31 页）：前者是网络/传输层访问控制；后者是子系统层应用访问控制；可叠加使用。

## 规范依据

- [导出资源定义，PDF 第 30 页](../_source/pages/page-030.md)
- [底层命名空间 / 端口 / 子系统定义，PDF 第 35 页](../_source/pages/page-035.md)
- [底层命名空间列表与端口列表 Identify 目录，PDF 第 368 页](../_source/pages/page-368.md)
- [Clear / Create Exported NVM Subsystem 命令，PDF 第 448 页](../_source/pages/page-448.md)
- [Associate / Disassociate Exported Namespace，PDF 第 460 页](../_source/pages/page-460.md)
- [访问模式与允许主机列表修改，PDF 第 463 页](../_source/pages/page-463.md)
- [Create / Delete Exported Port，PDF 第 467 页](../_source/pages/page-467.md)

## 相关阅读

- [admin-command-model.md](admin-command-model.md) - 导出资源管理命令的 opcode
- [controller-data-queues.md](controller-data-queues.md) - 主机内存分配与 PRP 用法
- [domains-and-divisions.md](domains-and-divisions.md) - 底层/导出资源的域归属
- [keep-alive.md](keep-alive.md) - Fabrics 传输的连接维护
