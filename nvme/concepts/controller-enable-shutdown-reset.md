# 控制器启用、关机与重置（Controller Enable, Shutdown, and Reset）

## 一句话说明

NVMe 控制器的"开/关/重启"由 `CC`（配置）/`CSTS`（状态）/`NSSR`（子系统重置）/`NSSD`（子系统关机）这四组寄存器驱动：主机配 `CC`、看 `CSTS` 握手、靠 `NSSR` / `CC.EN` 复位、用 `CC.SHN` 或 `NSSD` 关机——每条路径都对状态、范围、恢复方式有明确规定。

## 生活化类比

把 NVMe 控制器想成一台**商用电饭煲**：

- **按下"开始"（`CC.EN=1`）** → 煲进入"加热中"，等指示灯变"就绪"（`CSTS.RDY=1`）后才能下米。
- **"保温"（`CC.SHN=01b`）** → 正常关机，把剩饭吹凉再断电；
- **"急停"（`CC.SHN=10b`）** → 突然关机，跳过等待，可能丢一点点状态。
- **"拔总闸"（`NSSR`）** → 把整个厨房（子系统）的电全断一次再上电；
- **"局部关电"（`NSSD` + `CAP.CPS=10b`）** → 只关这一楼（Domain）的电。

每次按"开始"前都先关掉加热开关再重置；按"开始"和按"急停"不能同时按。

## 工作流程

```text
配 CC + AQA / ASQ / ACQ
            │
            ▼
[Disabled]   CC.EN=0, CSTS.RDY=0
   │  写 CC.EN=1
   ▼
[Enabling]   等待 CSTS.RDY=1（须在 CAP.TO / CRTO 内完成）
   │ RDY=1                       超时 → 错误处理
   ▼
[Ready]       可提交命令
   │  写 CC.EN=0  或  NSSR.NSSRC="NVMe"  或  NSSD.NSSC="Nrml"/"Abpt"
   ▼
[Reset / Disabling / Shutting Down]
   │ CSTS.RDY=0；I/O 队列删除；Admin 队列复位
   ▼
[Disabled / Shutdown Complete（CSTS.SHST=10b，可安全断电）]
```

简化说明：关机 (`SHN`) 与禁用 (`EN=0`) 是两个**独立**维度；关机完成要看 `CSTS.ST`（`0`=控制器级，`1`=子系统级）决定后续走 Controller Reset 还是 NVM Subsystem Reset。

## 初学者案例

**场景：服务器准备断电维护，操作员关停 NVMe 控制器以确保数据落盘。**

1. 主机停止下发新 I/O 命令，等所有 outstanding 命令完成。
2. 删除 I/O 提交队列，再删除对应的 I/O 完成队列（顺序不可乱）。
3. 写 `CC.SHN = 01b`（正常关机）。
4. 轮询 `CSTS`：
   - `SHST = 01b` → 关机进行中；
   - `SHST = 10b` 且 `ST = 0` → 控制器级关机完成，可安全断电。
5. 若 `ST = 1` → 子系统级关机完成，必须再做 **NVM Subsystem Reset** 才能再启用控制器。
6. 故障速查：若 `SHST` 卡在 `01b` 超过 `CAP.TO` → 多半是控制器卡在介质刷写，需要走"急停" `CC.SHN=10b` 或子系统重置。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 启用前握手 | 配好 `CC` + `AQA` + `ASQ` + `ACQ` 后才能写 `CC.EN=1` |
| 队列属性 | `AQA` / `ASQ` / `ACQ` 仅在 `CC.EN=0` 时可改；Controller Reset 不清零这些寄存器 |
| 启动后等待 | `CC.EN=1` 到 `CSTS.RDY=1` 之间**禁止**提交命令 |
| 重复操作未定义 | Ready 时再写 `EN=1`、未 Ready 时再写 `EN=0` 均为**未定义行为** |
| 启动超时 | 启用过程须在 `CAP.TO` 或 `CRTO` 内完成；超时进入错误处理 |
| `SHN` 三态 | `00b` 无关机；`01b` 正常关机；`10b` 突然关机 |
| `SHST` 进度 | `00b` 正常；`01b` 关机中；`10b` 关机完成（可断电） |
| `ST` 范围 | `0` 控制器级关机（恢复走 Controller Reset + 重新启用）；`1` 子系统级关机（恢复必须先 NVM Subsystem Reset） |
| NVM Subsystem Reset | 仅当 `CAP.NSSRS=1` 时可由主机触发；写 `NSSR.NSSRC = 4E564D65h`（ASCII "NVMe"），读返回 0；其它值无功能效果 |
| NSSD | 写 `4E726D6Ch`（"Nrml"）= 正常子系统关机；写 `41627074h`（"Abpt"）= 突然子系统关机；读返回 0 |
| 关机范围 | `CAP.CPS=10b` 仅影响本 Domain；`CAP.CPS=11b` 影响整个 NVM 子系统 |
| `NSSES` 行为 | `CAP.NSSES=1` 时，子系统关机上报期间 Controller Reset **保留** `CSTS.ST/SHST`；`NSSES=0` 时 CLR 禁用 |
| 关机与禁用不可同时 | 关机时再清 `CC.EN` 会同时触发 Reset 与 Shutdown，行为不可预测 |
| 启动后命令路径 | Memory-Based：Admin 队列重建 → `CC.EN=1` → 等 RDY → 配 I/O 队列；Fabrics：Connect 建队列 |
| 传输层关联保留 | Fabrics 控制器做 CLR / 动态移除前，Association 至少保留 2 分钟 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 关机 vs 禁用 | 关机是"准备断电"（`SHN` + `SHST`）；禁用是"停止处理命令"（`EN=0`） |
| Controller Reset vs NVM Subsystem Reset | CLR 只重置本控制器；NSSR 重置整个子系统/Domain |
| `ST=0` vs `ST=1` | `ST=0` 控制器级关机走 CLR；`ST=1` 子系统级关机必须先 NSSR |
| `NSSR` vs `NSSD` | NSSR 是子系统**重置**（保持上电）；NSSD 是子系统**关机**（准备断电） |
| `CAP.NSSRS` vs `CAP.NSSES` | NSSRS = 是否支持主机主动子系统重置；NSSES = 子系统关机/启用状态是否能分离 |
| Memory-Based 关机 vs Fabrics 关机 | 前者按 SQ→CQ 顺序删除队列再 `SHN`；后者用 Property Set 写 SHN，控制器不主动断连 |
| 正常关机 vs 突然关机 | 正常保证数据落盘；突然尽力完成，可能丢少量数据（用于电源预警） |

## 进阶细节

- **`CC` 寄存器布局**（规范 3.1.4 / PDF 第 79-82 页）：
  - `EN`（bit 0）：1→0 触发 Controller Reset；可随时写；
  - `CSS`（bits 3:1）：I/O 命令集选择，**仅禁用时可改**；
  - `MPS`（bits 6:4）：页大小 = 2^(12+MPS) 字节，**仅禁用时可改**；
  - `AMS`（bits 10:7）：仲裁机制，**仅禁用时可改**；
  - `SHN`（bits 15:14）：关机通知，可随时写；
  - `IOSQES`（bits 19:16）/ `IOCQES`（bits 23:20）：I/O 队列条目大小 = 2^N 字节，禁用时或**无对应 I/O 队列时**可改；
  - `CRIME`（bit 24）：就绪模式；仅当 `CAP.CRMS=11b` 时为 RW，否则 RO=0；由 `CC.EN: 0→1` 时的值决定启用哪种 Ready 模式；切换此位会改变 `CAP.TO` 报告时间（规范 3.1.4 / PDF 第 79 页）。
- **`CSTS` 寄存器字段**（规范 3.1.5 / PDF 第 82-84 页）：
  - `RDY`（bit 0）：命令就绪握手；
  - `CFS`（bit 1）：控制器致命状态；
  - `SHST`（bits 3:2）：`00b` 正常 / `01b` 关机进行 / `10b` 关机完成；
  - `NSSRO`（bit 4）：观察到子系统重置；
  - `PP`（bit 5）：处理临时暂停；
  - `ST`（bit 6）：`0` Controller Shutdown / `1` NVM Subsystem Shutdown。
- **Admin Queue 初始化**（规范 3.1.4.8-10 / PDF 第 85-86 页）：
  - `AQA`：Admin CQ Size | Admin SQ Size（0-based，范围 2..4096，即实际 3..4097 entries）；
  - `ASQ` / `ACQ`：页对齐的物理基地址，物理连续；
  - 队列 ID 固定 `0h`；Admin CQ 固定使用中断向量 0；
  - `AQA` / `ASQ` / `ACQ` **不受 Controller Reset 清零**。
- **Memory-Based 控制器正常关机推荐步骤**（规范 3.7.2 / PDF 第 133-134 页）：
  1. 停止新 I/O；
  2. 等所有 outstanding 命令完成；
  3. 顺序删除 SQ 后删除 CQ；
  4. 写 `CC.SHN=01b`；
  5. 轮询 `CSTS.ST=0` 且 `SHST=10b`；
  6. 安全断电。
  - 进入 PCIe D3 前**应**走正常关机路径；
  - 关机期间**不要**再清 `CC.EN`，避免与 Controller Reset 冲突。
- **Fabrics 关机**（规范 3.7.2 / PDF 第 134-135 页）：
  - 用 Property Set 写 `CC.SHN`；
  - 主机可选主动断开传输或轮询 `CSTS.SHST`；
  - 关机期间**仅处理 Fabrics 命令**；**Keep Alive 禁用**；
  - CLR / 动态移除前 Association 在 `CC.EN` 清零后**至少保留 2 分钟**。
- **Subsystem Shutdown 流程**（规范 3.7.3 / PDF 第 135-138 页）：
  - `NSSD` 发起请求 → 范围（Domain 或整个子系统）内所有控制器进入 `ST=1, SHST=01b`；
  - 完成条件：**整个受影响范围就绪后**，关机完成才被任一控制器首次观察到，并在**所有范围内控制器**上报告；
  - 上报延迟若为 0h 至少 30 秒；
  - NVM Subsystem Reset 动作：中止关机 → 清除 `ST/SHST`（范围内）→ 对每个范围内控制器启动 CLR → 禁用范围内 PMR → 执行传输层特定重置。
- **NVM Subsystem Shutdown 期间的 Controller Reset 行为**（规范 3.7.3 / PDF 第 135-138 页）：

| 条件 | 行为 |
|------|------|
| `CAP.NSSES=1` | CLR 仍启动；非 NSSR 引发的 CLR **保留** `CSTS.ST/SHST`；控制器可独立重置 |
| `CAP.NSSES=0` | 上报期间 CLR 被禁用；完成后的传输层特定 CLR 可能清 `ST/SHST` |

- **重置级别对比**（规范 3.7.1 / 3.7.2 / 3.7.3）：

| 级别 | 触发 | 作用 |
|------|------|------|
| NVM Subsystem Reset | 上电；写 `NSSR.NSSRC="NVMe"`（`NSSRS=1`）；管理方法/厂商事件 | 重置子系统/Domain 范围；范围内 CLR；禁用 PMR |
| Controller Level Reset | NSSR；`CC.EN: 1→0`；传输层重置 | 停止命令；删 I/O 队列；`RDY=0`；重置属性/状态（部分例外） |
| Queue Level Reset | 删后重建 I/O 队列 | 仅改队列属性，不动控制器 |

- **CLR 后的保留项**（规范 3.7.2 / PDF 第 139-141 页）：
  - Memory-Based：保留 Admin Queue 属性、PMR 属性、`CMBMSC` 寄存器；
  - Function Level Reset：仅保留 `CMBMSC`；
  - Message-Based（Fabrics）：CLR 无属性例外。
- **CRTO / CRIMT / CRWMT**（规范 3.5.3 / 3.5.4 / PDF 第 92 页）：
  - `CRIMT`（Controller Ready Independent of Media Timeout）：不依赖介质就绪模式的超时，500 ms 单位；
  - `CRWMT`（Controller Ready With Media Timeout）：含介质初始化的完整超时，≥ CRIMT，500 ms 单位；
  - 模式由 `CAP.CRMS` + `CC.CRIME` 共同决定（详见 `controller-ready-modes.md`）。
- **章节号**：CC 在 3.1.4 / 3.4.1；CSTS 在 3.1.5；NSSR 在 3.1.4.7；AQA/ASQ/ACQ 在 3.1.4.8-10；NSSD 在 3.1.4.x / 3.7.3；启动/禁用握手在 3.5.1-3.5.2；关机在 3.7.2-3.7.3；CRTO 在 3.5.3-3.5.4；Capabilities 在 3.1.3。

## 规范依据

- [CC 字段定义，PDF 第 79 页](../_source/pages/page-079.md)
- [CSTS 字段定义，PDF 第 82 页](../_source/pages/page-082.md)
- [NSSR 与 Admin Queue 属性，PDF 第 85 页](../_source/pages/page-085.md)
- [NSSD 与 Subsystem Shutdown，PDF 第 91 页](../_source/pages/page-091.md)
- [CRTO / CRIMT / CRWMT，PDF 第 92 页](../_source/pages/page-092.md)
- [关机交互矩阵，PDF 第 132 页](../_source/pages/page-132.md)
- [Memory-Based 关机与重启规则，PDF 第 133 页](../_source/pages/page-133.md)
- [Fabrics 关机与 Association 生命周期，PDF 第 134 页](../_source/pages/page-134.md)
- [NVM Subsystem Reset 与 Controller Level Reset 契约，PDF 第 139 页](../_source/pages/page-139.md)

## 相关阅读

- [controller-initialization.md](controller-initialization.md) - 启用前的启动与寄存器握手
- [controller-ready-modes.md](controller-ready-modes.md) - Ready 模式与 CRTO 超时
- [domains-and-divisions.md](domains-and-divisions.md) - 关机范围受 Domain 约束
- [keep-alive.md](keep-alive.md) - KATO 超时后控制器置 CFS 流程
