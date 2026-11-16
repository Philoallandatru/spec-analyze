# 关联与命令生命周期

**关联（Association）** 是一个主机与一个控制器之间排他的通信关系，囊括该控制器的 Admin Queue 与全部 I/O Queue；在关联建立后，每条命令都会经历"提交 → 候选 → 处理 → 完成"四个阶段。[PDF pp. 28-29](../_source/pages/page-028.md)

> **类比理解**：把关联想成酒店的专属 VIP 通行证——客人在前台出示证件（Host NQN / Subsystem NQN / Host ID）拿到房卡（Controller ID）即建立关联，同一张房卡同一时刻只服务一位客人；后续去健身房或会议室（I/O Queue）必须核对同一套身份。命令则像投进前台任务箱的工单，经过登记、排队、处理，最后把回执（CQE）放进收件箱才算真正完成。

---

## 生命周期拓扑

```text
关联生命周期（Fabrics）
[未关联]
    |  Connect Admin Queue (建立关联 + 设置 Host ID / Keep Alive Timeout)
    v
[与唯一主机已关联] <─── 同一主机可继续 Connect I/O Queue（继承身份契约）
    |
    +-- 控制器 Shutdown / Controller Level Reset
    +-- Admin Queue 传输连接断开
    `-- 不支持单独删除 I/O Queue 时，任一 I/O 连接断开
    v
[未关联]

命令生命周期
[主机构造 SQE]
    |  Memory-based：SQ Tail Doorbell 推进越过该槽位
    |  Message-based：主机将 Capsule 加入 SQ
    v
[已提交 submitted] ──(传输与准备就绪)──> [候选 candidate] ──(控制器处理)──> [已完成 completed]
                                                                        (CQE 状态写入 + 入队 CQ)
```

---

## 关联建立、身份契约与终止

| 阶段 | 核心契约与不变式 |
|---|---|
| **排他性** | 一个控制器同一时刻最多只与一个主机存在关联；只有该主机可以连接其 I/O Queue。[PDF p. 58](../_source/pages/page-058.md) |
| **Admin Connect** | 在 Fabrics 上，连接 Admin Queue 即建立关联，并同时设置当前 `Host Identifier` 与 `Keep Alive Timeout`。[PDF pp. 58, 476](../_source/pages/page-058.md) |
| **I/O Connect 一致性** | 任何 I/O Connect 必须发生在 Admin 关联已建立且控制器已使能之后，且必须沿用同一子系统端口、同一传输类型/地址、同一 `Host NQN`、同一 `Subsystem NQN`、同一 `Controller ID` 与兼容的 `Host Identifier`（如非零需匹配）；重复使用已占用 Queue ID 属于序列错误。[PDF pp. 58, 474](../_source/pages/page-058.md) |
| **NVM Set 绑定** | I/O Connect 还可将自身 SQ 关联到已宣告的 NVM Set。[PDF pp. 476-477](../_source/pages/page-476.md) |
| **带内认证门控** | 若 Connect 要求带内认证（in-band authentication），新建队列在认证完成前**仅接受** `Authentication Send/Receive` 命令。[PDF pp. 114-115](../_source/pages/page-114.md) |
| **无显式断联命令** | NVMe 没有专门的"终止关联"命令；`Disconnect` 只删除指定的 I/O Queue。仅当双方支持单 I/O 队列删除时，它才不影响关联；否则只能通过队列删除或传输连接终止来结束关联。[PDF pp. 58-59, 115-117](../_source/pages/page-058.md) |

### 控制器分配（Controller ID）策略

- **动态子系统**：主机必须在 Connect 中将 `CNTLID` 设为 `FFFFh`（请求任意可用动态控制器）。[PDF pp. 59, 473-474](../_source/pages/page-059.md)
- **静态子系统**：可指定特定控制器 ID，或使用 `FFFEh`（任意可用静态控制器），但**拒绝** `FFFFh`。静态分配预期持久，但控制器空闲时仍可能因实现原因被回收。[PDF pp. 59, 473-474](../_source/pages/page-059.md)
- **哨兵保留区间**：成功分配的 `CNTLID` 绝不会落在 `FFF0h..FFFFh`（其中 `FFFDh` 用于跨子系统分散命名空间）。[PDF pp. 473-474](../_source/pages/page-473.md)

> **常见排错（重连报错 `Invalid Field in Command`）**：网络瞬断重连时，若子系统已回收旧控制器并分配了新 `CNTLID`，主机继续沿用旧 `CNTLID` 发 I/O Connect 就会因身份不一致被拒。重连应使用 `FFFEh`/`FFFFh`，并以 Connect 实际返回的新 `CNTLID` 为准。

---

## 命令状态与传输恢复语义

- **完成（Completed）的严格定义**："完成"不等于内部执行结束，而是**处理已结束 + 状态已写入 CQE + 该 CQE 已发布到对应的 Completion Queue**，三者缺一不可。[PDF p. 29](../_source/pages/page-029.md)
- **Capsule（胶囊）**：NVMe over Fabrics 的基本交换单位，承载命令或响应，并可附带数据与 SGL。[PDF p. 29](../_source/pages/page-029.md)
- **通信丢失与幂等性（Idempotent）**：
  - 仍在传输或未完成时发生通信丢失，必须走"通信丢失恢复"流程，**不能盲目重提交**。[PDF pp. 115-117](../_source/pages/page-115.md)
  - **幂等命令**是指：重提交一次或多次、且**期间没有其他命令介入**时，控制器终态与返回结果保持一致。"无中间命令介入"是判断能否安全重试的关键前提。[PDF p. 31](../_source/pages/page-031.md)

---

## 易混概念对比

| 易混概念 | 实际区别 |
|---|---|
| **提交（Submitted） vs 候选（Candidate）** | 提交 = Doorbell 推进或 Capsule 入队；候选 = 已传输就绪、等待控制器挑选执行 |
| **关联终止 vs 队列删除** | `Disconnect` 只删单个 I/O 队列；终止关联需 Shutdown、CLR、Admin 断开或无单队列删除能力时的 I/O 断开 |
| **`FFFEh` vs `FFFFh`** | `FFFEh` = 任意可用静态控制器；`FFFFh` = 任意可用动态控制器；两者均不可作为 Connect 成功分配的返回值 |
| **Host NQN vs Host ID** | `Host NQN` 是可读字符串名称；`Host ID` 是 NVMe 内部使用的二进制标识符（如 128-bit） |
| **幂等 vs 盲目重试** | 幂等要求"期间无其他命令介入"；有中间命令或状态不明时盲目重试可能破坏数据一致性 |

---

## 关联与引申概念

- **Admin Queue / I/O Queue**：Admin Queue（ID=0）负责管理与建联，I/O Queue 负责数据读写；所有队列共享同一关联的身份契约。
- **Doorbell（门铃寄存器）**：Memory-based 传输（PCIe）下，主机写 SQ Tail Doorbell 通知控制器有新命令提交。
- **Capsule & SGL**：Fabrics 传输下替代内存环的交换单元，SGL（Scatter Gather List）用于描述分散的内存缓冲区。
- **Keep Alive**：由 Admin Connect 设定超时时间，用于检测主机与控制器之间的关联是否仍然存活。
- **Controller Level Reset（CLR）**：控制器级复位，会立即清理状态并终止当前关联。
- **延伸阅读**：
  - [controller.md](controller.md) — 控制器模型与静态/动态分类
  - [queue-pair.md](queue-pair.md) — Admin 与 I/O 队列对的创建与销毁
  - [command-sets.md](command-sets.md) — 关联建立后可协商使用的命令集
  - [fabrics-discovery-and-authentication.md](fabrics-discovery-and-authentication.md) — Fabrics 发现与带内认证流程

---

## 规范依据

- [关联与 candidate command 定义，PDF p. 28](../_source/pages/page-028.md)
- [Capsule、提交、完成定义，PDF p. 29](../_source/pages/page-029.md)
- [幂等命令定义，PDF p. 31](../_source/pages/page-031.md)
- [Fabrics 关联建立与队列连接身份约束，PDF p. 58](../_source/pages/page-058.md)
- [关联终止与控制器分配请求，PDF p. 59](../_source/pages/page-059.md)
- [Message-based Connect 身份、认证与 I/O 队列删除范围，PDF pp. 114-117](../_source/pages/page-114.md)
- [Connect 控制器分配与身份校验，PDF pp. 473-477](../_source/pages/page-473.md)
