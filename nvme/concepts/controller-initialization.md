# 控制器初始化（Controller Initialization）

## 一句话说明

控制器初始化是"transport-specific"的启动流程：Memory-Based（PCIe）走"等 RDY=0 → 配 AQA/ASQ/ACQ → 写 CC.EN=1 → 等 RDY=1"，Message-Based（Fabrics）走"建链 → Connect 建 Admin 队列 → Property Get/Set 写 CC → EN=1 → 等 RDY=1"；之后统一进入"Identify → 枚举 NS → 建 I/O 队列 → 配置 AER"四步。

## 生活化类比

把控制器初始化想成**新员工入职**：

- **Memory-Based 员工**（本地坐班）：先在工位上铺好入职材料（`AQA`/`ASQ`/`ACQ`），再按下"上班打卡"（`CC.EN=1`），等工牌灯亮（`CSTS.RDY=1`）。
- **Message-Based 员工**（远程办公）：先打开视频会议（建链 + Connect 建 Admin 队列），可能要先"亮工牌"（`AUTHREQ` 触发的认证），再打卡上班。
- 入职后**两人都做同一件事**：先自我介绍（Identify Controller），认识团队（命令集 / Profile）、认识项目（命名空间）、领工单（I/O 队列），最后订邮件提醒（异步事件 AER）。

## 工作流程

```text
Memory-Based (PCIe)                      Message-Based (Fabrics)
─────────────────────────────────        ─────────────────────────────────
等 CSTS.RDY=0                            建传输层连接
配 AQA / ASQ / ACQ                       发 Connect 建 Admin 队列 + Association
按 CAP.CSS 选 CC.CSS                     若 AUTHREQ≠0 → 必做认证
选 AMS / MPS                             Property Get/Set CAP, CC；写 CC.EN=1
写 CC.EN=1；轮询 CSTS.RDY=1              轮询 CSTS.RDY=1
              \                          /
               +---> 共同后续：Identify Controller / 命令集 Profile
                     枚举活动 NSID → 读 Identify Namespace
                     协商/创建 I/O 队列
                     配置 AER
```

简化说明：两路在"建立 Admin 通道"上分叉，在"识别 + 队列 + AER"上汇合。

## 初学者案例

**场景：驱动加载时按 PCIe 流程初始化一个本地 NVMe SSD。**

1. 上电后等 `CSTS.RDY=0`，确认上次的 Controller Reset 已结束。
2. 写 `AQA`/`ASQ`/`ACQ`：分别配置 Admin SQ/CQ 的条目数（2..4096，0-based）、物理基地址（页对齐）。
3. 读 `CAP.CSS`：若 `NOIOCSS=1` → `CC.CSS=111b`；若 `IOCSS=1` → `CC.CSS=110b`；若 `NCSS=1` → `CC.CSS=000b`。
4. 写 `CC`：选好 `AMS`、`MPS`、合法的 `IOSQES`/`IOCQES`，再 `EN=1`。
5. 轮询 `CSTS.RDY=1`（不可在 1 之前提交命令）。
6. 发 `Identify Controller`（`CNS=1`）→ 读 `Identify Namespace`（遍历 NSID）→ `Set Features Number of Queues` → `Create I/O CQ` → `Create I/O SQ` → 提交 AER。
7. 故障速查：`RDY=1` 长时间不到 → 看 `CAP.TO` 是否到；不到就查 `CFS`/`PP` 与 PCIe 链路状态。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 启动前提（Memory-Based） | 必须等 `CSTS.RDY=0`（上次重置结束）再开始 |
| 启动前提（Fabrics） | 先建传输层连接 + Connect 成功，才能访问 CC/CAP |
| Admin 队列配置时机 | `AQA`/`ASQ`/`ACQ` 仅在 `CC.EN=0` 时可改 |
| CSS 选取 | `CC.CSS` 必须根据 `CAP.CSS` 选取：`NOIOCSS=1`→`111b`；`IOCSS=1`→`110b`；`NCSS=1`→`000b` |
| 启用等待 | `CC.EN=1` 后到 `CSTS.RDY=1` 之间**禁止**提交任何命令 |
| 队列条目大小 | `IOSQES`/`IOCQES` 必须在创建 I/O 队列前初始化为合法值；否则队列创建报 Invalid Queue Size |
| 队列创建顺序（Memory-Based） | 先 `Create I/O CQ` → 后 `Create I/O SQ` |
| 队列创建（Fabrics） | 用 Connect 命令一次建 SQ + CQ 队列对 |
| 队列数量协商 | 创建 I/O 队列前先用 `Set Features Number of Queues` 协商（Memory-Based）；Fabrics 通过 `CAP.MQES` 限制 |
| 认证要求 | Fabrics 下若 `AUTHREQ≠0`，**必须**在正常初始化前完成 in-band 认证 |
| 关联超时 | Fabrics 主机在 Connect 后 2 分钟内未置 `CC.EN=1`，控制器可移除 Association |
| 命名空间发现 | 用 `Identify CNS=0h` 逐 NSID 探测 或 `CNS=2h` 一次拉 1024 个活动 NSID；用 `CNS=10h` 拉已分配 NSID |
| 异步事件 | 控制器就绪后**任意时刻**可配置 AER；建议在创建 I/O 队列前先发 AER 兜底 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 启动 vs 启用 | "启动"包含建链 + 配 Admin 队列；"启用"特指 `CC.EN=1` |
| 启动 vs 重置 | 启动是首次可用化；重置是运行中复位（CLR/NSSR） |
| Memory-Based vs Fabrics 初始化 | 前者用 MMIO 寄存器；后者用 Property Get/Set + Connect |
| `CC.CSS=110b` vs `111b` | 110b = 多命令集组合（按 I/O Command Set Profile 选）；111b = 无 I/O 命令集（只跑 Admin） |
| `RDY=1` vs `CFS=0` | RDY=1 是"已就绪处理命令"；CFS=0 是"无致命错误"——两件事要一起看 |
| 关联保留 vs 关联超时 | CLR 前关联保留 2 分钟；启动期间未 `EN=1` 也可能 2 分钟后被移除 |

## 进阶细节

- **规范 3.5.1 Memory-Based 初始化序列**（PDF 第 124-125 页）：
  1. 等 `CSTS.RDY=0`；
  2. 配 `AQA` / `ASQ` / `ACQ`；
  3. 读 `CAP.CSS`，按 `NOIOCSS` / `IOCSS` / `NCSS` 三种情形设置 `CC.CSS`；
  4. 选 `AMS` / `MPS` / `IOSQES` / `IOCQES`（在 `CC.EN=0` 时合法可写）；
  5. 写 `CC.EN=1`，轮询 `CSTS.RDY=1`；
  6. 发 `Identify Controller`；
  7. 若 `CAP.CSS.IOCSS=1`，识别可用命令集组合并选 Profile；
  8. 对每个使能的 I/O 命令集做命令集特定的 Identify；
  9. 枚举活动 NSID，识别 Namespace 数据结构；
  10. 创建 I/O CQ / SQ（先 CQ 后 SQ）；
  11. 任意时刻配置并提交 AER。
- **`CAP.CSS` → `CC.CSS` 选取规则**（规范 3.5.1 / PDF 第 124 页）：
  - `CAP.CSS.NOIOCSS=1` → `CC.CSS=111b`（无 I/O 命令集）；
  - `CAP.CSS.IOCSS=1` → `CC.CSS=110b`（I/O 命令集组合）；
  - `CAP.CSS.IOCSS=0` 且 `CAP.CSS.NCSS=1` → `CC.CSS=000b`（NVM 命令集）。
- **Figure 82 时序**（规范 3.5.2 / PDF 第 126-127 页）：Fabrics 成功的 Connect 在任何必需 in-band 认证交换**之前**就建出 Admin / I/O 队列；主机 Association 建立后 2 分钟内未 `CC.EN=1`，控制器可移除该 Association。
- **Fabrics 启动序列**（规范 3.5.2 / PDF 第 125-127 页）：
  1. 建立传输层连接；
  2. 发 Connect 命令 → 建 Admin 队列 + Association；
  3. 若 `AUTHREQ≠0`，完成 in-band 认证；
  4. Property Get CAP / Property Get CC / Property Set CC（写 `EN=1`）；
  5. 轮询 `CSTS.RDY=1`；
  6. Identify Controller / Identify Namespace；
  7. 用 Connect 建 I/O 队列对（每对一个 QID）。
- **Discovery 控制器初始化**（Figure 83 / 规范 3.5.3 / PDF 第 128 页）：
  - 收到 Connect → 若**请求变更通知 + 非零 KATO** → 控制器**可**调 KATO + 预分配 AER/AEN 资源 → Connect 成功 → 主机提交 AER；
  - 否则 → 控制器使用**固定** KATO；若不支持该通知请求则 Connect 报错；
  - 控制器不一定支持可变 KATO（"MAY"），可保持固定 KATO。
- **Discovery 启动后顺序**（规范 3.5.3 / PDF 第 128 页）：
  1. 如需认证，先做认证；
  2. 读 Controller Capabilities；
  3. 写 CC（含 `EN=1`）；
  4. 等 `RDY=1`；
  5. 识别相应 Controller 数据结构；
  6. 读 Discovery Log Page。
- **Create I/O 队列前置**（规范 3.3.1 / PDF 第 79 页）：在 `CC.EN=1` 之前若未初始化 `CC.IOCQES` / `CC.IOSQES`，Create I/O CQ/SQ 命令会以 `Invalid Queue Size` 中止。
- **`CC.CSS=110b` 时的工作**（规范 3.5.1 / PDF 第 124-125 页）：主机识别可用命令集组合并选择合适 Profile（I/O Command Set Profile，见 5.1.25.1.17）。
- **章节号**：Memory-Based 在 3.5.1；Fabrics 在 3.5.2；Ready Modes 在 3.5.3-3.5.4；Connect 命令在 3.3.2；Identify CNS 在 5.1.13；Admin 队列寄存器在 3.1.4.8-10；CC 在 3.1.4；CSTS 在 3.1.5；CAP 在 3.1.3。

## 规范依据

- [Memory-Based 初始化起点，PDF 第 124 页](../_source/pages/page-124.md)
- [Memory-Based 完成流程与 Fabrics 启动，PDF 第 125 页](../_source/pages/page-125.md)
- [Figure 82 时序与 Fabrics 初始化，PDF 第 126 页](../_source/pages/page-126.md)
- [Fabrics 初始化延续与超时处理，PDF 第 127 页](../_source/pages/page-127.md)
- [Figure 83 与 Discovery 初始化，PDF 第 128 页](../_source/pages/page-128.md)
- [CC / CSTS / CAP 定义，PDF 第 79 页](../_source/pages/page-079.md)

## 相关阅读

- [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) - 启用/禁用/重置的寄存器契约
- [controller-ready-modes.md](controller-ready-modes.md) - RDY=1 的就绪模式选择
- [asynchronous-event-reporting.md](asynchronous-event-reporting.md) - 初始化末尾配置 AER
- [admin-command-model.md](admin-command-model.md) - Identify 等 Admin 命令模型
