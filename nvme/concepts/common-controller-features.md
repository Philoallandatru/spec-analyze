# 控制器通用特性（Common Controller Features）

## 一句话说明

"通用控制器特性"是所有 NVMe 控制器都会实现的一组 FID，覆盖仲裁、电源、温度、写缓存、时间戳、热管理等基础策略，是日常调优和诊断的入口。

## 生活化类比

把通用特性想成**汽车仪表盘上的常用旋钮**：

- **仲裁（01h）** = "动力分配"：高速/中速/低速档的进油比例。
- **电源管理（02h）** = "档位选择"：手动指定当前档位。
- **温度阈值（04h）** = "水温报警阈值"：高于/低于某温度就报警。
- **易失性写缓存（06h）** = "急加速缓冲区"开关：开就冒险但快，关就保守但稳。
- **APST（0Ch）** = "自动启停"：空闲 N 毫秒后自动熄火。
- **HCTM（10h）** = "发动机降功率"：达到温升时自动限制输出。
- **Timestamp（0Eh）** = "行车电脑时钟"：控制器自带计时器。

## 工作流程

```text
  Set Features (FID)
       |
       +-- 01h Arbitration          调整 HPW/MPW/LPW/AB
       +-- 02h Power Management     切到指定 PS（可带 WH）
       +-- 04h Temperature Threshold 过温/低温阈值 + 迟滞
       +-- 06h Volatile Write Cache WCE 开关
       +-- 0Ch APST                 256B 表：32 个非操作态超时
       +-- 0Eh Timestamp            主机/控制器间 48-bit 毫秒计时
       +-- 0Fh Keep Alive           见 Keep Alive Timer
       +-- 10h HCTM                 两级降功率阈值 TMT1/TMT2
       +-- 12h Read Recovery Level  NVM Set / 子系统作用域
       +-- 1Ah Spinup Control       旋转介质启停
       +-- 1Bh Power Loss Signaling 掉电响应模式
       +-- 80h Software Progress    预启动计数器
       v
  控制器生效（后续命令；正在执行的命令可能用旧值）
```

## 初学者案例

**场景：想让空闲 SSD 自动进低功耗，又怕后台 GC 失败。**

1. `nvme get-feature -f 0x0c` 看 APST 当前状态：CDW11 `APSTE=0`（默认禁用）。
2. `nvme get-feature -f 0x11` 看 NOPPME（FID `11h`）：`NOPPME=1`。
3. 准备 256B 物理连续的 APST 表：每 8 字节一项，共 32 项；目标非操作态的项写非零 `ITPT`（毫秒） + `ITPS`（目标 PS 索引）；不支持的 PS 项必须清零。
4. `nvme set-feature -f 0x0c -v 1`（`APSTE=1`）并把表指针写到 DPTR。
5. 此时 `APSTE=1 + NOPPME=1` 进入 Figure 394 "宽松模式"：允许后台 GC 在非操作态短暂突破功耗上限，便于热管理与性能恢复。
6. 若想"严格模式"（`APSTE=0 + NOPPME=0`），后台工作不得超出功耗限制，代价是热管理可能受限。

> 排错提示：APST 表必须页对齐且物理连续；填错的 `ITPS` 应指向非操作态，否则控制器会忽略。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| Arbitration 字段（CDW11，Figure 386） | `31:24` HPW / `23:16` MPW / `15:08` LPW（0's based）/ `02:00` AB（2 的幂，`111b`=无限制） |
| Power Management（02h，Figure 387） | `04:00` PS（必须 NPSS 支持）；`07:05` WH（Workload Hint）；进入非操作态期间功耗可超该态标称值 |
| Temperature Threshold（04h） | 复合温度过温阈值**强制**实现；传感器 1-8 在非零时也必须支持过温+低温 |
| TMPSEL 字段（Figure 389） | `0h`=Composite；`1h-8h`=Sensor 1-8；`9h-Fh`=保留 |
| THSEL 字段 | `00b`=Over Temperature；`01b`=Under Temperature；`10b-11b`=保留 |
| TMPTHH 字段 | `000b`=无迟滞；`TMPTHMH=0` 时此字段必须 `000b` |
| 默认过温阈值 | Composite 来自 Identify `WCTEMP`；各 Sensor 1-8 默认 `FFFFh` |
| 默认低温阈值 | Composite 由实现定义；各 Sensor 1-8 默认 `0h` |
| 迟滞触发/恢复 | 过温：≥ 阈值起；< 阈值 - TMPTHH 止；低温：≤ 阈值起；> 阈值 + TMPTHH 止 |
| Volatile Write Cache（06h） | 仅 WCE 一位；非易失缓存不属于此特性；无易失缓存时 Get/Set 都回 `Invalid Field in Command` |
| APST（0Ch） | 256B 物理连续数据，32×8 字节；CDW11 `APSTE`(bit 0) 启用；不支持的 PS 项清零 |
| APST 条目 | `63:32` 保留；`31:08` ITPT（毫秒，`0`=禁用）；`07:03` ITPS（非操作态） |
| APST × NOPPME 交互 | Figure 394 四种组合决定"是否能自动进入非操作态"和"后台工作是否允许超功耗" |
| Timestamp（0Eh） | 48-bit 毫秒；`Origin`/`SYNC` 标记；复位后可能回退；**不**用于安全场景 |
| HCTM（10h） | TMT1/TMT2 必须在 Identify 范围内；任一为 0 禁用该级；同时非零要求 TMT1 < TMT2 |
| Read Recovery Level（12h） | 支持 NVM Set 时作用单 NVM Set；否则作用整个子系统；NS 继承所属 NVM Set |
| Spinup Control（1Ah） | 仅适用于含旋转介质的 EG；无则返回错误；配置持久化 |
| Power Loss Signaling（1Bh） | 模式 Disabled / Emergency Power Fail / Forced Quiescence；`Forced Quiescence` 处理中**不可**改此配置 |
| Software Progress Marker（80h） | 8-bit 饱和计数器；预启动成功 +1，OS 启动后清零；持久化 |
| 多控制器冲突 | 多个控制器请求冲突 PS 时，最终 Domain PS 未指定；建议主机协调 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 仲裁权重 vs AB | 权重决定分配比例；AB 决定单次最多从某 SQ 取多少条 |
| 仲裁 HPW vs WRR High | 仲裁 HPW 是 WRR 加权值；WRR 优先级是 SQ 创建时定的"档位" |
| Power Mgmt PS vs APST ITPS | PS 是手动指定；ITPS 是空闲超时的目标 |
| THSEL vs TMPSEL | THSEL=过温/低温；TMPSEL=复合/哪个传感器 |
| 迟滞起/止 vs 阈值触发 | 起/止是"带迟滞的两条边界"；阈值触发只是单条边界 |
| 复合温度过温 vs 传感器过温 | 复合过温强制；传感器过温"实现后才必须" |
| APST 启用 vs APST 表内容 | APSTE=1 必须 + 表正确；只设 APSTE 不给表无效 |
| WCE=0 vs 无易失缓存 | WCE=0 表示有但禁用；无易失缓存则 Get/Set 都报错 |
| 紧急掉电 vs 强制静默 | 紧急掉电快速保存；强制静默有序关闭；具体语义见规范 8.1.17 |
| Timestamp vs Keep Alive Timer | Timestamp 是 48-bit 计数器；Keep Alive 是定期间隔 |
| TMT1 vs TMT2 | TMT1=轻度降功率；TMT2=重度降功率；TMT1<TMT2 |

## 进阶细节

- **Figure 386 Arbitration CDW11**（规范 5.1.25.1.1）：
  - `31:24` HPW（0's based）
  - `23:16` MPW（0's based）
  - `15:08` LPW（0's based）
  - `07:03` 保留
  - `02:00` AB：`000b=1`, `011b=8`, `111b`=无限制
- **Figure 387 Power Mgmt CDW11**：`31:08` 保留；`07:05` WH；`04:00` PS。
- **Figure 388 Power Mgmt CQE DW0**：返回**当前**或**正在转入**的 PS。
- **Figure 389 Temp Threshold CDW11**：
  - `31:25` 保留
  - `24:22` TMPTHH（开尔文）
  - `21:20` THSEL（00b/01b）
  - `19:16` TMPSEL（0h-8h 有效，9h-Fh 保留）
  - `15:00` Temperature Threshold Value（开尔文）
- **Figure 392 APST CDW11**：`31:01` 保留；`00` APSTE。
- **Figure 393 APST 条目（每条 64 bit）**：
  - `63:32` 保留
  - `31:08` ITPT（毫秒）
  - `07:03` ITPS（非操作态 PS）
  - `02:00` 保留
- **APST 表约束**：256B，32 项，物理连续；不支持 PS 对应条目清零；项 0 对应 PS 0，依次递增。
- **Figure 394 APSTE×NOPPME**（4 种组合）：
  - `1,1` 自动+允许超功耗（宽松）
  - `0,1` 仅主机请求+允许超功耗
  - `1,0` 自动+不允许超功耗
  - `0,0` 仅主机请求+不允许超功耗（严格）
- **HCTM 行为**：观察复合温度；TMT1 触发"性能影响最小"的降温；TMT2 触发"无论性能影响"的强制降温；恢复需 < TMT1（具体迟滞由厂商定）。
- **Forced Quiescence 处理中**禁止通过 Set Features 改 `1Bh`。
- **Spinup Control 错误码**：子系统无含旋转介质的 EG 时返回 `Invalid Field in Command`。
- **APST 默认**：APSTE 默认 `0`（禁用）。
- **Timestamp 行为**：复位后时间可能回退；`SYNC=1` 提示存在厂商特定间隔未计入（如深度休眠期间）。

## 规范依据

- [通用特性 Set/Get 边界与可变性，PDF 第 395 页](../_source/pages/page-395.md)
- [Arbitration / Power Management 字段定义，PDF 第 396 页](../_source/pages/page-396.md)
- [温度阈值 + 迟滞，PDF 第 397 页](../_source/pages/page-397.md)
- [温度选择器与易失性写缓存，PDF 第 398 页](../_source/pages/page-398.md)
- [APST 表 + APSTE×NOPPME Figure 394，PDF 第 400-401 页](../_source/pages/page-400.md)
- [Timestamp / Keep Alive，PDF 第 402-403 页](../_source/pages/page-402.md)
- [HCTM / NOPPME / Read Recovery，PDF 第 403-405 页](../_source/pages/page-403.md)
- [Spinup Control / Power Loss Signaling，PDF 第 411-412 页](../_source/pages/page-411.md)
- [Software Progress Marker，PDF 第 422 页](../_source/pages/page-422.md)

## 相关阅读

- [特性值与作用域](feature-values-and-scope.md) - Feature 状态机与作用域
- [主机内存缓冲区](host-memory-buffer.md) - FID 0Dh 启用与描述符
- [电源状态描述符](power-state-descriptors.md) - FID 02h 切换 PS 索引
- [主机元数据与管理地址](host-metadata-and-management-addresses.md) - 7Dh-7Fh 主机元数据
- [断电信号](power-loss-signaling.md) - FID 1Bh 失电行为配置
