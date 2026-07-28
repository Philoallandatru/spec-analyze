# 预留通知日志（Reservation Notification Log）

## 一句话说明

预留通知日志是 NVMe 控制器维护的**破坏性读取**（Destructive Read）日志队列，专门记录挂接到本控制器的命名空间上发生的"未屏蔽"预留变更事件，主机用 `Get Log Page`（LID `80h`）按时间顺序逐条消费。

## 生活化类比

把控制器想成**物业前台**：

- **每个命名空间** = 一间办公室
- **预留变更**（被注册/被释放/被抢占）= 办公室门上换锁、撕封条、强拆工位
- **物业前台** = 控制器自带的**事件小黑板**，每发生一次就在小黑板上记一行
- **租户（主机）** 来前台时，前台**撕下最早那行**给他（破坏性读取），并在黑板底下补"已处理"标记
- **小黑板写满了** = 队列溢出，前台不会拒绝登记，只会把**最早的那一行划掉**，但"已划掉的次数"会记在**最新一行**的右上角
- 租户一比较：哎？我前一条是第 102 号，现在变成第 110 号——说明有 7 条**我没看到**

## 工作流程

```text
  命名空间上发生"未屏蔽"预留变更事件
                |
                v
   +-----------+----------+-----------+-----------+     Get Log Page LID 80h
   |  LPC=100  |  LPC=101 |  LPC=102  |  LPC=103 |  <------ 返回最旧并移除
   +-----------+----------+-----------+-----------+
                                       ^ 
                                       |
                                  LPC 持续递增
                                  (即使队列溢出也递增)

   LPC 字段语义：
   0h               = 队列空，返回全零 64 字节页
   1h..FFFF...FFFFh = 事件编号
   绕过全 1 后回绕  = 下一次重置为 1h
```

**字段级流程**：

1. 主机对控制器下发 `Get Log Page`，`LID = 80h`。
2. 控制器取出队列头部最旧的一条记录返回，同时**从队列中移除**。
3. 主机解析 64 字节日志页：LPC、RNLPT、NALP、NSID。
4. 主机继续读取直到 `NALP = 0`，表示队列清空。
5. 队列空时返回全零 64 字节页（所有字段为 0h）。

## 初学者案例

**场景：多主机共享一个命名空间，怎么知道别的机器把我的预留抢走了？**

1. 主机 A 在 NS1 上注册、并下了预留（Reservation）。
2. 主机 B 发 `Reservation Register` 把 A 抢占走（Registration Preempted）。
3. 控制器在队列里入队一条记录：`LPC=42`、`RNLPT=01h`（Registration Preempted）、`NSID=1`、`NALP=0`。
4. 主机 A 轮询日志（或收到异步事件触发）→ 发 `Get Log Page LID 80h`。
5. 控制器返回这条 LPC=42 的记录，并从队列移除。
6. 主机 A 看到自己被抢占了，决定让出锁或重注册。

> 关键点：日志是**控制器本地**的（每个控制器一份），只覆盖挂接到**本控制器**的命名空间；用 `Reservation Notification Mask`（Feature `82h`）可以屏蔽不想关心的事件类型。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 破坏性读取 | 每次读出后**立即从队列移除**，不可重复读；NALP 告诉主机"还剩多少条没读" |
| 队列在控制器本地 | 每个控制器维护自己的队列，与 NVM 子系统、命名空间无关；跨控制器看不到对方的事件 |
| LPC 单调递增 | 64 位序列号，从 `1h` 起递增；`0h` 保留表示空；`FFFF...FFFFh` 后回绕到 `1h` |
| 溢出计数在最新条 | 队列满时仍持续递增 LPC；最近一条的 LPC 反映"包括已丢失事件在内的累计计数" |
| 仅未屏蔽事件入队 | 已被 `Reservation Notification Mask`（Feature `82h`）屏蔽的事件**不会**生成日志记录 |
| 屏蔽在事件产生时生效 | 屏蔽是"出生前过滤"，不是"出生后丢弃"——主机不能事后读到被屏蔽的事件 |
| NSID 指向源命名空间 | 每条记录带命名空间 ID，主机能直接定位是哪间办公室出的事 |
| 命名空间独立配置 Mask | Feature `82h` 按命名空间生效；支持广播 Set 但**不支持广播 Get** |
| Reservation Persistence 单独配 | Feature `83h`（PTPL）控制预留是否跨断电保留；不可被 Save；与 Mask 是不同维度 |
| Controller Level Reset 清零 LPC | 控制器级重置会把 LPC 计数器清零，主机要意识到序列号会"重置" |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 预留通知日志 vs 普通快照日志 | 通知日志读一次就消失；普通日志（如 Error Info）读 N 次都一样 |
| NALP vs LPC | NALP 告诉你"这条读完还剩多少条"；LPC 是"事件的唯一编号"。NALP = 0 时队列空，**不是** LPC 归零 |
| RNLPT 值 `1h` vs `3h` | `1h` = Registration Preempted（注册被抢）；`3h` = Reservation Preempted（预留被抢）；`2h` = Reservation Released（释放） |
| Feature `82h` Mask vs Feature `83h` PTPL | Mask 决定"记不记"；PTPL 决定"断电后还在不在"；两者都按命名空间配置但**作用不同** |
| 广播 Set vs 广播 Get | Feature `82h`/`83h` 都支持"一次设全部挂接命名空间"；**都不支持**广播 Get |
| LPC 跳跃 vs 事件丢失 | 连续读取时 LPC 跳号 = 队列溢出丢事件；**LPC 不跳但 NALP 大于实际期望** 也说明有异常 |
| PTPL 启用 vs 预留持久 | PTPL 启用 = 断电后 Reservations + Registrants 都保留；PTPL 禁用 = 全部释放 + 清除注册者 |
| Save 操作 vs 不可 Save | Feature `83h`（PTPL）**不能**用 Save 命令持久化（non-saveable） |

## 进阶细节

- **64 字节页布局**（规范 Figure 290）：
  - Bytes `07:00` = LPC（Log Page Count, 64 位）
  - Byte `08` = RNLPT（Reservation Notification Log Page Type）
  - Byte `09` = NALP（Number of Additional Log Pages, 8 位，>255 时饱和为 255）
  - Bytes `11:10` = Reserved
  - Bytes `15:12` = NSID（Namespace Identifier, 32 位）
  - Bytes `63:16` = Reserved
- **LPC 边界处理**（规范 5.1.12.1.32）：
  - 队列空时返回 `0h` 标识的"空日志页"；LPC 从非零值（最后一条记录的 LPC）继续递增。
  - 跨越 `FFFF_FFFF_FFFF_FFFFh` 时，下次递增回到 `1h`，并产生新日志页。
- **RNLPT 编码**：
  - `0h` = Empty Log Page
  - `1h` = Registration Preempted
  - `2h` = Reservation Released
  - `3h` = Reservation Preempted
  - `4h..FFh` = Reserved
- **NALP 饱和计数**：当未读记录 > 255 时，NALP 返回 255；主机不应假定"读到 NALP=0 才结束"，应循环到 LPC 出现 `0h` 标识的空页。
- **Mask 屏蔽语义**（规范 5.1.25.1.18）：屏蔽在事件**产生时**生效，控制器根本不创建日志页；这与"产生后丢弃"在主机侧不可见的事件流上**结果相同**，但实现上不会占用队列槽。
- **PTPL 与 Reservations 关系**（规范 5.1.25.1.19）：PTPL=1 时 Reservations 和 Registrants 在断电后保留；PTPL=0 时断电后全部释放，控制器重启后没有持久化预留状态。
- **Persistent Event Log 关联**：预留通知日志里的事件**同时**会在 Persistent Event Log 中以"Reservation Notification"事件类型持久化，跨掉电后可重建时间线。

## 规范依据

- [Reservation Notification 队列创建、破坏性读取与溢出 LPC 计数，PDF 第 301 页](../_source/pages/page-301.md)
- [Figure 290 字段布局（LPC/RNLPT/NALP/NSID）与回绕，PDF 第 302 页](../_source/pages/page-302.md)
- [Reservation Notification Mask 特性（Feature `82h`），PDF 第 424 页](../_source/pages/page-424.md)
- [Reservation Persistence 特性（Feature `83h`），PDF 第 424 页](../_source/pages/page-424.md)
- [Persistent Event Log 事件类型定义，PDF 第 301 页](../_source/pages/page-301.md)

## 相关阅读

- [namespace-reservation-lifecycle.md](namespace-reservation-lifecycle.md) - 产生通知的预留操作
- [log-page-retrieval.md](log-page-retrieval.md) - Get Log Page 读取机制
- [persistent-event-log.md](persistent-event-log.md) - 跨断电事件补全
- [asynchronous-event-reporting.md](asynchronous-event-reporting.md) - AER 通知主机来读取
