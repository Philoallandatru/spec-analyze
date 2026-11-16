# 控制器启用、关机与重置（Controller Enable, Shutdown, and Reset）

NVMe 控制器的启用、关机与重置由四组核心寄存器驱动：**`CC`**（配置）、**`CSTS`**（状态）、**`NSSR`**（子系统重置）和 **`NSSD`**（子系统关机）。主机通过它们完成“上电配置 → 握手就绪 → 安全关机 / 复位恢复”的全生命周期控制。[PDF pp. 79-85](../_source/pages/page-079.md)

可以把它想成**商用电饭煲的控制面板**：写 `CC.EN=1` 是按下“开始”，等 `CSTS.RDY=1` 是指示灯变绿；`CC.SHN` 区分“正常保温落盘关机”和“紧急断电”；`CC.EN=0` 是单机复位，而 `NSSR` 则是“拔掉整个厨房总闸”。

## 启用 / 禁用状态机

```text
           第一步：CC.EN=0 时配置 CC + AQA / ASQ / ACQ
                              |
                              v
[Disabled 禁用态]  CC.EN=0, CSTS.RDY=0
       |  写 CC.EN=1
       v
[Enabling 启用中]  等待 CSTS.RDY=1（须在 CAP.TO / CRTO 超时前完成）── 超时 ─> 错误处理
       |  CSTS.RDY=1
       v
[Ready 就绪态]     可正常提交 Admin 与 I/O 命令
       |  写 CC.EN=0（或 NSSR / 传输层重置）
       v
[Resetting 复位中] 停止命令、删除所有 I/O 队列、复位 Admin 队列 → CSTS.RDY=0 → [Disabled]
```

[PDF pp. 82, 84](../_source/pages/page-082.md)

**核心规则**：
1. 在 `CC.EN=1` 之后、`CSTS.RDY=1` 之前，主机**禁止**提交任何命令。
2. 在 Ready 态重复写 `CC.EN=1`，或在 Not Ready 态重复写 `CC.EN=0`，均为**未定义行为**。
3. **Admin 队列寄存器（`AQA`/`ASQ`/`ACQ`）** 仅在 `CC.EN=0` 时可修改（`AQA` 0-based 范围 `2..4096` 条目，`ASQ`/`ACQ` 须页对齐且物理连续，Admin CQ 固定使用中断向量 0）；Controller Reset **不会**清零这组寄存器。[PDF pp. 82, 85-86](../_source/pages/page-085.md)

## 核心寄存器：`CC` 与 `CSTS`

### `CC`（Controller Configuration）

```text
位域： 31:25   24      23:20    19:16   15:14  13:11  10:07  06:04  03:01  00
     +-----+-------+--------+--------+------+------+------+------+------+--+
     |  R  | CRIME | IOCQES | IOSQES | SHN  | 保留 | AMS  | MPS  | CSS  |EN|
     +-----+-------+--------+--------+------+------+------+------+------+--+
```

| 字段 | 名称与说明 | 允许修改时机 |
|---|---|---|
| `EN` | `1→0` 触发 Controller Reset | 任何时候 |
| `CSS` | 选择 I/O 命令集（需 `CAP.CSS` 支持） | 仅禁用时（`CC.EN=0`） |
| `MPS` | 内存页大小 = $2^{(12+\text{MPS})}$ 字节 | 仅禁用时 |
| `AMS` | 仲裁机制选择 | 仅禁用时 |
| `SHN` | 关机通知：`00b`=无，`01b`=正常关机，`10b`=突然关机 | 任何时候 |
| `IOSQES` / `IOCQES` | I/O SQ/CQ 条目大小 = $2^N$ 字节 | 禁用时或无对应 I/O 队列时 |
| `CRIME` | 就绪模式（`0`=依赖介质，`1`=不依赖介质），仅当 `CAP.CRMS=11b` 时可写 | 禁用时（切换会改变 `CAP.TO`） |

[PDF pp. 79-82](../_source/pages/page-079.md)

### `CSTS`（Controller Status）

| 字段 | 含义 | 关键状态值 |
|---|---|---|
| `RDY` (bit 0) | 就绪标志 | `1`=可处理命令；`0`=未就绪 |
| `CFS` (bit 1) | 控制器致命错误 | `1`=无法继续报告错误，需重置 |
| `SHST` (bits 3:2) | 关机进度 | `00b`=正常，`01b`=关机中，`10b`=关机完成（可安全断电） |
| `NSSRO` (bit 4) | 子系统重置观察 | `1`=观察到 NVM Subsystem Reset |
| `PP` (bit 5) | 处理暂停 | `1`=命令处理临时暂停 |
| `ST` (bit 6) | 关机类型 | `0`=Controller Shutdown（走 CLR 恢复）；`1`=Subsystem Shutdown（须先 NSSR） |

[PDF pp. 82-84](../_source/pages/page-082.md)

## 关机流程（Shutdown）

**`CC.EN`/`CSTS.RDY` 与 `CSTS.SHST` 是两个独立维度**：关机时控制器可能处于 `EN/RDY=1/1` 或 `0/0`。但**禁止在发起关机的同时清零 `CC.EN`**，否则会引发关机与重置冲突。[PDF pp. 132-133](../_source/pages/page-132.md)

### 1. 控制器级关机（Memory-Based vs Fabrics）

- **Memory-Based 正常关机（推荐用于进入 PCIe D3 前）**：停止新 I/O → 等待未完成命令结束 → **先删 SQ 后删 CQ** → 写 `CC.SHN=01b` → 轮询 `CSTS.ST=0, SHST=10b` → 安全断电。
- **Memory-Based 突然关机**：跳过队列清空，直接写 `CC.SHN=10b` → 等 `SHST=10b`（尽力完成，可能丢失部分数据）。[PDF pp. 133-134](../_source/pages/page-133.md)
- **Fabrics 关机**：通过 Property Set 写 `CC.SHN`，主机可主动断开传输或轮询 `SHST`；关机期间仅处理 Fabrics 命令并禁用 Keep Alive；CLR 或动态移除前 Association 在 `CC.EN` 清零后至少保留 **2 分钟**。[PDF pp. 134-135](../_source/pages/page-134.md)
- **恢复方式（`ST=0`）**：执行 Controller Reset 后重新配置启用；若当时 `CC.EN=0`，也可通过单次写同时置 `EN=1` 并清 `SHN=00b` 来恢复。

### 2. NVM 子系统关机（`NSSD`）

写入 `NSSD.NSSC` 触发（`4E726D6Ch`="Nrml" 正常，`41627074h`="Abpt" 突然，读取返回 0），会**覆盖**正在进行的控制器关机：[PDF p. 91](../_source/pages/page-091.md)
- **范围**：`CAP.CPS=10b` 影响当前 Domain，`11b` 影响整个子系统。只有当**整个受影响范围就绪**后，范围内所有控制器才会同步报告 `ST=1, SHST=10b`。[PDF pp. 135-138](../_source/pages/page-135.md)
- **恢复方式（`ST=1`）**：普通启用无效，**必须先执行 NVM Subsystem Reset（NSSR）**。
- **`CAP.NSSES` 影响**：`NSSES=1` 时子系统关机期间允许独立执行 CLR 并保留 `ST/SHST`；`NSSES=0` 时 CLR 被禁用。

## 重置级别对比

| 重置级别 | 触发方式 | 作用与保留状态 |
|---|---|---|
| **NVM Subsystem Reset (NSSR)** | 上电；写 `NSSR.NSSRC = 4E564D65h` ("NVMe"，需 `CAP.NSSRS=1`)；管理事件 | 中止关机、清 `ST/SHST`、对范围内控制器启动 CLR、禁用 PMR、触发传输层重置（PCIe 可能 Link Down） [PDF p. 85](../_source/pages/page-085.md) |
| **Controller Level Reset (CLR)** | `CC.EN: 1→0`；NSSR；传输层重置（如 FLR） | 停止命令、删 I/O 队列、`RDY=0`。Memory-Based 保留 Admin 队列属性、PMR 属性和 `CMBMSC`；FLR 仅保留 `CMBMSC`；Fabrics 无例外 [PDF pp. 139-141](../_source/pages/page-139.md) |
| **Queue Level Reset** | 删除后重建 I/O SQ/CQ | 仅重置特定队列，不影响控制器状态 |

## 关联与引申概念

- **控制器初始化（[Controller Initialization](controller-initialization.md)）**：重置后重新配置 `AQA`/`ASQ`/`ACQ`、`CC` 并建立 I/O 队列的完整引导流程。
- **就绪模式与超时（[Controller Ready Modes](controller-ready-modes.md)）**：`CAP.CRMS`、`CC.CRIME` 与 `CRTO`（`CRIMT`/`CRWMT`）共同决定 `CC.EN=1` 后等待 `CSTS.RDY=1` 的超时上限。[PDF p. 92](../_source/pages/page-092.md)
- **Domain 与子系统划分（[Domains and Divisions](domains-and-divisions.md)）**：`CAP.CPS` 决定了 `NSSD` 和 `NSSR` 在多 Domain 子系统中的影响边界。
- **Keep Alive（[Keep Alive](keep-alive.md)）**：心跳超时会置位 `CSTS.CFS=1`，通常需要通过 CLR 或 NSSR 恢复。
- **通信丢失与重试（[Communication Loss and Command Retry](communication-loss-and-command-retry.md)）**：阐述重置优先级（优先 CLR，谨慎使用 NSSR 以免 PCIe Link Down）。

## 规范依据

- [CC 与 CSTS 寄存器字段，PDF 第 79-84 页](../_source/pages/page-079.md)
- [NSSR 与 Admin Queue 属性，PDF 第 85-86 页](../_source/pages/page-085.md)
- [NSSD 与 CRTO 定义，PDF 第 91-92 页](../_source/pages/page-091.md)
- [关机交互矩阵与 Memory-Based/Fabrics 关机，PDF 第 132-135 页](../_source/pages/page-132.md)
- [Subsystem/Domain Shutdown 范围与完成条件，PDF 第 135-138 页](../_source/pages/page-135.md)
- [NVM Subsystem 与 Controller Level Reset 契约，PDF 第 139-141 页](../_source/pages/page-139.md)
