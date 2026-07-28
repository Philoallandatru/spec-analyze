# Keep Alive（保活机制）

## 一句话说明

Keep Alive 是 NVMe 控制器内嵌的一种"看门狗"机制：每个控制器各自维护一个定时器，用来发现主机和控制器之间何时已经失联，并在必要时主动宣告自己进入致命状态。

## 生活化类比

把 Keep Alive 想象成**物业保安的巡逻打卡**：

- **巡逻岗亭** = 控制器；岗亭里有一块倒计时牌 = Keep Alive 定时器
- **倒计时起点** = 岗亭刚刚和业主联系上
- **每次收到业主消息**（命令或 Keep Alive 命令）= 保安按下"重置"按钮，倒计时回到设定值
- **倒计时归零前没有任何消息** = 岗亭认定业主失联，按应急预案拉响警报（设置 `CSTS.CFS=1`）

> 这个机制的关键是"双向"：主机不仅要主动发"我还活着"的信号（Command Based 模式），控制器也得在没人说话时主动判断是不是掉线了。

## 工作流程

```text
+-----------------------------+       +-----------------------------+
|        控制器侧定时器        |       |          主机侧              |
+-----------------------------+       +-----------------------------+
        |                                       |
   上电/复位/CC.EN 置 1                         |
        |                                       |
   CSTS.RDY = 1, KATO ≠ 0  -->  定时器激活     |
        |                                       |
   监控活动信号（命令/Keep Alive）                |
        |                                       |
        |        <-- Keep Alive 命令 周期发送 --|
        |   收到命令/流量   -->  重启定时器      |
        |                                       |
   持续 KATT 时间无活动                          |
        |                                       |
   CQT 时间内：                                 |
   1) 写错误日志 (Keep Alive Timeout Expired)   |
   2) CSTS.CFS = 1 致命状态                      |
   3) Fabrics: 终止关联/连接                     |
        |                                       |
   主机按 communication-loss 恢复流程重试        |
        +---------------------------------------+
```

**模式差异**：

| 模式 | 触发条件 | 重启依据 |
|------|---------|----------|
| Command Based（默认）| 收到 Keep Alive 命令并成功完成 | 仅 Keep Alive 命令 |
| Traffic Based（`TBKAS=1`）| 一个 `KATT` 间隔内取到任意 Admin/I/O 命令 | 任意命令流量 |

## 初学者案例

**场景：RDMA 链路抖动，SSD 是否会自己"认错"？**

1. 一台服务器的 NVMe-oF 目标通过 RDMA 连接主机。某条链路发生间歇性丢包。
2. 主机可能毫不知情：本地 TCP 栈还在重传，但 NVMe 控制器一侧已经连续 `KATT` 时长（默认 120 秒）没看到命令。
3. 控制器立即：
   - 在 Error Information Log 写一条 "Keep Alive Timeout Expired" 记录
   - 设置 `CSTS.CFS=1`，停止处理新命令
   - 对 Fabrics 传输：终止 Association，释放传输连接
4. 主机的 NVMe 驱动检测到完成队列静默、连接被远端关闭，按"通信丢失"流程重置并重试。
5. 重试在新的 Association 上重新建立；命名空间和命名空间状态不受影响。

> 关键收获：Keep Alive 把"链路已经死了但双方都不知道"变成"控制器主动宣告死亡并清理状态"，避免主机无限等待一个永远不会来的完成响应。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| KAS 非零才支持 | 控制器只在 `KAS` 通告为非零值时支持 Keep Alive；零值表示不提供 |
| KATO 单位毫秒 | Set Features 中的 `KATO` 是毫秒，必须为 `KAS` 的整数倍 |
| 默认值随传输而异 | 不强制该机制的传输默认 0；RDMA/TCP 强制要求 120000 ms |
| 设置值不可越界 | 低于下限被提升；超过上限返回 "Keep Alive Timeout Invalid" |
| 激活四条件 | `CC.EN=1` ∧ `CSTS.RDY=1` ∧ `CC.SHN=00b` ∧ `CSTS.SHST=00b` ∧ `KATO≠0` |
| 致命化超时 | 超时后写错误日志，置 `CSTS.CFS=1`，对 Fabrics 终止 Association |
| Traffic 模式最坏 `2×KATT` | 命令在检查点后到达时，检测最坏延迟 `2×KATT` |
| 主机预留 Admin SQ 空间 | 定时器激活期间应为 Keep Alive 命令预留 Admin SQ 槽位 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| KAS vs KATO vs KATT | `KAS` 控制器通告的支持能力；`KATO` 主机配置值；`KATT` 控制器当前生效值 |
| Command Based vs Traffic Based | 前者只认 Keep Alive 命令；后者认任何命令流量；模式由 `TBKAS` 位决定 |
| 控制器超时 vs 主机超时 | 控制器超时会置 CFS、写日志；主机超时只需走 communication-loss 流程 |
| CQT vs KATT | `KATT` 是 Keep Alive 间隔；`CQT`（Completion Queue Timeout）是超时后清理宽限 |
| `CC.SHN` vs `CSTS.SHST` | `SHN` 是主机发起的关闭通知；`SHST` 是控制器当前的关闭状态 |

## 进阶细节

- **配置接口**（规范 5.28.1）：仅 Fabrics Connect 与 Set Features 可设置 `KATO`；Get Features 读取生效值。
- **KAS 粒度**（规范 5.28.2）：所有协议强制 `KAS` 为 `100ms` 的整数倍；非零 `KATO` 必须向上对齐到 `KAS` 倍数。
- **协议差异**（规范 Figure 87 / 88）：PCIe 默认不要求；RDMA/TCP 默认 120000 ms；其他传输按 TP 规范。
- **Traffic Based 模式**（规范 5.28.3）：主机侧可强制走 Command Based 风格；控制器端若走 Traffic Based，则命令可能在检查点后到达，造成最坏 `2×KATT` 延迟。
- **超时清理**（规范 5.28.4）：`CQT` 时长内必须完成"日志 + `CSTS.CFS=1` + 终止 Association"三步；Fabrics 控制器在终止 Association 后允许通过新的 Admin Connect 重建。
- **与 Keep Alive 命令的关系**：Keep Alive 命令是一个普通 Admin 命令（Opc=18h），其提交/完成都受 Admin 队列仲裁；保持 Admin SQ 至少 1 个槽位空闲是主机责任。
- **致命化对应用层的影响**：一旦 `CSTS.CFS=1`，现有 I/O 命令不会被完成；驱动必须等命令超时后或经 Controller Reset 重新初始化。

## 规范依据

- [Keep Alive 能力与传输策略，PDF 第 146 页](../_source/pages/page-146.md)
- [KAS 粒度、配置接口与激活条件，PDF 第 147 页](../_source/pages/page-147.md)
- [Command Based 模式与主机发送建议，PDF 第 148 页](../_source/pages/page-148.md)
- [Traffic Based 模式与最坏 `2×KATT` 时序，PDF 第 150 页](../_source/pages/page-150.md)
- [超时清理、写错误日志与 Fabrics 终止关联，PDF 第 151 页](../_source/pages/page-151.md)

## 相关阅读

- [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) - CFS/SHN 状态字段联动
- [communication-loss-and-command-retry.md](communication-loss-and-command-retry.md) - 超时后的通信丢失恢复
- [controller-initialization.md](controller-initialization.md) - Fabrics 启动与 KAS 配置
- [asynchronous-event-reporting.md](asynchronous-event-reporting.md) - 关联终止的 AEN 上报
