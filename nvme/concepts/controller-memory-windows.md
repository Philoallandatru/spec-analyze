# 控制器内存窗口（Controller Memory Windows）

## 一句话说明

控制器内存窗口指 NVMe 控制器暴露给主机的两种可选内存：CMB（Controller Memory Buffer）可放队列/列表/数据，但**不保证持久**；PMR（Persistent Memory Region）在 CMB 基础上额外提供**持久化 + 健康状态 + 写屏障**语义。

## 生活化类比

把 CMB/PMR 想成**酒店给客人的两种"房卡权限"**：

- **普通房卡（CMB）** = 让客人能进特定楼层、用会议室、放行李（队列/列表/数据），但**出门后东西可能被人挪走**（不保证持久）。
- **保险柜钥匙（PMR）** = 在普通房卡权限之上**多一个保险柜**：贵重物品放进去后保证**断电也不丢**，还能看到保险柜"健康度"（OK / 只读 / 不可靠）。
- **楼层导航（CMBLOC）** = 告诉客人这间房从哪条走廊进去、楼梯起点在哪
- **走廊口大屏（CMBSZ）** = 写明"会议室可容纳多少人、行李能放多少"

客人要住一晚（高速读写）就拿普通房卡；要存重要文件就开保险柜；两种权限可以叠加但绝不能混淆。

## 工作流程

```text
              CMB 地址模型（PCI BAR）
   PCI BAR 由 CMBLOC.BIR 选中
   +-----------------------------------------------+
   | BAR base                                      |
   |   + CMBLOC.OFST * CMBSZ.SizeUnit              |
   |        +-------------- CMB -----------------+ |
   |        | 队列 / PRP-SGL List / 读-写数据    | |
   |        +------------------------------------+ |
   +-----------------------------------------------+

   可选"控制器地址视图":
     CMBMSC.CRE=1 -> CMBLOC/CMBSZ 有效
     CMBMSC.CMSE=1 + 有效 CBA -> 该地址范围的访问走 CMB
```

```text
            PMR 启用/就绪握手
   [PMR disabled / NRDY=1]
              |  PMRCTL.EN=1
              |  等待 PMRCAP.PMRTO * PMRTU
              v
   [PMR enabled, wait] ---- PMRSTS.CBAI/健康错误 ---> [诊断]
              |
              |  PMRSTS.NRDY=0
              v
   [PMR ready, 可被 PCIe 读/写]
              |
              |  持久化屏障 (memory read 完成 + PMRSTS read 完成)
              v
   [此前写入已持久]
```

## 初学者案例

**场景：把 I/O 提交队列放进 CMB**

1. 主机读 Identify Controller：`CAP.CMBS=1`（支持 CMB）。
2. 主机读 `CMBLOC` 与 `CMBSZ`：`BIR=2`（用 BAR2），`OFST=0`，`SizeUnit=1 MiB`，`SZ=64` → CMB 共 64 MiB。
3. 主机检查能力位：`CMBSZ.SQS=1`（支持 SQ）、`CMBSZ.CQS=1`（支持 CQ）、`CMBSZ.WDS=1`（支持写数据）。
4. 主机把 BAR2 映射到自己虚拟地址空间，按 `OFST` 算出 CMB 起始。
5. 主机创建 I/O SQ 时，把队列基地址指向 CMB 内的某块（同时设 `CC.IOSQES=...`）。
6. 控制器直接读写 CMB 内的 SQ/CQ，节省一次 PCIe TLP。
7. 重启后 CMB 内容**不一定保留**（依赖控制器实现），所以不要把关键数据放 CMB。

> 速查：CMB 写穿控制器内部弹性缓冲的"持续写吞吐"由 `CMBSTS` 报告；0 表示"未提供此信息"，不等于"没有 CMB"。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| CMB 是可选 | `CAP.CMBS=1` 才存在；不存在时 `CMBLOC/CMBSZ/CMBMSC/CMBSTS` 全部 0/保留 |
| BAR 4 KiB 对齐 | BAR 地址低 12 位为 0 |
| 实际大小 | `CMBSZ.SZ * SizeUnit`；但若 `OFST + 大小` 超过 BAR 范围，实际可用被截断 |
| 能力位推断 | 能力位清零 = 对应放置限制仍生效；**不可**仅凭 CMB 存在推断某能力被支持 |
| 控制器地址视图 | `CMBMSC.CRE=1` 才使 `CMBLOC/CMBSZ` 有效；`CMSE=1` + 有效 CBA 才把该地址范围映射到 CMB |
| `CMBSTS.CBAI=1` | 控制器地址空间启用失败，原因是配置的基地址无效 |
| PMR 持久化 | PMR 显式支持持久化 + 健康状态 + 写屏障；CMB 不承诺 |
| PMR 启用 | `PMRCTL.EN=1` 启动；超时 `PMRCAP.PMRTO * PMRTU`（500ms 或分钟单位） |
| PMR 就绪 | `PMRCTL.EN=1` 且 `PMRSTS.NRDY=0` 才可被 PCIe 访问 |
| PMR 写屏障 | 至少支持一种：memory read 完成 + PMRSTS read 完成 |
| PMR 控制器地址 | 上 32 位在 `PMRMSCU.CBA`，下 20 位在 `PMRMSCL.CBA`，低 12 位隐式为 0 |
| PMR 地址范围 | 不能溢出 64 位，不能与已启用 CMB 控制器地址范围重叠 |
| `PMRSTS.ERR` | 非零即"发生过写入错误"，**直到 PCI Function reset** 才清零 |
| 健康状态 4 类 | `000b` 正常；`001b` 持久但恢复可能错；`010b` 只读；`011b` 不可靠 |
| 0 值含义 | `value=0` 表示"控制器未提供该信息"，**不等于**该特性不存在 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| CMB vs PMR | CMB 注重低延迟访问；PMR 注重持久化 + 健康 + 屏障 |
| 主机地址 vs 控制器地址 | 主机用 BAR 映射访问；控制器地址视图由 `CRE/CMSE` 启用后，主机访问该地址范围会被路由到 CMB/PMR |
| `CAP.CMBS=1` vs `CMBSZ.SZ=0` | 前者表示支持 CMB 能力；后者表示 CMB 长度字段为 0（可能仍可用） |
| `PMRSTS.NRDY=1` vs PMR 未启用 | NRDY=1 表示"启用但未就绪"；未启用时 `PMRCTL.EN=0` |
| `PMRSTS.HSTS=001b` vs `ERR≠0` | HSTS 是健康状态（持久但可能错）；ERR 是写入错误历史 |
| `CMBSTS` 字段 0 vs 不支持 | 0 = 控制器未提供该信息；不可推断"无 CMB 弹性缓冲" |

## 进阶细节

- **CMB 能力维度对照**（Figures 47-48）：

  | 能力 | 字段 |
  |------|------|
  | 放 SQ | `CMBSZ.SQS` |
  | 放 CQ | `CMBSZ.CQS` |
  | 放 PRP/SGL List | `CMBSZ.LISTS` |
  | 主机→控制器数据 | `CMBSZ.WDS` |
  | 控制器→主机数据 | `CMBSZ.RDS` |
  | 不连续/混合内存队列 | `CMBLOC.CQPDS`, `CMBLOC.CQMMS` |
  | 数据/元数据/列表混合放置 | `CMBLOC.CDMMMS`, `CDPCILS`, `CDPMLS` |
  | 队列 dword 对齐 | `CMBLOC.CQDA` |

- **PMR 健康状态机**（`PMRSTS.HSTS`）：

  | 编码 | 含义 |
  |------|------|
  | `000b` | 正常 |
  | `001b` | 持久且工作，但恢复可能不正确 |
  | `010b` | 只读 |
  | `011b` | 不可靠；数据或持久化可能无效 |

- **PMR 控制器地址属性**（`PMRMSCU` / `PMRMSCL`）：
  - `PMRMSCU.CBA`：CBA 上 32 位。
  - `PMRMSCL.CBA`：CBA 接下来 20 位。
  - 低 12 位隐式为 0。
  - `PMRMSCL.CMSE` 启用要求：地址范围不溢出 64 位，且不与启用的 CMB 控制器地址范围重叠。
- **CMB vs PMR 对比（速查）**：

  | 维度 | CMB | PMR |
  |------|-----|-----|
  | 目的 | 低延迟控制器内存 | 持久化控制器内存 |
  | 持久化 | 不保证 | 显式屏障 + 健康 |
  | 启用/状态 | `CRE`/`CMSE` + `CBAI` | `EN`/`NRDY` + `CBAI`/`HSTS`/`ERR` |
  | 放置 | 按能力位放队列/列表/数据 | 按能力放读/写数据 |

## 规范依据

- [CMB 位置与能力字段（CMBLOC/CMBSZ，Figures 47-48），PDF 第 86 页](../_source/pages/page-086.md)
- [CMB 控制器地址空间与状态（CMBMSC/CMBSTS），PDF 第 89 页](../_source/pages/page-089.md)
- [CMB 吞吐与子系统关机（CMBSTS 字段），PDF 第 90 页](../_source/pages/page-090.md)
- [PMR 能力/控制/状态寄存器（PMRCAP/PMRCTL/PMRSTS），PDF 第 92 页](../_source/pages/page-092.md)
- [PMR 健康状态与错误历史，PDF 第 94 页](../_source/pages/page-094.md)

## 相关阅读

- [host-memory-buffer.md](host-memory-buffer.md) - 主机侧对偶机制，反向借用 DRAM
- [controller-properties.md](controller-properties.md) - CMB/PMR 寄存器在属性空间
- [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) - 启用与复位会改变 CMB/PMR 状态
- [power-state-descriptors.md](power-state-descriptors.md) - 低功耗下 CMB 数据的保留规则
