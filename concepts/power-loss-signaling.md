# 断电信号机制（Power Loss Signaling）

## 概述

断电信号机制（Power Loss Signaling，简称 PLS）是 NVMe 规范中一项重要的电源管理功能，作用范围覆盖整个 Domain（域）。该机制基于 PCIe 的能力实现，允许系统在电源即将断开时提前通知控制器（Controller），从而为数据保护争取宝贵时间。

**工作原理：**
- **传输层（Transport）** 通过断言（assert）**断电通知信号（Power Loss Notification，PLN）** 来告知即将发生的电源损失
- **控制器（Controller）** 可选择性地暴露**断电应答信号（Power Loss Acknowledge，PLA）** 来响应此通知
- **域配置（Domain Configuration）** 决定采用以下三种操作模式之一：
  - 强制静默模式（Forced Quiescence，FQ）
  - 紧急断电模式（Emergency Power Fail，EPF）
  - 禁用 PLS 功能

> 规范参考：[PDF pp. 615-616](../_source/pages/page-615.md)

---

## 状态转换模型

下图展示了 PLS 机制的完整状态流转过程，帮助理解系统在不同情况下的行为：

```text
上电 / 复位 / 关机事件
         |
         v
   [PLS 未就绪状态]
   (PLS Not Ready)
         |
         | SHST=00b (正常工作状态)
         v
    [PLS 就绪状态]
    (PLS Ready)
         |
         | 检测到断电信号 PLN + 根据配置选择模式
         |
         +------------------------+------------------------+
         |                        |                        |
         v                        v                        v
   [FQ 处理中]            [EPF 处理中]              [EPF 处理中]
 (Forced Quiescence)      端口保持启用             端口已禁用
         |                        |                        |
         |                        v                        v
         v                 [EPF 完成状态]           [EPF 完成状态]
   [FQ 完成状态]            端口保持启用             端口已禁用
         |                        |                        |
   解除 PLN 信号                   |                        |
         |                        v                        v
         |                   复位或关机               需要重新上电
         v                  返回未就绪状态              才能恢复
   返回 PLS 就绪
```

**关键说明：**
- 该状态机保留了规范图 660 中互斥的 EPF 端口分支设计，以及复位/重新上电的返回路径
- 每次上电循环后，所有状态都会返回到"PLS 未就绪"状态
- 执行控制器复位或关机操作（非零 `CSTS.SHST` 值）时，已启用端口的 FQ 状态也会返回到"未就绪"状态

> 规范参考：[PDF pp. 617-620](../_source/pages/page-617.md)

---

## 信号定义与支持要求

### 核心信号说明

| 信号名称 | 设置方 | 含义说明 |
|---------|--------|---------|
| **PLN = Asserted**<br>（断电通知已断言） | NVMe 传输层<br>(NVMe Transport) | 系统检测到电源即将断开，<br>发出预警通知 |
| **PLA = Asserted-EPF-Enabled**<br>（断电应答-EPF启用） | 控制器<br>(Controller) | 控制器已激活 EPF 模式，<br>PCIe 端口通信继续进行 |
| **PLA = Asserted-EPF-Disabled**<br>（断电应答-EPF禁用） | 控制器<br>(Controller) | 控制器已激活 EPF 模式，<br>PCIe 端口通信已停止 |

**信号复位规则：**
- 执行控制器级别复位（Controller Level Reset）后，`PLN` 和 `PLA` 两个信号都会被重置为未断言（Deasserted）状态

### 支持要求与约束

**Domain 级别约束：**
- 同一个 Domain 内的所有控制器必须：
  - 披露相同的支持模式
  - 使用相同的选定模式
  - 同一时刻最多只能激活一种操作模式

**信号支持要求：**
- 当系统支持 PLS 时，`PLN` 信号是**强制必须**的
- `PLA` 信号是**可选**的，控制器可自行决定是否实现

**附加支持要求：**
支持 PLS 功能还需要满足以下条件：
- 实现 Domain 范围的配置功能（Configuration Feature）
- 提供命名空间 I/O 影响报告
- 在支持的操作模式下，至少有一个适用电源状态的非零时序字段

> 规范参考：[PDF pp. 615-616](../_source/pages/page-615.md)

### 特殊情况处理

**被忽略的 PLN 转换：**
以下情况下，PLN 信号的状态转换会被控制器忽略：
- 控制器级别复位（Controller Level Reset）正在进行时
- 关机（Shutdown）正在进行或已完成时

**时序值为零的情况：**
- 当相应的 vault（数据保护）或 recovery（恢复）时长配置为零时，实际时长将变为厂商自定义（vendor specific）

**断电后的恢复行为：**
- 在 EPF 完成状态发生电源损失后，首次恢复时可能会暴露出尚未就绪或降级的命名空间（namespace）
- 原因：控制器内部的数据恢复过程可能仍在后台进行

> 规范参考：[PDF pp. 616-617](../_source/pages/page-616.md)

---

## 状态与端口通信规则

### 各状态下的信号与通信行为

| 控制器状态 | PLA 信号状态<br>（如果控制器支持） | PCIe 端口通信 |
|-----------|---------------------------|-------------|
| **PLS 未就绪 / 就绪**<br>(PLS Not Ready / Ready) | 未断言<br>(Deasserted) | 正常处理命令 |
| **FQ 完成**<br>(FQ Complete) | 未断言<br>(Deasserted) | 正常处理命令 |
| **EPF 处理中-端口启用**<br>(EPF Processing Port Enabled) | 已断言-EPF启用<br>(Asserted-EPF-Enabled) | 正常处理命令 |
| **EPF 处理中-端口禁用**<br>(EPF Processing Port Disabled) | 已断言-EPF禁用<br>(Asserted-EPF-Disabled) | **不处理**命令 |
| **EPF 完成-端口禁用**<br>(EPF Complete Port Disabled) | 未断言<br>(Deasserted) | **不处理**命令 |

### 端口行为约束

**实现规则：**
- 每个控制器只能实现**一种** EPF 端口行为（端口启用 或 端口禁用）
- 同一个 Domain 内的所有控制器必须实现**相同的** EPF 端口行为

**端口禁用分支的限制：**
- 无法通过 PCIe 端口发起控制器复位或子系统复位
- 无法通过 PCIe 端口发起关机操作
- 从"EPF 完成-端口禁用"状态退出的**唯一方式**是重新上电

**端口启用分支的优势：**
- 可以在复位或关机时正常返回到"PLS 未就绪"状态
- 保持了控制器的可管理性

> 规范参考：[PDF pp. 618-620](../_source/pages/page-618.md)

---

## 两种断电处理模式详解

### Forced Quiescence（强制静默模式）

**处理流程：**
```text
断言 PLA 信号
    ↓
停止从队列中取指令（fetch）
    ↓
完成或中止所有已取出的命令 + 准备数据保护
    ↓
挂起后台操作（如垃圾回收、磨损均衡等）
    ↓
处理完成
    ↓
解除 PLA 信号断言
```

**模式特点：**
- ✅ **保持连通性**：PCIe 端口继续工作，可以接收新命令
- ✅ **命令完成保证**：应当完成或中止所有先前已取回的命令
- ✅ **可恢复性**：如果电源实际未断开且 PLN 信号解除，控制器会恢复正常的命令取指操作
- ⚠️ **配置锁定**：在已取指状态下，拒绝任何试图改变 PLS 配置的操作
- ⚠️ **后台挂起**：所有后台操作（垃圾回收、数据整理等）会被挂起，直到 PLN 解除断言

**中断处理：**
- 执行复位或关机操作会中止 FQ 处理流程

> 规范参考：[PDF p. 621](../_source/pages/page-621.md)

### Emergency Power Fail（紧急断电模式）

**处理流程：**
```text
断言 PLA 信号
    ↓
停止从队列中取指令
    ↓
丢弃所有已取指令 + 丢弃管理端点的带外命令
    ↓
根据选定的端口行为（启用/禁用）准备数据保护
    ↓
处理完成
    ↓
解除 PLA 信号断言
```

**模式特点：**
- ⚡ **快速响应**：直接丢弃已取指令和管理端点命令，不尝试完成
- 📊 **披露恢复时间**：成功完成 EPF 后，会披露首次初始化时的恢复间隔时间
- ⏱️ **恢复时机灵活**：实际恢复可能在 `CC.EN=1`（控制器使能）之前或之后开始
- ⚠️ **恢复时长不确定**：可能超过 `CSTS.RDY=1`（控制器就绪）的时长
- ⚠️ **非成功完成**：如果 EPF 未能成功完成，可能花费比披露值更长的时间

**命令丢弃范围：**
- 进入 EPF 处理前已取回的所有 I/O 命令
- 进入 EPF 处理前收到的所有管理端点（Management Endpoint）命令

> 规范参考：[PDF pp. 621-622](../_source/pages/page-621.md)

---

## 相关概念关联

本文档涉及的 PLS 机制与以下 NVMe 概念紧密相关：

| 相关文档 | 关联内容 | 规范位置 |
|---------|---------|---------|
| [电源状态描述符](power-state-descriptors.md) | 报告 FQ vault、EPF vault 和 EPF recovery 的时长参数 | [PDF pp. 616, 621-622](../_source/pages/page-616.md) |
| [控制器使能、关机与复位](controller-enable-shutdown-reset.md) | 定义导致 PLS 返回未就绪状态的复位和关机条件 | [PDF pp. 616-620](../_source/pages/page-616.md) |
| [通用控制器功能](common-controller-features.md) | 定义保存的 Domain 范围模式选择器配置 | [PDF pp. 411-412, 615](../_source/pages/page-411.md) |

---

## 规范溯源

以下是本文档内容在 NVMe 规范中的详细定位，便于深入研究：

| 主题内容 | 规范页码 |
|---------|---------|
| PLS 模式、作用域与变量表起点 | [PDF p. 615](../_source/pages/page-615.md) |
| PLA 表、支持约定与被忽略的转换 | [PDF p. 616](../_source/pages/page-616.md) |
| 审阅图 660 状态机（完整状态转换图） | [PDF p. 617](../_source/pages/page-617.md) |
| 审阅状态/PLA/端口矩阵及 Not Ready/Ready 转换 | [PDF p. 618](../_source/pages/page-618.md) |
| 审阅 FQ 与 EPF 分支转换详情 | [PDF pp. 619-620](../_source/pages/page-619.md) |
| FQ 与 EPF 处理序列详细说明 | [PDF pp. 621-622](../_source/pages/page-621.md) |
