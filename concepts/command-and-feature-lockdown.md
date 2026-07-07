# 命令与功能锁定

## 功能概述

命令与功能锁定（Command and Feature Lockdown）是 NVMe 子系统（Subsystem）提供的一项可选安全能力。它允许系统管理员主动禁止特定操作的执行，从而增强系统的安全性和可控性。

**可锁定的操作类型：**

- **管理命令**（Admin Commands）
- **管理接口操作**（Management Interface Operations）  
- **PCIe 命令**（PCIe Commands）
- **功能设置操作**（Set Features Operations）

该功能通过专门的日志页面（Log Page）提供查询能力，帮助用户明确区分两种不同的状态：

| 状态类型 | 英文术语 | 含义说明 |
|---------|---------|---------|
| **可锁定** | Lockable | 该操作支持被锁定功能管理 |
| **已禁止** | Prohibited | 该操作当前在指定路径上被禁止执行 |

禁止状态可以按访问路径独立设置：
- **带内路径**（In-band）：通过管理队列（Admin Queue）访问
- **带外路径**（Out-of-band）：通过管理端点（Management Endpoint）访问

> 📖 **规范参考**：[PDF pp. 284-286](../_source/pages/page-284.md)

## 工作原理

### 二维选择模型

命令与功能锁定采用二维选择架构，通过两个正交的维度来精确定位和控制目标操作：

```text
                    内容维度（Contents）
              +------------------+------------------+
              |    lockable      |    prohibited    |
              |   (可锁定能力)    |   (禁止状态)      |
              +------------------+--------+---------+
                                          |
                                          ├─ Admin Queue 路径
                                          └─ Management Endpoint 路径

作用域维度（Scope）
  ├─ Admin 操作码（Admin Opcodes）
  ├─ 功能标识符（Feature IDs）+ 可选厂商 UUID
  ├─ 管理接口操作码（Management Interface Opcodes）
  └─ PCIe 命令操作码（PCIe Command Opcodes）
```

**架构说明：**

- **作用域维度**（Scope Selector）：指定要查询或操作的命令类型空间
- **内容维度**（Contents Selector）：指定查询的是"锁定能力"还是"当前禁止状态"
- **路径分离**：禁止状态按访问路径独立管理，同一操作可以在不同路径上有不同的禁止状态

此架构模型已与 NVMe 规范中的 Figures 265–266 进行了验证对照。

> 📖 **规范参考**：[PDF pp. 285-286](../_source/pages/page-285.md)

### 查询接口说明

查询锁定状态时，需要通过日志页面接口提供选择器参数：

#### 主要选择器参数

| 选择器 | 可选值 | 含义 |
|-------|-------|------|
| **Contents**<br>（内容选择器） | `00b` | 查询该操作是否支持锁定（返回可锁定操作列表） |
| | `01b` | 查询该操作当前是否在 Admin Queue 路径上被禁止 |
| | `10b` | 查询该操作当前是否在 Management Endpoint 路径上被禁止 |
| **Scope**<br>（作用域选择器） | `0h` | Admin 命令操作码空间 |
| | `2h` | Feature 标识符空间 |
| | `3h` | Management Interface 操作码空间 |
| | `4h` | PCIe 命令操作码空间 |
| **UUID Index**<br>（可选参数） | 索引值 | 仅在 Scope=`2h`（Feature）时考虑<br>用于选择厂商特定的功能锁定信息 |

#### 查询返回格式

日志页面返回一个 512 字节的数据结构，包含：

- **参数回显**：所选的 Contents 和 Scope 值
- **列表长度**：标识符列表的字节总长度
- **标识符列表**：按数字升序排列的操作码或功能 ID 列表
- **空列表表示**：长度字段为零表示该类别下没有匹配项

#### 错误处理

| 错误场景 | 返回状态 |
|---------|---------|
| 子系统不包含 Management Endpoint，但查询选择了 ME 路径的禁止状态 | `Invalid Field in Command` |

> 📖 **规范参考**：[PDF pp. 285-286](../_source/pages/page-285.md)

## 与相关功能的关系

### 与 Effects Catalogs 的区别

两个功能从不同角度描述命令的可用性，互为补充但各有侧重：

| 功能特性 | Effects Catalogs | Command and Feature Lockdown |
|---------|------------------|------------------------------|
| **关注点** | 命令的支持情况和影响范围 | 命令的执行权限和禁止状态 |
| **回答的问题** | 这个命令存在吗？<br>会产生什么影响？ | 这个命令当前能执行吗？<br>在哪个路径上被禁止？ |
| **典型用途** | 发现子系统能力<br>理解命令副作用 | 访问控制<br>安全加固 |
| **状态性质** | 静态的能力描述 | 动态的权限状态 |

**理解要点：**

- Effects Catalogs 描述"能力边界"——子系统支持什么、可能影响什么
- Lockdown 描述"权限状态"——已支持的操作是否被允许、当前是否被禁止
- 两者独立运作：一个命令可以被支持（在 Catalog 中）但被禁止（在 Lockdown 中）

> 📖 **规范参考**：[PDF pp. 281-286](../_source/pages/page-281.md)

### 与 Lockdown 命令的关系

锁定功能由命令和日志两个互补接口组成：

| 接口类型 | 名称 | 角色 | 用途 |
|---------|------|------|------|
| **控制路径** | Lockdown 命令 | 写入端 | 设置和修改锁定策略 |
| **查询路径** | Lockdown 日志页面 | 读取端 | 查询锁定能力和当前状态 |

日志页面是命令操作结果的"权威清单"，反映当前生效的锁定配置。

> 📖 **规范参考**：[PDF p. 284](../_source/pages/page-284.md)

## 执行锁定操作

### 命令参数说明

通过 Lockdown 命令执行锁定操作时，需要指定以下参数：

| 参数字段 | 说明 | 备注 |
|---------|------|------|
| **Identifier** | 要锁定的操作码或功能标识符 | 具体值取决于 Scope |
| **Scope** | 标识符所属的命名空间 | Admin / Feature / Management Interface / PCIe |
| **Interface** | 锁定生效的接口路径 | • Admin Queue（带内）<br>• Management Endpoint（带外）<br>• 或两者同时 |
| **Action** | 执行的操作 | • Prohibit（禁止）<br>• Allow（允许） |
| **UUID Index** | 厂商功能选择器 | 可选参数<br>仅在 Scope=Feature 且双方都支持 UUID 时有效 |

> 📖 **规范参考**：[PDF pp. 370-371](../_source/pages/page-370.md)

### 命令执行结果

| 执行条件 | 命令结果 | 说明 |
|---------|---------|------|
| 目标操作不可锁定 | ❌ `Prohibition of Command Execution Not Supported` | 该操作不支持锁定功能 |
| 选择了不存在的 Management Endpoint | ❌ `Invalid Field in Command` | 子系统中不包含 ME |
| PCIe 作用域 + Admin Queue 路径组合 | ❌ `Invalid Field in Command` | PCIe 命令只能在 ME 路径上锁定 |
| 请求的状态已经生效 | ✅ 成功（幂等操作） | 不改变状态，但命令执行成功 |
| 正常执行 | ✅ 成功 | 状态改变，子系统范围内生效 |

### 执行流程与强制机制

```text
步骤 1：发起锁定请求
   │
   │  指定：identifier + scope + interface + action
   │
   ▼
步骤 2：命令执行与验证
   │
   ├─ 验证目标可锁定性
   ├─ 验证接口路径有效性
   └─ 验证参数组合合法性
   │
   ▼
步骤 3：子系统范围内强制执行（Subsystem-wide Enforcement）
   │
   │  ⚠️ 关键特性：成功执行后，子系统内所有相关的
   │     控制器（Controllers）和管理端点（Management Endpoints）
   │     立即强制执行新的禁止规则
   │
   ▼
步骤 4：路径特定的错误响应
   │
   ├─ In-band 路径（Admin Queue）
   │  │
   │  │  被禁止的 Admin 命令或 Set Features 操作
   │  │  返回 NVMe 状态错误：
   │  └─ "Command Prohibited by Command and Feature Lockdown"
   │
   └─ Out-of-band 路径（Management Endpoint）
      │
      │  被禁止的 Management Interface 或 PCIe 命令
      └─ 返回管理接口 "Access Denied" 响应

步骤 5：解除禁止（可选）
   │
   ├─ 方式 1：子系统电源循环（Power Cycle）
   │          → 所有禁止状态清除，恢复默认
   │
   └─ 方式 2：显式执行 Allow 命令
              → 针对性解除特定操作的禁止
```

### 关键行为特性

1. **接口独立评估**  
   禁止状态在操作到达的接口上独立评估。同一个操作可以：
   - 在 Admin Queue 路径上被禁止
   - 在 Management Endpoint 路径上允许执行
   - 或任意组合

2. **全局一致性强制**  
   一旦成功执行禁止命令，子系统内所有适用的控制器和管理端点都会同步强制执行该规则，确保策略的一致性。

3. **状态持久性**  
   禁止状态会持续保持，不受控制器复位影响，直到：
   - 整个子系统电源循环（断电重启）
   - 或执行显式的 Allow 命令解除禁止

4. **幂等性保证**  
   重复执行相同的 Prohibit 或 Allow 命令不会产生错误，命令会成功返回，这简化了脚本化管理。

5. **路径限制规则**  
   PCIe 命令作用域只能与 Management Endpoint 路径组合，不能在 Admin Queue 路径上锁定。

> 📖 **规范参考**：[PDF pp. 370-372, 516](../_source/pages/page-370.md)

## 能力发现与验证

### 支持性检查

Lockdown 功能的支持通过控制器属性标识：

- **标识位**：`OACS.CFLS = 1`（Optional Admin Command Support: Command and Feature Lockdown Support）
- **要求**：该位置 1 表示同时支持 Lockdown 命令和 Lockdown 日志页面

### 发现流程

锁定日志页面是查询以下信息的权威入口：

1. **厂商特定的可锁定性**：哪些操作在当前子系统上可以被锁定
2. **当前禁止状态**：哪些操作在指定接口路径上已被禁止

通过组合不同的 Scope 和 Contents 选择器，可以全面了解子系统的锁定能力和当前配置。

> 📖 **规范参考**：[PDF p. 516](../_source/pages/page-516.md)

## 典型应用场景

### 安全加固场景

| 场景 | 锁定目标 | 效果 |
|------|---------|------|
| **生产系统部署** | 锁定格式化、固件更新等危险命令 | 防止误操作导致的数据丢失或系统故障 |
| **固件开发保护** | 锁定固件下载和激活命令 | 防止未授权的固件修改 |
| **配置冻结** | 锁定关键的 Set Features 操作 | 确保系统配置不被意外更改 |

### 多租户隔离场景

| 场景 | 策略 | 实现 |
|------|------|------|
| **租户权限分离** | In-band 路径禁止管理命令<br>Out-of-band 路径保留管理能力 | 租户通过 Admin Queue 无法执行管理操作<br>运维通过 Management Endpoint 保留完整管理权限 |
| **命名空间隔离** | 锁定跨命名空间的管理操作 | 限制租户只能管理自己的命名空间 |

### 合规与审计场景

| 场景 | 要求 | 实现方式 |
|------|------|---------|
| **访问控制合规** | 证明关键操作受到访问限制 | 通过日志页面导出当前禁止配置 |
| **变更管理** | 防止未经审批的配置变更 | 锁定配置相关的 Set Features 操作 |
| **最小权限原则** | 仅开放必要的管理接口 | 默认禁止所有可锁定操作，按需开放 |

### 运维安全场景

| 场景 | 风险 | 防护措施 |
|------|------|---------|
| **防止误操作** | 管理员执行错误命令 | 在维护窗口外锁定高风险命令 |
| **自动化脚本安全** | 脚本错误导致批量破坏 | 脚本运行环境仅开放必要的命令集 |
| **临时访问控制** | 第三方维护期间的权限管理 | 临时开放特定命令，完成后重新锁定 |

## 规范引用索引

以下是文档中引用的 NVMe 规范章节的完整列表：

| 主题 | 规范位置 | 链接 |
|------|---------|------|
| 功能目的与 UUID 限定的 Feature 作用域 | PDF p. 284 | [查看](../_source/pages/page-284.md) |
| 日志特定参数选择器（Log Specific Parameter Selector） | PDF p. 285 | [查看](../_source/pages/page-285.md) |
| 返回列表布局与排序规则 | PDF p. 286 | [查看](../_source/pages/page-286.md) |
| Effects Catalogs 与 Lockdown 的关系 | PDF pp. 281-286 | [查看](../_source/pages/page-281.md) |
| Lockdown 命令选择器与子系统强制执行机制 | PDF pp. 370-371 | [查看](../_source/pages/page-370.md) |
| Lockdown 命令状态码 | PDF p. 372 | [查看](../_source/pages/page-372.md) |
| 子系统持久性与接口特定的强制执行 | PDF p. 516 | [查看](../_source/pages/page-516.md) |
