# 通用命令格式（Common Command Format）

## 一句话说明

NVMe 规定所有管理命令和 I/O 命令的提交队列条目（SQE）都用同一套 64 字节布局，区别只在"命令特定字段"那 24 字节，控制器靠 `CDW0` 里的操作码和传输方向决定怎么解释。

## 生活化类比

把通用命令格式想成**统一的快递面单**：

- **64 字节固定大小** = 顺丰/中通/EMS 用同一张面单，只是"内容字段"由各家业务自填。
- **CDW0** = 面单上的"服务类型 + 是否需要签收"标签，搬运中心一看就知道怎么走流程。
- **NSID 字段** = "收件人 ID"，标明这个快件要送到哪个货架/房间。
- **PRP / SGL 指针** = "取件地址"，PRP 是按页算的固定地址，SGL 是任意分散地址清单。
- **CDW10-15** = 各家自填的备注栏：普通快递填重量，顺丰生鲜填温度，NVMe 里填 LBA、长度、命令特定标志。

## 工作流程

```text
  SQE 通用布局（64 字节，Figure 92）

  字节偏移  0       4       8      16      24              40              64
          +-------+-------+-------+-------+---------------+---------------+
          | CDW0  | NSID  |CDW2/3 | MPTR  | DPTR          | CDW10..CDW15  |
          +-------+-------+-------+-------+---------------+---------------+
          | OPC   | 命名  |       | 元数据| PRP1+PRP2     | 命令特定字段  |
          | FUSE  | 空间ID|       | 指针  | 或 SGL1       |               |
          | PSDT  |       |       |       |               |               |
          +-------+-------+-------+-------+---------------+---------------+

  CDW0 关键位（Figure 91）：
    bit 31:16  CID    命令 ID（与 SQ ID 联合唯一）
    bit 15:14  PSDT   PRP/SGL 选择
    bit 13:10  保留
    bit 09:08  FUSE   融合操作位置
    bit 07:02  OPC.FN opcode 功能码
    bit 01:00  OPC.DTD 传输方向
```

## 初学者案例

**场景：写命令主机把数据放到 PRP1，但控制器回"PRP 越界"。**

1. 你提交一条 NVM Write（OPC=`01h`，DTD=`10b` 主机→控制器）。
2. PRP1 指向主机内存第一页（4KB），PRP2 写的是第二页地址。
3. 命令长度是 8KB，跨两个 4KB 页，按规范 PRP2 应该指向**第二页**。
4. 但你的 PRP2 实际指向第三页（越过了第二页末尾），控制器回 `Invalid Field in Command`。
5. 检查 host 端 PRP 拆解逻辑，确保 PRP2 是"刚好跨一个页"时填第二页、跨多页时填 PRP List 指针。
6. 同样在 Fabrics 模式，所有 Admin/I/O 命令强制用 SGL（`PSDT=01b`），不能再用 PRP。

> 排错提示：`PRP2` 跨 1 页 vs PRP List 是 SQE 阶段最容易混淆的，按命令长度 / 起始页偏移严格判断。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 固定大小 | 64 字节（4 个 dword 头 + 8 字节 MPTR + 16 字节 DPTR + 6 个 dword 命令特定） |
| 共同字段 | CDW0、NSID、MPTR、PRP1、PRP2、SGL1、Metadata SGL Segment Pointer |
| 命令特定 | 字节 40-63（即 CDW10-CDW15）由具体命令规范定义 |
| PSDT 编码 | `00b`=PRP；`01b`=SGL（MPTR 为缓冲区地址）；`10b`=SGL（MPTR 为单描述符段）；`11b`=保留 |
| PSDT 协议约束 | PCIe 上 Admin 命令**必须**用 PRP；Fabrics 上 Admin/I/O **必须**用 SGL (`01b`) |
| FUSE 编码 | `00b`=普通；`01b`=融合第一条；`10b`=融合第二条；`11b`=保留 |
| DTD 编码 | `00b`=无传输；`01b`=主机→控制器；`10b`=控制器→主机；`11b`=双向 |
| NSID 取值 | 0=未用；1-FFFFFFFEh=具体 NS；FFFFFFFFh=广播（仅在命令明确支持时） |
| NSID 错误 | 对非活跃 NS→`Invalid Field in Command`；对不存在 NS→`Invalid Namespace or Format` |
| CID 警告 | 避免 FFFFh——错误信息日志用此值表示"无对应命令" |
| PRP2 用法 | 未跨页=保留；跨 1 页=第二页地址；跨多页=PRP List 指针 |
| 厂商命令 | 可选：CDW10-11 用作传输长度字段，保留公共指针布局 |
| Fabrics SQE | 操作码固定 `7Fh`；FCTYPE 占字节 4；用 SGL；没有融合形式 |
| Fabrics PSDT | `10b` 显式标记"一次传输"；`00b` 表示无传输 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| PRP vs SGL | PRP 是按 4KB 页地址描述；SGL 是任意分散描述符链 |
| MPTR 三种含义 | 取决于 PSDT：禁用/连续地址/单描述符段 |
| DPTR 两种含义 | 取决于 PSDT：PRP1+PRP2 / SGL1 |
| FUSE vs PSDT | FUSE 在位 8-9（融合）；PSDT 在位 14-15（传输方式选择） |
| CDW0 vs CDW2/3 | CDW0 全局定义；CDW2/3 由命令/命令集定义 |
| Admin 命令 vs I/O 命令 | 共享 64 字节布局；Admin 走 Admin SQ，I/O 走 I/O SQ；传输方式约束不同 |
| 通用 SQE vs Fabrics SQE | 通用 64 字节布局相同，但 Fabrics 用 `OPC=7Fh`+FCTYPE 二次分派 |
| 字节偏移 8-15 vs 16-23 | 字节 8-15 = CDW2/CDW3（命令相关）；字节 16-23 = MPTR |

## 进阶细节

- **CDW0 完整布局**（规范 4.1.1，Figure 91）：
  - `31:16` CID
  - `15:14` PSDT
  - `13:10` 保留
  - `09:08` FUSE
  - `07:02` OPC.FN
  - `01:00` OPC.DTD
- **SGL 对齐**：`PSDT=01b` 时 MPTR 对齐要求见 Identify Controller `SGLS.MBA`（Metadata Buffer Alignment）。
- **SGL 段**：`PSDT=10b` 时 MPTR 是 qword 对齐、含恰好一个 SGL 描述符的段地址。
- **传输层支持值**：具体传输可能仅支持 PSDT 子集，参见对应 NVMe Transport binding 规范。
- **PRP List**：跨多页时 PRP2 指向 PRP List 物理地址（PRP Entry 数组，最后一项目的位 0 = 1）。
- **CDW2/CDW3 用途**：常为数据块数 / LBA 低位（如 Read/Write）；由具体命令定义。
- **厂商特定命令**：可选扩展方式——用 CDW10/CDW11 当传输长度，保持公共指针布局，简化兼容。
- **未来扩展性**：新 I/O 命令集可能定义**非通用**的 SQE 格式；当前 Common 格式是"当前最广用"而非"唯一"。
- **Fabrics SQE 字段**：
  - 字节 0-3：CDW0（`OPC=7Fh`）
  - 字节 4：`FCTYPE`（6 位功能码 + 2 位传输方向）
  - 字节 5-23：保留
  - 字节 24-39：传输 / 键控 SGL 描述符（一次完整传输）
  - 字节 40-63：Fabrics 命令类型特定字段
- **完成匹配**：CID + SQID 在 CQ 条目里复现，控制器借此把命令和完成条目配对。
- **错误信息日志**：`CID=FFFFh` 在 Error Information log 中表示"非特定命令错误"。

## 规范依据

- [通用 SQE 概述与 Figure 91 CDW0，PDF 第 155 页](../_source/pages/page-155.md)
- [Figure 92 SQE 完整布局，PDF 第 156-158 页](../_source/pages/page-156.md)
- [PRP2 使用规则与 SGL 描述符，PDF 第 157-158 页](../_source/pages/page-157.md)
- [Fabrics SQE 布局 Figures 94-95，PDF 第 159-160 页](../_source/pages/page-159.md)

## 相关阅读

- [数据指针布局](data-pointer-layouts.md) - DPTR 字段按 PSDT 切 PRP/SGL
- [完成队列条目与状态](completion-queue-entry-and-status.md) - SQE 与 CQE 通过 CID 配对
- [命令排序与仲裁](command-ordering-and-arbitration.md) - FUSE 位影响命令原子性
- [识别命令模型](identify-command-model.md) - Identify 也用通用 SQE 格式
