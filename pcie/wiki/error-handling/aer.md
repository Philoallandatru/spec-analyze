# AER（Advanced Error Reporting，高级错误报告）

**AER（Advanced Error Reporting，高级错误报告）** 是 PCIe 规范定义的一套标准化错误检测、记录与上报机制。如果把 PCIe 链路比作一条高速公路，AER 就像是安装在每个路口的**“黑匣子 + 分级报警系统”**：轻微颠簸（可纠正错误）由硬件自动修复并记入台账；严重事故（不可纠正错误）则立即抓拍现场“案发包头”（Header Log），并通过专用报警报文逐级上报给 **Root Complex（根复合体，CPU 侧 PCIe 总线控制器）**，交由操作系统内核决定如何恢复。

---

## 为什么需要 AER

在早期 PCI 总线时代，设备出错通常只能通过简陋的 `PERR#` / `SERR#` 引脚粗暴拉低报警，系统既不知道**谁出的错**，也不知道**出了什么错**。AER 通过标准化的配置空间寄存器和报文机制解决了这一痛点：

- **精准定责**：通过错误报文中的 **Requester ID（BDF：Bus/Device/Function）**，Root Complex 能精确锁定出错的设备。
- **现场留证**：通过 **Header Log** 寄存器保存导致错误的 **TLP（Transaction Layer Packet，事务层数据包）** 前 4 个双字（DW），方便事后定位。
- **分级响应**：将错误划分为三个严重等级，避免“小病大治”（因轻微误码重启整机）或“大病漏治”（数据静默损坏）。

| 错误等级 | 英文名称 | 严重性 | 硬件/软件处理方式 | 典型例子 |
|---------|---------|-------|------------------|---------|
| **可纠正错误** | Correctable Error | 低 | 硬件自动纠正（如链路层重传），不丢数据；记入日志供链路健康监控 | Receiver Error、Bad TLP、Replay Timer Timeout |
| **不可纠正非致命错误** | Uncorrectable Non-Fatal | 中 | 无法自动纠正，可能影响单次事务，但链路仍可用；驱动可尝试恢复 | Completion Timeout、Poisoned TLP、Unsupported Request |
| **不可纠正致命错误** | Uncorrectable Fatal | 高 | 链路或设备内部状态已不可靠；需停止新事务并执行复位（FLR / Hot Reset） | Data Link Protocol Error、Surprise Down、Malformed TLP |

---

## AER 寄存器布局（Extended Capability `0x0001`）

AER 位于 PCIe 扩展配置空间（偏移 `0x100` 及以上，通过 Extended Capability 链表串联，Cap ID 为 `0x0001`）。其中 `RW1C`（Read/Write 1 to Clear）表示软件向对应位写 `1` 即可清除该错误标志。

| 偏移 | 寄存器名称 | 适用设备 | 核心作用 |
|------|-----------|---------|---------|
| `+0x00` | Extended Capability Header | 所有设备 | Cap ID = `0x0001`，Version = `0x2`，指向下一个扩展能力 |
| `+0x04` | **Uncorrectable Error Status** | 所有设备 | 记录已发生的不可纠正错误（`RW1C`） |
| `+0x08` | **Uncorrectable Error Mask** | 所有设备 | 屏蔽位：`1` = 屏蔽该错误不上报，`0` = 允许上报 |
| `+0x0C` | **Uncorrectable Error Severity** | 所有设备 | 严重性配置：`1` = Fatal（致命），`0` = Non-Fatal（非致命） |
| `+0x10` | **Correctable Error Status** | 所有设备 | 记录已发生的可纠正错误（`RW1C`） |
| `+0x14` | **Correctable Error Mask** | 所有设备 | 可纠正错误屏蔽位（`1` = 屏蔽） |
| `+0x18` | **Advanced Error Cap & Control** | 所有设备 | First Error Pointer（首个错误位索引）、**ECRC（端到端 CRC）** 生成/检查使能 |
| `+0x1C`~`+0x28` | **Header Log（4 DW）** | 所有设备 | 锁存触发首个不可纠正错误的 TLP 头部（128-bit） |
| `+0x2C` | **Root Error Command** | 仅 Root Port | 控制 Root Port 收到对应错误报文时是否触发系统中断（AER IRQ） |
| `+0x30` | **Root Error Status** | 仅 Root Port | 汇总是否收到 `ERR_COR`、`ERR_FATAL/NONFATAL` 及是否发生多次错误 |
| `+0x34` | **Error Source Identification** | 仅 Root Port | 记录发送 `ERR_COR`（低 16 位）与 `ERR_FATAL/NONFATAL`（高 16 位）的设备 BDF |

> **名词速览**：
> - **BDF（Bus:Device.Function）**：PCIe 设备在总线拓扑中的唯一门牌号（16-bit Requester ID）。
> - **ECRC（End-to-End CRC）**：事务层可选的 32 位端到端校验码，用于保护 TLP 在经过多个 Switch 后依然保持内容完整；区别于链路层逐跳校验的 **LCRC**。

---

## 错误上报与处理流程

当 **Endpoint（终端设备，如 NVMe SSD、网卡）** 检测到异常时，AER 会按以下链路逐级上报至操作系统：

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Endpoint 检测到错误                                       │
│    • 置位 UE/CE Status 对应 bit；若为不可纠正错误，锁存 Header Log │
│    • 检查 Mask：若被屏蔽则仅记录状态，不上报                   │
│    • 查 Severity 表判定级别，构造 Error Message TLP            │
│      (0x30 = ERR_COR, 0x31 = ERR_NONFATAL, 0x32 = ERR_FATAL) │
└──────────────────────────────┬──────────────────────────────┘
                               │ 隐式路由（Implicitly Routed to RC）
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Root Port（根端口）接收 Error Message TLP                  │
│    • 置位 Root Error Status                                  │
│    • 将报文中的 Requester ID 写入 Error Source Identification  │
│    • 若 Root Error Command 使能，触发系统 AER 中断（IRQ）      │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. OS AER 驱动处理（如 Linux drivers/pci/pcie/aer.c）         │
│    • 读取 Root Error Status 与 Error Source ID 定位出错设备    │
│    • 读取该设备的 UE/CE Status 与 Header Log，打印 dmesg 日志    │
│    • 分级恢复：Correctable 仅记账；Non-Fatal 回调驱动恢复；      │
│      Fatal 执行设备/链路复位（FLR 或 Hot Reset）               │
│    • 向状态寄存器写 1（RW1C）清零，准备下一次监控               │
└─────────────────────────────────────────────────────────────┘
```

### 常见错误位速查

**Correctable Error Status (`+0x10`)**：
- `Bit 0` **Receiver Error**：物理层接收端检测到符号/解码异常，已由链路层恢复。
- `Bit 6` **Bad TLP** / `Bit 7` **Bad DLLP**：收到 **LCRC** 或 **DLLP CRC** 校验失败的数据包并已丢弃。
- `Bit 8` **REPLAY_NUM Rollover** / `Bit 12` **Replay Timer Timeout**：重传计数器溢出或重传定时器超时，提示链路信号质量可能下降。
- `Bit 13` **Advisory Non-Fatal**：建议性非致命错误（某些可安全忽略的非致命异常会降级记入此处）。

**Uncorrectable Error Status (`+0x04`)**：
- `Bit 4` **Data Link Protocol Error** / `Bit 5` **Surprise Down**：数据链路层协议崩溃或设备意外掉线（默认 Fatal）。
- `Bit 12` **Poisoned TLP Received**：收到被标记为“中毒”（`EP=1`）的数据包，提示数据本身已不可信。
- `Bit 13` **Flow Control Protocol Error** / `Bit 17` **Receiver Overflow**：流控信用违规或接收缓冲区溢出（默认 Fatal）。
- `Bit 14` **Completion Timeout**：发出非投递请求（如 Memory Read）后超时未收到响应。
- `Bit 15` **Completer Abort** / `Bit 16` **Unexpected Completion** / `Bit 20` **Unsupported Request**：完成端中止、收到未预期的完成包或访问了不支持的地址/功能。
- `Bit 18` **Malformed TLP**：TLP 头部格式严重违规（默认 Fatal）。
- `Bit 19` **ECRC Error**：端到端 CRC 校验失败。

---

## 常用诊断与观测命令

在 Linux 环境中，无需编写代码即可直接通过 `lspci`、`setpci` 和 `dmesg` 检查设备 AER 状态：

```bash
# 1. 查看设备 AER 寄存器详情（- 表示未置位，+ 表示置位或使能）
lspci -vvv -s 01:00.0 | grep -A 8 "Advanced Error Reporting"
# 重点关注：
#   UESta / CESta : 当前未清除的不可纠正 / 可纠正错误
#   UEMsk / CEMsk : 错误屏蔽掩码
#   UESvrt        : 不可纠正错误严重性（+ 为 Fatal，- 为 Non-Fatal）

# 2. 使用 setpci 直接读取 AER 状态（ECAP_AER 为自动解析的扩展能力偏移）
setpci -s 01:00.0 ECAP_AER+0x04.L   # 读取 Uncorrectable Error Status
setpci -s 01:00.0 ECAP_AER+0x10.L   # 读取 Correctable Error Status

# 3. 查看内核 AER 报错日志
dmesg | grep -iE "aer|pcie.*error"
```

> **运维建议**：如果生产环境中频繁出现 `RxErr` 或 `BadTLP` 等可纠正错误，虽然业务暂时未中断，但往往意味着线缆、金手指或插槽存在信号完整性隐患，应尽早排查，避免演变为链路掉速或致命错误。

---

## 关联与引申概念

AER 是 PCIe 可靠性（RAS，Reliability, Availability, Serviceability）体系的中枢，与以下概念紧密协作：

- **[PCIe 错误类型](error-types.md)**：详细拆解 Correctable、Non-Fatal 与 Fatal 各类错误的具体成因与影响矩阵。
- **[PCIe 错误检测](detection.md)**：解释物理层（8b/10b、128b/130b）、数据链路层（LCRC、序列号）和事务层（ECRC、TLP 格式检查）如何发现错误并喂给 AER。
- **[PCIe 错误恢复](recovery.md)**：说明 AER 上报之后，驱动和硬件如何通过 **FLR（功能级复位）**、**Hot Reset（热复位）** 以及 **DPC（下行端口隔离）** 完成故障隔离与恢复。
- **[PCIe 错误注入](error-injection.md)**：介绍如何通过 `aer-inject` 或协议分析仪人为触发 AER 流程，用于驱动和系统稳定性测试。
- **[Extended Capabilities](../configuration/extended-capabilities.md)**：了解 PCIe `0x100+` 扩展配置空间的链表结构与查找方式。
- **[链路训练与重训练](../physical-layer/link-training.md)**：当可纠正错误率过高时，物理层如何通过链路重训练恢复信号质量。
