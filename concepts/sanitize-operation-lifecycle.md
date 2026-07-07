# Sanitize 操作生命周期

## 概述

Sanitize（数据清除）操作用于启动或恢复一次覆盖整个子系统（Subsystem）范围的后台数据擦除。该操作支持三种擦除方式：块擦除（Block Erase）、加密擦除（Crypto Erase）或覆写（Overwrite）。此外，还可以选择性地启用介质验证（Media Verification）以及验证后的空间释放（Post-Verification Deallocation）。

**重要限制**：在导出型 NVM 子系统（Exported NVM Subsystem）上禁止执行此操作。

[PDF pp. 387-390](../_source/pages/page-387.md)

## 核心概念模型

为了帮助理解，我们先用简化的状态转换图来说明 Sanitize 操作的基本流程：

```text
                         成功处理 + 启用介质验证
[空闲] -- 开始擦除 ------------------------------> [介质验证]
   |             |                                         |
   |             `-- 不验证 -------------------------> [空闲]
   |                                                       |
   `-- 退出失败模式                                         | 退出验证
                                                           v
                                         [验证后空间释放]
                                                           |
                                                           v
                                                        [空闲]
```

这个解释性预览重构了 Sanitize 命令显式选择的状态转换。完整的受限/非受限失败状态机保留在规范第 8.1.24 节，将在处理该源范围时集成。

[PDF pp. 387-390](../_source/pages/page-387.md)

## 命令与后台操作的边界

### 命令完成的含义

**重要概念**：Sanitize 命令的成功完成并不意味着擦除操作已完成。

**实际行为**：
- 命令成功只是启动了后台操作
- 在命令完成队列条目（CQE, Completion Queue Entry）返回之前，系统会更新 Sanitize Status 日志
- 任何不成功的命令都不会启动任何操作，既不修改日志也不改动用户数据，保留先前的状态

[PDF pp. 389-390](../_source/pages/page-389.md)

### 命令参数与控制契约

| 操作/控制参数 | 说明 |
|-------------|------|
| **Block/Crypto/Overwrite**<br/>擦除类型 | 启动选定的后台数据清除处理 |
| **EMVS**<br/>启用介质验证 | 在成功完成块擦除或加密擦除且执行正常空间释放后，进入介质验证状态 |
| **Exit Media Verification**<br/>退出介质验证 | 不启动新的清除操作；仅在当前处于验证状态时转换到空间释放阶段 |
| **NDAS**<br/>不释放空间 | 请求保留分配状态，除非控制器禁止该行为 |
| **AUSE**<br/>允许非受限退出 | 选择受限（Restricted）或非受限（Unrestricted）完成模式 |
| **Overwrite 参数** | • Pass 数量（`0` 表示 16 次）<br/>• 可选的模式反转<br/>• 32 位数据模式 |

### 参数有效性约束

**介质验证的限制**：
在以下情况下，介质验证（Media Verification）无效：
- 使用 Overwrite 擦除方式
- 设置了 `NDAS=1`（不释放空间）
- 控制器能力不支持验证功能

**退出介质验证的限制**：
Exit Media Verification 命令仅在目标当前处于介质验证状态时有效。

[PDF pp. 388-390](../_source/pages/page-388.md)

## 启动条件与序列化

### 命令拒绝条件

在以下情况下，Sanitize 命令会被拒绝：

| 条件 | 说明 |
|-----|------|
| **PMR 已启用** | 持久内存区域（Persistent Memory Region）处于启用状态 |
| **Namespace 写保护** | 任何命名空间被设置为写保护 |
| **固件激活待定** | 需要重置的固件激活（Firmware Activation）仍在等待执行 |
| **控制器挂起** | 子系统中任一控制器被挂起（Suspended） |
| **多域限制** | 被分割的多域（Multi-Domain）子系统不能启动该操作 |
| **队列放置限制** | 使用了不支持的 CMB 队列放置方式 |

**固件操作互斥**：在 Sanitize 操作期间，禁止新的固件激活操作。

[PDF pp. 387-390](../_source/pages/page-387.md)

### 能力与选项验证

**能力控制**：`SANICAP` 字段控制：
- 支持的擦除操作类型
- No-Deallocate 行为选项
- 验证支持情况

**错误处理原则**：
不支持的操作或非法的选项组合会直接以 "Invalid Field" 失败，而不是静默降级为其他擦除方法。这确保了操作的可预测性和审计要求的满足。

[PDF pp. 387-388](../_source/pages/page-387.md)

## 相关配置与功能

### Sanitize Config 配置

**配置命令**：Sanitize Config (`17h`)

**功能**：选择当 No-Deallocate 行为被禁止时，控制器如何处理 `NDAS=1` 请求。

| 模式 | 行为 |
|-----|------|
| **Error Mode**<br/>错误模式 | 拒绝命令 |
| **Warning Mode**<br/>警告模式 | 允许处理，成功完成后记录 "Sanitized Unexpected Deallocate"<br/>（清除时意外释放空间） |

**作用范围**：此配置是子系统范围的设置，仅在 No-Deallocate 被禁止时有效。

[PDF p. 409](../_source/pages/page-409.md)

## 相关主题

### 状态监控

- [Sanitize Operation Status](sanitize-operation-status.md) - 提供后台生命周期的持久进度和结果观察接口。这是监控 Sanitize 操作执行状态的主要方式。[PDF pp. 302-305](../_source/pages/page-302.md)

### 序列化依赖

以下特性定义了与 Sanitize 操作的独立序列化门控：

- [Firmware Update Lifecycle](firmware-update-lifecycle.md) - 固件更新生命周期
- [Controller Migration](controller-migration.md) - 控制器迁移
- [Controller Memory Windows](controller-memory-windows.md) - 控制器内存窗口

[PDF pp. 388-390](../_source/pages/page-388.md)

### 对比：Format NVM

- [Format NVM Lifecycle](format-nvm-lifecycle.md) - 定义命名空间级别的格式化安全擦除。**重要区别**：Format NVM 是针对单个命名空间的操作，而 Sanitize 是覆盖整个子系统范围的清除操作，两者是不同的概念。[PDF pp. 218-220](../_source/pages/page-218.md)

## 规范引用

以下是本文档引用的 NVMe 规范页面清单：

### 命令与基本流程
- [Sanitize 范围、操作类型、后台阶段与能力门控，PDF p. 387](../_source/pages/page-387.md)
- [介质验证转换与序列化门控，PDF p. 388](../_source/pages/page-388.md)
- [命令启动原子性与选项字段，PDF p. 389](../_source/pages/page-389.md)
- [操作值、CQE/日志顺序与状态，PDF p. 390](../_source/pages/page-390.md)
