# 预留通知日志（Reservation Notification Log）

预留通知日志（LID `80h`）是每个 NVMe 控制器本地维护的**破坏性读取（Destructive Read）**事件队列，用来记录挂接到本控制器的命名空间上发生的"未屏蔽"预留变更事件（如注册被抢占、预留被释放或被抢占）。主机通过 `Get Log Page`（LID `80h`）按时间顺序逐条读取并消费这些通知。[PDF pp. 301-302](../_source/pages/page-301.md)

可以把控制器想成**共享办公楼的物业前台**：
- **命名空间** = 共享办公室，多台主机可注册或抢占使用权（Reservation）
- **日志队列** = 前台的事件小黑板，记录哪间办公室换了锁或被强占
- **破坏性读取** = 租户来前台读取时，前台撕下最早那行记录交给他（读完即从黑板移除）
- **溢出计数** = 小黑板写满时，新事件会顶掉最旧记录，但事件编号（LPC）继续递增；租户比对前后编号差，就能发现中间漏读了几条

---

## 队列结构与核心字段

```text
  未屏蔽的预留变更事件（仅限挂接到本控制器的命名空间）
            |
            v
     +---------+---------+---------+       Get Log Page (LID 80h)
     | LPC=100 | LPC=101 | LPC=102 |  ---------------------------> 返回最旧记录并移除
     +---------+---------+---------+
            ^
            |  队列满时覆盖旧记录，但 LPC 持续递增；
            |  最新记录的 LPC 反映包含已丢失事件在内的累计计数
```

每条日志页固定为 **64 字节**（Figure 290），核心字段如下：[PDF p. 302](../_source/pages/page-302.md)

| 字段 | 字节偏移 | 说明 |
|------|----------|------|
| **LPC**（Log Page Count） | `07:00` | 64 位单调事件编号：从 `1h` 起递增，达 `FFFF...FFFFh` 后回绕到 `1h`；**控制器级别重置时清零**；`0h` 保留表示队列空（返回全零 64 字节页） |
| **RNLPT**（通知类型） | `08` | `0h`=空页；`1h`=Registration Preempted（注册被抢占）；`2h`=Reservation Released（预留被释放）；`3h`=Reservation Preempted（预留被抢占）；`4h–FFh`=保留 |
| **NALP**（额外日志页数） | `09` | 读完当前记录后队列中剩余的未读记录数，**最大饱和值为 255**（>255 时仍报 255） |
| **NSID**（命名空间 ID） | `15:12` | 发生预留变更的命名空间标识符 |

### 消费与溢出检测流程
1. 主机收到异步事件通知（AER）或主动轮询，下发 `Get Log Page`（LID `80h`）。
2. 控制器返回队列头部最旧的 64 字节记录并**立即移除**；若队列为空，返回全零页（`LPC=0h`）。
3. 由于 `NALP` 在 255 处饱和，主机应循环读取直至读到 `NALP=0` 或空页（`LPC=0h`）。
4. 主机比对连续两条记录的 `LPC` 差值：例如前一条 `LPC=102`、下一条 `LPC=110`，说明队列曾溢出并丢失了中间 7 个事件。

---

## 相关特性配置（Mask 与 Persistence）

| 特性 | Feature ID | 作用与关键规则 |
|------|------------|----------------|
| **Reservation Notification Mask** | `82h` | 控制屏蔽哪类预留事件。屏蔽在**事件产生时生效**（根本不入队，非事后过滤）。按命名空间配置，支持**广播 Set**，不支持广播 Get。[PDF p. 424](../_source/pages/page-424.md) |
| **Reservation Persistence** | `83h` | 控制断电持久化（`PTPL`）：`PTPL=1` 断电后保留预留与注册者；`PTPL=0` 则全部释放清除。按命名空间配置，支持**广播 Set**，不支持广播 Get，且**不可 Save（non-saveable）**。[PDF pp. 424-425](../_source/pages/page-424.md) |

### 常见易混淆点
- **破坏性读取 vs 快照日志**：Error Info 等普通日志可重复读取且内容不变；预留通知日志读一条少一条。
- **控制器本地作用域**：日志按控制器独立维护，跨控制器看不到对方的预留通知事件。
- **跨断电补全**：预留通知日志在控制器重置后 `LPC` 会清零；若需跨断电审计预留事件时间线，需结合 **Persistent Event Log（持久事件日志）**。

---

## 关联与引申概念

- **Reservation（命名空间预留）**：多主机共享同一个命名空间时用于协调排他访问锁的机制，抢占或释放预留时会产生本日志事件。
- **AER（Asynchronous Event Reporting，异步事件报告）**：当预留通知日志中有新事件入队时，控制器可通过 AER 通知主机前来读取排空。
- **Persistent Event Log（持久事件日志）**：将重要事件（包括 Reservation Notification）持久化记录在介质上的日志机制，弥补通知日志在重置后丢失或溢出的不足。
- **相关文档**：
  - [命名空间预留生命周期](namespace-reservation-lifecycle.md) — 触发预留通知的具体命令与状态转换
  - [日志页获取](log-page-retrieval.md) — `Get Log Page` 通用命令机制
  - [持久事件日志](persistent-event-log.md) — 跨断电事件时间线重建
  - [异步事件报告](asynchronous-event-reporting.md) — AER 通知与触发读取

## 规范依据

- [预留通知日志创建、破坏性读取、空队列与溢出计数，PDF p. 301](../_source/pages/page-301.md)
- [Figure 290 字段布局（LPC/RNLPT/NALP/NSID）与计数器回绕，PDF p. 302](../_source/pages/page-302.md)
- [Reservation Notification Mask（FID 82h）与 Persistence（FID 83h），PDF pp. 424-425](../_source/pages/page-424.md)
