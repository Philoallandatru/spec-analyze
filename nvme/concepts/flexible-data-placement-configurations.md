# 灵活数据放置配置（Flexible Data Placement Configurations）

## 一句话说明

FDP Configurations 日志（LID `20h`）是**耐久度组（EG）级别**的"静态目录"，列出该 EG 当前**可被启用**的 FDP 放置配置；主机从目录里挑一条生效，才能进入动态的 Usage/Statistics/Events 观测。

## 生活化类比

把耐久度组想成一个**分拣中心**：

- **目录（FDP Configurations log）** = 中心墙上挂的"分拣方案表"：每张方案卡写明分拣格（回收组）数、每个格有几个篮子（回收单元句柄）、篮子之间怎么隔开放置。
- **运行时日志** = 实时看板：哪个篮子刚被装满、累计装了多少、最近一次重定向了哪个把手。
- **FDP Feature（`1Dh`）** = 现场主管的"按下哪张方案卡"的开关。

只有挂在墙上的方案才能被启用；启用时方案里规定的"隔开放置级别"（普通/初始隔离/持续隔离）也立刻决定数据后续怎么混。

## 工作流程

```text
耐久度组 (ENDGID) 选择
 └─ FDP 配置描述符 (FDPCV=1 ?)
      ├─ NRG (回收组数) + RGIF (RGID 占 Placement ID 多少 MSB)
      ├─ NRUH (回收单元句柄数) + 按 RUH ID 升序排列的句柄类型
      ├─ 命名空间上限 (NNS, 受 MNAN 或 NN 限制)
      └─ 标称 RU 容量 + ERUTL (句柄重定向时间上限)
              │
              ▼  Set Features FDP (1Dh) 选一个有效 index 启用
              │
   ┌──────────┼─────────────────────┐
   ▼          ▼                     ▼
RU Handle   FDP Statistics       FDP Events
Usage       (LID 22h)            (LID 23h)
(LID 21h)   累计 128-bit 计数    4 KiB 旧→新事件历史
```

简化说明：上面左边是"静态目录"，下面三块是"运行时观测"，都按 EG 范围生效。

## 初学者案例

**场景：主机想给热数据开 FDP，但写命令被回 `FDP Disabled`。**

1. 主机先 `nvme get-log /dev/nvme0n1 --log-id=0x20 --log-len=...` 读 FDP Configurations log（按 ENDGID 选 EG），看有几个描述符。
2. 发现只有一个描述符的 `FDPCV=0`——表示"现在这条不可用"，其它也都没有。这是耐久度组刚建好、还没启用 FDP。
3. 主机 `nvme set-feature /dev/nvme0 -f 0x1d -v <index>`：选目录里 `FDPCV=1` 的 index，并把"使能"位置 1。
4. 此后写命令才允许带 `DPD`（Data Placement Directive）字段；FDP 关闭时强行 DPD 会触发 FDP Disabled 错误。
5. 故障速查：被回 FDP Disabled 时，先确认目录里 `FDPCV=1` 的 index 是否被实际启用，而非只读目录不启用。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 日志作用域 | FDP Configurations log（LID `20h`）按 EG 范围，按 `ENDGID` 选择 |
| `NUMFDPC` | 0-based 的配置描述符数量 |
| `FDPCV=1` | 该配置**当前可用**；`FDPCV=0` 时主机应忽略 |
| `NRG` / `NRUH` | 两者均非 0 |
| `RGIF` | 仅在 NRG > 1 时非 0；决定 Placement Identifier 中"RGID 占几位 MSB" |
| `MAXPIDS` | 0-based 上限，且必须 `< NRG * NRUH` |
| `NNS` | 命名空间上限，非 0 时受 `MNAN` 约束，否则受 `NN` 约束 |
| 句柄列表顺序 | 描述符内按 Reclaim Unit Handle Identifier 升序排列 |
| 描述符变长 | 各描述符长度可变，末尾补 0 至 8 字节边界 |
| `FDPVWC=1` | 该配置存在易失写缓存，会强制 Identify Controller `VWCP=1`（即便控制器整体无 VWC） |
| `ERUTL` | 句柄重定向时间上限；0 = 未上报 |
| 启用副作用 | Set Features FDP（`1Dh`）切换配置可能改变 Identify 的 User Data Format 字段，主机需重新发现 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| FDP Configurations log vs FDP Feature `1Dh` | 日志是"目录"；Feature 是"选目录里的某条 + 启用" |
| 初始隔离 vs 持续隔离 | 初始隔离：GC 后可能同组同 type 句柄数据混住；持续隔离：GC 后仍在同一 RUH 内的 RU |
| `RGIF=0` vs `RGIF>0` | RGIF=0 → Placement ID 全部是 Placement Handle（NRG=1）；RGIF>0 → 高位为 RGID，低位为 PHNDL |
| Usage vs Statistics vs Events | Usage = 句柄当前被谁占用；Statistics = 累计字节/擦除计数；Events = 4 KiB 事件历史 |
| `FDPVWC=1` vs 控制器真有 VWC | `FDPVWC=1` 只让 `VWCP=1` 报告"有 VWC"，与一般控制器有无 VWC 解耦 |

## 进阶细节

- **LID `20h` 头部**（规范 5.1.12.1.28 / PDF 第 294 页）：
  - `NUMFDPC`（16 位，0-based）：配置描述符数量；
  - `VER`（8 位）：当前必须为 0；
  - `SZE`（32 位）：整页字节数；
  - 16 字节头后是变长描述符列表。
- **描述符 DSZE + FDPA 属性字节**（规范 5.1.12.1.28 / PDF 第 294 页）：
  - `DSZE`（16 位）：本描述符字节数；
  - `FDPCV`（bit 7）：配置当前是否可用；
  - `FDPVWC`（bit 4）：本配置存在 VWC → Identify Controller `VWCP=1`。
- **句柄类型 `02h` Persistently Isolated**（规范 5.1.12.1.28 / PDF 第 296 页）：本句柄写入的用户数据原本就隔离在引用 RU 内；若控制器因 GC 等厂商特定操作搬动数据，则**只能搬到同一 RG 内、只含同一 RUH 写入数据的 RU**。
- **句柄类型 `00h/01h`**：`00h` = 未指定；`01h` = Initially Isolated（GC 后同 type 句柄在同 RG 内可能共址）。
- **Placement Identifier 格式**（规范 5.1.12.1.28 / PDF 第 296 页）：
  - `NRG=1` 且 `RGIF=0` → 整个 Placement Identifier = Placement Handle（Figure 282，16 位 PHNDL）；
  - `RGIF>0` → 高位是 RGID、低位是 PHNDL（Figure 283）。
- **运行时三件套**（规范 5.1.12.1.29-31 / PDF 第 296-298 页）：

| 日志 | LID | 稳定契约 |
|------|-----|----------|
| RU Handle Usage | `21h` | 每 RUH 一条描述符：`unused` / 主机显式占用 / 单条控制器选择（最多 1 条控制器选择） |
| FDP Statistics | `22h` | 128-bit 主机写入 / 介质写入 / 介质擦除字节计数（饱和）；切换 FDP Feature 清零；固件更新**不清零** |
| FDP Events | `23h` | 固定 4 KiB 旧→新事件历史；按事件类型筛选；仅在对应句柄的事件被启用时记录 |

- **日志启用前提**（规范 5.1.12.1.29-31）：FDP 启用时 `NSID` 被保留；FDP 关闭时读这些日志返回 `FDP Disabled`。Statistics 聚合 EG 内"自 FDP 上次启用以来存在过"的所有命名空间。
- **事件历史行为**（规范 5.1.12.1.31 / PDF 第 298-299 页）：
  - 历史满时，新启用的事件会丢弃最旧条目；
  - FDP 从禁用→启用时事件计数清零；
  - 顺序按"发生顺序"，不严格按时间戳——因 Controller Level Reset 后 Timestamp 配置可能变。
- **事件有效性位**（规范 5.1.12.1.31 / PDF 第 300-301 页）：Placement Identifier / NSID / RG 与 RUH 位置各自独立有效；主机事件涵盖：提前句柄重定向、时限到期、Reset 引发重定向、非法放置；控制器事件涵盖：介质再分配、隐式重定向。
- **FDP Events Feature（`1Eh`）**（规范 5.1.25.1.30 / PDF 第 413-414 页）：按 EG 内某命名空间"经由某 Placement Handle 触达"的 RUH 启用有序事件类型；FDP 关闭时该 Feature 无效；拒绝广播 NSID；同 EG 内跨命名空间共享同一 RUH；Get Features 返回"升序支持类型 + 每类启用状态"。
- **Set Features FDP（`1Dh`）**（规范 5.1.25.1.29 / PDF 第 412 页）：为指定 EG 选一个有效配置 index 并启用/禁用 FDP；属 mandatory-save；切换后可能影响 Identify 报告的 User Data Format 字段。
- **章节号**：日志定义在 5.1.12.1.28-31；Feature 在 5.1.25.1.29-30；主机使用流程在 8.1.10。

## 规范依据

- [FDP Configurations 头部与属性，PDF 第 294 页](../_source/pages/page-294.md)
- [配置上限、变长布局与句柄类型，PDF 第 295 页](../_source/pages/page-295.md)
- [持续隔离、Placement Identifier 拆分与 Usage 日志，PDF 第 296 页](../_source/pages/page-296.md)
- [Usage 描述符与 FDP Statistics 边界，PDF 第 297 页](../_source/pages/page-297.md)
- [Statistics 计数器与 FDP Events 保留，PDF 第 298 页](../_source/pages/page-298.md)
- [FDP Feature（`1Dh`）启用，PDF 第 412 页](../_source/pages/page-412.md)
- [FDP Events Feature（`1Eh`）选择器，PDF 第 413 页](../_source/pages/page-413.md)

## 相关阅读

- [NVM 集与耐久度组](nvm-sets-and-endurance-groups.md) - FDP 替代 NVM Set 组织
- [存储资源层次结构](storage-resource-hierarchy.md) - 资源层级背景
- [特性值与作用域](feature-values-and-scope.md) - FDP Feature 1Dh 启用
- [日志页读取](log-page-retrieval.md) - FDP 日志靠 Get Log Page
- [指令交换](directive-exchange.md) - 也涉及 placement 标识
