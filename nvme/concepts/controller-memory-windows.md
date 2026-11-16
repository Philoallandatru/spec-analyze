# 控制器内存窗口（Controller Memory Windows）

NVMe 控制器可通过 PCIe BAR 向主机暴露两种可选的控制器侧内存窗口：**CMB（Controller Memory Buffer）**用于低延迟存放队列、PRP/SGL 列表或命令数据（不保证掉电持久）；**PMR（Persistent Memory Region）**则在 CMB 的基础上额外提供**持久化语义、写屏障与健康状态监控**。[PDF pp. 86-94](../_source/pages/page-086.md)

可以把它想成**酒店给客人的两种权限**：CMB 像“高速寄存室”，方便随时存取队列与临时数据，但断电不保；PMR 像“带健康监测的保险柜”，放进去的数据在执行写屏障后断电不丢。

## CMB：地址模型与能力维度

```text
PCI BAR（由 CMBLOC.BIR 选中，BAR 地址按 4 KiB 对齐）
+----------------------------------------------------------+
| BAR base                                                 |
|   + CMBLOC.OFST * CMBSZ.SizeUnit                         |
|        +---------------- CMB ----------------------+      |
|        | 队列 / PRP-SGL List / 读-写数据            |      |
|        +-------------------------------------------+      |
+----------------------------------------------------------+

可选“控制器地址视图”：
  CMBMSC.CRE=1              -> CMBLOC/CMBSZ 有效
  CMBMSC.CMSE=1 + 有效 CBA  -> 主机访问该地址范围路由至 CMB
```

- **可用大小**：`CMBSZ.SZ * SizeUnit`；若“偏移 + 大小”超出所选 BAR 边界，实际可用空间会被截断。[PDF pp. 86-89](../_source/pages/page-086.md)
- **能力位判断原则**：`CAP.CMBS=1` 才存在 CMB；某个能力位清零意味着对应的放置限制仍然生效，**不能**仅凭“CMB 存在”就推断支持某种用途。[PDF pp. 86-88](../_source/pages/page-086.md)

| CMB 能力维度 | 声明字段 |
|---|---|
| 存放 SQ / CQ | `CMBSZ.SQS`、`CMBSZ.CQS` |
| 存放 PRP/SGL 列表 | `CMBSZ.LISTS` |
| 主机→控制器写数据 / 控制器→主机读数据 | `CMBSZ.WDS`、`CMBSZ.RDS` |
| 非连续或混合内存队列 | `CMBLOC.CQPDS`、`CMBLOC.CQMMS` |
| 数据/列表/命令混合放置 | `CMBLOC.CDMMMS`、`CDPCILS`、`CDPMLS` |
| 队列 dword 对齐 | `CMBLOC.CQDA` |

- **`CMBSTS.CBAI=1`**：表示控制器地址空间启用失败（基地址无效）。弹性缓冲大小、旁路行为与持续写吞吐均独立报告，**`0` 表示“未提供此信息”**，不代表特性不存在。[PDF pp. 89-91](../_source/pages/page-089.md)

## PMR：启用握手、写屏障与健康状态

```text
[PMR Disabled / NRDY=1]
          |  写 PMRCTL.EN=1；等待 PMRCAP.PMRTO * PMRTU（500 ms 或分钟）
          v
[PMR Enabled, 等待] ---- PMRSTS.CBAI / 健康错误 ---> [诊断处理]
          |  PMRSTS.NRDY=0（仅 EN=1 且 NRDY=0 时才算就绪）
          v
[PMR Ready：可被 PCIe 读/写]
          |  执行写屏障（内存读完成 + PMRSTS 读完成）
          v
[此前写入已持久化]
```

[PDF pp. 92-94](../_source/pages/page-092.md)

| `PMRSTS.HSTS` | 健康状态含义 |
|---|---|
| `000b` | 正常工作 |
| `001b` | 已持久且运行中，但恢复出的数据可能不正确 |
| `010b` | 只读（Read-Only） |
| `011b` | 不可靠；数据或持久化可能无效 |

- **错误历史**：`PMRSTS.ERR` 非零表示过去发生过写入错误，**直到 PCI Function Level Reset（FLR）才清零**。[PDF p. 94](../_source/pages/page-094.md)
- **PMR 控制器地址视图**：基地址拆分为 `PMRMSCU.CBA`（高 32 位）与 `PMRMSCL.CBA`（中 20 位），低 12 位隐式为 0；启用 `PMRMSCL.CMSE` 时，地址范围既不能溢出 64 位空间，也不能与已启用的 CMB 地址范围重叠。[PDF p. 96](../_source/pages/page-096.md)
- **吞吐与缓冲编码**：PMR 的弹性缓冲与持续写吞吐采用 `value * unit` 编码，`0` 同样表示未提供信息。[PDF p. 95](../_source/pages/page-095.md)

## CMB vs PMR 对比

| 维度 | CMB | PMR |
|---|---|---|
| **核心用途** | 低延迟控制器内存（绕过主机 DRAM 往返） | 持久化控制器内存（类 NVDIMM/字节寻址存储） |
| **持久化与屏障** | 不保证（重启后内容不一定保留） | 显式支持写屏障 + 健康状态监控 |
| **启用与状态** | `CMBMSC.CRE/CMSE` + `CMBSTS.CBAI` | `PMRCTL.EN` / `PMRSTS.NRDY` + `CBAI`/`HSTS`/`ERR` |
| **允许放置内容** | 按能力位放 SQ/CQ、PRP/SGL List、读/写数据 | 按能力位放读/写数据 |

## 关联与引申概念

- **主机内存缓冲（[Host Memory Buffer](host-memory-buffer.md)）**：与 CMB/PMR 方向相反的对偶机制——由主机借出系统 DRAM 给无本地 DRAM 的控制器使用。
- **控制器属性（[Controller Properties](controller-properties.md)）**：`CMBLOC`/`CMBSZ`/`CMBMSC`/`CMBSTS` 与 `PMRCAP`/`PMRCTL`/`PMRSTS` 均位于控制器属性空间。
- **启用、关机与重置（[Controller Enable, Shutdown, and Reset](controller-enable-shutdown-reset.md)）**：NVM Subsystem Reset 会禁用范围内的 PMR；CLR 保留 PMR 属性与 `CMBMSC`，而 FLR 仅保留 `CMBMSC`。
- **功耗状态（[Power State Descriptors](power-state-descriptors.md)）**：规定了控制器进入低功耗状态时 CMB 数据的保留行为。
- **PCIe BAR（Base Address Register）**：PCIe 设备向系统声明其内存或 I/O 地址窗口的寄存器，CMB/PMR 即映射在 BAR 空间内。

## 规范依据

- [CMB 位置与能力字段（Figures 47-48），PDF 第 86-88 页](../_source/pages/page-086.md)
- [CMB 控制器地址空间、状态与吞吐，PDF 第 89-90 页](../_source/pages/page-089.md)
- [PMR 能力/控制/状态（PMRCAP/PMRCTL/PMRSTS），PDF 第 92-94 页](../_source/pages/page-092.md)
- [PMR 吞吐编码与控制器地址视图，PDF 第 95-96 页](../_source/pages/page-095.md)
