# Format NVM 生命周期

## 概述

Format NVM 命令用于修改存储介质的格式属性，并可以选择性地执行用户数据擦除（User Data Erase）或加密擦除（Cryptographic Erase）。当该命令成功完成后，之前存储在受影响命名空间（Namespace）中的用户数据将无法被读取。

**规范参考：**[PDF pp. 217-220](../_source/pages/page-217.md)

## 操作流程

Format NVM 命令的执行流程可以用以下决策模型来理解：

```text
                    SES = 不擦除          SES = 安全擦除
                           |                       |
作用域能力 ----------+-----------------------+
                           |
            FNS 选择格式化作用域 / SENS 选择擦除作用域
                           |
              NSID 选择单个、已连接集合或子系统集合
                           ↓
            [格式化介质 + 可选的安全擦除]
                           |
                  操作完成
```

### 流程说明

这个决策流程简化了规范中 Figure 188 的渲染逻辑：

- **FNS 字段**：仅在 `SES=000b`（无安全擦除）时生效，用于控制格式化作用域
- **SENS 字段**：仅在 `SES` 非零时生效，用于控制擦除作用域

**规范参考：**[PDF pp. 218-220](../_source/pages/page-218.md)

## 作用域规则

Format NVM 命令可以作用于不同范围的命名空间，具体由作用域位（Scope Bit）和 NSID 参数共同决定：

| 作用域位 | NSID 参数值 | 受影响的命名空间范围 |
|---------|------------|-------------------|
| 适用作用域位为 `0` | 一个已分配的 NSID | 仅指定的单个命名空间 |
| 适用作用域位为 `0` | `FFFFFFFFh` | 控制器（Controller）已连接的所有命名空间 |
| 适用作用域位为 `1` | 已分配的 NSID 或 `FFFFFFFFh` | NVM 子系统（NVM Subsystem）中的所有命名空间 |

### 详细说明

- **适用作用域位**：对于普通格式化操作使用 `FNS` 字段，对于安全擦除使用 `SENS` 字段
- **广播限制**：如果 `FNVMBS=1`，则禁止使用广播 NSID（`NSID=FFFFFFFFh`）
- **空作用域处理**：当有效的广播作用域内不包含任何命名空间时，命令会成功完成但不执行任何实际操作

**规范参考：**[PDF pp. 218-219](../_source/pages/page-218.md)

## 安全擦除与格式化字段

### 安全擦除设置（SES）

| SES 值 | 操作效果 | 说明 |
|--------|---------|------|
| `000b` | 无安全擦除 | 仅执行格式化操作，不擦除数据 |
| `001b` | 用户数据擦除（User Data Erase） | 控制器可以在所有受影响数据均已加密的情况下使用加密擦除 |
| `010b` | 加密擦除（Cryptographic Erase） | 通过删除加密密钥使原数据无法恢复 |

### 格式化参数

命令的格式索引由 `LBAFU` 和 `LBAFL` 字段组合而成：

- **保护信息和元数据字段**：这些字段的具体含义取决于所使用的 I/O 命令集（I/O Command Set）
- **LBAFU 字段**：当主机行为支持（Host Behavior Support）禁用 LBA 格式扩展（LBA Format Extension）时，`LBAFU` 字段会被忽略

**规范参考：**[PDF pp. 219-220](../_source/pages/page-219.md)

## 并发控制与错误处理

在执行 Format NVM 命令时，需要注意以下并发和错误情况：

### 命令冲突

- **正在进行的 I/O 冲突**：如果有正在进行的 I/O 操作与 Format NVM 命令冲突，Format NVM 命令可能会返回"命令序列错误"（Command Sequence Error）状态
- **格式化期间的新 I/O**：在格式化过程中，发送到受影响命名空间的新 I/O 命令可能会返回"格式化进行中"（Format in Progress）错误状态

**规范参考：**[PDF pp. 218-219](../_source/pages/page-218.md)

### 多域子系统限制

在分割的多域（Multi-Domain）子系统环境中，如果控制器无法访问所选的命名空间：

- 命令会返回 ANA（Asymmetric Namespace Access，非对称命名空间访问）不可访问状态
- 或返回持久丢失（Persistent Loss）路径状态

**规范参考：**[PDF p. 218](../_source/pages/page-218.md)

### 常见错误状态

Format NVM 命令可能返回以下错误状态：

| 错误场景 | 返回状态 |
|---------|---------|
| 禁用或不支持的格式 | Invalid Namespace or Format（无效的命名空间或格式） |
| 命名空间处于写保护状态 | Namespace Is Write Protected（命名空间被写保护） |
| 无效的广播操作 | Invalid Field（无效字段） |
| 无效的格式参数 | Invalid Format（无效格式） |

**规范参考：**[PDF pp. 219-220](../_source/pages/page-219.md)

## 相关概念与关联

### 与其他命令的交互

- **管理命令限制**：Format NVM 执行期间，会临时缩减允许执行的管理命令（Admin Command）白名单
- **I/O 命令交互**：格式化过程会影响 I/O 命令的处理流程

**规范参考：**[PDF pp. 192-193, 218-219](../_source/pages/page-192.md)

### 状态查询

- 成功执行 Format NVM 后，新的格式设置会通过 **Identify Namespace** 数据结构上报

**规范参考：**[PDF p. 219](../_source/pages/page-219.md)

### 与 Sanitize 的区别

- **Secure Erase（安全擦除）**：作为 Format NVM 命令的一个选项，用于在格式化时擦除数据
- **Sanitize（数据清理）**：是一个独立的、更广义的数据清理操作，具有独立的状态机

**规范参考：**[PDF pp. 218-220](../_source/pages/page-218.md)

## 规范证据索引

以下是本文档涉及的规范章节引用：

- [Format 操作边界与固件下载完成，PDF p. 217](../_source/pages/page-217.md)
- [Figure 188：Format 和安全擦除作用域矩阵，PDF p. 218](../_source/pages/page-218.md)
- [作用域边界情况与 Figure 189 起始部分，PDF p. 219](../_source/pages/page-219.md)
- [Figure 189 延续部分与 Figure 190 状态描述，PDF p. 220](../_source/pages/page-220.md)
