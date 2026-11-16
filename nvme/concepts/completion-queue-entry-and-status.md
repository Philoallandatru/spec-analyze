# 完成队列条目与状态（Completion Queue Entry and Status）

完成队列条目（CQE, Completion Queue Entry）是控制器向主机返回命令执行结果的 16 字节数据结构，用来关联已提交的命令、同步提交队列消费进度（`SQHD`），并通过分层状态码精确报告成功、失败原因与重试建议。[PDF pp. 160-161](../_source/pages/page-160.md)

可以把它想成**快递签收回执**：
- **DW0 / DW1**：业务备注（因命令而异的返回值）
- **DW2（`SQID` + `SQHD`）**：来自哪个网点、该网点已发出多少单（释放 SQ 槽位）
- **DW3（`CID` + `P` + `Status`）**：运单号、新旧水印（Phase Tag）以及签收成败状态

## 通用布局（16 字节）

```text
  管理与 I/O 通用 CQE（基于内存，Figure 96）：
  31                          16 15                          0
  +-----------------------------+----------------------------+
  |             DW0（命令特定） |             DW1（命令特定） |
  +-----------------------------+----------------------------+
  |  SQID (16 bit)              |  SQHD (16 bit)             |  ← DW2
  +-----------------------------+----------------------------+
  |  Status (15 bit)            |P(1)|  CID (16 bit)         |  ← DW3
  +-----------------------------+----------------------------+
   31                        17  16  15                     0
```

[PDF pp. 160-161](../_source/pages/page-160.md)

| 区域 | 基于内存的 CQE（PCIe） | Fabrics CQE（Figure 99） |
|---|---|---|
| **字节 0:7** | 命令特定的 DW0-DW1 | Fabrics 响应类型特定内容（FRTS） |
| **字节 8:11** | `SQID` + `SQHD`（消费快照） | `SQHD`（流控禁用时保留）+ 保留 |
| **字节 12:15** | `CID` + `Phase Tag (P)` + `Status` | `CID` + `Status Info`（`STATUS` 占 bit 15:1，bit 0 保留；无 `SQID`/`P`） |

- `SQID` 与 `CID` 联合唯一标识一条未完成命令（同 SQ 内最多同时 65535 条）。
- `SQHD` 是创建 CQE 时的快照，控制器可能在主机读取前已消费更多 SQ 条目，主机可借此一次性释放多个 SQ 槽位。[PDF pp. 160-161](../_source/pages/page-160.md)

## 状态字段与状态码空间

```text
DW3 位域：
  31    30    29:28        27:25          24:17
+-----+-----+------------+-------------+-----------+
| DNR |  M  |    CRD     |     SCT     |    SC     |
+-----+-----+------------+-------------+-----------+
```

[PDF pp. 161-163](../_source/pages/page-161.md)

| 字段 | 位宽 | 含义与主机处理规则 |
|---|---|---|
| **`DNR`** | 1 bit | **Do Not Retry**：`1` 表示重提交到子系统**任何控制器**预期仍会失败；`0` 表示重试**可能**成功（非保证） |
| **`M`** | 1 bit | **More**：`1` 表示 Error Information Log（Log ID `01h`）中有额外错误信息（日志可能被后续错误覆盖） |
| **`CRD`** | 2 bit | **Command Retry Delay**：仅在 `DNR=0` 且 Host Behavior Support `ACRE=1h` 时有效；`00b`=立即重试，`01b/10b/11b`=等待 `CRDT1/2/3`（单位 100ms）。**建议性字段**，提前重试不报错 |
| **`SCT`** | 3 bit | **Status Code Type**：`0h`=Generic，`1h`=Command Specific，`2h`=Media/Data Integrity，`3h`=Path Related，`4h-6h`=保留，`7h`=Vendor Specific |
| **`SC`** | 8 bit | **Status Code**：具体条件码。每族内 `00h-7Fh`=跨命令集通用，`80h-BFh`=I/O 命令集特定，`C0h-FFh`=Vendor Specific |

- **成功判定**：`SCT=0h` 且 `SC=00h`（此时 `DNR`/`M`/`CRD` 均为 0）。任何非零 `SCT`/`SC` 均视为非成功完成。[PDF pp. 161-163](../_source/pages/page-161.md)
- **多错误并发**：同时满足多个错误条件且未规定优先级时，控制器自行选择返回其中一个适用状态。

### 四大常用状态族速查

| 状态族 (`SCT`) | 覆盖范围 | 典型状态与主机动作 |
|---|---|---|
| **Generic (`0h`)** | 协议级通用错误 | 无效操作码/字段、`CID` 冲突、SGL/PRP 错误、Abort/Cancel 中止（`07h`）、SQ 删除中止（`08h`）；命令中断/格式化中需设 `DNR=0` [PDF pp. 163-166](../_source/pages/page-163.md) |
| **Command Specific (`1h`)** | 特定命令约定/门控 | Admin/I/O 生命周期门控拒绝；Fabrics Connect/Auth 失败（`B0h..BFh` 保留给传输绑定） [PDF pp. 166-169](../_source/pages/page-166.md) |
| **Media & Data Integrity (`2h`)** | 介质与端到端保护 | 读/写介质失败、CRC/Guard/AppTag/RefTag 校验失败（通常不建议原样重试） |
| **Path Related (`3h`)** | 主机-控制器路径状态 | 内部路径错误、ANA 状态转换（转换后重试）、持久丢失（换其他控制器/路径，勿在同路径重试） [PDF pp. 169-171](../_source/pages/page-169.md) |

**排错顺序示例**：收到 `Status=0x000E0F02`（拆位：`DNR=0, M=0, CRD=0, SCT=3h, SC=02h`）→ 先看 `SCT=3h` 知是路径错误 → 再看 `DNR=0` 知可重试 → 查其他 ANA 路径或短暂等待后重试。若 `M=1`，则用 `nvme get-log` 拉取 Log Page `01h`。

## Phase Tag（阶段标签）生命周期

在基于内存的环形完成队列中，主机通过比较槽位中的 `P` 位与**当前期望的 Phase 值**来识别控制器新发布的 CQE，无需在消费后清零槽位：[PDF pp. 160, 171-173](../_source/pages/page-171.md)

```text
主机初始化 CQ 所有槽位的 P = 0（Admin CQ 在 CC.EN→1 前；I/O CQ 在 Create CQ 前）
        |
        v
控制器首次从槽位 0 写入 CQE → 设置 P = 1（主机期望值 = 1，识别为新条目）
        |
        v
控制器推进 CQ Tail 指针；主机消费并推进 CQ Head 门铃（不重写 P 位）
        |
        v
CQ Tail 回绕到槽位 0 → 控制器发布 P 翻转为 0（主机期望值同步翻转为 0）
```

- **多写原子性**：若控制器分多次写入一个 CQE 的各字段，**必须在最后一次写入时才更新 `P` 位**，防止主机读到写了一半的 CQE。[PDF pp. 171-173](../_source/pages/page-171.md)

## 概念辨析

| 容易混淆 | 实际区别 |
|---|---|
| `SQHD` vs 头门铃 | `SQHD` 是控制器写入 CQE 的“SQ 已消费位置”快照；头门铃是主机写给控制器的“CQ 已读位置”通知 |
| `DNR=0` vs 保证成功 | `DNR=0` 仅表示“重试可能成功”；`CRD` 提供建议等待时间，提前重试不报错 |
| `P` 位翻转时机 | 同一轮队列中连续多个 CQE 的 `P` 位相同，仅在**队列尾指针回绕**时翻转一次 |

## 关联与引申概念

- **通用命令格式（[Common Command Format](common-command-format.md)）**：SQE（64 字节）与 CQE（16 字节）通过 `CID` 与 `SQID` 精确配对。
- **命令排序与流控（[Command Ordering and Arbitration](command-ordering-and-arbitration.md)）**：利用 CQE 中的 `SQHD` 回收 SQ 槽位，实现主机-控制器之间的队列流控。
- **通信丢失与命令重试（[Communication Loss and Command Retry](communication-loss-and-command-retry.md)）**：通信中断时结合停止屏障与 CQE 的 `DNR`/`CRD` 制定安全重试策略。
- **异步事件上报（[Asynchronous Event Reporting](asynchronous-event-reporting.md)）**：控制器通过完成 Admin CQ 上挂起的 AER 命令（返回特定 CQE DW0）主动通知主机异常事件。
- **ANA（Asymmetric Namespace Access）**：多路径访问状态机制，Path Related 状态码（`SCT=3h`）常与 ANA 路径切换联动。

## 规范依据

- [通用 CQE 布局（Figures 96-98），PDF 第 160-161 页](../_source/pages/page-160.md)
- [Fabrics CQE 布局（Figure 99），PDF 第 161 页](../_source/pages/page-161.md)
- [Status 字段位定义与状态族（Figures 100-101），PDF 第 162-163 页](../_source/pages/page-162.md)
- [Generic 状态码（Figure 102），PDF 第 163-166 页](../_source/pages/page-163.md)
- [Command Specific 状态分支（Figures 103-105），PDF 第 166-169 页](../_source/pages/page-166.md)
- [Media 与 Path Related 状态（Figures 106-107），PDF 第 169-171 页](../_source/pages/page-169.md)
- [Phase Tag 生命周期（Figure 108），PDF 第 171-173 页](../_source/pages/page-171.md)
