# 控制器就绪模式与超时（Controller Ready Modes and Timeouts）

## 一句话说明

NVMe 控制器"就绪"有两种口径：**Ready With Media**（命令 + 介质 + 命名空间全部可用）和 **Ready Independent of Media**（不依赖介质的命令立即可用，命名空间/介质稍后到位）；`CAP.CRMS` 决定是否可选，`CC.CRIME` 决定选哪个，`CRTO.CRIMT` 与 `CRTO.CRWMT` 决定等待上限。

## 生活化类比

把控制器想成一家**新开的快递分拣中心**：

- **Ready With Media 模式**：等所有货架都装好、扫码枪都通了电，再开门营业——顾客来办业务一办就能办。
- **Ready Independent of Media 模式**：先把"客服窗口"（不依赖货架的命令）开门迎客；后台货架还在慢慢补货——买货的顾客要稍等，但寄件、查单、改地址的先开张。
- 老板（`CAP.CRMS`）决定分拣中心能不能"分两阶段开张"；主管（`CC.CRIME`）决定这次选哪种开张方式；招聘合同（`CRTO.CRIMT`/`CRTO.CRWMT`）写明最长等多久。

## 工作流程

```text
CC.EN: 0 -> 1
   │
   ├─ Ready With Media（CC.CRIME=0）
   │      └─ 在 CRTO.CRWMT 之前：RDY=1 + 命令/介质全部就绪
   │
   └─ Ready Independent of Media（CC.CRIME=1）
          ├─ 在 CRTO.CRIMT 之前：RDY=1 + 不依赖介质的命令就绪
          └─ 在 CRTO.CRWMT 之前：附加命名空间/介质就绪
```

简化说明：两种模式"开门时间"不同；CRIMT 一定 ≤ CRWMT；任一模式下若超过 CRWMT 仍未就绪，控制器仍必须置 `CSTS.RDY=1`，并记录 `Controller Ready Timeout Exceeded` 事件。

## 初学者案例

**场景：服务器冷启动后主机轮询 `CSTS.RDY`，但 Admin 命令一直返回 `Admin Command Media Not Ready`。**

1. 主机读 `CAP.CRMS`：`11b` 表示两种模式都支持。
2. 写 `CC.CRIME=1` 选 "Ready Independent of Media" 模式。
3. 写 `CC.EN=1`，轮询 `RDY=1` —— 这次 `RDY=1` 表示"非介质命令就绪"，早于介质就绪。
4. 在介质就绪（`CRTO.CRWMT`）之前下发 Sanitize/Format 类的 Admin 命令：可能返回 `Admin Command Media Not Ready`，主机应**重试**。
5. 故障速查：若 `RDY=1` 后**超过 `CRTO.CRWMT`** 介质仍未就绪——这是"超时违规"，控制器仍会置 `RDY=1`，但要在 Persistent Event Log 记 `Controller Ready Timeout Exceeded`；主机应按硬件错误处置。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| `CAP.CRMS` 含义 | `00b` 旧版（无选择，遵循 Revision < 2.0）；`01b` 仅 Ready With Media；`11b` 两种都支持（Rev 2.1 控制器必选） |
| `CC.CRIME` 生效时机 | 由 `CC.EN: 0→1` 时的值决定启用哪种模式；之后改 `CRIME` 无效 |
| `CC.CRIME` 可写性 | 仅当 `CAP.CRMS=11b` 时为 RW；其余情况 RO=0 |
| `CC.CRIME` 与 `CAP.TO` | 改 `CRIME` 可能影响 `CAP.TO` 报告值，但**不**改变 `CRTO.CRWMT` / `CRTO.CRIMT` |
| Ready With Media 语义 | `CSTS.RDY=1` 时：所有附加命名空间、Admin 所需介质、命令处理全部就绪 |
| Ready Indep. of Media 语义 | `CSTS.RDY=1` 时：不依赖附加命名空间/介质的命令可处理；依赖介质的 Admin/I/O 可能返回 `Namespace Not Ready` / `Admin Command Media Not Ready`（可重试） |
| CRIMT 单位 | 500 ms 一档 |
| CRWMT 单位 | 500 ms 一档，且 `CRWMT ≥ CRIMT` |
| 超时违规 | 错过适用截止时间时控制器**仍**必须在 `CRWMT` 之前置 `CSTS.RDY=1`；若支持 Persistent Event Log 需记 `Controller Ready Timeout Exceeded`（`NVM Subsystem Hardware Error`） |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| `CAP.CRMS` vs `CC.CRIME` | CRMS 是"能力"（是否可选）；CRIME 是"本次启用时选哪种" |
| `CAP.TO` vs `CRTO.CRWMT` | `CAP.TO` 是传统单段超时；`CRTO.CRWMT` 是 Rev 2.1 引入的两段超时之一 |
| `CRIMT` vs `CRWMT` | CRIMT 是"非介质命令就绪"上限；CRWMT 是"全部就绪"上限 |
| `RDY=1` vs 介质就绪 | RDY=1 仅代表"开门"；介质就绪还要等 CRWMT |
| `CFS=1` vs Ready Timeout | CFS=1 是控制器致命错误；Ready Timeout 是启动期超时违规 |
| `Admin Command Media Not Ready` vs `Namespace Not Ready` | 前者针对 Admin 命令；后者针对 I/O 命令访问未就绪命名空间；两者均**可重试** |

## 进阶细节

- **规范 3.5.3 两种模式**（PDF 第 128 页）：
  - **Controller Ready With Media**：`CC.EN: 0→1` 后，到 `CSTS.RDY: 0→1` 时，控制器能无错处理所有命令（规范 3.5.4.1），所有附加命名空间与 Admin 所需介质已就绪。
  - **Controller Ready Independent of Media**：`CC.EN: 0→1` 后：
    - `CRIMT` 之前 → `CSTS.RDY=1` 且能处理**不访问附加命名空间**的命令 + **不依赖介质**的 Admin 命令；
    - `CRWMT` 之前 → 附加命名空间与 Admin 所需介质就绪。
- **`CAP.CRMS` 三态**（规范 3.5.3 / PDF 第 130 页）：
  - `00b`：传统（无选择），可读性，保持就绪；适用于符合 Revision < 2.0 的控制器；
  - `01b`：仅 Ready With Media，`CC.CRIME` RO=0；
  - `11b`：两种都支持，Rev 2.1 控制器必设；主机在启用前选好 `CC.CRIME`。
- **`CC.CRIME` 行为**（规范 3.1.4 / 3.5.3 / PDF 第 79、130 页）：仅 `CAP.CRMS=11b` 时为 RW；`CC.EN: 0→1` 时的 `CRIME` 值决定本轮启用模式；改 `CRIME` **不**改 `CRTO` 字段但可能改 `CAP.TO`。
- **`CRTO` 字段**（规范 3.1.4.x / PDF 第 92 页）：
  - `CRIMT`（Controller Ready Independent of Media Timeout）：500 ms 单位；
  - `CRWMT`（Controller Ready With Media Timeout）：500 ms 单位，且 `≥ CRIMT`。
- **Figure 84 Admin 命令白名单**（规范 3.5.3 / PDF 第 129-130 页）：明列在 Ready Indep. of Media 模式下"可返回 `Admin Command Media Not Ready`"的 Admin 命令；Device Self-test 与 Get Log Page 有更窄限制。完整白名单以 PDF Figure 84 为准，不在本文档复刻。
- **超时违规处理**（规范 3.5.4.1 / PDF 第 131-132 页）：若错过适用截止时间（非零 `CAP.CRMS`）：
  - 控制器**仍**必须在 `CRWMT` 之前置 `CSTS.RDY=1`；
  - 若支持 Persistent Event Log，记录 NVM Subsystem Hardware Error，事件 code = `Controller Ready Timeout Exceeded`。
- **章节号**：Ready 模式定义在 3.5.3；错误处理在 3.5.4.1；`CC` 在 3.1.4；`CAP` 在 3.1.3；`CRTO` 在 3.1.4.x。

## 规范依据

- [就绪模式定义与 Discovery 初始化，PDF 第 128 页](../_source/pages/page-128.md)
- [Ready Indep. of Media 命令限制与 Figure 84 起点，PDF 第 129 页](../_source/pages/page-129.md)
- [Figure 84 续、模式选取与超时定义，PDF 第 130 页](../_source/pages/page-130.md)
- [超时映射与失败处理，PDF 第 131 页](../_source/pages/page-131.md)
- [CRTO 寄存器字段定义，PDF 第 92 页](../_source/pages/page-092.md)

## 相关阅读

- [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) - CC.EN 与 RDY 握手细节
- [controller-initialization.md](controller-initialization.md) - 启动流程的就绪阶段
- [format-nvm-lifecycle.md](format-nvm-lifecycle.md) - 介质未就绪与 Format 关系
- [keep-alive.md](keep-alive.md) - 启动期间 Keep Alive 激活条件
