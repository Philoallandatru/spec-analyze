# 主机内存缓冲区（Host Memory Buffer, HMB）

**主机内存缓冲区（Host Memory Buffer, HMB）**是一段由主机分配、专供 NVMe 控制器独占使用的主机物理内存区域（常见于无板载 DRAM 的 **DRAM-less SSD**，用于存放 FTL 映射表或缓存）。HMB 通过 **Set Features（FID `0Dh`）** 启停，并以 Set Features 的 **CQE（完成队列项）完成**作为内存所有权在主机与控制器之间一次性移交的分界线。[PDF pp. 429-431](../_source/pages/page-429.md)

可以把 HMB 想象成**租给房客的房间**：`Set Features EHM=1` 完成就像交出钥匙，租期内控制器独占使用，房东（主机）不得再写入或修改；`Set Features EHM=0` 完成就像房客搬走退房，控制器保证之后不再访问，主机方可安全修改或回收内存。

## 所有权流转与启停规则

```text
主机分配内存区域 + 物理连续描述符列表
        |
        | Set Features (FID 0Dh, EHM=1)
        v
[控制器独占 HMB；主机禁止写入描述符列表及关联缓冲]
        |
        | Set Features (FID 0Dh, EHM=0) 完成 (CQE 返回)
        v
[控制器停止一切访问；主机可安全修改或回收内存]
```

| 操作 / 字段 | 规范契约 |
|---|---|
| **重复启用（`EHM=1`）** | 对已启用的 HMB 再次设 `EHM=1` 会返回 `Command Sequence Error` |
| **重复禁用（`EHM=0`）** | 对已禁用的 HMB 再次设 `EHM=0` 命令成功但无实际动作；`EHM=0` 时控制器忽略 CDW12–CDW15 |
| **禁用同步边界** | 控制器在提交禁用 CQE 前**应当**取回必要数据；提交禁用 CQE 之后**不得**再访问 HMB，直到再次启用为止 |
| **Memory Return（`MR=1`）** | 表示"还回"重置或 Runtime D3 之前的那份内存：总大小（`HSIZE`）、描述符列表地址（`HMDLLA/HMDLUA`）、条目数（`HMDLEC`）、描述符内容及缓冲内容必须与控制器上次看到的完全一致 |
| **新内存（`MR=0`）** | 表示提供的是新分配的内存，缓冲内容未定义；`CTZ`（CDW11 bit 3）可用于请求清零 |

> **排错提示**：禁用 HMB 时，主机必须等 `Set Features (EHM=0)` 的 CQE 返回后才能释放或重用内存，切勿提前覆盖。[PDF pp. 429-431](../_source/pages/page-429.md)

## 非操作态电源访问限制

若 Identify Controller 的 `CTRATT.HMBR=1`（支持 HMB 非操作态访问限制），主机可通过 Set Features CDW11 的 `HMNARE`（bit 2）在进入非操作态电源状态时限制控制器访问 HMB（否则置 `HMNARE=1` 会报错 `Invalid Field in Command`）：[PDF p. 430](../_source/pages/page-430.md)

- **主机主动转入非操作态**：立即禁止控制器访问 HMB（Admin 命令工作及 Admin 启动的后台操作除外）。该限制独立于 `NOPPME`（非操作态电源许可模式），也不改变对外报告的进入延迟（`ENLAT`）。
- **自主转换（Autonomous Transition）**：仅要求控制器把对 HMB 的访问"最小化"，并不立即完全禁止。

## 描述符契约与 Get Features 观测

Set Features（`EHM=1`）通过 CDW12–CDW15 传入 HMB 参数：

- **CDW12（`HSIZE`）**：以控制器内存页（`CC.MPS`）为单位的 HMB 总大小。
- **CDW13–CDW14（`HMDLLA` / `HMDLUA`）**：64 位描述符列表物理地址，必须 **16 字节对齐**（低 4 位为 0），且列表驻留在物理连续的主机内存中。
- **CDW15（`HMDLEC`）**：描述符条目数，**必须非 0**（否则返回 `Invalid Field in Command`）。

```text
物理连续描述符列表 (Entry 0 .. Entry HMDLEC-1)
  每条描述符固定 16 字节 (Figure 453)：
  +-------------+-------------------+-------------------------+
  | Bits 127:96 | Bits 95:64        | Bits 63:00              |
  +-------------+-------------------+-------------------------+
  | Reserved    | BSIZE (CC.MPS 页) | BADD (按 CC.MPS 页对齐) |
  +-------------+-------------------+-------------------------+
  注：若 BSIZE=0，控制器忽略该条描述符。
```

**Get Features（FID `0Dh`）**在 CQE Dword 0 中返回当前 `HMNARE`（bit 2）与 `HMNAR`（bit 3，当前是否正在限制访问），并在 4 KiB 数据缓冲中返回当前大小、描述符列表地址与有效条目数；这些均为**只读观测值**，不构成新的所有权移交。[PDF pp. 431-432](../_source/pages/page-431.md)

## 规范依据

- [HMB 所有权与启停规则（Figures 445–447），PDF p. 429](../_source/pages/page-429.md)
- [非操作态限制与 Memory Return 语义，PDF p. 430](../_source/pages/page-430.md)
- [HSIZE / HMDLLA / HMDLUA / HMDLEC 与描述符布局（Figures 448–453），PDF pp. 431-432](../_source/pages/page-431.md)
- [Get Features 属性与 CQE DW0（Figure 454），PDF p. 432](../_source/pages/page-432.md)

## 关联与引申概念

- **DRAM-less SSD**：省略板载 DRAM 缓存的低成本/低功耗 NVMe SSD，依赖 HMB 借用主机内存加速地址映射与小块缓存。
- **Identify 命令模型**：Identify Controller 通过 `HMPRE`（首选大小）、`HMMIN`（最小大小，0 表示任意可行）、最大描述符条目数以及 `CTRATT.HMBR` 通告 HMB 能力；注意 `HMPRE=0` 表示无 HMB 能力。详见 [识别命令模型](identify-command-model.md)。
- **电源管理与通用特性**：主机主动与自主电源状态转换决定了 `HMNARE` 何时生效。详见 [通用控制器特性](common-controller-features.md) 与 [特性值与作用域](feature-values-and-scope.md)。
- **数据指针布局（SGL）**：HMB 描述符（16 字节固定页粒度）与 SGL 描述符结构相似但用途不同（SGL 用于单条命令数据传输，支持任意字节对齐）。详见 [数据指针布局](data-pointer-layouts.md)。
