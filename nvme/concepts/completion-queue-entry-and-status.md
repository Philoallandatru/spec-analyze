# 完成队列条目与状态（Completion Queue Entry and Status）

## 一句话说明

完成队列条目（CQE）把"哪条命令完成、结果如何"打包成一个 16 字节的结构返回给主机，状态字段再用 3 层（`SCT`+`SC`+`DNR/M/CRD`）精确表达成败原因与重试建议。

## 生活化类比

把 CQE 想成**快递的"签收回执"**：

- **DW0 / DW1** = 快递公司内部备注（"签收人姓名"等），格式因业务而异。
- **DW2.SQID + SQHD** = "这单来自哪个网点 + 该网点已经发完几单"（用于主机判断哪些 SQ 槽位可重发）。
- **DW3.CID** = 运单号尾号。
- **DW3.Phase Tag（P）** = 回执单右下角的"对号"水印，每次新单翻转一次，主机一眼能看出"这张是新的还是旧的"。
- **DW3.Status（DNR/M/CRD/SCT/SC）** = "签收状态"细分：成功 / 失败类型 / 是否可重试 / 建议等待时间。

## 工作流程

```text
  通用 CQE 布局（管理 + I/O，Figure 96）— 至少 16 字节

  31                          16 15                          0
  +-----------------------------+----------------------------+
  |             DW0             |             DW1            |  ← 命令特定
  +-----------------------------+----------------------------+
  |  SQID (16 bit)              |  SQHD (16 bit)             |  ← DW2
  +-----------------------------+----------------------------+
  |  Status (15 bit)            |  P (1) |  CID (16 bit)     |  ← DW3
  +-----------------------------+----------------------------+
   31  17                    17 16  15                    0

  Fabrics CQE（Figure 99）：
    字节  0-7   Fabrics 响应类型特定 (FRTS)
    字节  8-9   SQHD（流量控制禁用时保留）
    字节 10-11  保留
    字节 12-13  CID
    字节 14-15  Status Info: STATUS(15:1) | 保留(0)

  状态字段（Figure 100）位域：
    31    DNR      1=不要重试
    30    M        1=错误信息日志有更多内容
    29:28 CRD      重试延迟槽（仅 DNR=0 + ACRE=1 时有意义）
    27:25 SCT      状态码类型
    24:17 SC       状态码
```

## 初学者案例

**场景：写盘返回 `0x000E0F02`——主机要不要重试？**

1. 主机收到 CQE，DW3 状态字段 `0x000E0F02`。
2. 拆位：`DNR=0`（可重试），`M=0`（无附加错误日志），`CRD=0`（立即重试），`SCT=0b011=3`（路径相关），`SC=0x02`。
3. 路径相关 SC `02h` 通常是"内部路径错误"，规范建议主机**先检查其他 ANA 路径**或**短暂等待后重试**。
4. 如果 `DNR=1`，主机**不应**重试相同命令到同一控制器；如果 `SCT=2h`（介质/数据完整性）且 SC=`81h`（CRC 错误），**直接换介质/重映射**，不要重试。
5. 排错流程：先看 `SCT` 分大类 → 看 `SC` 找具体条件 → 看 `DNR` 决定是否重试 → 看 `M` 决定是否拉错误信息日志。

> 排错提示：永远先拆 `DNR`，决定重试策略前再读错误信息日志（`M=1` 时）。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 通用 CQE 大小 | 至少 16 字节（DW0-DW3） |
| DW0/DW1 | 命令特定；不用则保留 |
| DW2.SQID | 提交队列标识符；Fabrics 保留 |
| DW2.SQHD | SQ 已消费头指针；Fabrics 在流量控制禁用时保留 |
| DW3.CID | 命令 ID；与 SQID 联合唯一 |
| DW3.P | 阶段标签；Fabrics 保留 |
| 状态位域 | `DNR`(31) `M`(30) `CRD`(29:28) `SCT`(27:25) `SC`(24:17)，bit 16 之后是 CID |
| SCT 类型 | `0h`=Generic；`1h`=Command Specific；`2h`=Media/Data Integrity；`3h`=Path Related；`4h-6h`=保留；`7h`=Vendor Specific |
| SC 范围 | 每族内 `00h-7Fh`=跨命令集；`80h-BFh`=I/O 命令集特定；`C0h-FFh`=Vendor Specific |
| 成功状态 | SC=`00h` 且 `DNR=0`、CRD=`00b` |
| DNR=1 | 重新提交到子系统内任何控制器**预期仍会失败** |
| DNR=0 | **可能**成功，不保证 |
| M=1 | 错误信息日志页（Log ID `01h`）有该命令的额外信息 |
| CRD 条件 | 仅当 `DNR=0` 且 Host Behavior Support `ACRE=1h` 时有效 |
| CRD 值 | `00b`=立即重试；`01b/10b/11b`=查 Identify Controller `CRDT1/2/3`（单位 100ms） |
| 多错误适用 | 控制器自行选择返回哪个状态码 |
| 阶段标签 | 主机初始化 0；控制器首次发布置 1；每次环绕翻转 1 次 |
| P 写入时机 | 多写 CQE 时，**最后一次**写才更新 P 位 |
| 阶段初值 | Admin CQ 在 `CC.EN→1` 之前初始化；I/O CQ 在创建命令前初始化 |
| 主机消费 | 只推进头指针，**不重写** P 位 |
| Fabrics CQE | `STATUS` 占 bit 15:1；bit 0 保留 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| SC=00h vs 全 0 CQE | 任何非零 SCT/SC 都视为非成功；即使 DNR/M=0 |
| DNR=0 vs 保证成功 | DNR=0 是"可能"；CRD 给出建议延迟 |
| CRD vs 强制延迟 | CRD 是建议性，提前重试**不报错** |
| M 位 vs 错误日志 | M=1 表示"有"，不代表"必读"；日志可能被覆盖 |
| SQID vs CID | SQID 表"哪条队列"；CID 表"队列内第几条" |
| SQHD vs 头门铃 | SQHD 是控制器"已消费"快照；头门铃是主机"已读"通知 |
| SCT 0h vs 1h | 0h=协议通用错误；1h=与具体命令绑定的错误 |
| P 位翻转 vs 完成到达 | 同一队列连续多完成条目，P 位只在"环绕"时翻 |
| 通用 vs Fabrics CQE | 通用 16 字节布局相同；Fabrics 在 DW3 P 位保留、状态字段搬到 FRTS 头部 |
| Path Related 错误 | 通常与多路径/ANA 转换相关；应尝试其他路径 |

## 进阶细节

- **DW2 详细位**（Figure 97）：`31:16` SQ Identifier；`15:0` SQ Head Pointer。
- **DW3 详细位**（Figure 98）：`31:17` Status（15 位）；`16` Phase Tag；`15:0` CID。
- **状态字段位**（Figure 100）：DNR(31)；M(30)；CRD(29:28)；SCT(27:25)；SC(24:17)。当 SCT=0 且 SC=0 时这几位应为 0。
- **SCT 完整表**（Figure 101）：0h=Generic；1h=Command Specific；2h=Media/Data Integrity；3h=Path Related；4h-6h=Reserved；7h=Vendor Specific。
- **Generic SC 常见值**（Figure 102）：
  - 00h Successful Completion
  - 01h Invalid Command Opcode
  - 02h Invalid Field in Command
  - 03h Command ID Conflict
  - 04h Data Transfer Error
  - 05h Commands Aborted due to Power Loss Notification
  - 06h Internal Error
  - 07h Command Abort Requested（Abort/Cancel）
  - 08h Command Aborted due to SQ Deletion
  - 09h Command Aborted due to Failed Fused Command
  - 0Ah Command Aborted due to Missing Fused Command
  - 0Bh Invalid Namespace or Format
  - 0Ch SGL Descriptor Type / Offset / Length / Subtype / Specific Field / Last Segment / 等无效
  - 0Dh Command ID Conflict（取消时）
- **ACRE 启用条件**（规范 5.1.25.1.14）：Host Behavior Support 特性（Feature ID `E0h`）字段；`ACRE=1h` 时 CRD 有效。
- **错误信息日志**：Log Page ID `01h`；M=1 时主机应用 `nvme get-log` 拉取。
- **路径相关状态**（Figure 106-107）：含 ANA 状态变化、持久丢失、内部路径错误等；主机通常应尝试其他路径或换控制器。
- **P 位与多写**：当控制器分多次写 CQE 各字段，P 位只在最后写时翻转（保证主机不会"提前读到半完成条目"）。
- **CQE 多写完成的可见性**：P 位翻转之前，主机按 Phase Tag 判定，**不应**误读为新条目。
- **CID 唯一性范围**：`CID`+`SQID` 联合唯一；同 SQ 内同时最多 65535 条未完成命令。
- **DNR 含义扩展**："子系统内任何控制器"都预期失败；不是仅当前控制器。

## 规范依据

- [Common CQE 布局 Figures 96-98，PDF 第 160-161 页](../_source/pages/page-160.md)
- [Fabrics CQE 布局 Figure 99，PDF 第 161 页](../_source/pages/page-161.md)
- [Status 字段位定义 Figure 100，PDF 第 162 页](../_source/pages/page-162.md)
- [SCT 值 Figure 101，PDF 第 163 页](../_source/pages/page-163.md)
- [Generic SC 值 Figure 102，PDF 第 163-166 页](../_source/pages/page-163.md)
- [Command Specific / Media / Path SC，PDF 第 166-171 页](../_source/pages/page-166.md)
- [Phase Tag 生命周期 Figure 108，PDF 第 171-173 页](../_source/pages/page-171.md)

## 相关阅读

- [通用命令格式](common-command-format.md) - CQE 与 SQE 通过 CID 配对
- [命令排序与仲裁](command-ordering-and-arbitration.md) - SQHD 跟踪 SQ 头指针
- [异步事件上报](asynchronous-event-reporting.md) - AER 也复用 CQE 通道
- [通信丢失与命令重试](communication-loss-and-command-retry.md) - CRD 给出重试延迟
