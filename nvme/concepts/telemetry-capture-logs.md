# 遥测捕获日志（Telemetry Capture Logs）

## 一句话说明

遥测捕获日志是 NVMe 暴露的"控制器内部状态快照"，由 512 字节的**标准化头** + 多个 512 字节的**数据块**组成，分主机发起（LID `07h`）和控制器发起（LID `08h`）两类：主机发起在收到 `CTHID=1` 的 Get Log Page 时触发捕获，控制器发起由厂商特定条件触发并保留一份"已抓好的"快照供主机读取。

## 生活化类比

把 NVMe 控制器想成**一台装了行车记录仪的赛车**：

- **行车记录仪的 SD 卡** = 遥测日志（LID `07h` 或 `08h`）
- **车手主动按录制键** = 主机发 `Get Log Page` 并设 `CTHID=1`（Create Telemetry Host-Initiated Data）
- **车手在弯道失控时行车记录仪自动保存一段** = 控制器内部条件触发，自动写入 LID `08h`
- **录像总是先有一段"封面"**（厂商、时间戳、码流版本） = 512 字节标准化头
- **录像主体是若干 512 字节块** = 多个 Telemetry Data Block
- **车手回家插电脑看视频** = 主机连续读多块数据（offset 必须是 512 的倍数）

## 工作流程

```text
   主机发 Get Log Page (LID=07h 或 08h)
              |
              v
   +---------+-------------+-----------------------+
   |  byte0  ... byte511   |  block 1 | block 2 | ...|  标准化头  +  数据块
   +---------+-------------+-----------------------+
       ^                       ^
       |                       |
   厂商 OUI / area 边界 /     Area 1..N 数据
   范围 / 代号 / 原因
```

**端到端流程**：

1. **主机发起（`07h`）**：主机发 Get Log Page 设 `CTHID=1` → 控制器立即捕获内部状态；可附带 `MCDA`（仅当 `MCDAS=1`）限定最大创建到 Area 1/2/3/4。
2. **数据读取**：数据稳定到"下一次捕获/Firmware Commit/POR"为止；主机按 512 字节倍 offset 读。
3. **控制器发起（`08h`）**：厂商条件触发保存；Area 1-3 跨所有重置，Area 4 可能跨 Controller Level Reset。
4. **ack**：`08h` 的 availability=1 时，主机用 `RAE=0` 成功读取后清掉 availability 标志。
5. **Area 4 协商**：`DA4S=1`（控制器支持）+ `ETDAS=1`（主机支持）→ 日志按 Area 4 算大小；否则按 Area 3。

## 初学者案例

**场景：SSD 出现偶发卡顿，FAE 想抓现场怎么办？**

1. 现场工程师接到 FAE 指令："主机发起一次遥测抓取"。
2. 工程师执行 `nvme telemetry-log /dev/nvme0n1 --host-generate=1`（对应 `CTHID=1`）。
3. 工具把 `MCDA` 设成 `100b`（如果该控制器报告 `MCDAS=1`），告诉控制器"我只要到 Area 4 就够"。
4. 控制器立刻拍"快照"——把内部状态写入 LID `07h` 区域。
5. 工具读头 512 字节看 `IEEE`（OUI）、`THDA1LB`...`THDA4LB`（各 Area 最后块号）、`THDGN`（Generation Number）。
6. 工具按 `THDA1LB` 算 offset，把 Area 1 数据 dump 到本地文件。
7. 工具按 `TCDA`（来自 LID `08h`）判断有没有控制器主动抓的快照，如果有就一并取走。
8. 把 dump 寄给 FAE，FAE 用对应 OUI 的解码工具分析。

> 注意：若想"避免覆盖旧快照"，先把旧数据读出来再发 `CTHID=1`；`CTHID=1` 立刻把旧内容清掉。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 头是 512 字节 | 每个遥测日志都有 512 字节的标准化头；即使无数据头也存在 |
| 块是 512 字节 | 所有 Telemetry Data Block 严格 512 字节；Log Offset 必须是 512 倍数 |
| 区域嵌套而非分段 | Area 1 ⊂ Area 2 ⊂ Area 3 ⊂ Area 4；最后块号单调不递减 |
| 主机发起由 `CTHID` 触发 | `CTHID=1` 立即捕获；`CTHID=0` 仅读已有快照 |
| 主机发起失效条件 | 下一次 `CTHID=1`、Firmware Commit 或 POR 都会让旧快照失效 |
| `MCDA` 受 `MCDAS` 门控 | `MCDAS=0` 时 `MCDA` 被忽略（不能选最大 Area） |
| 控制器发起 Area 1-3 跨重置 | Area 1/2/3 数据在所有重置后都保留 |
| 控制器发起 Area 4 可跨 CLR | Area 4 可能跨 Controller Level Reset 保留（不保证） |
| 8 位 Generation Number | `FFh` 后回绕到 `00h`；用于检测"我读到的是不是最新的" |
| availability 仅 in-band 清 | 控制器发起日志的 `available=1` 由成功的 in-band Get Log Page `RAE=0` 清除；out-of-band 看到的不一定清 |
| Header 镜像两种捕获 | 主机发起日志的 Header 同时镜像了控制器发起日志的 availability + generation，便于单次读对齐 |
| 范围是 Controller 或 Subsystem | 2.1+ 规范下 THS 不再是 `0h`；常见值 `01h`（Controller）、`02h`（NVM Subsystem） |
| 偏移必须是 512 倍数 | 不满足时控制器返回 Invalid Field in Command |
| Area 4 双标志协商 | 控制器 `DA4S=1` 且主机 `ETDAS=1` → Area 4 生效；缺一就按 Area 3 截断 |
| 数据超出最后块未定义 | 超过 last-block 边界的数据由控制器决定，规范不保证有意义 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| `07h` vs `08h` | `07h` = Host-Initiated（主机发起）；`08h` = Controller-Initiated（控制器发起） |
| `CTHID=1` vs `MCDA` | `CTHID=1` 触发"是否做捕获"；`MCDA` 限制"最多创建到哪个 Area" |
| `MCDAS` vs `MCDA` | `MCDAS` 是控制器能力（Supported Log Pages log 里）；`MCDA` 是本次请求值 |
| `DA4S` vs `ETDAS` | `DA4S` = 控制器是否支持 Area 4；`ETDAS` = 主机是否愿意消费 Area 4；两者**都**为 1 才算 Area 4 启用 |
| `RAE=0` vs `RAE=1` | `RAE=0` 在控制器发起日志上会清掉 availability；`RAE=1` 不清（仅查询） |
| Generation Number 回绕 | 8 位，`FFh`→`00h`；不是 16 位；回绕后主机要靠其他标志（如 availability 翻转）判断 |
| Area 1 vs Area 2 关系 | 不是"Area 2 是 Area 1 之后"；是 Area 2 包含 Area 1 的全部内容 + 更多 |
| Telemetry vs Self-Test | Telemetry 是控制器内部状态快照；Self-Test 是控制器自检过程+结果 |
| in-band vs out-of-band | in-band 通过 NVMe 队列读；out-of-band 通过 Management Endpoint；后者可见性不受 `RAE=0` 影响 |
| 主机发起日志稳定性 | 稳定到"下一次 CTDID=1 / Firmware Commit / POR"——不是无限期 |
| 标准化头 vs 厂商头 | 头是规范定义的固定字段；数据块是厂商定义的"应用数据" |

## 进阶细节

- **Telemetry Host-Initiated Log Specific Parameter 字段**（Figure 213, 规范 5.1.12.1.8）：
  - Bits `14:12` = Reserved
  - Bits `11:09` = MCDA（最大创建 Area；`001b`=Area 1，`010b`=Area 1-2，`011b`=Area 1-3，`100b`=Area 1-4）
  - `MCDA` 仅在 `CTHID=1` 且 `MCDAS=1` 时生效
- **Telemetry Host-Initiated Header 字段布局**（Figure 214, 规范 5.1.12.1.8）：
  - Byte `00` = LID（必须 `07h`）
  - Bytes `04:01` = Reserved
  - Bytes `07:05` = IEEE OUI
  - Bytes `09:08` = THDA1LB（Area 1 Last Block；0 = 无数据）
  - Bytes `11:10` = THDA2LB（≥ THDA1LB）
  - Bytes `13:12` = THDA3LB（≥ THDA2LB）
  - Bytes `15:14` = Reserved
  - Bytes `19:16` = THDA4LB（若 `DA4S=1` 则 ≥ THDA3LB）
  - Bytes `379:20` = Reserved
  - Byte `380` = THS（Telemetry Host-Initiated Scope，2.1+ 不为 `0h`）
  - Byte `381` = THDGN（8-bit Generation Number，`FFh` 后回 `00h`）
  - Byte `382` = TCDA（Telemetry Controller-Initiated Data Available 镜像）
  - Byte `383` = TCDGN（Telemetry Controller-Initiated Generation Number 镜像）
  - Bytes `511:384` = RID（Reason Identifier，厂商特定）
- **MCDAS 字段**（Figure 215, 规范 5.1.12.1.8.1）：在 Supported Log Pages log 的 LID Specific Parameter 里，bit 0 = `MCDAS`；`1` = 支持 `MCDA`，`0` = 不支持。
- **Telemetry Controller-Initiated 字段**（Figure 216, 规范 5.1.12.1.9）：与 Host-Initiated 头结构类似但 LID=`08h`；另有 Telemetry Controller-Initiated Data Generation Number 与 Availability 字段。
- **DA4S 协商**（规范 5.1.12.1.9）：
  - `DA4S=0` → 永远 3 个 Area；日志大小按 Area 3 算
  - `DA4S=1` + `ETDAS=1`（Host Behavior Support Feature）→ 4 个 Area；按 Area 4
  - `DA4S=1` + `ETDAS=0` → 4 个 Area **存在**，但主机可见大小按 Area 3 截断
- **回绕语义**（规范 5.1.12.1.8）：THDGN 8 位，`FFh` 后回 `00h`；不是简单的 `+1`。
- **Host Behavior Support Feature**（规范 5.1.25.1.14）：`ETDAS` 字段需要主机在 Host Behavior Support 中明确启用；这是为了避免老主机把"长日志"误读。
- **in-band ack 行为**（规范 5.1.12.1.9）：当 `available=1`，主机用 `RAE=0` 成功读完后控制器**清掉** availability；out-of-band Management Endpoint 看到的可读性**不**受此清位影响。
- **快照触发序列**（规范 5.1.12.1.8）：`CTHID=1` 触发新捕获 → generation 自增 → 头字段刷新 → 数据区域重写；旧 generation 编号视为"已被新快照覆盖"。
- **Firmware Commit 的副作用**（规范 5.1.12.1.8 + 5.1.8）：Firmware Commit 会让 Host-Initiated 快照失效；这是为了避免新旧固件"内容含义不一致"造成误读。
- **偏移量约束**：所有 Log Page Offset Lower 必须是 512 的倍数；`MCDA` 与 `CTHID` 都必须在 Log Specific Parameter 字段中同时设置才生效。

## 规范依据

- [Telemetry Host-Initiated 概述、`CTHID` 与 `MCDA` 触发与 Self-test 边界，PDF 第 241 页](../_source/pages/page-241.md)
- [主机发起保留、块大小、Area 4 协商与 MCDAS 字段，PDF 第 242 页](../_source/pages/page-242.md)
- [Host-Initiated Header 字段布局（OUI/Area Last Block/Scope/Generation/Reason），PDF 第 243 页](../_source/pages/page-243.md)
- [Controller-Initiated 保留、头、Availability、Generation 与 Area 协商，PDF 第 244 页](../_source/pages/page-244.md)
- [Asynchronous Event Reporting 触发读取 `08h`，PDF 第 198 页](../_source/pages/page-198.md)

## 相关阅读

- [log-page-retrieval.md](log-page-retrieval.md) - 通用 Get Log Page 路径
- [asynchronous-event-reporting.md](asynchronous-event-reporting.md) - 触发读取的 AER
- [error-and-health-logs.md](error-and-health-logs.md) - 错误日志与遥测互补
- [persistent-event-log.md](persistent-event-log.md) - 事件诊断的另一通道
