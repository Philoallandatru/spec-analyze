# 主机内存缓冲区（Host Memory Buffer）

## 一句话说明

HMB 让主机划一块"独占内存"给控制器当 scratch 空间；启用后**所有权一次性移交控制器**（主机禁止写入），禁用后**控制器完成 Set Features 时一次性还回**（主机可改可回收）。

## 生活化类比

把 HMB 想成**租给租客的房间**：

- **Set Features EHM=1** = "钥匙交给你（控制器），租期内你独占使用；房东（主机）不许再开门进来摆东西"。
- **Set Features EHM=0 完成** = "租约结束，租客搬走（控制器停止访问），房东可以进去打扫、改造、重新出租"。
- **描述符列表** = 房间清单（地址 + 大小），按 16 字节一页登记。
- **HM613MR=1** = "我按上次你住进去时的样子还给你"（房间内容 + 清单原封不动）；`MR=0` = "我提供新房间，家具随便放"。
- **非操作态访问限制** = "租客出差/睡觉时"，房东可以锁门不准进。

## 工作流程

```text
  Set Features (FID 0Dh)：
    CDW11: bit 0 = EHM  启用位
           bit 1 = MR   Memory Return
           bit 2 = HMNARE  非操作态访问限制启用
           bit 3 = CTZ   清零
    CDW12: HSIZE   HMB 总大小（CC.MPS 页单位）
    CDW13: HMDLLA  描述符列表地址低 32 位（16 字节对齐）
    CDW14: HMDLUA  描述符列表地址高 32 位
    CDW15: HMDLEC  描述符条目数（非 0）

  描述符列表（Figure 452，物理连续）：
    +----------+----------+----+
    |  15:0    | ...      | 末 |
    +----------+----------+----+
    | Entry 0  | Entry 1  |... |

  每个描述符（16 字节，Figure 453）：
    +----------+----------+--------+
    | 127:96   | 95:64    | 63:0   |
    +----------+----------+--------+
    | Reserved | BSIZE    | BADD   |
    |          | (页单位) | 页对齐 |
    +----------+----------+--------+
    BSIZE=0 → 该描述符被忽略

  Get Features 4 KiB 缓冲：返回当前 size / 描述符地址 / 有效条目数
  CQE DW0：bit 2 HMNARE，bit 3 HMNAR
```

## 初学者案例

**场景：驱动设了 HMB，但某些主机日志报"控制器还在用旧地址"。**

1. 你的驱动 `nvme set-feature -f 0x0d -v EHM=0` 禁用 HMB。
2. 立刻又把同一块内存 mmap 给另一个进程使用。
3. 控制器日志里"在我提交 Disable 的 CQE 之前，我还在读这块内存"——这是规范允许的（"should retrieve any necessary data before posting the completion"），但**禁用 CQE 提交后**控制器保证**不再访问**。
4. 严格做法：禁用 Set Features 的 CQE 返回后，再等待一个"小延迟"或轮询控制器不再访问。
5. 另一种情况是 `MR=1` 的"还回"路径：必须用**和之前**完全一样的 size / 描述符地址 / 描述符内容 / 缓冲内容；否则控制器视为不一致。
6. 排错：检查 `HSIZE`、`HMDLLA/HMDLUA`、`HMDLEC`、每条描述符 `BADD`/`BSIZE` 是否与上次匹配。

> 排错提示：禁用 HMB 之后**不要立刻**释放或重用内存；至少等 Set Features 的 CQE 回来再动。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| Feature ID | `0Dh` Host Memory Buffer |
| 启用位 | CDW11 bit 0 `EHM`（`1`=启用，`0`=禁用） |
| 重复启用 | 已启用时再 Set EHM=1 → `Command Sequence Error` |
| 重复禁用 | 未启用时 Set EHM=0 成功但无动作 |
| 禁用时 CQE 后 | 控制器保证**不再访问** HMB；CQE 提交前"应"取回必要数据 |
| 内存返回 MR | CDW11 bit 1；`MR=1` 表示还回，必须与上次完全一致；`MR=0` 内容未定义 |
| 非操作态限制 HMNARE | CDW11 bit 2；需 `CTRATT.HMBR=1` 才支持；否则 `Invalid Field in Command` |
| 描述符列表地址 | 64-bit，16 字节对齐（bit 3:0 = 0） |
| 描述符总数 | `HMDLEC` 必须非 0；`0` → `Invalid Field in Command` |
| 描述符条目 | 16 字节；`BADD` 64-bit 页对齐；`BSIZE` 32-bit CC.MPS 页单位；`BSIZE=0` 忽略该条 |
| 描述符列表位置 | 物理连续，驻留在主机内存 |
| 描述符列表修改 | HMB 启用期间，**禁止**主机写描述符列表和关联内存 |
| 启用后 CQE 含义 | "CQE 完成 = 所有权已移交" |
| 禁用完成含义 | "CQE 完成 = 控制器不再访问，可安全修改/回收" |
| Get 4 KiB 缓冲 | 返回当前 size / 描述符地址 / 有效条目数（观察值，不触发移交） |
| CQE DW0 | bit 2 = HMNARE，bit 3 = HMNAR |
| 非操作态访问 | HMNARE 启用后：仅 Admin 命令和 Admin 启动的后台操作可访问 HMB |
| 与 NOPPME 关系 | HMNARE 启用/禁用**不影响** NOPPME；HMNARE 不改变 ENLAT 报告值 |
| 自主转非操作态 | HMNARE 启用时，自主转换"不立即禁止" HMB 访问，应"最小化" |
| 主机转非操作态 | HMNARE 启用时，主机转非操作态"立即禁止" HMB 访问（除 Admin） |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| EHM=1 vs EHM=0 | 1=启用（控制器独占）；0=禁用（CQE 后可回收） |
| MR=1 vs MR=0 | 1=按上次状态"还回"；0=新分配（内容未定义） |
| HMNARE 启用 vs 未启用 | 启用后非操作态 HMB 不可访问；未启用则可访问 |
| 自主转 vs 主机转非操作态 | 自主转"应最小化访问"；主机转"立即禁止访问"（除 Admin） |
| HMB 描述符 vs SGL 描述符 | HMB 描述符 16 字节固定；SGL 描述符 16 字节但有多种 type/subtype |
| BADD vs SGL Data Block 地址 | BADD 64-bit 页对齐 + BSIZE 页单位；SGL Data Block 任意对齐 + 字节长度 |
| HMB 启用期间 vs 禁用期间 | 启用：主机禁写描述符 + 缓冲；禁用：主机可写可改可回收 |
| HMB 描述符列表 vs Identify 报告能力 | 列表是主机当前声明；Identify 报告 preferred/min/descriptor limits |
| Get Features 4 KiB 缓冲 vs SQE DPTR | 4 KiB 是 Get 的"返回缓冲"；SQE DPTR 是 Set 的输入 |

## 进阶细节

- **Figure 447 CDW11**：
  - `31:04` 保留
  - `03` CTZ（清零）
  - `02` HMNARE（需 `CTRATT.HMBR=1` 才支持；`HMBR=0` 时设 1 报 `Invalid Field in Command`）
  - `01` MR（Memory Return）
  - `00` EHM（Enable Host Memory）
  - `EHM=0` 时控制器**忽略** CDW12-15
- **Figure 448 CDW12**：`31:00` HSIZE（HMB 总大小，以 `CC.MPS` 页为单位）
- **Figure 449 CDW13**：`31:00` HMDLLA（描述符列表地址低 32 位）
- **Figure 450 CDW14**：`31:00` HMDLUA（描述符列表地址高 32 位）
- **Figure 451 CDW15**：`31:00` HMDLEC（条目数，**必须非 0**）
- **Figure 452 描述符列表**：物理连续；每条 16 字节；总条数 = HMDLEC。
- **Figure 453 描述符条目**：
  - `127:96` 保留
  - `95:64` BSIZE（CC.MPS 页单位；`0`=忽略该条）
  - `63:00` BADD（主机物理地址，按 `CC.MPS` 对齐；低 `n:0`=0）
- **Figure 454 CQE DW0**：
  - `31:04` 保留
  - `03` HMNAR（"当前是否正在限制 HMB 访问"）
  - `02` HMNARE（"限制是否启用"）
  - `01:00` 保留
- **HMNARE 限制范围**：非操作态期间除 Admin 命令和 Admin 启动的后台操作外，控制器**不得**访问 HMB。
- **HMNARE 与 NOPPME**（规范 5.1.25.1.10）：HMNARE 启用/禁用**不影响** Non-Operational Power State Permissive Mode；HMNARE 启用**不改变** Identify 报告的 ENLAT，但控制器可"超出 ENLAT 取回必要数据"。
- **MR 严格一致**：HSIZE、HMDLLA/HMDLUA、HMDLEC、各条 BADD/BSIZE、缓冲内容都要与"控制器可见的上次状态"完全相同。
- **CTRATT.HMBR**：HMB 非操作态限制能力位；`HMBR=0` 表示控制器不支持此功能，HMNARE 写 1 必报错。
- **Get 4 KiB 缓冲**：当前 size、描述符地址、有效条目数（观察值，**不是**新的所有权移交）。
- **重复禁用**：未启用时 Set `EHM=0` 成功且无动作；CQE 仍正常返回。
- **重复启用**：已启用时 Set `EHM=1` 报 `Command Sequence Error`。
- **同步要求**：主机必须等 Set Features CQE 回来才能修改/回收 HMB；之前 CQE 没回来前控制器可能仍在读。
- **能力报告**：Identify Controller 报告 preferred size / minimum size / max descriptor entries / HMBR 能力位。

## 规范依据

- [HMB 所有权与启停规则 Figures 445-447，PDF 第 429 页](../_source/pages/page-429.md)
- [非操作态限制与 Memory Return 语义，PDF 第 430 页](../_source/pages/page-430.md)
- [HSIZE / HMDLLA / HMDLUA / HMDLEC Figures 448-451，PDF 第 431 页](../_source/pages/page-431.md)
- [描述符列表 Figure 452 + 描述符条目 Figure 453，PDF 第 431-432 页](../_source/pages/page-431.md)
- [Get Features CQE DW0 Figure 454 + 4 KiB 属性缓冲，PDF 第 432 页](../_source/pages/page-432.md)
- [CTRATT HMBR 能力位与 Identify 字段，PDF 第 333-337 页](../_source/pages/page-333.md)
- [Host-requested / Autonomous 电源转换与 HMB 关系，PDF 第 400-405 页](../_source/pages/page-400.md)

## 相关阅读

- [特性值与作用域](feature-values-and-scope.md) - FID 0Dh 的 Set Features 机制
- [通用控制器特性](common-controller-features.md) - HMB FID 行为详解
- [数据指针布局](data-pointer-layouts.md) - 描述符结构与 SGL 类似
- [控制器内存窗口](controller-memory-windows.md) - 另一种主机内存区机制
- [主机元数据与管理地址](host-metadata-and-management-addresses.md) - 同属主机让渡资源
