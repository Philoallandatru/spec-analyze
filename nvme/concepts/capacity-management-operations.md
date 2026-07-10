# 容量管理操作（Capacity Management Operations）

## 概述

容量管理操作是 NVMe 规范中用于配置存储容量分配的机制。它允许管理员通过两种方式来组织和分配 NVM 子系统（NVM Subsystem）的存储容量：

- **固定配置模式**：选择预定义的容量配置方案
- **动态分配模式**：按需创建和删除容量实体，灵活调整存储架构

通过这些操作，可以配置持久性组（Endurance Group）和 NVM 集合（NVM Set）的容量分配关系。[PDF pp. 203-206](../_source/pages/page-203.md)

## 两种管理模式

### 核心概念对比

NVMe 容量管理提供了两种不同的管理思路，它们在配置方式和灵活性上各有特点：

```text
┌─────────────────────────────────┬─────────────────────────────────┐
│       固定配置模式               │       动态分配模式               │
│   (Fixed Configuration)         │   (Variable Allocation)         │
├─────────────────────────────────┼─────────────────────────────────┤
│  使用容量配置 ID                 │  指定具体的字节容量大小          │
│  (Capacity Configuration ID)    │  (64-bit byte capacity)         │
│              ↓                  │              ↓                  │
│  选择预定义的 Endurance Group    │  在 Domain 中创建 Endurance Group│
│              ↓                  │              ↓                  │
│  选择预定义的 NVM Set            │  在 Endurance Group 中创建 NVM Set│
└─────────────────────────────────┴─────────────────────────────────┘
```

**关键术语说明：**
- **Domain（域）**：存储资源的顶层容器
- **Endurance Group（持久性组）**：管理存储介质耐久性和寿命的逻辑单元
- **NVM Set（NVM 集合）**：可以包含多个命名空间的存储资源集合
- **Namespace（命名空间）**：对主机可见的存储卷

### 层级包含关系

**重要特性：删除操作具有级联效应**

当删除一个父级对象时，其下属的所有子对象都会被自动删除：

- 删除 Endurance Group → 同时删除其下的所有 NVM Set 和 Namespace
- 删除 NVM Set → 同时删除其下的所有 Namespace

这种设计遵循容量管理的包含语义，确保资源管理的一致性。[PDF pp. 203-205](../_source/pages/page-203.md)

## 操作命令详解

### 操作类型与参数

容量管理命令通过操作码（Operation Code）来区分不同的操作类型。下表详细说明了每种操作及其参数含义：

| 操作类型 | `ELID` 字段的含义 | 容量字段用途 |
|---|---|---|
| **选择容量配置**<br/>Select Configuration | 容量配置标识符<br/>（Capacity Configuration Identifier）<br/>• `0h` = 清除当前配置 | 保留字段<br/>(reserved) |
| **创建持久性组**<br/>Create Endurance Group | Domain 标识符<br/>（Domain Identifier）<br/>• `0h` = 由控制器自动选择 | 请求的容量大小<br/>（64 位字节数） |
| **删除持久性组**<br/>Delete Endurance Group | 要删除的 Endurance Group 标识符 | 保留字段<br/>(reserved) |
| **创建 NVM 集合**<br/>Create NVM Set | 父级 Endurance Group 标识符<br/>• `0h` = 由控制器自动选择 | 请求的容量大小<br/>（64 位字节数） |
| **删除 NVM 集合**<br/>Delete NVM Set | 要删除的 NVM Set 标识符 | 保留字段<br/>(reserved) |

**命令格式说明：**
- **CDW10**（Command Dword 10）包含 `ELID` 字段和操作码
- **CDW11** 和 **CDW12** 分别包含容量值的低 32 位（`CAPL`）和高 32 位（`CAPU`）
- 容量字段仅在创建操作时有效，用于指定请求的 64 位字节容量
- 这些字段的定义对应规范中的图 156-158

[PDF pp. 203-204](../_source/pages/page-203.md)

## 固定配置模式工作流程

### 配置选择生命周期

固定配置模式使用预定义的配置描述符（Configuration Descriptor）来一次性配置整个存储架构：

```text
[初始状态：无已选配置]
        |
        | 执行操作：选择一个有效的配置描述符
        ↓
[自动创建阶段]
  按照描述符依次创建：
  1. 创建每个 Endurance Group
  2. 创建每个 NVM Set
        |
        ├── 情况 1：重复选择同一个配置 ID
        |      → 操作成功，但不会产生任何变化
        |
        └── 情况 2：选择配置 ID = 0h（清除配置）
               → 按顺序执行删除：
                 1) 删除所有 Namespace
                 2) 删除所有 NVM Set
                 3) 删除所有 Endurance Group
                 4) 清空"已选配置"状态
               → 返回到初始状态
```

**重要限制：**
- 当已有配置生效时，不能直接切换到另一个非零配置 ID
- 必须先选择 `0h` 清除当前配置，然后才能选择新的配置
- 尝试选择未知的配置标识符会被拒绝，不会改变任何 Media Unit 的配置状态

这些规则确保了配置变更的可控性和安全性。[PDF p. 204](../_source/pages/page-204.md)

## 动态分配模式详解

### 容量分配机制

动态分配模式允许按需创建存储资源，遵循以下分配逻辑：

**创建 Endurance Group 时：**
1. 如果指定了 Media Unit（媒体单元），则选择该 Media Unit
2. 否则，从指定的 Domain 中分配请求的容量

**创建 NVM Set 时：**
1. 如果指定了 Media Unit，则选择该 Media Unit
2. 否则，从父级 Endurance Group 中分配请求的容量

[PDF p. 205](../_source/pages/page-205.md)

### 错误处理与失败场景

在动态分配过程中，可能会遇到以下错误情况：

| 失败场景 | 返回的完成状态 | 详细说明 |
|---|---|---|
| **标识符耗尽** | Identifier Unavailable | 系统中没有可用的非零 Endurance Group 或 NVM Set 标识符 |
| **容量不足** | Insufficient Capacity | 请求的容量超过了可用的最大容量或剩余容量<br/>错误信息日志（Error Information Log）可能会报告可创建的最大容量值 |
| **删除操作参数错误** | Invalid Field in Command | 使用了零值标识符或不存在的标识符 |
| **不支持 NVM Set 功能** | Invalid Field in Command | 执行了 NVM Set 相关操作，但控制器不支持 NVM Set 功能 |

**创建成功时的返回值：**

当成功创建 Endurance Group 或 NVM Set 后，完成队列项（Completion Queue Entry, CQE）的 Dword 0 的位 15:0 会返回新创建实体的标识符。这个标识符可用于后续的管理操作。其他操作类型的返回值字段为保留字段。

[PDF pp. 205-206](../_source/pages/page-205.md)

## 与其他功能的关联

### NVM 容量模型

容量管理操作直接修改 NVM 容量模型（NVM Capacity Model）中描述的分配层级关系：

```
Domain → Endurance Group → NVM Set → Namespace
```

这个四层的容量分配架构是 NVMe 规范中存储资源组织的核心模型。容量管理命令通过创建和删除操作来改变这个层级结构中的实体及其关系。[PDF pp. 141-146, 203-205](../_source/pages/page-141.md)

### 配置描述符

在固定配置模式下，系统会：
- 读取支持的容量配置描述符（Supported Capacity Configuration Descriptor）
- 更新 Media Unit 状态（Media Unit Status）中的"已选配置"字段值

这些描述符定义了预定义配置方案的具体内容。[PDF p. 204](../_source/pages/page-204.md)

### 错误信息日志

当容量不足导致命令失败时：
- 命令完成状态码会返回 `Insufficient Capacity`
- 错误信息日志（Error Information Log）中会包含命令特定的容量信息
- 这些信息帮助管理员了解可用的最大容量限制

[PDF pp. 205-206](../_source/pages/page-205.md)

## 规范引用索引

本文档内容基于 NVMe 规范的以下部分：

- **操作命令格式**：[图 156-157、操作码与容量字段定义，PDF p. 203](../_source/pages/page-203.md)
- **固定配置流程**：[图 158 与配置选择序列，PDF p. 204](../_source/pages/page-204.md)
- **分配与失败规则**：[Endurance Group 和 NVM Set 的创建/删除规则，PDF p. 205](../_source/pages/page-205.md)
- **完成状态与返回值**：[图 159-160、命令状态码与创建的标识符，PDF p. 206](../_source/pages/page-206.md)

---

**文档说明：**  
本文档是对 NVMe 规范中容量管理操作部分的中文解读，旨在帮助中文读者更好地理解相关概念。在实际实现时，请务必参考 NVMe 官方规范原文以获取权威和完整的技术细节。
