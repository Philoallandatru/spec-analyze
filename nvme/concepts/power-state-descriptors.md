# 电源状态描述符（Power State Descriptors）

## 一句话说明

电源状态描述符是 Identify Controller 数据结构里 32 个 32 字节的条目，描述每个电源状态的"是否操作态、进出延迟、相对性能、空闲/活动/最大功耗、断电时序"，让主机能在功耗、性能和响应时间之间做权衡。

## 生活化类比

把电源状态描述符想成**汽车档位说明书**：

- **NOPS** = "这档能不能开（P 档 vs D 档）"。
- **ENLAT / EXLAT** = "换进/换出这档要几秒"。
- **RRL / RWT / RWL / RWT** = "这档读/写有多快、延迟多大"。
- **IDLP** = "挂这档不动时的怠速油耗"。
- **ACTP** = "这档地板油时的瞬时油耗"。
- **MP** = "这档允许的最大持续功率"。
- **FQVT / EPFVT / EPFRT** = "断电时这档能多快把数据搬进保险箱 / 失电后恢复时间"。

## 工作流程

```text
  Identify Controller 字节 2048-3071：32 个 32 字节 PSD
  (PSD0 必选，PSD1-31 可选)

  每个 PSD（Figure 313，32 字节）：
    +------+------+------+------+------+------+------+------+
    | 字节 0       | 字节 4       | 字节 8       | 字节 12      |
    | (bit 31:0)   | (63:32)      | (95:64)      | (127:96)     |
    | MP+MXPS+NOPS | ENLAT        | EXLAT        | RRT/RRL      |
    |              | (µs)         | (µs)         | (相对)       |
    +--------------+--------------+--------------+-------------+
    | 字节 16      | 字节 20      | 字节 24      | 字节 28      |
    | (159:128)    | (191:160)    | (223:192)    | (255:224)    |
    | RWT/RWL/IDLP | IPS/ACTP/APS | EPFRT/FQVT   | EPFVT/EPFRTS |
    |              | /IDLP        | /EPFVT       | /保留        |
    +--------------+--------------+--------------+-------------+

  bit 25  NOPS   0=操作态；1=非操作态
  bit 24  MXPS   0=0.01W；1=0.0001W（用于 MP）
  bit 151:150 IPS 00=未报告；01=0.0001W；10=0.01W；11=保留（用于 IDLP）
  bit 183:182 APS  Active Power Scale（用于 ACTP）
```

## 初学者案例

**场景：选低功耗档时，应用 QPS 下降。**

1. 主机 `nvme get-feature -f 0x02 -v 5` 把控制器切到 PS 5。
2. PS 5 的 PSD 描述符里：`NOPS=0`（操作态）、`ENLAT=50`（进 50µs）、`EXLAT=100`（出 100µs）、`RRL=5`（相对读延迟较大）、`RWT=2`（写吞吐较高）、`IDLP=3000` + `IPS=10b`（`0.01W`×3000 = 30W 空闲）。
3. 应用跑了 5 分钟后，`RRL=5` 决定读延迟变大，QPS 从 100k 跌到 60k。
4. 主机读所有 PSD 找到 `RRL=0` 的最低延迟态（PS 0）切回去，性能恢复。
5. 若要选 `NOPS=1` 的非操作态做深度省电，先看 `EXLAT` 是否大（可能秒级）；不切 `NOPS=1` 给读延迟敏感的应用。

> 排错提示：选 PS 不能只看 `IDLP`；还要看 `RRL`/`RWT`/`RWL`/`RWT` 和进出延迟，否则性能反噬。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 数量 | Identify Controller 含 32 个 PSD（PSD0 必选，PSD1-31 可选） |
| 大小 | 每个 PSD = 32 字节 |
| 位置 | Identify Controller 字节 2048-3071；PSD0 字节 2048-2079；PSD31 字节 3040-3071 |
| NOPS（位 25） | 0=可处理 I/O（操作态）；1=不处理 I/O（非操作态） |
| ENLAT（位 63:32） | 进入该 PS 的最大延迟，单位微秒；0=未报告 |
| EXLAT（位 95:64） | 退出该 PS 的最大延迟，单位微秒；0=未报告 |
| RRL（位 108:104） | 相对读延迟；值 < 支持 PS 数；值越小延迟越低 |
| RRT（位 100:96） | 相对读吞吐；值越小吞吐越高 |
| RWL（位 124:120） | 相对写延迟；值越小延迟越低 |
| RWT（位 116:112） | 相对写吞吐；值越小吞吐越高 |
| IDLP（位 143:128） | 空闲功耗（30 秒窗口，前置 10 秒空闲）；0=未报告 |
| IPS（位 151:150） | IDLP 比例：`00b`=未报告；`01b`=0.0001W；`10b`=0.01W；`11b`=保留 |
| ACTP（位 175:160） | 该 PS 在指定工作负载下，10 秒窗口内最大平均功耗；0=未报告 |
| APS（位 183:182） | ACTP 比例（值定义见 Figure 314 之外的 ACTP 比例表） |
| MP（位 15:0） | 该 PS 的持续最大功耗；0=未报告 |
| MXPS（位 24） | MP 比例：`0`=0.01W；`1`=0.0001W |
| EPFVT（位 207:200） | 紧急断电 vault 时间（仅在 `PLSEPF=1` 时报告；否则 0） |
| EPFVTS（位 215:208） | EPFVT 比例（Figure 314） |
| FQVT（位 199:192） | 强制静默 vault 时间（仅在 `PLSFQ=1` 时报告；否则 0） |
| FQVTS（位 213:208 区域） | FQVT 比例（Figure 314） |
| EPFRT（位 191:184） | 紧急断电后首次初始化时间（仅在 `PLSEPF=1` 时报告） |
| EPFRTS（位 211:208） | EPFRT 比例；`EPFRT=0` 时此位必须 0 |
| Time Scale 编码（Figure 314） | `0h`=1µs；`1h`=10µs；`2h`=100µs；`3h`=1ms；`4h`=10ms；`5h`=100ms；`6h`=1s；`7h`=10s；`8h`=100s；`9h`=1000s；`Ah`=10000s；`Bh`=100000s；`Ch`=1000000s；`Dh-Fh`=保留 |
| 比例与时序值 | 时间 = 时序值 × Scale；`0=未报告` 时 Scale 必须 0 |
| Active Workload | 测量 ACTP 的工作负载定义在 Identify 的 `Active Power Workload` 字段 |
| 测量窗口 | IDLP=30 秒（前 10 秒空闲）；ACTP=10 秒平均；不含命令处理/属性访问/后台/Sanitize/自检 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| NOPS=1 vs 控制器禁用 | NOPS=1 是非操作态（不可发 I/O，但控制器仍启用）；`CC.EN=0` 是整个控制器关闭 |
| ENLAT vs EXLAT | ENLAT=进；EXLAT=出 |
| IDLP vs ACTP vs MP | IDLP=空闲；ACTP=10s 平均最大；MP=持续最大（上限指导） |
| RRL vs RRT | RRL=读延迟（越小越好）；RRT=读吞吐（越小越高） |
| RWL vs RWT | RWL=写延迟；RWT=写吞吐 |
| MXPS vs APS | MXPS=MP 比例；APS=ACTP 比例；IPS=IDLP 比例 |
| MXPS 0 vs 1 | 0=0.01W；1=0.0001W（高分辨率） |
| IPS 11b | 保留；写 11b 视为未报告 |
| EPFVT=0 vs EPFVT=0+PLSEPF=1 | `EPFVT=0`=未报告；`PLSEPF=0` 强制 `EPFVT=0` |
| Time Scale Dh-Fh | 保留；不应用于实际值 |
| PSD vs PS 索引 | PSD 是 4 KiB 描述符数组；PS 是 Set Features 选的索引 |
| PSD0 vs 其他 PSD | PSD0 必选；其他可选 |

## 进阶细节

- **Figure 313 完整位定义**：
  - `15:00` MP
  - `23:16` 保留
  - `24` MXPS（`0`=0.01W；`1`=0.0001W）
  - `25` NOPS（`0`=操作态；`1`=非操作态）
  - `31:26` 保留
  - `63:32` ENLAT（µs）
  - `95:64` EXLAT（µs）
  - `100:96` RRT
  - `103:101` 保留
  - `108:104` RRL
  - `111:109` 保留
  - `116:112` RWT
  - `119:117` 保留
  - `124:120` RWL
  - `127:125` 保留
  - `143:128` IDLP
  - `149:144` 保留
  - `151:150` IPS（`00`=未报告；`01`=0.0001W；`10`=0.01W；`11`=保留）
  - `159:152` 保留
  - `175:160` ACTP
  - `181:176` 保留
  - `183:182` APS
  - `191:184` EPFRT
  - `199:192` FQVT
  - `207:200` EPFVT
  - `211:208` EPFRTS
  - `255:212` 保留
- **Figure 314 Time Scale 完整表**：
  - `0h` 1µs；`1h` 10µs；`2h` 100µs；`3h` 1ms；`4h` 10ms；`5h` 100ms；`6h` 1s；`7h` 10s；`8h` 100s；`9h` 1000s；`Ah` 10000s；`Bh` 100000s；`Ch` 1000000s；`Dh-Fh` 保留
- **PLSEPF / PLSFQ**（Identify Controller "Power Loss Signaling Information"）：决定 EPFVT/EPFRT/FQVT 是否报告；`=0` 时相关字段清 0。
- **Active Power Workload**：Identify Controller 中定义工作负载类型；ACTP 测量依赖此字段。
- **时间值=0 时 Scale 必须=0**：规则防止"未报告"误读。
- **RRL/RRT/RWL/RWT 范围**：`0 ≤ value < NPSS`；NPSS 是支持的 PS 数（0-based，至少 1）。
- **测量语义**：IDLP 不含命令处理、属性访问、后台工作、Sanitize、自检；ACTP 是 10 秒窗口平均。
- **平台扩展**：Base 规范给的是"指导值"；平台/外形规范可能附加更细的测量和报告要求。
- **Power Management Feature**（FID `02h`）：Set Features PS 索引与 PSD 数组一致；选 PS 时**必须**是控制器支持的 PS（否则 `Invalid Field in Command`）。
- **APST ITPS**：APST 条目中的 ITPS 指向"非操作态" PSD；APST 启用时会参考 `ENLAT`/`EXLAT`。
- **HMNARE 交互**：HMNARE 启用时，控制器从非操作态返回前可能"超出 ENLAT"以取回 HMB 必要数据（规范 5.1.25.1.2.4）。
- **Identify vs Get Log Page**：PSD 在 Identify Controller 内；运行时电源状态改变可通过 Get Log Page 读 SMAR/Health 或 Controller Data Queue。

## 规范依据

- [Identify Controller PSD 数组 Figures 312-313，PDF 第 350 页](../_source/pages/page-350.md)
- [PSD 字段 Figure 313 续，PDF 第 351 页](../_source/pages/page-351.md)
- [Active Power / Idle Power / 相对性能，PDF 第 352 页](../_source/pages/page-352.md)
- [Max Power + Time Scale Figure 314，PDF 第 353 页](../_source/pages/page-353.md)
- [Power Management Feature 02h，PDF 第 396 页](../_source/pages/page-396.md)
- [APST 与 PSD ITPS 关系，PDF 第 400-401 页](../_source/pages/page-400.md)

## 相关阅读

- [识别命令模型](identify-command-model.md) - PSD 嵌在 Identify Controller 内
- [通用控制器特性](common-controller-features.md) - FID 02h 切换 PS 索引
- [特性值与作用域](feature-values-and-scope.md) - PS 由 Set Features 选择
- [控制器就绪模式](controller-ready-modes.md) - 上电后的初始 PS 选择
- [断电信号](power-loss-signaling.md) - 失电时 PS 切换行为
