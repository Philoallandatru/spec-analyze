# 断电信号机制（Power Loss Signaling）

## 一句话说明

断电信号机制（Power Loss Signaling, PLS）是 NVMe 在 PCIe 上提供的"提前预警"能力：当系统检测到电源即将断开时，通过断言 PLN（Power Loss Notification）信号通知控制器；控制器在 Domain 范围内统一选择"强制静默（FQ）"或"紧急断电（EPF）"模式，在断电前完成数据保护工作。

## 生活化类比

把 Domain 内的控制器们想成**一群收银员**：

- **商场断电预警广播** = PLN 信号
- **收银员按商场规则处理**：
  - **FQ 模式** = 收银员把手头的单子结完，暂停接新单（但收银机还亮着）
  - **EPF 模式** = 收银员把手头的单子直接丢进保险箱（数据保护），可以关闭收银机
- **两种 EPF 子模式**：
  - **端口保持启用** = 收银机屏幕还亮，管理员可远程看进度或下发命令
  - **端口禁用** = 收银机彻底关屏，只能等下次市电恢复
- **断电后再次上电** = 收银员上班，先做"恢复"工作（vault 保护 + recovery）

> 关键：所有收银员（Domain 内所有控制器）必须听同一份规则——同一时刻要么全 FQ、要么全 EPF，绝不能各行其是。

## 工作流程

```text
   上电 / 复位 / 关机事件
            |
            v
      [PLS Not Ready]
            |
            | SHST=00b (正常)
            v
      [PLS Ready]  <----------+
            |                |
            | PLN asserted   | 解除 PLN
            | 选择 FQ / EPF  | 复位/重上电
            v                |
    +-------+-------+--------+----+
    |               |             |
    v               v             v
 [FQ 处理中]   [EPF 处理中]   [EPF 处理中]
 (端口保持启用)  (端口保持启用) (端口禁用)
    |               |             |
    v               v             v
 [FQ Complete] [EPF Complete]  [EPF Complete]
    |            (端口保持启用)    (端口禁用)
    +--> 解除 PLN, 回到 Ready  |   |
                              v   v
                          复位/重上电 只能重上电
                          回到 Not    才能恢复
                          Ready
```

**两种模式对比**：

| 维度 | FQ（强制静默）| EPF（紧急断电）|
|------|--------------|----------------|
| 已取命令 | 完成或中止 | 直接丢弃 |
| 端口通信 | 保持 | 可保持 或 禁用 |
| 后台操作 | 挂起 | 丢弃 |
| 断电恢复 | 解除 PLN 即可 | 披露恢复时长 |
| 退出方式 | 解除 PLN / 复位 | 复位 / 关机 / 重上电 |

## 初学者案例

**场景：机房 UPS 即将耗尽，NVMe SSD 怎么自保？**

1. UPS 控制器检测到电池快用完，向 PCIe 链路断言 PLN。
2. 同一 Domain 内的所有 NVMe 控制器收到 PLN，根据之前保存的配置选择模式。
3. **如果配的是 FQ**：
   - 控制器停止从队列取新命令
   - 把手头正在处理的写命令完成或中止
   - 把需要的数据保护信息（vault）写入非易失存储
   - 完成后解除 PLA 信号，PCIe 端口仍正常工作
4. **如果配的是 EPF（端口禁用）**：
   - 直接丢弃已取命令和管理端点命令
   - 写入 vault
   - 关闭 PCIe 端口通信
   - 此时主机已无法下发任何命令
5. 市电恢复后，控制器在 CQE 中披露 EPF recovery 间隔；命名空间初始化时可能尚未完全恢复。
6. 驱动层应根据此报告延迟挂载数据敏感型命名空间。

> 关键收获：PLS 不是"断电保护"本身，而是"在真正断电前，给控制器若干毫秒或数秒完成保护动作"。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 作用域是 Domain | 同一 Domain 内所有控制器必须披露相同支持、选择相同模式 |
| PLN 强制 | 系统支持 PLS 时，PLN 信号必须实现 |
| PLA 可选 | 控制器可选择是否实现 PLA 信号 |
| 一种 EPF 端口行为 | 每个控制器只能实现"端口启用"或"端口禁用"之一；Domain 内必须一致 |
| 已取状态下锁配置 | FQ 处理过程中拒绝改变 PLS 配置 |
| 后台挂起 | FQ 期间所有后台操作（GC、磨损均衡）挂起 |
| 复位清信号 | Controller Level Reset 会把 PLN/PLA 都重置为未断言 |
| 状态返回条件 | 复位或关机（非零 SHST）会让 FQ 状态返回 Not Ready |
| 端口禁用退出 | 端口禁用 EPF 完成后只能"重上电"退出 |
| 恢复时序 | EPF 完成状态下首次上电可能暴露未就绪/降级的命名空间 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| FQ vs EPF | FQ 优雅关门；EPF 直接丢单 |
| EPF 端口启用 vs 端口禁用 | 前者保留可管理性；后者只能重上电 |
| PLN vs PLA | PLN 由传输层发出；PLA 由控制器响应 |
| Domain 模式 vs 控制器实现 | 模式在 Domain 范围一致；端口行为是每个控制器的实现选择 |
| SHST vs SHN | `SHN` 主机发起的关闭通知；`SHST` 控制器当前关闭状态 |
| Not Ready vs Ready | Not Ready 屏蔽 PLN；Ready 才允许响应 |
| Power State Descriptor vs PLS | 前者定义电源状态；后者定义断电行为 |

## 进阶细节

- **三状态阶段**（规范 Figures 660-661）：
  - Not Ready：屏蔽所有 PLN 转换
  - Ready：监控 PLN 转换
  - FQ Complete / EPF Complete：终态
- **变量表**（规范 5.2.27）：Domain 范围配置决定当前生效模式（FQ / EPF / Disabled）；保存值在 Reset 后恢复。
- **PLA 信号三态**（规范 Table 92）：
  - Deasserted：未在 EPF 模式
  - Asserted-EPF-Enabled：EPF 模式，PCIe 端口通信继续
  - Asserted-EPF-Disabled：EPF 模式，PCIe 端口已停止
- **FQ 处理序列**（规范 5.2.27.3）：
  1. 断言 PLA
  2. 停止从队列取命令
  3. 完成/中止已取命令 + 准备 vault
  4. 挂起后台操作
  5. 解除 PLA 断言
- **EPF 处理序列**（规范 5.2.27.4）：
  1. 断言 PLA
  2. 停止从队列取命令
  3. 丢弃已取命令 + 丢弃管理端点命令
  4. 准备 vault（按选定的端口行为）
  5. 解除 PLA 断言
- **被忽略的 PLN 转换**（规范 5.2.27.5）：
  - Controller Level Reset 进行中
  - Shutdown 进行中或已完成
- **EPF Recovery 报告**（规范 5.2.27.6）：EPF 完成后，Identify Controller 中披露首次初始化的恢复间隔；实际恢复可在 `CC.EN=1` 之前或之后开始；非成功完成可能超出披露值。
- **时序值为零的语义**（规范 5.2.27.7）：若 vault / recovery 时长配置为 0，则实际时长为厂商自定义。
- **多 Domain 协调**：不同 Domain 的 PLS 状态独立；一个 Domain 进入 EPF 不影响其他 Domain。
- **与 ANA 关系**：EPF 端口禁用时，ANA 状态应反映 inaccessible；其他 Domain 不受影响。

## 规范依据

- [PLS 模式与变量表，PDF 第 615 页](../_source/pages/page-615.md)
- [PLA 表与支持约定，PDF 第 616 页](../_source/pages/page-616.md)
- [状态机 Not Ready / Ready / 处理中，PDF 第 617 页](../_source/pages/page-617.md)
- [状态 / PLA / 端口矩阵，PDF 第 618 页](../_source/pages/page-618.md)
- [FQ 与 EPF 分支转换，PDF 第 619 页](../_source/pages/page-619.md)
- [FQ 与 EPF 处理序列细节，PDF 第 621 页](../_source/pages/page-621.md)

## 相关阅读

- [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) - SHN/SHST 状态联动
- [power-state-descriptors.md](power-state-descriptors.md) - 电源状态定义基础
- [domains-and-divisions.md](domains-and-divisions.md) - Domain 范围与一致性
- [persistent-event-log.md](persistent-event-log.md) - 意外断电事件归档
