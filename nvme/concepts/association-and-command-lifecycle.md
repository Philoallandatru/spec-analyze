# 关联与命令生命周期（Association and Command Lifecycle）

## 一句话说明

关联（Association）是一个主机与一个控制器之间的排他通信关系：连上 Admin 队列就建立关联，再连 I/O 队列必须保持身份一致；命令的生命周期是"提交 → 候选 → 处理 → 完成"四步。

## 生活化类比

把关联想成**酒店 VIP 通道**：

- **建联** = 客人到前台出示身份证（Host NQN + Subsystem NQN + Host ID），前台开房（绑定 Controller ID）并发一张房卡——这张卡就是关联。
- **排他** = 房卡只能你一个人用，别人无法同时开你的房间（同一控制器同一时刻只能关联一个主机）。
- **后续连接** = 用同一张身份去开会议室、健身房（I/O 队列），前台核对"身份证 + 房号"必须一致。
- **退联** = 退房（Shutdown）、断电（CLR）、或钥匙卡失联（传输断开）。

命令的生命周期就像"前台任务"：客人填单（写 SQE）→ 投到任务箱（敲 Doorbell / 加 capsule）→ 前台处理 → 结果回执入收件箱（CQE）。

## 工作流程

```text
  关联生命周期
  [未关联]
      |  Connect Admin Queue
      v
  [与一个主机关联]  ←──── 同一主机可连 I/O Queue
      |
      |  Shutdown / CLR / Admin 断开 / (无单队列删除能力时) I/O 断开
      v
  [未关联]

  命令生命周期
  [主机构造命令]
      |  内存: SQ Tail Doorbell 推进
      |  消息: 把 capsule 加入 SQ
      v
  [已提交 submitted]
      |  传输 + 准备就绪
      v
  [候选 candidate]
      |  控制器处理
      v
  [完成 completed]  ← CQE 状态已更新 + CQE 已写入 CQ
```

简化说明：命令"完成"= 处理完 + CQE 状态写入 + CQE 已发到 CQ，三件事都做才算。

## 初学者案例

**场景：NVMe-oF 客户端重连后命令报"Invalid Field in Command"**

1. 客户端第一次 Connect 用 `CNTLID=0x1234` 关联到控制器；工作正常。
2. 网络瞬断，客户端重连，仍用 `CNTLID=0x1234`。
3. 子系统已把控制器 `1234h` 释放；这次 Connect 实际返回的是新控制器 `5678h`。
4. 客户端继续用旧身份发命令 → 后续 I/O Connect 因身份不一致报 `Invalid Field in Command`。
5. 解决：重连时用 `CNTLID=FFFEh`（任意可用静态控制器）或 `FFFFh`（任意动态控制器），**使用 Connect 返回的真实 CNTLID**。

> 排错口诀：**关联的身份四件套（Host NQN / Subsystem NQN / Controller ID / Host ID）必须一致**；重连后一定用新返回的 ID。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 关联定义 | 一个主机与一个控制器之间的排他通信关系 |
| 包含范围 | 该控制器的 Admin Queue + 所有 I/O Queue |
| 关联排他 | 一个控制器同一时刻只与一个主机关联 |
| 建立方式（Fabrics） | Connect Admin Queue 即建立关联 |
| Connect 必填 | Host NQN + Subsystem NQN + Host Identifier + 可指定 Controller ID |
| 后续 I/O Connect 一致 | 同一子系统端口 + 同一传输类型/地址 + 同一 Host NQN + 同一 Subsystem NQN + 同一 Controller ID + 兼容 Host ID |
| 错误使用 | 重复使用 Queue ID = 序列错误 |
| 关联终止条件 | ① Shutdown；② Controller Level Reset；③ Admin 传输断开；④ 不支持单队列删除时任一 I/O 断开 |
| 没有显式断联命令 | Disconnect 只删除 I/O Queue；不终止关联 |
| 单 I/O 队列删除 | 双方支持时 Disconnect 只影响该 I/O 队列；否则可能终止关联 |
| 传输丢失恢复 | 传输丢失 + 未完成命令需走"通信丢失恢复"流程，**不能盲目重发** |
| 静态 vs 动态 | 静态控制器 ID 期望持久但实现可回收空闲控制器；动态用 `FFFFh` |
| 静态用 `FFFEh` | 静态子系统可接受 `FFFEh`（任意可用静态），**但拒绝 `FFFFh`** |
| 动态用 `FFFFh` | 动态子系统必须用 `FFFFh`；返回任意可用控制器 |
| 哨兵值 | Connect 不会返回 `FFF0h–FFFFh` 范围的 Controller ID |
| 队列 I/O 前置 | I/O Queue Connect 前必须已有 Admin 关联且控制器已使能 |
| 身份继承 | I/O Connect 的 Host NQN/Subsystem NQN/Controller ID 继承 Admin Connect |
| Host ID 兼容 | 后续 I/O Connect 的 Host ID 必须兼容 Admin Connect 时的值 |
| NVM Set 关联 | I/O Connect 可把其 SQ 关联到某 NVM Set |
| Keep Alive | Admin Connect 设置当前 Host Identifier 和 Keep Alive Timeout |
| 传输存在 | 消息控制器下，传输连接在 Connect 之前已存在；Admin Connect 创建关联 |
| 认证门控 | Connect 后若需 in-band 认证，新队列**只接受** Authentication Send/Receive |
| 命令完成定义 | 处理完毕 + CQE 状态已写入 + CQE 已发到 CQ |
| Capsule | Fabrics 下信息交换单位 = 命令或响应 + 可选数据 + SGL |
| 幂等命令 | 同一子系统终态 + 同一结果在无中间命令时重发；定义里"无中间命令"是关键 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 关联 vs 队列 | 关联是"主机-控制器"关系；队列是命令/完成通道 |
| Admin Queue vs I/O Queue | Admin 必有一个（标识 0）；I/O 可有多个 |
| 关联终止 vs 队列删除 | Disconnect 只删队列；终止关联需满足 4 个条件之一 |
| 静态 vs 动态 | 静态保留 ID/Feature；动态每次关联重新分配 |
| FFFEh vs FFFFh | FFFEh = 任意静态；FFFFh = 任意动态；两者都不可用于 Connect 返回 |
| FFFDh vs FFF0h–FFFCh | FFFDh = 跨子系统分散命名空间；FFF0h–FFFCh = 保留 |
| 提交 vs 候选 | 提交 = 命令已投放；候选 = 传输就绪、控制器可处理 |
| 候选 vs 完成 | 候选 = 等待处理；完成 = 处理完 + CQE 已发 |
| 显式断开 vs Disconnect | 没有"显式断联"命令；Disconnect 只是删 I/O 队列 |
| 通信丢失 vs 命令失败 | 通信丢失 ≠ 命令失败；前者走恢复流程，不能盲目重发 |
| 幂等 vs 重试 | 幂等 = 重发效果一致；重试不等于幂等（可能有中间命令） |
| Host ID vs Host NQN | Host NQN 是名字（字符串）；Host ID 是标识符（128-bit 等） |

## 进阶细节

- **关联定义**（规范 1.5.x / 3.1.2）：一个主机与一个控制器之间的排他通信关系；包含 Admin Queue + 所有 I/O Queue。
- **关联建立**（规范 3.1.2 / 6.3）：Fabrics Connect 命令建立关联，传入 Host NQN、NVM Subsystem NQN、Host Identifier、可指定 Controller ID。
- **I/O Connect 一致性要求**（规范 6.3 / 5.2.6）：
  - 同一子系统端口、NVMe Transport 类型、NVMe Transport 地址
  - 同一 Host NQN
  - 同一 NVM Subsystem NQN
  - 同一 Controller ID
  - 同一 Host Identifier，或 Host ID = 0h（如支持）
- **关联终止条件**（规范 3.1.3）：
  - ① 控制器 Shutdown（3.6.2）
  - ② Controller Level Reset
  - ③ NVMe Transport 连接对 Admin Queue 断开
  - ④ 不支持单 I/O 队列删除时任一 I/O Queue 断开
- **Connect 的 Controller ID 选择**（规范 6.3 / 5.2.6）：
  - 动态子系统：必须 `FFFFh`（任意可用）；返回任意动态控制器
  - 静态子系统：可指定特定 ID；或 `FFFEh`（任意可用静态）
  - 静态子系统**拒绝** `FFFFh`
  - 成功分配**绝不返回** `FFF0h–FFFFh`
- **Connect 前的传输存在**（规范 4.1.3）：消息控制器下，传输连接在 Connect 之前已存在；Admin Connect 创建关联。
- **Admin Connect 设置**（规范 6.3）：当前 Host Identifier + Keep Alive Timeout。
- **I/O Connect 增强**（规范 6.3）：可把 SQ 关联到某 NVM Set；身份字段继承 Admin Connect。
- **认证门控**（规范 4.1.3）：Connect 后若需 in-band 认证，新队列**只接受** Authentication Send/Receive 命令，直到认证完成。
- **命令投放语义**（规范 1.5.19 / 4.1）：
  - 内存映射：SQ Tail Doorbell 写跨过该命令槽位 = 提交
  - 消息：主机把 capsule 加入 SQ = 提交
- **命令完成定义**（规范 1.5.18 / 4.1）：
  - 处理完毕
  - CQE 状态已更新
  - CQE 已写入关联 CQ
  - 三件事都做才算"完成"
- **Capsule**（规范 1.5.15）：NVMe over Fabrics 的信息交换单位 = 命令/响应 + 可选数据 + 可选 SGL。
- **单 I/O 队列删除的边界**（规范 4.1.4 / 3.3.2.4）：双方支持时 Disconnect 只影响该 I/O 队列；否则可能终止关联。
- **传输丢失的恢复**（规范 4.1.5）：传输丢失 + 未完成命令需走"通信丢失恢复"流程；**不能盲目重发**。
- **幂等命令定义**（规范 1.5.x）：同一子系统终态 + 同一结果在**无中间命令**时重发；定义里"无中间命令"是关键条件。
- **Host NQN vs Host ID**：Host NQN 是字符串（人/系统可读），Host ID 是标识符（NVMe 内部引用）；不同主机可同 Host NQN 但 Host ID 必须不同。

## 规范依据

- [Association 与 candidate-command 定义，PDF 第 28 页](../_source/pages/page-028.md)
- [Capsule、submission、completion 定义，PDF 第 29 页](../_source/pages/page-029.md)
- [Fabrics 关联建立与队列身份，PDF 第 58 页](../_source/pages/page-058.md)
- [关联终止与控制器分配请求，PDF 第 59 页](../_source/pages/page-059.md)
- [Connect 控制器分配与身份验证，PDF 第 473-477 页](../_source/pages/page-473.md)

## 相关阅读

- [controller.md](controller.md) - 关联基于控制器与主机建立
- [queue-pair.md](queue-pair.md) - Admin 队列建联与终止
- [command-sets.md](command-sets.md) - 关联建立决定可发命令集
- [fabrics-discovery-and-authentication.md](fabrics-discovery-and-authentication.md) - 认证是建立关联的前提
