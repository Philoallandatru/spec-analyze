# 识别命令模型（Identify Command Model）

**Identify（Admin OPC `06h`）**是 NVMe 的核心只读发现命令，用于从控制器获取固定 **4,096 字节（4 KiB）** 的描述结构，涵盖 NVM 子系统、域（Domain）、控制器、命名空间或资源清单。命令中的 `CNS`（Controller or Namespace Structure）决定返回哪类结构，`NSID`、`CNTID`、`CSI`、`CNSSID` 和 `UIDX` 则按所选 `CNS` 的规则进一步限定目标。[PDF pp. 317-319](../_source/pages/page-317.md)

可以把 Identify 想象成**"设备信息公开窗口"**：`CNS` 是你要申请的表格编号（`01h` 是控制器营业执照、`00h` 是命名空间身份证、`02h` 是活跃命名空间清单）；填好窗口号与筛选条件后，窗口统一递回一张 4 KiB 的标准表格，未使用的字段全填 0。

## 命令选择器与基本契约

```text
Identify 命令 (Admin OPC 06h)
  |-- CDW10 [07:00] CNS    ---> 返回哪类 4 KiB 结构或列表？
  |-- CDW10 [31:16] CNTID  ---> 哪个控制器？(未用时清零；CNTID=0h 本身合法)
  |-- NSID                 ---> 哪个命名空间或列表游标？(依 CNS 而定)
  |-- CDW11 [31:24] CSI    ---> 哪个 I/O 命令集？(00h NVM / 01h KV / 02h ZNS / 03h SLM / 04h CP)
  |-- CDW11 [15:00] CNSSID ---> CNS 专属选择器（如 NVMSETID）
  `-- CDW14 [06:00] UIDX   ---> UUID 索引（仅支持 UUID 选择时有效）
```

- **数据指针与错误**：每次返回固定 4 KiB，未用条目填 0；使用 PRP 时**不是** PRP List 指针（最多跨 1 个页边界）。不支持的 `CNS` 返回 `Invalid Field in Command`；命令集不匹配返回 `Invalid I/O Command Set`。[PDF pp. 317-319](../_source/pages/page-317.md)
- **版本与传输边界**：对 1.0/1.1 控制器仅应发送该版本定义的 `CNS`；`CNS 1Dh`（Underlying NS list）**仅限 Fabrics**，若子系统内任一控制器使用内存传输（PCIe）则禁止使用。[PDF pp. 318-320](../_source/pages/page-318.md)

## CNS 选择器族与分页规则

| `CNS` 族 | 代表性 `CNS` 与结果 | 游标 / 边界契约 |
|---|---|---|
| **活跃命名空间与基础结构** | `00h` Identify NS、`01h` Identify Controller、`02h` Active NSID list、`03h` NS Identification Descriptors、`04h` NVM Set list、`05h/06h` CSI 特定 NS/Controller、`07h` CSI 过滤 Active NSID list、`08h` CS 无关 NS | `02h` 最多返回 1,024 个**严格大于**输入 `NSID` 的递增 NSID；`04h` 最多返回 31 个**大于等于** `NVMSETID` 的 128B 条目 |
| **控制器与命名空间管理** | `10h/11h` 已分配 NSID 列表/NS 数据、`12h/13h` 已挂载/I/O 控制器列表、`14h/15h` 主/次控制器、`16h/17h` NS 粒度/UUID 列表 | `10h` 游标严格大于；`12h/13h` 最多 2,047 个**大于等于** `CNTID` 的 ID；`17h` 到首个零条目结束 |
| **资源清单与视图** | `18h/19h` Domain/EG 列表、`1Ah/1Bh` CSI 已分配列表/NS、`1Ch` I/O 命令集组合、`1Dh` 底层 NS（仅 Fabrics）、`1Eh` Ports、`1Fh` CS 无关已分配 NS、`20h` 支持的控制器状态格式 | `18h/19h` 从资源游标起（含）；超过 `ENDGIDMAX` 返回空列表（成功） |

- **能力模板与非活跃 NS**：对命名空间结构传入 `NSID=FFFFFFFFh` 可返回用于创建/格式化的能力模板（非能力字段清 0）；非活跃（Inactive）命名空间返回全 0 结构。[PDF pp. 355-357](../_source/pages/page-355.md)
- **NS 识别描述符（`CNS 03h`）**：变长列表，以 `NIDL=0` 结束，同类型不重复，至少含一种全球唯一持久标识（`EUI64`、`NGUID` 或 `UUID`），并在多 I/O 命令集时携带 `CSI`；跨重置和 Format 保持不变。[PDF pp. 353-354](../_source/pages/page-353.md)
- **I/O 命令集组合（`CNS 1Ch`）与迁移格式（`CNS 20h`）**：`1Ch` 列出最多 512 个可同时使用的命令集向量（通过 Feature `19h` 切换）；`20h` 列出标准控制器状态版本与厂商 UUID 迁移格式索引，供 Migration Send/Receive 使用。[PDF pp. 364-366](../_source/pages/page-364.md)

## Identify Controller（`CNS 01h`）关键能力区段

```text
Identify Controller (4,096 字节) 分段概览 (Figure 312)：
  0..101    身份(VID/SSVID/SN/MN)、拓扑(CMIC)、传输上限(MDTS)、版本、OAES、CTRATT、RRLS
  102..315  能力目录：管理面(OACS/ACL/AERL)、固件/日志(FRMW/LPA)、电源/热(NPSS/APSTA)、HMB(HMPRE/HMMIN)、容量(TNVMCAP)、RPMB
  316..523  运行与 I/O 包络：自检(DSTO)、Sanitize(SANICAP)、ANA(ANACAP)、KPIOC、CQT、队列条目大小(SQES/CQES)、MAXCMD、NN、ONCS、FUSES
  524..587  Format(FNA)、缓存(VWC)、原子性(AWUN/AWUPF/ACWU)、写保护、Copy(CDFS)、SGL(SGLS)、MNAN、刷新、变更跟踪与 CDQ 上限
  768..1023 子系统 NQN (SUBNQN)
  1792..1806 Fabrics 属性 (Capsule 大小、ICDOFF、Discovery 类型)
  2048..3071 电源状态描述符数组 (PSD 0..31，必须以状态 0 开头)
```

读懂 Identify Controller 时最容易踩坑的边界规则：[PDF pp. 321-350](../_source/pages/page-321.md)

- **`0` 不一定代表不支持**：`MDTS=0` 表示单次数据传输大小无上限；`HMMIN=0` 表示 HMB 可请求任意可行大小（上限到 `HMPRE`，但 **`HMPRE=0` 才表示不支持 HMB**）；`MNAN=0` 表示回退受限于 `NN`；`CQT=0` 表示未报告静默时间。
- **`NN` vs `MNAN`**：`NN` 是最大有效 NSID，**不是**支持的命名空间数量上限（后者是 `MNAN`）。
- **`MAXCMD` vs SQ 深度**：`MAXCMD` 是单队列同时允许未完成命令数上限，可以大于单个 Submission Queue 的深度（Fabrics 必报，PCIe 可选）。
- **依赖门控**：`SGLS=00b` 时禁用所有 SGL 用法（忽略其他 SGL 位）；`FNA.FNVMBS=1` 禁止广播 Format 并强制清零格式/擦除范围位；`ONCS` 是所有已支持/继承命令集的汇总，具体命令集支持需结合 `Commands Supported and Effects` 日志（LID `05h`）确认；Discovery 控制器对部分身份/能力字段保留，`SN/MN` 不能直接假设为普通 I/O 控制器语义。

## 规范依据

- [Identify 概述、命令字段与 CNS/CSI 选择矩阵（Figures 306–311），PDF pp. 317-320](../_source/pages/page-317.md)
- [Identify Controller（Figure 312）分段详解，PDF pp. 321-350](../_source/pages/page-321.md)
- [Power State 描述符与 CNS 02h–08h 资源/命名空间视图，PDF pp. 351-360](../_source/pages/page-351.md)
- [CNS 10h–20h 已分配、控制器、UUID、域/EG、I/O CS 组合与状态格式，PDF pp. 361-366](../_source/pages/page-361.md)

## 关联与引申概念

- **身份、名称与列表格式**：规范了 Identify 返回的 VID/OUI/EUI64/NGUID/UUID/NQN 字节序与列表格式。详见 [身份、名称与列表格式](identity-name-and-list-formats.md)。
- **命名空间标识符**：区分 Active NSID 与 Allocated NSID 的生命周期。详见 [命名空间标识符](namespace-identifiers.md)。
- **电源状态描述符（PSD）**：Identify Controller 字节 2048 起定义各功耗状态的延迟与功耗。详见 [电源状态描述符](power-state-descriptors.md)。
- **命令支持与效果（LID `05h`）**：用于精确查询每个 I/O 命令集下各命令的支持情况。详见 [命令效果与支持](command-effects-and-support.md)。
- **数据指针布局**：`MDTS` 与 `SGLS` 共同约束读写命令的 PRP/SGL 构造。详见 [数据指针布局](data-pointer-layouts.md)。
