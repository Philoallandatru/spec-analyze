# 电源状态描述符（Power State Descriptors）

电源状态描述符（PSD）是 NVMe 控制器在 `Identify Controller` 数据结构中提供的 32 个 32 字节条目（`PSD0` 必选，`PSD1`–`PSD31` 可选），用来向主机描述每个电源状态的**运行能力、切换延迟、相对性能、功耗指标与断电保护时序**，帮助主机在性能、功耗和响应速度之间做出权衡。[PDF pp. 350-353](../_source/pages/page-350.md)

可以把电源状态描述符想成**汽车档位说明书**：
- **NOPS**：这档能不能开（P 档非操作态 vs D 档操作态）
- **ENLAT / EXLAT**：换进/换出这档需要多少时间
- **RRT / RRL / RWT / RWL**：这档读/写的相对吞吐量和延迟
- **IDLP / ACTP / MP**：怠速功耗、负载平均功耗、持续最大功耗上限
- **FQVT / EPFVT / EPFRT**：强制静默或紧急断电时保存数据与恢复就绪的时间

---

## 描述符布局与工作模型

32 个 PSD 位于 `Identify Controller` 数据结构的字节 `2048–3071`（`PSD0` 占 `2048–2079`，`PSD31` 占 `3040–3071`）。主机通过 `Set Features`（Feature ID `02h`）选择要切换的电源状态索引，或配置 APST（Autonomous Power State Transition，自主电源状态转换）在空闲时自动切换。

```text
  Identify Controller 字节 2048-3071：32 × 32 字节 PSD（Figure 313）
  +------------------+------------------+------------------+------------------+
  | Dword 0 (31:0)   | Dword 1 (63:32)  | Dword 2 (95:64)  | Dword 3 (127:96) |
  | MP, MXPS, NOPS   | ENLAT (µs)       | EXLAT (µs)       | RRT, RRL, RWT, RWL|
  +------------------+------------------+------------------+------------------+
  | Dword 4 (159:128)| Dword 5 (191:160)| Dword 6 (223:192)| Dword 7 (255:224)|
  | IDLP, IPS        | ACTP, APS, EPFRT | FQVT, EPFVT,     | 保留             |
  |                  |                  | EPFRTS/FQVTS     |                  |
  +------------------+------------------+------------------+------------------+
```

### 核心字段与位域说明

| 字段 | 位域 | 含义与编码规则 |
|------|------|----------------|
| **NOPS** | `25` | 操作态标识：`0` = 操作态（可处理 I/O）；`1` = 非操作态（不处理 I/O，但控制器仍启用） |
| **ENLAT** | `63:32` | 进入该电源状态的最大延迟（微秒）；`0` = 未报告 |
| **EXLAT** | `95:64` | 退出该电源状态的最大延迟（微秒）；`0` = 未报告 |
| **RRT / RRL** | `100:96` / `108:104` | 相对读吞吐 / 相对读延迟（`0 ≤ 值 < NPSS`，数值越小性能越好） |
| **RWT / RWL** | `116:112` / `124:120` | 相对写吞吐 / 相对写延迟（`0 ≤ 值 < NPSS`，数值越小性能越好） |
| **IDLP / IPS** | `143:128` / `151:150` | 空闲功耗与比例（严格空闲 10s 后 30s 窗口典型功耗）。`IPS`：`00b`=未报告，`01b`=0.0001W，`10b`=0.01W，`11b`=保留 |
| **ACTP / APS** | `175:160` / `183:182` | 活动功耗与比例（由 `Active Power Workload` 定义的负载下 10s 窗口最大平均功耗） |
| **MP / MXPS** | `15:0` / `24` | 持续最大功耗与比例（指导值）。`MXPS`：`0`=0.01W，`1`=0.0001W |

**功耗计算与测量约定**：
- 实际功耗 = `幅度值 × 比例系数`；幅度为 `0` 或比例为"未报告"时，表示主机无法获取该估算值。
- `IDLP` 测量窗口不含命令处理、属性访问、后台维护、Sanitize 和自检；`ACTP` 为 10 秒平均功耗；`RRL/RRT/RWL/RWT` 受 `NPSS`（支持的电源状态数，0-based，至少为 1）约束。
- 主机通过 `Set Features`（FID `02h`）切换电源状态时，必须选择控制器支持的 PS 索引，否则返回 `Invalid Field in Command`。

[PDF pp. 350-353](../_source/pages/page-350.md) | [PDF p. 396](../_source/pages/page-396.md)

- [Identify Controller PSD 数组 Figures 312-313，PDF 第 350 页](../_source/pages/page-350.md)
- [PSD 字段 Figure 313 续，PDF 第 351 页](../_source/pages/page-351.md)
- [Active Power / Idle Power / 相对性能，PDF 第 352 页](../_source/pages/page-352.md)
- [Max Power + Time Scale Figure 314，PDF 第 353 页](../_source/pages/page-353.md)
- [Power Management Feature 02h，PDF 第 396 页](../_source/pages/page-396.md)
- [APST 与 PSD ITPS 关系，PDF 第 400-401 页](../_source/pages/page-400.md)

## 相关阅读

仅当 `Identify Controller` 的断电信号能力（Power Loss Signaling Information：`PLSEPF` / `PLSFQ`）受支持时，PSD 才报告对应的断电保护时序；否则相关字段清零。[PDF pp. 350-353](../_source/pages/page-350.md)

| 时序字段 | 位域 | 比例字段 | 条件与说明 |
|----------|------|----------|------------|
| **EPFRT**（紧急断电恢复时间） | `191:184` | `EPFRTS` (`211:208`) | `PLSEPF=1` 时报告；从紧急断电恢复后重新就绪的时间 |
| **FQVT**（强制静默保存时间） | `199:192` | `FQVTS` (`213:208` 区域) | `PLSFQ=1` 时报告；强制静默信号后完成数据保存的时间 |
| **EPFVT**（紧急断电保存时间） | `207:200` | `EPFVTS` (`215:208`) | `PLSEPF=1` 时报告；紧急断电信号后完成数据保存的时间 |

- **时间计算**：`实际时间 = 时序值 × Time Scale`。当时间值为 `0`（未报告）时，对应的 Scale 也必须为 `0`。
- **Time Scale 编码（Figure 314）**：`0h`=1µs，`1h`=10µs，`2h`=100µs，`3h`=1ms，`4h`=10ms，`5h`=100ms，`6h`=1s，`7h`=10s，`8h`=100s，`9h`=1000s，`Ah`=10000s，`Bh`=100000s，`Ch`=1000000s（约 11.6 天），`Dh–Fh`=保留。

---

## 实践与排错要点

**场景：切换低功耗档后应用 QPS 骤降。**
1. 主机执行 `nvme set-feature /dev/nvme0 -f 0x02 -v 5` 将控制器切到 PS 5。
2. PS 5 虽然空闲功耗低（如 `IDLP=3000, IPS=10b` 即 30W），但 `RRL=5` 导致读延迟显著增加，使延迟敏感型应用 QPS 下跌。
3. 排查时应同时检查 `RRL/RWT/RWL/RRT` 与 `ENLAT/EXLAT`，切回 `RRL=0` 的高性能状态（如 PS 0）；对延迟敏感业务避免切入 `NOPS=1` 的非操作态（退出延迟可能达秒级）。

### 常见易混淆点

| 易混淆概念 | 实际区别 |
|------------|----------|
| **NOPS=1 vs 控制器禁用** | `NOPS=1` 仍保持控制器启用（`CC.EN=1`），只是不处理 I/O；`CC.EN=0` 是关闭整个控制器 |
| **ENLAT vs EXLAT** | `ENLAT` 是进入该电源状态的延迟；`EXLAT` 是退出该电源状态的延迟 |
| **RRL/RWL vs RRT/RWT** | 延迟与吞吐均是**数值越小性能越好**，范围 `0 ≤ 值 < NPSS` |
| **IDLP vs ACTP vs MP** | `IDLP` 是严格空闲功耗；`ACTP` 是 10s 负载平均功耗；`MP` 是持续最大功耗上限指导值 |
| **HMNARE 与 ENLAT** | 启用 HMNARE（Host Memory Buffer 相关特性）时，控制器从非操作态返回前可能超出 `ENLAT` 以取回 HMB 必要数据 |

---

## 关联与引申概念

- **APST（Autonomous Power State Transition）**：控制器在空闲超时后自动跳转到指定非操作态（ITPS）的机制，依赖 PSD 中的 `ENLAT`/`EXLAT` 评估转换代价。[PDF pp. 400-401](../_source/pages/page-400.md)
- **HMB（Host Memory Buffer）**：允许控制器借用主机内存作为缓存；从非操作态恢复时可能需要额外时间重新拉取 HMB 数据。
- **SMART / Health Log**：运行时电源状态变化或热节流结果可通过 `Get Log Page` 读取健康日志观察。
- **相关文档**：
  - [识别命令模型](identify-command-model.md) — PSD 数组嵌入在 Identify Controller 数据结构中
  - [控制器初始化](controller-initialization.md) — 从非操作态恢复或上电初始化时使用延迟与恢复时序
  - [通用控制器特性](common-controller-features.md) — FID `02h` 电源管理特性切换 PS 索引
  - [特性值与作用域](feature-values-and-scope.md) — `Set Features` 的作用域与持久化规则
  - [控制器就绪模式](controller-ready-modes.md) — 上电后的初始电源状态选择
  - [断电信号](power-loss-signaling.md) — `PLSEPF` / `PLSFQ` 与断电时的电源状态切换

## 规范依据

- [Identify Controller PSD 数组（Figures 312-313），PDF pp. 350-351](../_source/pages/page-350.md)
- [Active Power / Idle Power / 相对性能字段，PDF p. 352](../_source/pages/page-352.md)
- [Max Power 与 Time Scale（Figure 314），PDF p. 353](../_source/pages/page-353.md)
- [Power Management Feature（FID 02h），PDF p. 396](../_source/pages/page-396.md)
- [APST 与 PSD ITPS 关系，PDF pp. 400-401](../_source/pages/page-400.md)
