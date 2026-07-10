# Keep Alive（保活机制）

Keep Alive 是 NVMe 协议中的一项重要机制，用于检测主机（Host）与控制器（Controller）之间的通信故障。它的工作原理类似于看门狗定时器（Watchdog Timer），每个控制器独立维护一个定时器来监控通信状态。

## 核心概念

**KATT**（Keep Alive Timeout Time，保活超时时间）：控制器当前使用的超时时间值。

**KAS**（Keep Alive Support，保活支持）：控制器通告的一个非零值，表明该控制器支持 Keep Alive 功能。

**KATO**（Keep Alive Timeout，保活超时配置）：主机设置的超时时间配置值。

不同的传输协议（如 PCIe、RDMA、TCP）会定义各自允许的取值范围，以及该特性是否必须启用。只有通告了非零 `KAS` 值的控制器才支持 Keep Alive 定时器和 Keep Alive 命令。

**规范参考**：[PDF pp. 146-147](../_source/pages/page-146.md)

## 工作原理

### 状态转换模型

下面的状态图展示了 Keep Alive 定时器的激活条件和运行流程：

```text
未激活状态 ──┬─ 满足激活条件 ─→ 激活状态
            │   (启用 + 就绪 + 未关闭 + KATO>0)
            │                         │
            │                         │ 监测活动信号
            │                         │ (规则取决于工作模式)
            │                         ↓
            └─── 退出条件 ────── [监测间隔]
                (禁用/未就绪/关闭)      │
                                       │ 超时期间无有效活动
                                       ↓
                                  [超时处理流程]
```

**说明**：这是一个解释性的状态视图，综合展示了激活条件判断和两种看门狗模式，并非规范中的正式状态图。

**规范参考**：[PDF pp. 147-149](../_source/pages/page-147.md)

## 配置与激活

### 如何配置超时时间

主机可以通过以下两种方式设置 `KATO` 值：

1. **Fabrics Connect**（Fabrics 连接）：在建立 Fabrics 连接时配置
2. **Set Features 命令**：通过特性设置命令动态配置

### 配置规则

| 场景 | 行为 | 说明 |
|------|------|------|
| 设置值为 0 | 可能被拒绝 | 如果传输协议强制要求 Keep Alive，则无法用 0 禁用 |
| 设置值超过上限 | 被拒绝 | 返回 "Keep Alive Timeout Invalid" 错误，控制器端的值不变 |
| 设置值低于下限 | 自动调整 | 非零值会被提升到适用的最小值，然后向上取整到 `KAS` 粒度 |
| 查询生效值 | Get Features | 使用 Get Features 命令可以获取实际生效的值 |

**重要提示**：
- Set Features 命令中的 `KATO` 以**毫秒**为单位
- 不同传输协议的默认值不同：
  - 不要求该定时器的传输默认值为 0
  - RDMA 和 TCP 传输的默认值为 **120,000 ms**（120 秒），会向上取整到 `KAS` 粒度
  - 其他传输的具体要求由相应的传输绑定规范定义

**规范参考**：[PDF p. 147](../_source/pages/page-147.md), [PDF p. 403](../_source/pages/page-403.md)

### 定时器激活条件

控制器的 Keep Alive 定时器只有在**同时满足**以下所有条件时才会激活：

| 寄存器/字段 | 要求值 | 含义 |
|------------|--------|------|
| `CC.EN` | 1 | Controller Enable，控制器已启用 |
| `CSTS.RDY` | 1 | Controller Status Ready，控制器已就绪 |
| `CC.SHN` | 00b | Shutdown Notification，未处于关闭过程 |
| `CSTS.SHST` | 00b | Shutdown Status，关闭状态为正常 |
| 生效的 KATO | 非零 | 配置的超时时间不为 0 |

当定时器激活时，会将其初始化为 `KATT` 值。在定时器激活期间，主机应该为 Keep Alive 命令预留 Admin 提交队列（Admin Submission Queue）的空间。

**规范参考**：[PDF p. 147](../_source/pages/page-147.md)

## 工作模式

Keep Alive 支持两种工作模式，由 `TBKAS`（Traffic Based Keep Alive Support）位决定：

### 模式对比

| 特性 | Command Based 模式<br>（基于命令，`TBKAS=0`） | Traffic Based 模式<br>（基于流量，`TBKAS=1`） |
|------|----------------------------------------|----------------------------------------|
| **活动证据** | Keep Alive 命令成功完成 | 在一个监测间隔内，控制器获取到任意 Admin 或 I/O 命令 |
| **控制器截止时间** | 自上次启动/重启后经过 `KATT` 后超时 | 检查连续的 `KATT` 间隔 |
| **主机发送频率建议** | 按 `KATT/2` 频率发送 | 按 `KATT/4` 频率检查；正常完成的流量可能抑制 Keep Alive |
| **最坏情况延迟** | 在仅命令规则下约为 `KATT` | 最长 `2 × KATT` |

### Command Based 模式（基于命令）

在这种模式下：

1. **控制器行为**：每当成功完成一个 Keep Alive 命令，或成功完成一个设置非零 KATO 的 Set Features 命令时，控制器会重启定时器
2. **主机行为**：无论控制器使用哪种模式，主机都可以采用基于命令的行为
3. **超时检测**：主机应该将"提交 Keep Alive 命令后 `KATT` 时间内未收到完成响应"视为主机端检测到的超时

**规范参考**：[PDF pp. 147-149](../_source/pages/page-147.md)

### Traffic Based 模式（基于流量）

这种模式更加灵活，可以利用正常的命令流量来维持连接：

1. **前提条件**：只有当控制器也使用 Traffic Based 模式时，主机才可以使用这种模式
2. **监测机制**：
   - 控制器在每个间隔边界检查是否有命令活动
   - 如果在当前间隔内获取（fetched）到任何 Admin 或 I/O 命令，则阻止超时并开始新的间隔
   - 如果没有命令活动，则发生超时
3. **时序特性**：由于命令可能在检查刚结束后被获取，检测可能延迟到下一次检查，导致最长 `2 × KATT` 的延迟

**规范参考**：[PDF pp. 149-150](../_source/pages/page-149.md)

#### 时序示例

下图展示了 Traffic Based 模式下可能出现的最坏情况：

```text
检查点 1       最后一条命令      检查点 2                检查点 3
   |────────── KATT ──────────|────────── KATT ──────────|
                       ↑ 被获取      ✓ 检测到流量           ✗ 无流量
                                      继续                   超时
```

**说明**：这个精简的时序图重建自规范中的 Figure 90，保留了导致最坏情况发生的两个相邻 `KATT` 窗口。当命令在检查点 1 刚过后被获取时，需要等到检查点 2 才能检测到活动，然后在检查点 3 才检测到超时。

**规范参考**：[PDF p. 150](../_source/pages/page-150.md)

## 超时处理

### 控制器端处理流程

当控制器检测到 Keep Alive 超时后，会在 `CQT`（Completion Queue Timeout）时间内执行以下操作：

1. **记录错误**：在 Error Information Log（错误信息日志）中记录一条条目，错误类型为 "Keep Alive Timeout Expired"
2. **停止处理**：停止命令处理
3. **设置故障状态**：将 `CSTS.CFS`（Controller Fatal Status，控制器致命状态）设置为 1

### 特殊处理（基于消息的控制器）

对于使用消息传递机制的控制器（如 Fabrics），还会执行额外操作：

- **终止连接**：终止相关关联（Association）的传输连接
- **断开关联**：断开该关联
- **重新建立**：之后可以通过新的 Admin Queue Connect 建立新的关联

**规范参考**：[PDF p. 151](../_source/pages/page-151.md)

### 主机端处理建议

如果主机在仍有未完成命令时检测到超时，应该：

1. **遵循通信丢失恢复规则**：按照 communication-loss recovery 的标准流程处理
2. **谨慎提交依赖命令**：通过另一个控制器提交依赖于先前命令的工作之前，要充分考虑先前的命令可能仍然会与后续命令产生交互

**规范参考**：[PDF pp. 148, 150-151](../_source/pages/page-148.md)

## 与其他功能的关系

### 队列依赖

Keep Alive 机制运行在控制器上，在 Command Based 模式下会使用 Admin Queue（管理队列）来发送 Keep Alive 命令。

**规范参考**：[PDF pp. 146-147](../_source/pages/page-146.md)

### 状态耦合

定时器的激活与控制器的启用（enable）和就绪（readiness）状态紧密耦合，在控制器关闭（shutdown）期间会被抑制。

**规范参考**：[PDF p. 147](../_source/pages/page-147.md)

### 错误处理

控制器检测到的 Keep Alive 超时会成为：
- **致命状态事件**（fatal-status event）
- **错误恢复触发点**（error-recovery event）
- **关联销毁原因**（对于 Fabrics 传输）

**规范参考**：[PDF p. 151](../_source/pages/page-151.md)

## 规范引用

本文档内容基于 NVMe 规范的以下章节：

- [能力标识、传输策略、配置与激活，PDF pp. 146-147](../_source/pages/page-146.md)
- [Command Based 控制器与主机行为，PDF pp. 147-149](../_source/pages/page-147.md)
- [Traffic Based 行为与 Figure 90 时序图，PDF pp. 149-150](../_source/pages/page-149.md)
- [超时清理流程，PDF p. 151](../_source/pages/page-151.md)
