# 命名空间预留生命周期（Namespace Reservation Lifecycle）

## 一句话说明

命名空间预留（Namespace Reservation）是一种由命名空间维护、由主机用 64 位 Reservation Key 登记并竞选的访问控制机制：通过把"持有预留"的特权授予单一持有者、把"被准许操作"的特权授予多个注册者，实现多主机共享一个命名空间时的有序访问。

## 生活化类比

把命名空间想成**一个共享会议室**：

- **会议室门** = 命名空间
- **员工工卡** = 64 位 Reservation Key（每张卡只能借给一个员工）
- **登记（Register）** = 员工先去前台刷卡登记
- **预订（Acquire）** = 第一个成功预订的人成为"持有者"，独占会议
- **类型（Type）** = 预订规则：
  - 写独占 = 只有持卡人能用
  - 独占访问 = 持卡人独占整个会议室
  - 写独占-注册者 = 只有注册者能写
  - 写独占-所有注册者 = 所有注册者能写，持卡人独占
  - 独占访问-注册者 / 独占访问-所有注册者 = 同理升级到读
- **释放（Release）** = 会议结束，预订归还但登记保留
- **清场（Clear）** = 取消所有预订和登记
- **替换（Preempt）** = 强行替换现有预订

> 关键：每张卡（Key）属于一个员工（Host）；会议室（Namespace）只认卡不认人。

## 工作流程

```text
  [未注册]
       | Register (IEKEY 可跳过校验)
       v
  [注册者 registrant] -- Acquire(type) --> [预留持有者 reservation holder]
       |                  |                       |
       | Replace key      | Preempt               | Release
       | Unregister       | Preempt+Abort         v
       v                  v                  [注册者]（registrants 保留）
  [未注册]            抢占的 key 被移除        |
                                            Clear -> 所有状态清空
```

**预留类型**（6 种）：

| 类型 | 持卡人 | 注册者 | 其他主机 |
|------|--------|--------|----------|
| Write Exclusive | R/W | 无访问 | 无访问 |
| Exclusive Access | R/W | 无访问 | 无访问 |
| Write Exclusive – Registrants Only | R/W | R/W | 无访问 |
| Write Exclusive – All Registrants | R/W | R/W | R/W |
| Exclusive Access – Registrants Only | R/W | R | 无访问 |
| Exclusive Access – All Registrants | R/W | R | R |

> R = 可读；R/W = 可读写；"持卡人"也属于注册者集合。

## 初学者案例

**场景：两个主机共享一个 NVMe 卷，避免互相覆盖。**

1. 主机 A、B 各自生成一个 64 位 Key（`KEY_A`、`KEY_B`）。
2. 两台主机都向控制器 Register，控制器内部记录两条注册。
3. 主机 A 发送 Acquire(Write Exclusive – Registrants Only, KEY_A) → 成功成为 Holder。
4. 主机 B 尝试 Write → 控制器返回 `Reservation Conflict`。
5. 主机 A 完成工作后 Release(KEY_A) → 仍然注册但不再持有预留。
6. 现在两台主机都能写（因为没人持有）。
7. 主机 B 想要"独占写入窗口"，可 Acquire(Write Exclusive, KEY_B) 抢占。
8. 主机 A 突然断电？不影响——PTPL=1 时，注册与预留持续存在。

> 关键收获：预留的"谁能做、谁能看"完全由类型决定；Key 是身份的载体，类型是规则的载体。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| Key 属于 Host | 64 位 Reservation Key 是主机身份，由主机生成 |
| 预留属于 NS | Reservation 状态绑定在命名空间，不在控制器 |
| IEKEY 跳过校验 | Reservation Register 可设 `IEKEY=1` 跳过对当前 Key 的校验 |
| IEKEY 不能用于 Acquire | Acquire 命令 `IEKEY=1` 无效 |
| Preempt 指定目标 | 必须给出要注销的 key；Preempt+Abort 还会中止未完成命令 |
| Clear 清空全部 | Clear 同时清掉预留与所有注册 |
| Release 保留注册 | Release 仅释放预留，注册者关系保留 |
| 分散 NS 需 DISNSRS | 分散命名空间上的 Reservation 命令必须设 `DISNSRS=1` |
| 报告代次号 | Reservation Report 含回绕的 Generation Counter |
| 主机 ID 宽度匹配 | 选 64-bit 还是 128-bit Host ID 决定 Basic 还是 Extended 记录格式 |
| CNTLID 哨兵 | `FFFFh` = 该 subsystem 无控制器关联；`FFFDh` = 远端参与方 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Register vs Acquire | Register 决定"我是谁"；Acquire 决定"我占这个 NS" |
| Release vs Clear | Release 保留注册；Clear 完全清空 |
| Preempt vs Preempt+Abort | 前者只踢 key；后者还中止该 NS 上未完成命令 |
| IEKEY 在 Acquire 不可用 | 仅 Register 支持 `IEKEY=1` |
| Write Exclusive vs Exclusive Access | 前者只锁写；后者读写都锁 |
| Registrants vs All Registrants | "Registrants Only" 排斥未注册者；"All Registrants" 允许所有注册者 |
| Basic vs Extended Record | 64-bit Host ID → 24B Basic；128-bit Host ID → 64B Extended |
| 本地 CNTLID vs 哨兵 FFFDh | 本地是真实 ID；`FFFDh` 是"远端参与方"的占位 |

## 进阶细节

- **PTPL（Persist Through Power Loss）**（规范 8.1.3）：
  - `PTPL=0`：断电后预留释放 + 注册清空
  - `PTPL=1`：断电后预留与注册都保留
  - Register 命令的 `RREGA`/`IEKEY` 字段决定 PTPL 是否变更；若 Reservation Persistence Feature 可保存，则当前值与保存值同步更新
- **Reservation Report 结构**（规范 8.1.4）：
  - 前缀由 Generation Counter、Reservation Type、Registrant Count、PTPL 状态组成
  - 之后是每条 24 字节（Basic）或 64 字节（Extended）的注册者记录
  - Generation Counter 在注册变更、Clear、Preempt 成功后递增
- **CNTLID 哨兵值**（规范 8.1.5）：
  - `FFFFh`：表示该 host 在本子系统中没有控制器关联
  - `FFFDh`：表示该 host 的控制器在另一参与方（仅用于分散 NS）
- **Acquire 失败模式**：
  - 当前已存在预留但 Key 不匹配 → `Reservation Conflict`
  - 控制器无对应 Host Identifier 记录 → `Invalid Field in Command`
- **Preempt 的副作用**（规范 8.1.6）：被抢占的注册者若持预留，新持有者获得预留；可选 Abort 中止该 NS 上未完成命令。
- **Release 类型校验**（规范 8.1.7）：Release 必须给出与当前类型匹配的值，否则 `Reservation Conflict`。
- **Register 副作用**（规范 8.1.8）：同一条 Register 可同时注册/注销/替换 Key；PTPL 副作用取决于 `RREGA` 位。
- **Reservation Acquire 必填字段**（规范 8.1.9）：Reservation Type、Current Reservation Key、PRINFO 字段。
- **与多主机 I/O 顺序**：在多主机拓扑中，预留是规范推荐的串行化机制；不应依赖普通 Read/Write 来协调。

## 规范依据

- [Acquire 行为、Key 校验与预留类型，PDF 第 489 页](../_source/pages/page-489.md)
- [Register 行为与 PTPL 副作用，PDF 第 491 页](../_source/pages/page-491.md)
- [Release / Clear 与 DISNSRS 校验，PDF 第 492 页](../_source/pages/page-492.md)
- [Reservation Report 前缀与 Generation，PDF 第 493 页](../_source/pages/page-493.md)
- [Basic/Extended 记录格式与 CNTLID 哨兵，PDF 第 495 页](../_source/pages/page-495.md)

## 相关阅读

- [reservation-notification-log.md](reservation-notification-log.md) - 预留变更事件通知流
- [persistent-event-log.md](persistent-event-log.md) - Reservation 跨断电归档
- [dispersed-namespace-lifecycle.md](dispersed-namespace-lifecycle.md) - 远端 CNTLID 哨兵语义
- [namespace-management-lifecycle.md](namespace-management-lifecycle.md) - 命名空间本身生命周期
