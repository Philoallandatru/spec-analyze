# 识别命令模型（Identify Command Model）

## 一句话说明

Identify 是 NVMe 的"通用只读接口"：主机用一个 4 KiB 命令读出子系统、控制器、命名空间或资源目录等数据；`CNS` 选大类，`NSID`/`CNTID`/`CSI`/`CNSSID`/`UIDX` 按 `CNS` 规则细化选择器，返回值统一 4 KiB。

## 生活化类比

把 Identify 想成**"政府信息公开窗口"**：

- **CNS = 你要查什么档案**（00h=某个 NS 的身份证，01h=控制器的营业执照，02h=所有 NS 列表……）。
- **NSID = 查几号窗口的档案**（按需）。
- **CNTID = 查哪个子公司**（按需）。
- **CSI = 查哪个业务口**（NVM / KV / Zoned / SLM / CP）。
- **UIDX = 查"带 UUID 后缀"的版本**（按需）。
- **结果 = 一份 4 KiB 的标准表格**，多余字段填 0。

## 工作流程

```text
  Identify 命令（Admin OPC=06h）
  CDW10: CNTID(31:16) | CNS(7:0)
  CDW11: CSI(31:24)   | CNSSID(15:0)
  CDW14: UIDX(6:0)
       |
       v
  控制器按 CNS 选数据结构：
    00h  Identify Namespace（按 NSID/CSI）
    01h  Identify Controller
    02h  Active NSID list
    03h  NS Identification Descriptors
    04h  NVM Set list（按 CNSSID.NVMSETID）
    05h/06h  CSI-specific NS/Controller
    07h  CSI-filtered active NSID list
    08h  CSI-independent NS
    0Ah  Allocated NSID list
    10h/11h  Allocated NSID list/NS data
    12h/13h  Attached/I/O Controller list
    14h/15h  Primary/Secondary Controller
    16h/17h  NS Granularity/UUID list
    18h/19h  Domain/Endurance Group list
    1Ah/1Bh  CSI-allocated NSID list/NS data
    1Ch       I/O Command Set 数据（按 CNTID）
    1Dh       Underlying NS list（仅 Fabrics；PCIe 禁止）
    1Eh       Ports list
    1Fh       I/O CS Independent Allocated NS data
    20h       Supported Controller State Formats
       |
       v
  返回 4 KiB 数据结构（未用字段填 0）
```

## 初学者案例

**场景：想看自己 SSD 支持多大的 HMB，但读出来 HMPRE=0。**

1. `nvme id-ctrl` 走 CNS=`01h`、CNTID=0、CSI=0。
2. 看 `HMPRE` 字段（byte 262-263 附近）= 0。
3. 含义：HMPRE=0 是"advertise 无 HMB"，**不是**"HMB 不可用"；再查 `HMMIN`（byte 278 附近）= 0 表示任意可行。
4. 但因为 `HMPRE=0`，**没有 HMB 能力**，driver 就不应尝试 Set Features `0Dh`（EHM）。
5. 同时检查 `CTRATT.HMBR`（byte 96-99）= 0，说明"非操作态访问限制"也不支持，HMNARE 写 1 必报 `Invalid Field in Command`。
6. 排错：HMB 必须看多个字段（`HMPRE`+`HMMIN`+`CTRATT.HMBR`），不能只看一个。

> 排错提示：Identify Controller 是 4 KiB 的"能力目录"，**单字段 0 不一定 = 不可用**——必须看周围的能力位和关联字段。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 命令 | Admin 命令，OPC=`06h`（Identify） |
| 返回大小 | 固定 4 KiB |
| 缓冲 | PRP 至多跨 1 个页边界；**不是** PRP List 指针 |
| CNS 选择器 | 8-bit（CDW10 `07:00`） |
| CNTID | 16-bit（CDW10 `31:16`），不用的 `CNS` 必须置 0；`CNTID=0` 是合法值 |
| CSI | 8-bit（CDW11 `31:24`），不用的 `CNS` 置 0 |
| CNSSID | 16-bit（CDW11 `15:0`），由 `CNS` 决定含义 |
| UIDX | 7-bit（CDW14 `6:0`），需控制器支持 UUID 选择 |
| 不支持的 CNS | 回 `Invalid Field in Command` |
| 命令集不匹配 | `Invalid I/O Command Set`（NS 关联的 I/O CS 与 `CNS` 要求不一致时） |
| CSI 编码 | `00h`=NVM；`01h`=KV；`02h`=Zoned；`03h`=SLM；`04h`=Computational；`05h-2Fh`=保留；`30h-3Fh`=Vendor；`40h-FFh`=保留 |
| 传输分区 | 通用结构 / 仅内存传输 / 仅消息传输三组分别定义 |
| 1.0/1.1 控制器 | 主机只发该版本定义的 `CNS`，其他结果未定义 |
| Discovery 控制器 | 部分身份/能力字段保留或条件性，I/O/Admin 列不能假设 |
| CNS 列表规范 | Active NS(00-0Ah) / Mgmt(10-17h) / Inventory(18-1Eh) / Views(1Ah-20h) |
| 命名空间识别描述符 | 变长；以 `NIDL=0` 结束；每类型不重复；至少含一种全球唯一标识（EUI64/NGUID/UUID） |
| NSID 列表分页 | 最多 1024 个递增 NSID，**严格大于**输入 NSID |
| NVM Set 列表分页 | 最多 31 个 128 字节条目，**大于等于**输入 NVM Set ID |
| Controller 列表分页 | 最多 2047 个，**大于等于**输入 CNTID |
| Capability-only 模板 | `NSID=FFFFFFFFh` 时返回能力模板（仅能力字段，非能力字段清 0） |
| Inactive NS | 返回全 0 的 NS 结构 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Active NSID vs Allocated NSID | Active = 创建后已 Attach；Allocated = 已分配（无论是否 Attach） |
| `CNS 00h` vs `CNS 05h` vs `CNS 08h` | 00h=NVM 命令集特定；05h=CSI 特定；08h=CS 无关 |
| `CNS 02h` vs `CNS 0Ah` vs `CNS 10h` | 02h=Active；0Ah=Allocated 基线；10h=Allocated 详细 |
| `MDTS=0` vs `MDTS` 大 | 0 = "无上限"；非 0 = 2^n 内存页数上限 |
| `NN` vs `MNAN` | NN = 最大 NSID（不一定是 NS 数量）；MNAN = 支持的 NS 数量上限 |
| `MAXCMD` vs SQ size | MAXCMD = 单条 SQ 同时可处理命令数；可大于 SQ 大小 |
| `SGLS=00b` vs `SGLS` 位 | 00b 禁用 SGL（无论其他位）；位 1+ 仅 00b 后才解读 |
| `FNA.FNVMBS=1` 含义 | "广播 Format 不支持"，且强制其他 format/erase scope 位清 0 |
| `HMPRE=0` 含义 | "不广告 HMB 能力"，**不**是"HMB 不可用" |
| `CTRATT.HMBR=0` | 不支持 HMB 非操作态访问限制；`HMNARE=1` 必报错 |
| Discovery 控制器 vs 普通控制器 | Discovery 许多身份字段保留；`SN`/`MN` 不可用于构造唯一标识 |

## 进阶细节

- **CDW10 位定义**（Figure 307）：`31:16` CNTID；`07:00` CNS。
- **CDW11 位定义**（Figure 308）：`31:24` CSI；`15:00` CNSSID。
- **CDW14 位定义**（Figure 309）：`06:00` UIDX。
- **Figure 310 CNS 表**（部分）：
  - `00h` M Identify Namespace（NSID=Y, CNTID=N, CSI=N8）
  - `01h` M Identify Controller（NSID=N, CNTID=N, CSI=N）
  - `02h` M Active NSID list（NSID=Y, CNTID=N, CSI=N）
  - `03h` M NS Identification Descriptor list（NSID=Y）
  - `04h` O NVM Set list（NSID=Y, CNSSID=NVMSETID）
  - `05h` M1 CSI-specific NS（NSID=Y, CSI=Y）
  - `06h` O1 CSI-specific Controller（CSI=Y）
  - `07h` O1 CSI-filtered Active NSID list（NSID=Y, CSI=Y）
  - `08h` O1 CSI-independent NS（NSID=Y）
  - `09h-0Bh` O/M 各种 NS 列表与描述符
  - `0Ch/0Dh/0Eh` O 大型 NSID/UUID/NVM Set 列表
  - `10h/11h` O/M Allocated NSID list/NS data
  - `12h/13h` O Attached/I/O Controller list（CNTID=Y）
  - `14h/15h` O Primary/Secondary Controller
  - `16h/17h` O NS Granularity/UUID list
  - `18h/19h` O Domain/Endurance Group list
  - `1Ah/1Bh` O CSI-allocated NSID list/NS data
  - `1Ch` O I/O Command Set data（CNTID=Y）
  - `1Dh` O Get Underlying NS list（**仅 Fabrics**；PCIe 禁止）
  - `1Eh` O Get Ports list
  - `1Fh` O I/O CS Independent Allocated NS data
  - `20h` O Supported Controller State Formats
  - `21h-FFh` 保留
- **Capability 关键字段**：
  - 0-75：身份、固件、仲裁推荐
  - 76：CMIC（拓扑/多端口/ANA 报告）
  - 77：MDTS（最大数据传输）
  - 78-91：控制器身份、版本、Runtime D3 延迟
  - 92-95：OAES（可选异步事件）
  - 96-99：CTRATT（跨能力属性）
  - 100-101：RRLS（读取恢复等级）
  - 102-262：能力目录（管理、命令、日志等）
  - 263-271：电源/温度/激活时间
  - 272-311：HMB + 容量报告
  - 312-315：RPMB
  - 316-523：自检、固件、热、Sanitize、ANA、KPIOC、CQT
  - 524-535：Format/缓存/原子/写保护/Copy
  - 536-539：SGL 能力位图
  - 540-587：NS 附件 + refresh + 变更跟踪 + CDQ
  - 768-1023：子系统 NQN
  - 1792-1806：Fabrics capsule/连接/Discovery 属性
  - 2048-3071：电源状态描述符开始
- **重要字段定义**：
  - `MDTS=0` 含义是"无上限"；非 0 = `2^n` 内存页数
  - `MAXCMD` 可超过 SQ 大小；Fabrics 必报，PCIe 可选
  - `NN` = 最大 NSID（不=NS 数量）
  - `MNAN` = 支持 NS 数量上限；`MNAN=0` 时回退为 ≤ NN
  - `NPSS` = 支持的 PS 数，0-based，至少有 PS 0
  - `HMPRE` 非 0 时必须 ≥ `HMMIN`；为 0 时 HMB 不可用
  - `SANICAP` 非 0 时支持 Sanitize 命令
  - `KPIOC` 区分子系统范围 vs 单 NS 启用
  - `CQT` = 0 表示"未报告"；如要标 1ms 即 1
  - `DSTO.HIRS` 需要 Device Self-test 支持
  - `DSTO.SDSO` 选子系统单操作 vs 每控制器单操作
  - `MNTMT`/`MXTMT` 限制 HCTM 阈值范围
  - `VWC` 单独报告缓存存在和 broadcast Flush 是否合法
  - `AWUN`/`AWUPF`/`ACWU` 仅适用于逻辑块命令集
  - `CDFS` 枚举支持的 Copy 描述符格式
  - `SGLS` 0=禁用；位 1+ 在 00b 后才解读
- **NS 识别描述符**（CNS `03h`）：变长；以 `NIDL=0` 结束；每类型不重复；至少一个 EUI64/NGUID/UUID；多 CS 时含 CSI；跨 reset/format 持久。
- **I/O Command Set Combination**（CNS `1Ch`）：每个组合是"一组同时可用的 CS 向量"；索引 0 保留；Feature `19h` 选当前活跃组合；未选中的 CS 视为不支持。
- **Supported Controller State Formats**（CNS `20h`）：迁移目录，两部分：标准版本 + 厂商 UUID；Migration Receive/Send 用此索引。
- **特殊边界**：
  - `CNS 1Dh` 在任何控制器用内存传输的子系统中**禁止**
  - `FNA.FNVMBS=1` ⇒ broadcast Format 不支持
  - `HMPRE=0` ⇒ 无 HMB；`HMPRE`>0 ⇒ 必有 HMB 能力
  - 任何 `CNS` 不支持时 `Invalid Field in Command`

## 规范依据

- [Identify 概述与 Data Pointer，PDF 第 317 页](../_source/pages/page-317.md)
- [CDW10/CDW11/CDW14 + 选择错误规则 Figures 307-309，PDF 第 318 页](../_source/pages/page-318.md)
- [CNS 值矩阵 Figure 310，PDF 第 319-320 页](../_source/pages/page-319.md)
- [CSI 值 Figure 311，PDF 第 320 页](../_source/pages/page-320.md)
- [Identify Controller 身份与拓扑 Figure 312，PDF 第 321-326 页](../_source/pages/page-321.md)
- [能力目录中段，PDF 第 327-334 页](../_source/pages/page-327.md)
- [Operational / I/O envelope Figure 312 续，PDF 第 335-342 页](../_source/pages/page-335.md)
- [Format/缓存/SGL/CDQ/Fabrics 末段，PDF 第 343-350 页](../_source/pages/page-343.md)
- [Power-state 描述符开始 + CNS 02h-04h 资源列表，PDF 第 351-355 页](../_source/pages/page-351.md)
- [CSI-specific + Independent NS 视图，PDF 第 355-360 页](../_source/pages/page-355.md)
- [Allocated / Controller / UUID / Domain / EG 目录，PDF 第 361-366 页](../_source/pages/page-361.md)

## 相关阅读

- [身份名称与列表格式](identity-name-and-list-formats.md) - 读出的内容字段定义
- [通用命令格式](common-command-format.md) - Identify 是 Admin SQE 命令
- [电源状态描述符](power-state-descriptors.md) - Identify Controller 含 PSD 数组
- [控制器属性](controller-properties.md) - 控制器属性由 Identify 报告
- [NVM 集与耐久度组](nvm-sets-and-endurance-groups.md) - Identify 列出 EG/Set 列表
