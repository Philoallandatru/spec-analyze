# 控制器初始化（Controller Initialization）

控制器初始化是主机建立与 NVMe 控制器通信通道的引导过程：在基于内存的传输（PCIe）与基于消息的传输（Fabrics）上采用不同的管理通道建链方式，随后统一执行**“识别能力 → 枚举命名空间 → 创建 I/O 队列 → 挂起异步事件（AER）”**。[PDF pp. 124-127](../_source/pages/page-124.md)

可以把它想成**新员工入职**：本地员工（PCIe）先铺好桌面表格（`AQA`/`ASQ`/`ACQ`）再打卡上班（`CC.EN=1`）；远程员工（Fabrics）先连视频会议（Connect）并刷工牌认证（`AUTHREQ`）再打卡；上班后两人都走同一套流程——自我介绍（Identify）、认领项目（Namespace）和工单（I/O 队列）。

## 初始化流程总览

```text
基于内存的传输 (PCIe)                    基于消息的传输 (Fabrics)
─────────────────────────────            ─────────────────────────────
1. 等待 CSTS.RDY=0（上次复位结束）        1. 建立传输层连接
2. 在 CC.EN=0 时配 AQA / ASQ / ACQ       2. 发 Connect 命令建 Admin 队列 + Association
3. 依据 CAP.CSS 选 CC.CSS；配 AMS/MPS    3. 若 AUTHREQ != 0，完成带内身份认证
   以及合法的 IOSQES / IOCQES            4. Property Get/Set CAP & CC；写 CC.EN=1
4. 写 CC.EN=1；轮询 CSTS.RDY=1           5. 轮询 CSTS.RDY=1
              \                                    /
               +------> 5. Identify Controller + 命令集 Profile（若适用）
                        6. 枚举活动 NSID + 读取 Identify Namespace
                        7. 协商队列数 + 创建 I/O 队列（PCIe 先 CQ 后 SQ；Fabrics 用 Connect）
                        8. 任意时刻提交 AER（异步事件请求）
```

[PDF pp. 124-126](../_source/pages/page-124.md)

## 传输差异与核心规则

| 初始化阶段 | Memory-Based（PCIe） | Message-Based（Fabrics） |
|---|---|---|
| **建立 Admin 通道** | 在 `CC.EN=0` 时编程 `AQA`/`ASQ`/`ACQ` 寄存器 | 建链后发 Connect 命令创建 Admin 队列对和 Association |
| **属性访问** | 直接读写 MMIO 寄存器 | 使用 Fabrics `Property Get` / `Property Set` 命令 |
| **带内认证** | 不适用 | Connect 成功创建队列后，若 `AUTHREQ != 0` 须在正常初始化前完成认证（Figure 82） |
| **I/O 队列创建** | 1. `Set Features (Number of Queues)`<br>2. 配置中断向量<br>3. **先 `Create I/O CQ`，后 `Create I/O SQ`** | 1. 查询队列上限（Number of Queues 与 `CAP.MQES`）<br>2. 对每对 I/O 队列执行 `Connect` |

**必须牢记的约束**：
1. **`CC.CSS` 映射规则**（依据 `CAP.CSS`）：`NOIOCSS=1` → `111b`（仅 Admin）；`IOCSS=1` → `110b`（多命令集组合，后续需选 Profile）；`IOCSS=0 ∧ NCSS=1` → `000b`（NVM 命令集）。[PDF p. 124](../_source/pages/page-124.md)
2. **禁止抢跑**：`CC.EN=1` 之后、`CSTS.RDY=1` 之前**不得**提交任何命令；若此前未正确设置 `CC.IOSQES`/`CC.IOCQES`，创建 I/O 队列会报 `Invalid Queue Size`。[PDF p. 79](../_source/pages/page-079.md)
3. **2 分钟关联超时（Fabrics）**：建立 Association 后若主机在 **2 分钟**内未置位 `CC.EN=1`，控制器可主动移除该 Association。[PDF pp. 126-127](../_source/pages/page-126.md)
4. **命名空间枚举**：可通过 Identify `CNS=02h` 一次获取最多 1024 个活动 NSID，或 `CNS=10h` 获取已分配 NSID。

## Discovery 控制器初始化（NVMe-oF）

Discovery 控制器专用于发现存储目标，其初始化重点在于是否支持变更通知与 KATO（Keep Alive Timeout）调整（Figure 83）：[PDF p. 128](../_source/pages/page-128.md)

```text
收到 Connect 请求
  |-- 请求变更通知 + 非零 KATO --> 控制器可调整 KATO + 预分配 AER/AEN 资源 -> Connect 成功 -> 提交 AER
  `-- 否则 ----------------------> 使用固定 KATO（不支持通知则 Connect 报错）-> 完成连接（不提交 AER）
```

Connect 成功后依次执行：按需认证 → 读 Capabilities → 写 `CC`（含 `EN=1`）→ 等 `RDY=1` → Identify Controller → 读取 Discovery Log Page。

## 概念辨析

| 容易混淆 | 实际区别 |
|---|---|
| 初始化 vs 启用 | 初始化是从建链、配队列到枚举 NS 的全流程；启用特指写 `CC.EN=1` 到 `CSTS.RDY=1` 的握手 |
| `CC.CSS=110b` vs `111b` | `110b` 表示支持多种 I/O 命令集组合（需选 Profile）；`111b` 表示无 I/O 命令集（仅 Admin） |
| Association 保留 vs 超时 | CLR 前 Association 至少保留 2 分钟；启动时建链后 2 分钟不置 `EN=1` 则会被移除 |

## 关联与引申概念

- **控制器启用与重置（[Controller Enable, Shutdown, and Reset](controller-enable-shutdown-reset.md)）**：详细定义 `AQA`/`ASQ`/`ACQ`、`CC`/`CSTS` 寄存器位与超时恢复路径。
- **就绪模式（[Controller Ready Modes](controller-ready-modes.md)）**：`CC.CRIME` 决定控制器在 `CSTS.RDY=1` 时是否要求所有介质与命名空间就绪。
- **异步事件上报（[Asynchronous Event Reporting](asynchronous-event-reporting.md)）**：初始化完成后挂起 AER 命令，用于接收命名空间变更或健康告警。
- **Admin 命令模型（[Admin Command Model](admin-command-model.md)）**：Identify、Set Features、Create I/O CQ/SQ 等引导期命令均走 Admin 队列。
- **MMIO 与 BAR（PCIe 基础）**：Memory-Based 控制器通过 PCIe BAR0 的 MMIO（内存映射 I/O）空间访问 `CAP`/`CC`/`CSTS` 等寄存器。

## 规范依据

- [基于内存的初始化序列，PDF 第 124-125 页](../_source/pages/page-124.md)
- [Figure 82 时序与 Fabrics 初始化，PDF 第 126-127 页](../_source/pages/page-126.md)
- [Figure 83 与 Discovery 控制器初始化，PDF 第 128 页](../_source/pages/page-128.md)
- [CC / CSTS / CAP 定义，PDF 第 79 页](../_source/pages/page-079.md)
