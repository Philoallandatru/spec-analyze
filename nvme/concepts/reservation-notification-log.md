# Reservation Notification Log（预留通知日志）

## 概述

预留通知日志（Reservation Notification Log）是一个特殊的日志队列，用于记录 NVMe 命名空间上发生的预留变更事件。该日志具有以下特点：

- **控制器本地存储**：每个控制器（Controller）维护自己的日志队列
- **破坏性读取**：每次读取操作会返回最旧的通知记录，并将其从队列中移除
- **事件过滤**：只记录未被屏蔽的预留变更事件
- **作用范围**：仅涵盖挂接到该控制器的命名空间（Namespace）

通过 Get Log Page 命令并指定日志标识符（Log Identifier, LID）为 `80h` 来读取此日志。

**规范参考**：[PDF pp. 301-302](../_source/pages/page-301.md)

---

## 工作原理

### 队列结构示意图

下图展示了预留通知日志的工作机制：

```text
未屏蔽的预留变更事件
          |
          v
   +--------+----------+----------+       Get Log Page LID 80h
   | LPC n  | LPC n+1  | LPC n+2  |  --------------------------> 返回并移除最旧的记录
   +--------+----------+----------+       
          ^
          |
    队列溢出时，丢失的变更会使
    最新入队记录的计数器前进
```

**说明**：
- 事件按发生顺序入队，每个事件都有递增的日志页计数器（Log Page Count, LPC）
- 读取时返回队列头部（最旧）的记录，同时将其移除
- 当队列满时，新事件会覆盖旧事件，但会更新计数器以指示丢失了多少记录

**规范参考**：[PDF pp. 301-302](../_source/pages/page-301.md)

---

## 核心字段说明

### 日志记录结构

每条预留通知记录包含以下关键字段：

| 字段名称 | 说明 |
|---------|------|
| **LPC**（Log Page Count）<br>日志页计数器 | • 64 位序列号，用于标识每个通知事件<br>• 控制器级别重置（Controller Level Reset）时清零<br>• 从 `1h` 开始递增，达到全 `FFFFFFFFFFFFFFFFh` 后回绕到 `1h`<br>• `0h` 保留用于表示空结果（无可用记录） |
| **RNLPT**（Reservation Notification Log Page Type）<br>通知类型 | 指示事件类型：<br>• Registration Preempted（注册被抢占）<br>• Reservation Released（预留被释放）<br>• Reservation Preempted（预留被抢占） |
| **NALP**（Number of Additional Log Pages）<br>额外日志页数量 | • 指示读取当前记录后还有多少条未读记录<br>• 最大值为 255（饱和计数） |
| **NSID**（Namespace Identifier）<br>命名空间标识符 | 指示发生预留变更的命名空间 ID |
| **空队列标识** | 当队列为空时，返回全零的 64 字节日志页 |

**规范参考**：[PDF p. 302](../_source/pages/page-302.md)

### 溢出处理机制

当队列容量不足时，系统采用以下策略：

1. **序列号持续递增**：即使无法将事件入队，LPC 序列号仍会递增
2. **丢失检测**：通过比较连续读取的 LPC 值，主机可以检测到是否有记录丢失
3. **最新记录更新**：队列中最新的记录会更新其 LPC，反映包括已丢失事件在内的所有事件计数

**举例说明**：
- 假设队列当前有 LPC 为 100、101、102 的三条记录（队列已满）
- 新事件发生（LPC 应为 103），但队列无空间
- 系统移除 LPC 100 的记录，将 LPC 103 的事件入队
- 主机读取时会发现 LPC 从 102 跳到 103，可以推断没有丢失记录
- 若跳跃幅度更大（如从 102 到 110），则表明丢失了中间的 7 个事件

**规范参考**：[PDF pp. 301-302](../_source/pages/page-301.md)

---

## 相关特性与配置

### 1. Reservation Notification Mask（预留通知屏蔽，Feature ID `82h`）

**功能说明**：
控制哪些类型的预留变更事件会被记录到日志中。

**可屏蔽的事件类型**：
| 事件类型 | 说明 |
|---------|------|
| Registration Preempted<br>注册被抢占 | 主机的注册被其他主机抢占 |
| Reservation Released<br>预留被释放 | 预留被主动释放 |
| Reservation Preempted<br>预留被抢占 | 预留被其他主机抢占 |

**配置特点**：
- **作用范围**：每个命名空间独立配置
- **广播设置**：支持 Broadcast Set 操作，可一次性配置所有已挂接且具备预留能力的命名空间
- **广播读取**：不支持 Broadcast Get 操作
- **屏蔽机制**：在事件产生时阻止其创建，而非事后过滤

**规范参考**：[PDF p. 424](../_source/pages/page-424.md)

### 2. Reservation Persistence（预留持久化，Feature ID `83h`）

**功能说明**：
控制预留信息在断电后是否保留（Persist Through Power Loss, PTPL）。

**配置选项**：
| PTPL 值 | 断电后的行为 |
|---------|-------------|
| **1**（启用） | 保留预留（Reservations）和注册者（Registrants）信息 |
| **0**（禁用） | 释放预留并清除注册者信息 |

**配置特点**：
- **作用范围**：每个命名空间独立配置
- **广播设置**：支持 Broadcast Set，影响所有已挂接且具备预留能力的命名空间
- **广播读取**：不支持 Broadcast Get 操作
- **保存限制**：此特性不可被保存（Non-saveable，不支持 Save 操作）

**规范参考**：[PDF pp. 424-425](../_source/pages/page-424.md)

### 3. 与其他日志的区别

预留通知日志与普通日志的重要区别：

| 特性 | 预留通知日志 | 普通快照式日志 |
|------|-------------|---------------|
| **读取效果** | 破坏性读取（移除记录） | 非破坏性读取（保留数据） |
| **NALP 字段** | 指示是否还有更多记录待读取 | 通常不需要 |
| **使用场景** | 主机需持续排空队列 | 主机可重复读取相同数据 |

**NALP**（Number of Additional Log Pages）字段告诉主机是否应该继续发起读取请求，直到队列清空。

**规范参考**：[PDF pp. 301-302](../_source/pages/page-301.md)

---

## 典型使用流程

### 主机端操作步骤

1. **配置屏蔽**：根据需要设置 Reservation Notification Mask（Feature `82h`）
2. **配置持久化**：根据需要设置 Reservation Persistence（Feature `83h`）
3. **读取日志**：
   - 发送 Get Log Page 命令，LID = `80h`
   - 处理返回的通知记录
4. **检查 NALP**：
   - 如果 NALP > 0，继续读取
   - 如果 NALP = 0，队列已空
   - 空队列返回全零 64 字节页
5. **检测丢失**：比较连续记录的 LPC 值，发现序列号跳跃则表明有记录丢失

### 注意事项

- 只有挂接到当前控制器的命名空间的事件才会出现在该日志中
- 已屏蔽的事件类型不会生成日志记录
- 控制器重置会清零 LPC 计数器，导致序列号重新开始
- 队列容量有限，重要事件应及时读取以避免丢失

---

## 规范引用

本文档内容基于以下 NVMe 规范章节：

- [队列创建、破坏性读取、空行为与溢出计数，PDF p. 301](../_source/pages/page-301.md)
- [渲染的图 290 字段布局与计数器回绕，PDF p. 302](../_source/pages/page-302.md)
- [Reservation Notification Mask 特性，PDF p. 424](../_source/pages/page-424.md)
- [Reservation Persistence 特性，PDF pp. 424-425](../_source/pages/page-424.md)
