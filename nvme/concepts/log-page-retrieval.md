# 日志页读取（Log Page Retrieval）

## 一句话说明

`Get Log Page` 是 NVMe 的"**通用读日志命令**"——靠 **LID（Log Page Identifier）** 决定返回什么数据、靠 **scope 字段**决定看哪一层（子系统、域、控制器、命名空间），靠 **RAE/OT/LPO** 决定怎么消费（清事件、按字节读还是按索引读）。

## 生活化类比

把 `Get Log Page` 想成**万能取件机**：

- **取件码 = LID**：00h 报"哪些取件码有效"、01h 报"错误台账"、02h 报"健康档案"、… 一切靠取件码区分。
- **取件窗口大小 = NUMDL:NUMDU**：要拿几个 dword 的货；不够要补，太多不补零。
- **取件起始位置 = LPO**：
  - `OT=0` → 按**字节偏移**取（按 dword 对齐理解）。
  - `OT=1` → 按**条目索引**取（仅当该 LID 在 Supported Log Pages 中标 `IOS=1`）。
- **取件人级别 = NSID**：从"全酒店"到"单间"四种粒度。
- **是否取件后清台账 = RAE**：`RAE=0` 成功获取后清对应异步事件；`RAE=1` 不清；命令失败也**不清**。
- **取件人偏好渠道 = CDW14.CSI/UIDX**：选了才生效，乱填被忽略或拒绝。

万能取件机只负责"按规则搬货"，**不创造数据**——支持与否、长度多少、字段含义全部由 LID 决定。

## 工作流程

```text
   Host                       Controller                       Log object
     |                             |                              |
     |  Get Log Page               |                              |
     |  LID + scope(NSID)          |                              |
     |  NUMD dwords, LPO offset    |                              |
     |  OT (offset type)           |  select supported LID --->  |
     |  RAE (retain async event?)  |  choose byte/index start    |
     |  CDW14.CSI / UIDX           |                              |
     |---------------------------->|  transfer dword range        |
     |<-- data buffer -------------|<-----------------------------|
     |  successful with RAE=0 may  |                              |
     |  clear the corresponding AER|                              |
```

简化说明：上图是"信封式"流程，未还原任何编号图。

### 命令字段约定

| 关注点 | 契约 |
|--------|------|
| 传输大小 | `NUMDU:NUMDL` 是 0 基 dword 计数；请求**超出末尾**则返回完整日志 + 未定义尾部 dword |
| 事件消费 | 成功完成 + `RAE=0` → 清除对应异步事件；`RAE=1` 保留；命令失败**总保留** |
| 字节偏移 | `OT=0`：`LPOU:LPOL` 是字节偏移，按 dword 对齐；越界 = `Invalid Field in Command` |
| 索引偏移 | `OT=1`：`LPOU:LPOL` 是**条目索引**；仅在 LID 在 Supported Log Pages 中标 `IOS=1` 时合法 |
| 命令集选择 | `CDW14.CSI` 仅当所选 LID 支持且 `CC.CSS=110b` 时生效 |
| UUID 选择 | `CDW14.UIDX` 仅在该 LID 支持 UUID 选择时生效 |
| 字段位置 | 命令字段集中在 CDW10–CDW14；其他命令专属字段为保留 |

### Supported Log Pages（LID 00h）布局

```text
+--------------------------------------+
| LID 00h | LID 01h | ... | LID FFh   |   1024 字节
+--------------------------------------+
   每个 LID 一条 32-bit Supported & Effects:
   31             16 15          2 1   0
  +----------------+--------------+---+---+
  |  LID 专属       |   保留       |IOS|SUP|
  +----------------+--------------+---+---+

  - SUP=0 -> 主机应忽略其余位
  - IOS=1 表示支持索引偏移 (OT=1)
  - IOS 必须为 0 当且仅当不支持扩展 Get Log Page 数据
```

## 初学者案例

**场景：主机想读 SMART/Health，但只要后半段（温度历史）**

1. 主机提交 `Get Log Page`：
   - `LID=02h`、`NUMDL=N`、`NSID=FFFFFFFFh`（控制器级）。
   - `RAE=0`（如果后端配置允许用此读路径清对应 AER，但**这里通常配 RAE=1**以免误清事件）。
2. 因为要读的是**后半段**：
   - 设 `OT=0`、把 `LPO` 设为"温度历史偏移"。
   - 偏移不是 4 的倍数时：控制器可以拒绝；若接受，按"低 2 位视为 0"处理。
3. 控制器返回 `[LPO .. LPO + N*4)` 字节数据。
4. 主机发现 `NUMDL` 太大 → 控制器仍按日志末尾返回，**多读不补零**。
5. 若想把 `CSI` 切成别的命令集：必须先 `CC.CSS=110b`，且 `LID` 自身标注支持 CSI。

> 故障速查：返回 `Invalid Field in Command` 且 LPO 看起来"差不多"——很可能 LPO 低 2 位没清零；或在不支持 `IOS=1` 的日志上把 `OT=1` 强行用了。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 命令归属 | `Get Log Page` 是 Admin 命令，CQE 从 Admin CQ 走 |
| LID 是第一维度 | 同一命令靠 LID 选择数据，**LID 即功能** |
| 范围字段遵循日志定义 | 子系统/Domain/控制器/命名空间四种 scope 由日志自身规定 |
| `RAE=0` 才清 | 仅"命令成功"且 `RAE=0` 时清除对应异步事件 |
| OT 与 IOS 配对 | `OT=1`（索引模式）必须建立在 `IOS=1` 之上 |
| LPO 对齐 | 字节偏移模式按 dword 对齐；越界 = 错误 |
| CSI 条件 | `CDW14.CSI` 仅在 `CC.CSS=110b` 且日志支持时生效 |
| UUID 条件 | `CDW14.UIDX` 仅在所选 LID 支持 UUID 选择时生效 |
| Supported 矩阵 | `LID 00h` 给出 256 个 32-bit 描述（`SUP` + `IOS` 等） |
| 接口实例差异 | 不同 Admin Queue 或 Management Endpoint 上**支持的日志可以不同** |
| 非法 LID | 不支持的 LID 返回 `Invalid Log Page`（除命令专属例外） |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| `RAE=0` vs 命令失败 | `RAE=0` 成功才清；命令失败**永远不清** |
| 字节偏移 vs 索引偏移 | `OT=0` 字节；`OT=1` 索引；后者依赖 `IOS=1` |
| `NUMDL` 越界 vs 越界读取 | 越界读取 = 控制器**返回末尾 + 未定义尾部**；越界偏移（LPO 超出）= 错误 |
| `Supported Log Pages` vs `Commands Supported` | 前者报 LID 支持（`LID=00h`），后者报命令支持（LID 07h），**别混为同一物** |
| `CDW14.CSI` vs `CC.CSS` | `CC.CSS` 是"启用哪个命令集"；`CDW14.CSI` 是"这一次 Get Log Page 选哪个命令集"，后者仅在 `CC.CSS=110b` 时生效 |
| `UIDX` vs 厂商日志 | 厂商日志不一定支持 UUID 选择；要看 LID 自己的定义 |
| `LID=00h` vs `LID=01h` | 00h 是"我能读哪些日志"；01h 是"实际错误条目" |
| `OT=1` 必须 `IOS=1` | 不是"建议"，是"强制"——否则 `Invalid Field in Command` |
| 多接口矩阵差异 | Admin CQ 与 Management Endpoint 支持的日志**可不同**，不要假设 |

## 进阶细节

- **超出末尾的契约**：`NUMDL` 超出实际日志长度时，控制器返回**整条日志**，**不会**填零、**不会**填充任何额外数据；这给主机一种"读多少算多少"的安全网。
- **`LPO` 低 2 位的契约**：控制器**可以**拒绝低 2 位非零的偏移；**如果接受**，必须按"低 2 位视为 0"的方式解释。
- **索引模式的 `IOS`**：`Supported Log Pages` 条目的 `IOS` 位表示该 LID 是否支持按索引访问；不支持时 `OT` 必须为 0。
- **多 scope 的 NSID 解释**：
  - 控制器/子系统 scope：除 `0h` 或 `FFFFFFFFh` 之外的 NSID **非法**。
  - 命名空间 scope：必须指定一个有效 NSID。
  - 多 scope 日志（Domain/子系统/控制器等）：按日志定义 + Domain 支持/NSID 解析**实际 scope**。
- **CSI/UIDX 在不适用时的行为**：被忽略；不会造成命令失败，但**也不会**改变选择。
- **CSER 等扩展位**：`Commands Supported and Effects` 本身通过 `Get Log Page(LID=05h)` 读，命令处理流程本身**就是 `Get Log Page` 的递归应用**。
- **与 AER 的握手**：
  - AER 通知主机"某事件发生" → 主机用 `Get Log Page` 读详情 + `RAE=0` 消费。
  - `RAE=1` 留给"我想看，又不想让别的主机错过"的场景。
- **与 Persistent Event Log 的差异**：后者自带"建立快照 + 多次分片 + 代数一致"流程；`Get Log Page` 不做快照、不做代数——它读到的总是**当前视图**。
- **失败时不清事件的副作用**：主机若依赖 `RAE=0` 路径轮询，命令失败/超时会把事件**保留**到下次重试，避免"看起来清了但其实没清"。

## 规范依据

- [Get Log Page 命令总述与完成，PDF 第 223 页](../_source/pages/page-223.md)
- [Data Pointer、长度、RAE、LID 与日志专属选择器，PDF 第 224 页](../_source/pages/page-224.md)
- [LPO / OT / CSI / UIDX 偏移与选择器细节，PDF 第 225 页](../_source/pages/page-225.md)
- [LID 注册表与 scope 说明，PDF 第 226–227 页](../_source/pages/page-226.md)
- [Supported Log Pages 数据结构 Figure 204，PDF 第 227–228 页](../_source/pages/page-227.md)

## 相关阅读

- [error-and-health-logs.md](error-and-health-logs.md) - 典型 LID 01h/02h 消费方
- [persistent-event-log.md](persistent-event-log.md) - LID 0Dh 的特殊流式读取
- [telemetry-capture-logs.md](telemetry-capture-logs.md) - LID 07h/08h 多块读取实践
- [command-effects-and-support.md](command-effects-and-support.md) - LID 05h 自指代应用
