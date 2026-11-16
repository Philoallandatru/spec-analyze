# Admin 命令模型

Admin 命令集（Admin Command Set）定义了提交到 **Admin 提交队列（Admin Submission Queue, Admin SQ）** 的全部管理命令。它把"所有传输通用"、"仅 Memory-based（如 PCIe）"与"仅 Message-based（如 Fabrics）"三组命令合并在一起，套用统一的 SQE/CQE 信封，再通过各自专属的 dword 字段区分具体操作。[PDF pp. 191-192](../_source/pages/page-191.md)

> **类比理解**：把 Admin SQ 想成酒店前台——通用命令（Identify、Get Log Page、Set Features 等）是常驻服务台，Memory-based 命令是本地柜台，Message-based 命令是远程柜员；所有请求都装进统一的 SQE 信封，由 `opcode` 决定交给谁、`DTD` 决定数据往哪边走。

---

## 命令分类与调度模型

```text
                    Admin Submission Queue
                    +-------------------------+
                    |  SQE 共用部分           |  (opcode, NSID, 数据指针)
                    |  CDW10..CDW15 命令专属  |  (由各命令定义)
                    +-----------+-------------+
                                |
                                v
                       +--------+--------+
                       |  opcode 选命令   |
                       |  DTD 选数据方向  |
                       +--------+--------+
                                |
        +-----------------------+-----------------------+
        |                       |                       |
        v                       v                       v
  5.1 通用命令            5.2 Memory-based        5.3 Message-based
  (所有传输通用)          (仅 PCIe 等)            (仅 Fabrics)
  Identify                Create/Delete I/O Queue Discovery
  Get/Set Features        Virtualization Mgmt     Fabric Zoning
  Get Log Page / FW       Doorbell Buffer Config  Exported NVM Resource
  Format / Sanitize
  Fabrics (opcode 7Fh)
```

- **统一信封与命令专属区**：所有 Admin 命令复用通用 SQE/CQE 结构（`opcode`、`NSID`、数据指针、`CID` 等），Admin 章节负责赋予 `CDW10..CDW15` 具体含义。[PDF pp. 155-159, 191](../_source/pages/page-155.md)
- **队列独立性**：Admin 命令的处理必须独立于 I/O 队列的拥塞状况。例如，即使 I/O 完成队列（CQ）已满，也不应阻塞 `Delete I/O Submission Queue` 等管理命令。[PDF p. 191](../_source/pages/page-191.md)
- **与支持矩阵配合**：某 `opcode` 在表中已分配，不代表所有控制器都必须支持；控制器类型是否支持该命令，还需结合独立的命令支持矩阵判定。[PDF pp. 65-67, 191](../_source/pages/page-065.md)

---

## Opcode 与选择子契约

| 维度 | 规范契约 |
|---|---|
| **Opcode 构成** | 8 位组合值 = `Function[5:0]`（功能码）+ `Data Transfer Direction[1:0]`（DTD，数据传输方向） |
| **Opcode 分段** | `00h..3Fh` 标准通用管理命令；`40h..7Eh` 保留；`7Fh` 专属 Fabrics；`80h..BFh` I/O 命令集专属或保留（如 `80h` Format NVM、`84h` Sanitize）；`C0h..FFh` 厂商自定义 |
| **未列即保留** | Figure 141 未列出的 `opcode` 一律保留，提交会被控制器以 `Invalid Opcode` 拒收 |
| **NSID 处理** | 不使用 NSID 的命令必须将其清零；使用 NSID 的命令一般支持 `FFFFFFFFh`（广播），除非脚注另有收窄 |
| **Fabrics 命令** | 所有 Fabrics 命令共用 `opcode = 7Fh`，具体操作由 Fabrics 命令格式内部字段区分 |
| **导出资源命令** | Clear/Create/Manage Exported NVM Subsystem、Manage Exported Namespace/Port 等命令**禁止**出现在仅使用 Memory-based 传输的子系统控制器上 |

---

## Format 与 Sanitize 期间的命令闸门

当命名空间正在执行 **Format NVM** 或控制器正在执行 **Sanitize** 时，并非所有 Admin 命令都能照常运行，规范通过允许表（allowlist）收窄可执行集合（Figure 142）：[PDF pp. 192-193](../_source/pages/page-192.md)

```text
[普通 Admin 命令]
        |
        +-- 命名空间处于 Format 中？—— 不在允许表中 --> 可能中止：Format In Progress
        |
        `-- Sanitize 进行中？—— 不在允许表中 --> 禁止 / 命令专属闸门

[Format NVM 请求]
        `-- 受影响命名空间上仍有非允许表 Admin 操作 --> 可能中止：Command Sequence Error
```

- **始终允许的核心控制/恢复类命令**：`Abort`、`AER`、`Create/Delete I/O Queue`、`Identify`、`Keep Alive`、`Get Features`、多数 `Set Features`、特定 `Get Log Page`、部分 `Fabrics` 命令。
- **厂商命令**：仅在"既不影响也不读取用户数据"时被允许。
- **各自额外限制**：
  - **Format 期间**：`Set Features` 中的 `Namespace Write Protection (NWP)` 被禁止；中止判定仅针对被该 Format 影响的命名空间，其他命名空间不受牵连。
  - **Sanitize 期间**：`Vendor Specific` 日志与 `Persistent Event Log` 被禁止；`Sanitize` 命令本身在 `SANACT != 101b` 时被禁止。

---

## 常见排错与易混点

| 易混/排错点 | 实际区别与排查方向 |
|---|---|
| **自定义命令报 `Invalid Opcode`** | 厂商自定义命令必须使用 `C0h..FFh`（`11xx_xxxxb`），落在未列出的保留区间会被直接拒收；`DTD` 方向填错也会导致 `opcode` 不匹配 |
| **opcode vs Function** | `opcode` 是 8 位完整组合值，`Function` 只是其中高 6 位，同一功能结合不同 `DTD` 可对应不同 `opcode` |
| **DTD vs NSID** | `DTD` 决定数据流向（无数据 / Host→Ctrl / Ctrl→Host / 双向），`NSID` 决定作用哪个命名空间，互不相干 |
| **厂商区 vs 保留区** | `C0h..FFh` 是允许厂商自定义的空间；其余未在 Figure 141 列出的 `opcode` 一律禁止使用 |
| **Abort vs AER** | `Abort` 是同步管理命令，提交即尝试中止目标命令；`AER` 是长寿命占位命令，挂在队列中等待事件发生才返回 |

---

## 关联与引申概念

- **Admin Queue（管理队列）**：控制器上编号固定为 0 的队列对，专门承载 Admin 命令与完成响应，独立于 I/O 队列运行。
- **SQE / CQE（提交/完成队列条目）**：主控交换命令（64 字节 SQE）与结果（16 字节 CQE）的标准信封，详见 [common-command-format.md](common-command-format.md)。
- **NSID（命名空间标识符）**：用于指定命令作用的命名空间（`0` 表示不使用或控制器级，`FFFFFFFFh` 表示广播）。
- **Fabrics（NVMe-oF）**：基于消息（Message-based）的远程传输模型，所有 Fabrics 管理操作复用 `opcode = 7Fh`。
- **Format NVM / Sanitize**：格式化与安全擦除命令，执行期间会触发 Figure 142 允许表闸门，限制并发的 Admin 命令。
- **延伸阅读**：
  - [command-effects-and-support.md](command-effects-and-support.md) — 命令支持矩阵与效果日志（CSUPP / CSE）
  - [command-and-feature-lockdown.md](command-and-feature-lockdown.md) — 运行时命令与 Feature 动态锁定
  - [asynchronous-event-reporting.md](asynchronous-event-reporting.md) — AER 异步事件上报机制
  - [command-abort-semantics.md](command-abort-semantics.md) — Abort 命令中止语义

---

## 规范依据

- [Figure 141：Admin opcode 矩阵与脚注，PDF pp. 191-192](../_source/pages/page-191.md)
- [Figure 142：Format/Sanitize 期间 Admin 允许表，PDF pp. 192-193](../_source/pages/page-192.md)
- [通用 SQE 结构与字段定义，PDF pp. 155-159](../_source/pages/page-155.md)
- [控制器类型与命令支持矩阵，PDF pp. 65-67](../_source/pages/page-065.md)
