# 安全协议交换（Security Protocol Exchange）

## 概述

安全协议交换机制为主机（Host）和控制器（Controller）之间提供了一个标准化的安全通信通道。通过 **Security Send** 和 **Security Receive** 这两个配对命令，主机可以向控制器发送安全相关的命令和参数，并接收控制器返回的状态和数据。

**核心设计理念：**
- NVMe 只负责提供命令的"信封"（Command Envelope），即传输机制
- 具体的请求/响应配对关系和数据载荷（Payload）的含义由所选择的安全协议（Security Protocol）自行定义
- 这种设计实现了协议层面的解耦，使 NVMe 能够支持多种不同的安全协议

> 参考规范：[PDF pp. 390-392](../_source/pages/page-390.md)

## 工作原理

### 交互模型

下图展示了主机和控制器之间的典型安全协议交互流程：

```text
主机（Host）                  控制器（Controller）/ 安全协议处理模块
    |                                           |
    |---- Security Send (发送安全命令和数据) ---->|
    |                                           |
    |                   [控制器执行协议定义的操作] |
    |                                           |
    |<--- Security Receive (接收状态和结果) ------|
    |                                           |

注意：命令的配对关系和保留机制由具体的安全协议定义
```

**关键特性：**

1. **不对称方向性**：Send 和 Receive 是两个独立的命令，由协议决定如何配对
2. **协议主导**：配对关系和数据保留策略完全由安全协议规范定义
3. **数据易失性**：通过 Receive 获取的数据在发生通信中断（Communication Loss）或控制器级别重置（Controller Level Reset）时可能丢失

> 参考规范：[PDF pp. 390-392](../_source/pages/page-390.md)

## 命令结构

### 命令字段说明

Security Send 和 Security Receive 命令共享相似的字段结构，但用途略有不同：

| 字段名称 | Security Send | Security Receive | 说明 |
|---------|--------------|------------------|------|
| **SECP**<br>（Security Protocol） | 指定 Security Protocol Out 的语义 | 指定 Security Protocol In 的语义 | 选择要使用的安全协议类型 |
| **SPSP**<br>（Security Protocol Specific） | 16 位协议特定选择器 | 使用相同的选择器命名空间 | 协议内部的子功能选择 |
| **NSSF**<br>（NVMe Security Specific） | 仅在协议 `EAh` 时使用 | 仅在协议 `EAh` 时使用 | 对于非 `EAh` 协议，此字段为保留字段 |
| **长度字段** | 指定传输长度（Transfer Length） | 指定分配长度（Allocation Length） | Send 表示发送的数据量<br>Receive 表示预留的接收缓冲区大小 |
| **数据方向** | 主机到控制器 | 控制器到主机 | 数据流向 |

**错误处理：**
- 如果 SECP 字段指定的协议值为保留值或控制器不支持的值，命令将失败并返回 **Invalid Field in Command** 错误状态

> 参考规范：[PDF pp. 391-392](../_source/pages/page-391.md)

## 协议发现与 NVMe 专用协议

### 安全协议发现（Protocol Discovery）

**协议 `00h`** 具有特殊用途，专门用于发现控制器支持的安全协议列表：

- **使用场景**：主机需要了解控制器支持哪些安全协议时
- **操作方式**：发送 Security Receive 命令，SECP 设置为 `00h`
- **特点**：
  - 这是一个独立的 Receive 请求，不需要先发送对应的 Send 命令
  - 不与任何先前的 Send 命令关联
  - 返回数据包含控制器支持的所有安全协议列表

### NVMe 专用安全协议（Protocol `EAh`）

**协议 `EAh`** 是专门分配给 NVMe 接口使用的安全协议：

| 子协议参数 | 用途 | 说明 |
|-----------|------|------|
| **SPSP = 0001h** | 选择 Replay Protected Memory Block (RPMB) | RPMB 是一种具有重放保护的安全存储区域 |
| **NSSF** 字段 | 选择特定的 RPMB 目标（Target） | 控制器可能支持多个 RPMB 目标 |

> 参考规范：[PDF p. 391](../_source/pages/page-391.md)

## 相关概念

### 与其他功能的关联

**RPMB 能力声明：**
- 通过 [Identify Command Model](identify-command-model.md) 中的 Identify Controller 数据结构，控制器会声明：
  - 支持的 RPMB 目标数量
  - RPMB 目标之间的共享能力
- **强制要求**：如果控制器声明的 RPMB 目标数量非零，则必须支持 Security Send 和 Security Receive 命令

> 参考规范：[PDF pp. 333-334](../_source/pages/page-333.md)

**协议规范引用：**
- NVMe Base Specification 仅定义命令传输机制（信封）
- 具体协议的详细定义由外部规范定义：
  - SPC（SCSI Primary Commands）规范
  - ACS（ATA Command Set）规范
- NVMe 规范不重复定义这些协议的具体细节，而是直接引用相关规范

> 参考规范：[PDF pp. 390-392](../_source/pages/page-390.md)

## 使用场景示例

1. **发现支持的协议**：主机启动时发送 SECP=`00h` 的 Security Receive 命令，获取支持列表
2. **RPMB 访问**：使用 SECP=`EAh`、SPSP=`0001h` 访问重放保护内存块
3. **TCG Opal 加密**：使用特定的 Security Protocol 值实现全盘加密功能

## 参考资料

- [Security Receive 命令用途与保留边界说明，PDF p. 390](../_source/pages/page-390.md)
- [Security Receive 字段详解、协议发现机制、NVMe EAh 协议与 RPMB 选择，PDF p. 391](../_source/pages/page-391.md)
- [Security Send 命令字段与响应关系说明，PDF p. 392](../_source/pages/page-392.md)
