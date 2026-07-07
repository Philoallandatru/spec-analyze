# 控制器启用、关机与重置

## 概述

NVMe 控制器（Controller）的启用、关机和重置操作是主机（Host）与控制器之间通过寄存器握手完成的核心流程。主机通过配置 `CC`（Controller Configuration，控制器配置寄存器）和读取 `CSTS`（Controller Status，控制器状态寄存器）来完成这些操作。

**核心机制：**

- **控制器启用**：主机配置好队列参数后，通过设置 `CC.EN=1` 启动控制器，等待 `CSTS.RDY=1` 表示就绪
- **关机流程**：通过 `CC.SHN` 字段发起正常或突然关机，确保数据持久化后再断电
- **重置操作**：通过清除 `CC.EN` 触发控制器重置，或通过 `NSSR` 寄存器触发子系统重置

**关键约束：**

- 队列配置（Queue Configuration）只能在控制器禁用（Disabled）状态下进行
- 关机完成后，需要执行相应级别的重置才能恢复命令执行
- 不同的关机类型（控制器关机 vs 子系统关机）需要不同的恢复路径

[规范参考：PDF pp. 79-85](../_source/pages/page-079.md)

---

## 启用/禁用状态机

控制器的启用和禁用遵循严格的状态转换规则，主机必须按照状态机的要求操作：

```text
                 第一步：配置 CC + AQA/ASQ/ACQ
                            |
                            v
[Disabled 禁用态] CC.EN=0, CSTS.RDY=0
     |  第二步：将 CC.EN 设为 1
     v
[Enabling 启用中] ────── 等待就绪（需在 CAP.TO / CRTO 超时前完成）───┐
     | CSTS.RDY=1                                            超时 |
     v                                                          v
[Ready 就绪态] 可提交命令                                   进入错误处理
     | 第三步：清除 CC.EN=0
     v
[Controller Reset / Disabling 重置/禁用中]
     | 删除所有 I/O 队列；复位 Admin 队列
     | CSTS.RDY=0
     └─────────────────────────────> [Disabled 禁用态]
```

**重要规则：**

1. **等待就绪**：在启用控制器（设置 `CC.EN=1`）之后、`CSTS.RDY=1` 之前，主机**不得**提交任何命令
2. **避免重复操作**：
   - 在控制器已就绪（Ready）时重复设置启用是**未定义行为**
   - 在控制器未就绪（Not Ready）时重复禁用也是**未定义行为**
3. **超时处理**：启用过程必须在 `CAP.TO` 或 `CRTO` 规定的时间内完成，否则需要进行错误处理

[规范参考：PDF pp. 82, 84](../_source/pages/page-082.md)

---

## 控制器配置寄存器（CC）

`CC` 寄存器是主机配置控制器行为的主要接口，包含多个关键字段：

### 寄存器布局

```text
位域：  31:25   24      23:20   19:16  15:14  13:11  10:07  06:04  03:01  00
      +-----+-------+-------+-------+------+-------+------+-------+------+--+
      |  R  | CRIME |IOCQES |IOSQES | SHN  | AMS   | MPS  | CSS   |  R   |EN|
      +-----+-------+-------+-------+------+-------+------+-------+------+--+
```

### 字段说明

| 字段 | 名称 | 说明 | 修改时机 |
|------|------|------|---------|
| **EN** | Enable | 控制命令处理的启用/禁用。<br>• `1` → `0`：触发控制器重置（Controller Reset） | 任何时候 |
| **CSS** | I/O Command Set Selected | 选择 I/O 命令集（如 NVM Command Set） | 仅在禁用时 |
| **MPS** | Memory Page Size | 内存页大小（2^(12+MPS) 字节） | 仅在禁用时 |
| **AMS** | Arbitration Mechanism Selected | 仲裁机制选择 | 仅在禁用时 |
| **SHN** | Shutdown Notification | 关机通知：<br>• `00b`：无关机<br>• `01b`：正常关机<br>• `10b`：突然关机 | 任何时候 |
| **IOSQES** | I/O Submission Queue Entry Size | I/O 提交队列条目大小（2^IOSQES 字节） | 禁用时或无 I/O 队列时 |
| **IOCQES** | I/O Completion Queue Entry Size | I/O 完成队列条目大小（2^IOCQES 字节） | 禁用时或无 I/O 队列时 |
| **CRIME** | Controller Ready Independent of Media Enable | 选择就绪模式：<br>• 不依赖介质就绪<br>• 依赖介质就绪 | 仅当 `CAP.CRMS` 支持时 |

### 配置约束

- **队列条目大小**：必须是 2 的幂，一旦存在对应的 I/O 队列就不应修改
- **能力支持**：`AMS`、`MPS`、`CSS` 的设置值必须被 `CAP`（Controller Capabilities）寄存器支持
- **修改时机**：大部分字段只能在控制器禁用（`CC.EN=0`）时修改

[规范参考：PDF pp. 79-82](../_source/pages/page-079.md)

---

## 控制器状态寄存器（CSTS）

`CSTS` 寄存器反映控制器的当前运行状态，主机通过轮询该寄存器来监控状态变化：

### 寄存器布局与字段

```text
CSTS: [ ST | PP | NSSRO | SHST(2位) | CFS | RDY ]
        │    │     │        │         │     └── RDY: 命令就绪握手信号
        │    │     │        │         └──────── CFS: 致命错误状态
        │    │     │        └────────────────── SHST: 关机进度（2位）
        │    │     └─────────────────────────── NSSRO: 子系统重置已观察到
        │    └───────────────────────────────── PP: 处理临时暂停
        └────────────────────────────────────── ST: 关机类型标识
```

### 字段详解

| 字段 | 名称 | 状态值 | 含义 |
|------|------|--------|------|
| **RDY** | Ready | `0` / `1` | 控制器未就绪 / 已就绪，可处理命令 |
| **CFS** | Controller Fatal Status | `1` | 控制器进入致命错误状态 |
| **SHST** | Shutdown Status | `00b` | 正常操作（无关机） |
|  |  | `01b` | 关机进行中 |
|  |  | `10b` | 关机已完成 |
| **NSSRO** | NVM Subsystem Reset Occurred | `1` | 观察到子系统重置 |
| **PP** | Processing Paused | `1` | 命令处理临时暂停 |
| **ST** | Shutdown Type | `0` | 控制器级关机（Controller Shutdown） |
|  |  | `1` | NVM 子系统关机（Subsystem Shutdown） |

### 关机状态解读

- **正常操作**：`SHST=00b` 表示控制器正常运行
- **关机中**：`SHST=01b` 表示正在执行关机流程
- **关机完成**：`SHST=10b` 表示关机完成，可以安全断电
- **关机类型**：
  - `ST=0`：控制器级关机，需要 **Controller Level Reset + 重新启用**
  - `ST=1`：子系统级关机，需要 **NVM Subsystem Reset + 重新启用**

[规范参考：PDF pp. 82-84](../_source/pages/page-082.md)

---

## 关机流程

NVMe 提供两种关机方式：**正常关机**（Normal Shutdown）和**突然关机**（Abrupt Shutdown）。

### 关机路径

```text
正常关机:  CC.SHN=01b ──> CSTS.SHST=01b ──> CSTS.SHST=10b
突然关机:  CC.SHN=10b ──> CSTS.SHST=01b ──> CSTS.SHST=10b

关机完成后的恢复路径：
├─ ST=0 (控制器关机) ──> Controller Level Reset + 重新启用
└─ ST=1 (子系统关机) ──> NVM Subsystem Reset + 重新启用
```

### 关机类型对比

| 特性 | 正常关机（Normal） | 突然关机（Abrupt） |
|------|-------------------|-------------------|
| **触发方式** | `CC.SHN=01b` | `CC.SHN=10b` |
| **准备时间** | 较长，确保数据持久化 | 较短，快速关机 |
| **适用场景** | 正常关闭系统、进入低功耗模式 | 紧急情况、电源丢失预警 |
| **数据安全** | 保证所有数据写入持久存储 | 尽力完成，但可能丢失部分数据 |

### 关机与启用的交互

**重要注意事项：**

1. **避免冲突**：在发起关机的同时清除 `CC.EN` 可能导致关机和控制器重置同时发生，产生不可预测的行为
2. **子系统关机的限制**：当子系统关机正在进行或已完成时（`ST=1`），普通的启用操作**无法**使控制器变为就绪状态，必须先执行 **NVM Subsystem Reset**
3. **独立维度**：启用状态（`CC.EN` / `CSTS.RDY`）和关机状态（`CSTS.SHST`）是相关但独立的两个维度
   - 关机时 `EN/RDY` 可能是 `0/0`（已禁用）
   - 关机时 `EN/RDY` 也可能是 `1/1`（仍启用）

[规范参考：PDF pp. 80, 82-84](../_source/pages/page-080.md)

---

## 重置机制

NVMe 提供多个级别的重置机制，用于不同的恢复场景：

### NVM Subsystem Reset 触发

如果控制器能力 `CAP.NSSRS=1`（支持子系统重置），可以通过以下方式触发：

```text
写入 NSSR.NSSRC = 4E564D65h（ASCII "NVMe"）──> 启动 NVM Subsystem Reset
写入其他值                              ──> 无功能性效果
读取 NSSR.NSSRC                        ──> 返回 0
```

[规范参考：PDF p. 85](../_source/pages/page-085.md)

### NVM Subsystem Shutdown Property

通过 `NSSD` 寄存器可以触发子系统级别的关机：

```text
写入 NSSD.NSSC = 4E726D6Ch（ASCII "Nrml"）──> 正常子系统关机
写入 NSSD.NSSC = 41627074h（ASCII "Abpt"）──> 突然子系统关机
写入其他值                              ──> 无功能性效果
读取 NSSD.NSSC                        ──> 返回 0
```

**影响范围：**

- `CAP.CPS=10b`：仅影响该控制器所在的 Domain
- `CAP.CPS=11b`：影响整个 NVM 子系统

[规范参考：PDF p. 91](../_source/pages/page-091.md)

---

## 控制器就绪超时

控制器提供两个超时参数来限定就绪时延：

### 超时参数

| 参数 | 全称 | 说明 | 单位 |
|------|------|------|------|
| **CRTO.CRIMT** | Controller Ready Independent of Media Timeout | 不依赖介质模式下的就绪超时<br>（仅需要不访问介质的命令就绪） | 500 ms |
| **CRTO.CRWMT** | Controller Ready With Media Timeout | 包含附加命名空间/介质的完整就绪超时<br>（≥ CRIMT） | 500 ms |

### 就绪模式

控制器支持两种就绪模式（由 `CAP.CRMS` 能力位和 `CC.CRIME` 配置位控制）：

1. **依赖介质就绪模式**（Ready With Media）：控制器需要完成介质初始化才能就绪
2. **不依赖介质就绪模式**（Ready Independent of Media）：控制器可以在介质初始化完成前就绪，部分命令立即可用

相关映射关系详见：[Controller Ready Modes and Timeouts](controller-ready-modes.md)

[规范参考：PDF pp. 92, 128-131](../_source/pages/page-092.md)

---

## 关机处理详解

### 关机状态转换

```text
正常操作（SHST=00b）
       │ 主机写入 CC.SHN=01b / 10b，或通过 NSSD 请求
       v
Shutdown 进行中（SHST=01b）
       │ 可通过 Predictable Latency Namespace (PLN) abort 命令中止
       │ 等待关机完成
       v
Shutdown 完成（SHST=10b）──> 可以安全断电

关机类型标识：
├─ ST=0: Controller Shutdown（控制器关机）
└─ ST=1: Subsystem Shutdown（子系统关机，覆盖控制器关机）
```

### 关机并发规则

**单一关机机制**：一个控制器同时只能有一种关机机制生效

- **覆盖规则**：NVM Subsystem Shutdown 请求会覆盖正在进行的 Controller Shutdown
- **状态独立性**：`CC.EN` / `CSTS.RDY` 与 `CSTS.SHST` 是独立维度
  - 启用状态可能是 `0/0`（禁用）或 `1/1`（启用）
  - 无论启用状态如何，`SHST` 都能独立反映关机状态

[规范参考：PDF pp. 132-133](../_source/pages/page-132.md)

### Memory-Based 传输的关机流程

**正常关机（Normal Shutdown）推荐步骤：**

1. **停止新的 I/O**：主机停止提交新命令
2. **等待未完成命令**：让所有 outstanding 命令完成
3. **删除队列**：按照先删除 Submission Queue、后删除 Completion Queue 的顺序删除队列
4. **发起关机**：写入 `CC.SHN=01b`
5. **等待完成**：轮询直到 `CSTS.ST=0, SHST=10b`
6. **安全断电**：可以移除电源

**突然关机（Abrupt Shutdown）步骤：**

1. **跳过队列清空**：直接写入 `CC.SHN=10b`
2. **等待完成**：轮询直到 `CSTS.SHST=10b`
3. **断电**：可以移除电源

**重要注意事项：**

- **进入 PCIe D3 前**：应遵循正常关机路径
- **避免在关机期间禁用**：通过 `CC.EN` 禁用控制器会触发 Controller Reset，与关机流程冲突
- **通信丢失防范**：在所有 completion 或关机完成前发生通信丢失，需要遵守通信丢失防范措施

[规范参考：PDF pp. 133-134](../_source/pages/page-133.md)

### Fabrics 传输的关机流程

**关机步骤：**

1. **发起关机**：通过 Property Set 命令写入 `CC.SHN`
2. **主机选项**：
   - 在传输层上主动断开连接
   - 或轮询 `CSTS.SHST` 等待完成
3. **控制器行为**：控制器不会主动发起断开

**关机期间的限制：**

- 仅处理 Fabrics 命令
- 禁用 Keep Alive 机制

**Controller Level Reset 或动态控制器移除前：**

- 关联（Association）在 `CC.EN` 清零后至少保留 **2 分钟**

[规范参考：PDF pp. 134-135](../_source/pages/page-134.md)

### 关机完成后的恢复

**Controller Shutdown 完成后（ST=0）：**

命令执行只能通过以下方式恢复：

1. **指定的重置/重新启用路径**：
   - 执行 Controller Reset（`CC.EN=0`）
   - 重新配置并启用（`CC.EN=1`）

2. **单一写操作恢复**（如果 `CC.EN=0`）：
   - 通过单次写操作同时设置 `EN=1` 并清除 `SHN=00b`
   - 随后完成初始化

**重要提醒：**

在所有 completion 或 shutdown-complete 状态之前发生通信丢失，必须遵守通信丢失防范措施，因为 outstanding 命令可能通过另一个控制器与后续命令产生交互。

[规范参考：PDF pp. 134-135](../_source/pages/page-134.md)

---

## NVM Subsystem Shutdown 与 Reset

### Subsystem Shutdown 流程

```text
NSSD 发起正常/突然关机请求
           │
           v
[ST=1, SHST=01b：Subsystem/Domain Shutdown 进行中]
           │ 等待上报的延迟（如果为 0h，则至少 30 秒）
           v
[ST=1, SHST=10b，在受影响范围内的每个控制器上都显示]
           │ 受影响范围现在可以安全断电
           v
NVM Subsystem Reset ──> 中止关机、清除 ST/SHST、重置所有范围内控制器
```

### 受影响范围

| CAP.CPS 值 | 受影响范围 |
|-----------|-----------|
| `10b` | 发起关机的控制器所在的 Domain |
| `11b` | 整个 NVM 子系统（单 Domain 子系统） |

**完成条件：**

- 仅当**整个受影响范围就绪**时，关机完成才会首次被任一控制器观察到
- 该完成状态会在范围内**所有控制器**上报告

[规范参考：PDF pp. 135-139](../_source/pages/page-135.md)

### Subsystem Shutdown 期间的 Controller Reset 行为

| 条件 | 行为说明 |
|------|---------|
| **CAP.NSSES=1**<br>（支持子系统关机和启用状态分离） | • Controller Reset 仍会启动 CLR<br>• 非子系统重置的 CLR **保留** `CSTS.ST/SHST`<br>• 控制器可以独立重置而不影响子系统关机状态 |
| **CAP.NSSES=0**<br>（不支持分离） | • 关机上报期间 Controller Reset 被禁用<br>• 完成后的传输层特定 CLR 可能清除 `ST/SHST` |

### 推荐的 Host 行为

**便携性最佳实践：**

在每次 NVM Subsystem Shutdown 之后执行一次 **NVM Subsystem Reset**

**NVM Subsystem Reset 的作用：**

1. 中止正在进行的关机
2. 清空 `ST/SHST`（在其范围内）
3. 对每个范围内控制器启动 Controller Level Reset
4. 禁用其范围内的 PMR（Persistent Memory Region）
5. 执行传输层特定的重置动作

**重置范围：**

- **单 Domain 重置**：覆盖整个子系统
- **多 Domain 子系统**：可能仅覆盖一个 Domain，或覆盖整个子系统（取决于实现）

[规范参考：PDF pp. 135-140](../_source/pages/page-135.md)

---

## 重置级别对比

| 重置级别 | 触发方式 | 主要作用 |
|---------|---------|---------|
| **NVM Subsystem Reset**<br>（子系统重置） | • 系统上电<br>• 写入 `NSSR.NSSRC`（当支持时）<br>• 管理方法或厂商事件 | • 重置子系统/Domain 范围<br>• 对所有范围内控制器启动 CLR<br>• 禁用范围内的 PMR |
| **Controller Level Reset**<br>（控制器级重置） | • 子系统重置<br>• `CC.EN: 1 → 0`<br>• 传输层特定重置 | • 停止命令<br>• 删除所有 I/O 队列<br>• 控制器回到空闲状态，`RDY=0`<br>• 重置属性/状态（部分例外） |
| **Queue Level Reset**<br>（队列级重置） | 删除后重建 I/O 队列或队列对 | 只更改队列属性，不重置控制器 |

### Controller Reset 后的恢复

对于基于内存的控制器执行 Controller Reset：

- **保留项**：Admin Queue 属性、PMR 属性、`CMBMSC` 寄存器
- **Function Level Reset 保留项**：仅保留 `CMBMSC`
- **基于消息的控制器**：CLR 没有属性例外

**恢复步骤：**

1. 恢复传输层/属性状态
2. 启用控制器并等待就绪
3. 使用 Admin 命令重新配置
4. 重建 I/O 队列

[规范参考：PDF pp. 139-141](../_source/pages/page-139.md)

---

## Admin Queue 初始化

Admin Queue（管理队列）是控制器的核心队列，用于执行管理命令。初始化步骤如下：

### 初始化流程

```text
前提条件：CC.EN=0（控制器处于禁用状态）

步骤 1：配置 Admin Queue 属性
  AQA ← 设置 Admin Completion Queue Size | Admin Submission Queue Size
        （零基值，范围 2..4096 entries）
  
步骤 2：配置 Admin Submission Queue 地址
  ASQ ← 设置页对齐的物理基地址

步骤 3：配置 Admin Completion Queue 地址
  ACQ ← 设置页对齐的物理基地址（中断向量为 0）

步骤 4：配置控制器并启用
  CC  ← 设置合法的 entry sizes / modes / EN=1
  
步骤 5：等待就绪
  轮询 CSTS.RDY，等待变为 1
```

### Admin Queue 特性

| 特性 | 说明 |
|------|------|
| **Queue Identifier** | 固定为 0 |
| **内存布局** | 物理地址连续 |
| **修改时机** | `AQA`、`ASQ`、`ACQ` 只能在 `CC.EN=0`（禁用状态）时修改 |
| **重置保留** | 在 Controller Reset 中，这些配置会被保留 |
| **队列大小** | 2 到 4096 个条目（零基值，实际大小为配置值+1） |
| **中断向量** | Admin Completion Queue 固定使用中断向量 0 |

[规范参考：PDF pp. 82, 85-86](../_source/pages/page-082.md)

---

## 规范参考

本文档内容基于以下 NVMe 规范章节：

- [Controller Configuration 字段详解，PDF pp. 79-82](../_source/pages/page-079.md)
- [Controller Status 字段详解，PDF pp. 82-84](../_source/pages/page-082.md)
- [NVM Subsystem Reset 与 Admin Queue 属性，PDF p. 85](../_source/pages/page-085.md)
- [Admin Completion Queue 与 CMB Location，PDF p. 86](../_source/pages/page-086.md)
- [NVM Subsystem Shutdown 与 Controller Ready Timeouts，PDF pp. 91-92](../_source/pages/page-091.md)
- [Shutdown 交互矩阵，PDF pp. 132-133](../_source/pages/page-132.md)
- [Memory-Based Shutdown 与 Restart 规则，PDF pp. 133-134](../_source/pages/page-133.md)
- [Fabrics Shutdown 与 Association 生命周期，PDF pp. 134-135](../_source/pages/page-134.md)
- [Subsystem/Domain Shutdown 范围与完成条件，PDF pp. 135-138](../_source/pages/page-135.md)
- [NVM Subsystem 与 Controller Level Reset 契约，PDF pp. 139-141](../_source/pages/page-139.md)
