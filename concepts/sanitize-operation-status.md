# Sanitize 操作状态

## 概述

本文档介绍 NVMe 规范中的 Sanitize Status（清理状态）日志，该日志用于跟踪数据清理操作的进度和结果。Sanitize 操作是一种彻底清除存储介质上数据的安全机制，确保数据无法被恢复。

## Sanitize Status 日志

Sanitize Status 日志（Log Identifier 81h）是用于持久化观察 Sanitize 操作进度和最近一次 Sanitize 结果的接口。当控制器（Controller）报告支持 Sanitize 功能时，必须提供此日志。该日志具有以下特性：

- **持久性**：在控制器重置（Reset）和电源循环（Power Cycle）后仍然保留
- **有效性**：当控制器状态寄存器 `CSTS.RDY=1` 时包含有效数据
- **必需性**：支持 Sanitize 功能的控制器必须实现此日志

[规范引用：PDF p. 302](../_source/pages/page-302.md)

## 操作流程模型

理解 Sanitize 操作的状态转换，可以参考以下流程：

```text
       从未启动（Never Started）
             |
             v
 [清理处理中] ───> [介质验证] ───> [验证后释放]
 Sanitize         Media           Post-Verify
 Processing      Verification     Deallocate
      |              |                 |
      +──────────────+─────────────────+
                     v
           成功或失败结果

状态观察：通过 LID 81h 获取进度 + 当前/最近状态 + 擦除证据
```

### 状态说明

- **清理处理中（Sanitize Processing）**：正在执行数据清理操作
- **介质验证（Media Verification）**：验证清理操作是否成功
- **验证后释放（Post-Verify Deallocate）**：验证后释放已分配的存储空间

以上流程为简化的概念模型，完整的操作状态机定义请参考规范性章节。[规范引用：PDF pp. 302-303](../_source/pages/page-302.md)

## 关键字段说明

### 进度指示：SPROG 字段

`SPROG`（Sanitize Progress）字段表示 Sanitize 操作的进度，其含义取决于当前操作阶段：

| 操作阶段 | SPROG 含义 |
|---------|-----------|
| 清理处理中（Sanitize Processing） | 进度值（分子），分母为 65,536 |
| 验证后释放（Post-Verify Deallocate） | 进度值（分子），分母为 65,536 |
| 介质验证（Media Verification） | 固定值 `FFFFh`（不适用） |
| 未执行 Sanitize 操作 | 固定值 `FFFFh`（不适用） |

**注意**：`SPROG` 并非适用于所有状态的通用百分比指示器，仅在特定阶段有效。

[规范引用：PDF p. 302](../_source/pages/page-302.md)

### 状态观察字段

| 字段 | 全称 | 含义 |
|-----|------|------|
| `SOS` | Sanitize Operation Status | 清理操作状态，可能的值包括：<br>• 从未启动（Never Started）<br>• 已清理（Sanitized）<br>• 清理中（Sanitizing）<br>• 失败（Failed）<br>• 已清理但意外释放（Sanitized with Unexpected Deallocation） |
| `MVCNCLD` | Media Verification Canceled | 指示请求的介质验证操作是否被取消：<br>• 因子系统组成（Subsystem Composition）变化而取消<br>• 因指定的重置（Reset）条件而取消 |
| `GDE` | Global Data Erased | 全局数据擦除指示器，满足以下条件时为真：<br>• 自制造后首次 Sanitize 之前未写入用户数据<br>• 自最近一次成功 Sanitize 后未写入用户数据<br>• 期间未启用 PMR（Persistent Memory Region） |
| `OPC` | Overwrite Pass Count | 已完成的覆写遍数；仅用于 Overwrite 操作，其他操作类型此字段为零 |

[规范引用：PDF p. 303](../_source/pages/page-303.md)

### SOS 特殊值说明

`SOS=100b`（二进制值）表示一种特殊的结果状态："**已清理但意外释放**（Sanitized with Unexpected Deallocation）"：

- **场景**：使用了 No-Deallocate（不释放）选项的 Sanitize 请求成功完成
- **异常行为**：尽管请求不释放空间，但所有用户数据（User Data）占用的介质空间均被释放
- **状态机状态**：操作状态机处于空闲（Idle）状态

**保留值**：`101b` 至 `111b` 为保留值，规范未定义其含义。

[规范引用：PDF pp. 303-304](../_source/pages/page-303.md)

## 时间估计字段

Sanitize Status 日志提供各阶段操作的预估完成时间，帮助主机（Host）了解操作进度。

### 标准操作时间估计

以下字段估计不包含特殊 No-Deallocate 介质修改路径的操作时间：

| 字段 | 全称 | 估计的操作类型 | 哨兵值含义 |
|-----|------|---------------|-----------|
| `ETO` | Estimated Time for Overwrite | 覆写（Overwrite）操作的后台处理时间 | `0h`：命令完成时预计完成<br>`FFFFFFFFh`：无法提供估计 |
| `ETBE` | Estimated Time for Block Erase | 块擦除（Block Erase）操作的后台处理时间 | `0h`：命令完成时预计完成<br>`FFFFFFFFh`：无法提供估计 |
| `ETCE` | Estimated Time for Crypto Erase | 加密擦除（Crypto Erase）操作的后台处理时间 | `0h`：命令完成时预计完成<br>`FFFFFFFFh`：无法提供估计 |

### No-Deallocate 介质修改时间估计

当请求 No-Deallocate 选项且控制器能力（Capability）选择该行为时，以下字段估计包含额外介质修改的操作时间：

| 字段 | 全称 | 估计的操作类型 | 哨兵值含义 |
|-----|------|---------------|-----------|
| `ETODMM` | Estimated Time for Overwrite with Deallocate Media Modification | 覆写操作包含额外介质修改的时间 | `0h`：命令完成时预计完成<br>`FFFFFFFFh`：无法提供估计 |
| `ETBENMM` | Estimated Time for Block Erase with No-Deallocate Media Modification | 块擦除操作包含额外介质修改的时间 | `0h`：命令完成时预计完成<br>`FFFFFFFFh`：无法提供估计 |
| `ETCENMM` | Estimated Time for Crypto Erase with No-Deallocate Media Modification | 加密擦除操作包含额外介质修改的时间 | `0h`：命令完成时预计完成<br>`FFFFFFFFh`：无法提供估计 |

### 验证后释放时间估计

| 字段 | 全称 | 估计的操作类型 | 哨兵值含义 |
|-----|------|---------------|-----------|
| `ETPVDS` | Estimated Time for Post-Verification Deallocation | 验证后释放（Post-Verification Deallocation）操作时间<br>（若操作进入此阶段） | `FFFFFFFFh`：无法提供估计 |

[规范引用：PDF pp. 304-305](../_source/pages/page-304.md)

### 哨兵值说明

- **`0h`**：表示操作预计在命令完成（Completion）时同步完成，无需后台处理
- **`FFFFFFFFh`**：表示控制器无法提供时间估计

## 详细状态信息

### SSI 字段（Sanitize Status Information）

`SSI.SANS`（Sanitize Action Not Started）和 `SSI.FAILS`（Sanitize Action Failed Status）字段提供更细粒度的状态信息：

#### SANS 字段

仅当版本有效性（Version Validity）位允许时，`SSI.SANS` 才暴露当前的详细状态：

- **空闲**（Idle）
- **受限处理中**（Restricted Processing）
- **受限失败**（Restricted Failure）
- **非受限处理中**（Unrestricted Processing）
- **非受限失败**（Unrestricted Failure）
- **介质验证中**（Media Verification）
- **验证后释放中**（Post-Verification Deallocation）

#### FAILS 字段

- **当 `SOS` 报告失败时**：`SSI.FAILS` 记录具体的失败状态码
- **当 `SOS` 未报告失败时**：此字段值为零

**重要提示**：以上说明仅为观察字段的映射关系，不能替代规范第 8.1.24.3 节中定义的规范性状态转换机制。完整的状态机行为必须参考规范文档。

[规范引用：PDF p. 305](../_source/pages/page-305.md)

## 相关概念

### 与其他概念的关联

- **[Sanitize Operation Lifecycle](sanitize-operation-lifecycle.md)**  
  定义了 Sanitize 命令的启动条件（Gate）和状态转换逻辑，本日志观察的即是该生命周期的状态。  
  [规范引用：PDF pp. 387-390](../_source/pages/page-387.md)

- **与 Format NVM 的区别**  
  Sanitize 操作与 Format NVM 命令的 Secure Erase 选项是不同的数据清除机制。Sanitize Status 日志仅观察 Sanitize 操作，不涉及 Format NVM 操作。  
  [规范引用：PDF p. 302](../_source/pages/page-302.md) | [PDF pp. 218-220](../_source/pages/page-218.md)

- **状态机映射关系**  
  `SOS`、取消（Cancellation）、进度（Progress）与操作状态之间的精确对应关系，由 Sanitize Status 日志定义所引用的状态机章节详细规定。  
  [规范引用：PDF pp. 302-303](../_source/pages/page-302.md)

## 规范引用

以下是本文档涉及的 NVMe 规范页面，供深入研究参考：

- **支持性、持久性、就绪状态与进度语义**  
  [PDF p. 302](../_source/pages/page-302.md)

- **Sanitize Status 位字段定义与初始状态值**  
  [PDF p. 303](../_source/pages/page-303.md)

- **其他结果状态、操作参数与时间估计字段**  
  [PDF p. 304](../_source/pages/page-304.md)

- **No-Deallocate 时间估计与状态信息编码**  
  [PDF p. 305](../_source/pages/page-305.md)
