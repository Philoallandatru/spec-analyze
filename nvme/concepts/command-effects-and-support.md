# 命令效果与支持（Commands Supported and Effects）

## 一句话说明

"命令支持与效果日志"（Commands Supported and Effects Log, LID=05h）是一个 4096 字节的结构，每个 Admin 操作码（Opcode）和每个 I/O 操作码对应一个 32 位描述符，告诉主机"这个命令支不支持 + 它可能影响哪些资源"。

## 生活化类比

把命令效果日志想成**餐厅的菜单副作用说明**：

- **每一道菜** = 一个命令（Read / Write / Format NVM / Create Namespace …）
- **菜名旁的小字** = 描述符的 Scope（影响范围：包间？楼层？整栋楼？）
- **"本菜可能临时停水"** = Capability Change（能力变更）
- **"本菜可能换桌布"** = Data Change（数据变更）
- **"本菜需要整层楼暂停"** = Command Submission and Execution（提交与执行建议）
- **"已售罄"** = CSUPP = 0（不支持）

服务员（主机）点单前先扫一眼副作用说明，避免在婚宴上突然下单"全场灯光改造"（控制器能力变更）这种大动静。

## 工作流程

```text
主机拿日志（Get Log Page, LID=05h, 长度 4096）
              |
              v
+---------------------------------------------+
| 字节 3:0    ACS0   (Admin Opcode 0 描述符)  |
| 字节 7:4    ACS1   (Admin Opcode 1 描述符)  |
| ...                                         |
| 字节 1023:1020  ACS255                       |
| 字节 1027:1024 IOCS0  (I/O Opcode 0 描述符) |
| ...                                         |
| 字节 2043:2040  IOCS255                     |
| 字节 4095:2048  保留                         |
+---------------------------------------------+
              |
              v
按描述符做协调决策:
  - 暂停相关 Namespace 操作
  - 等待命令完成
  - 必要时重新 Identify / 重新枚举
```

简化说明：日志描述的是"命令**可能**的总体效果"，包括可选行为；主机应保守对待所有被设置的位。

## 初学者案例

**场景：Format NVM 之前，主机要不要做协调？**

1. 主机想对 NSID=1 调用 Format NVM。
2. 主机先 `nvme get-log /dev/nvme0 -i 5 -l 4096` 读 Commands Supported and Effects。
3. 找到 IOCS 对应 Format NVM 描述符（按当前 I/O Command Set 解释）。
4. 描述符位指示：Scope = Namespace；Capability Change = 单个 NS 能力变更；CSE = 命名空间内串行化。
5. 主机执行协调动作：
   - 暂停对 NSID=1 的所有 I/O；
   - 排空 SQ；
   - 提交 Format NVM；
   - 等待 CQE；
   - 重新发 Identify 检查 NSID=1 的新容量/格式化参数。
6. 继续恢复业务。

> 速查：多控制器共享同一 Namespace 时，所有关联主机都需要参考 CSE 字段做协调。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 日志长度 | 固定 4096 字节（256 个 Admin + 256 个 I/O + 2048 字节保留） |
| 每条目 32 位 | Admin Opcode 0..255 与 I/O Opcode 0..255 各对应一个描述符 |
| CSUPP=0 即"不支持" | 描述符其他所有位全为 0 |
| 描述"可能的"效果 | 含可选行为；主机应保守对待所有被设置的位 |
| I/O 部分按命令集解释 | 由 `CC.CSS` 选定；`CC.CSS=110b` 时由 CDW14 的 CSI 动态选择 |
| 范围多选 | 命令效果日志的 Scope 可同时设多个位（NS / Controller / NVM Set / EG / Domain / Subsystem） |
| 串行化建议 | CSE/CSER 提示主机在 NS 范围或全局范围串行化；非零放宽策略会覆盖 CSE |
| 能力变更后 | 主机应暂停相关操作 → 等待 CQE → 重新 Identify / 重新枚举 |
| 多控制器共享 NS | 各主机应协调命令提交，满足 CSE 字段要求 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 命令效果日志 vs 特性效果日志 | 前者按 Opcode 索引；后者按 FID / NVMe-MI Opcode 索引 |
| 范围位可多选 vs 范围位唯一 | 命令效果日志 Scope 可多选；特性/MI 效果范围只能"恰好一位为 1"（含 CDQ） |
| CSUPP=0 vs 字段=0 | CSUPP=0 = 不支持；CSUPP=1 时其余字段也可为 0（表示"无效果"） |
| CSE vs CSER | CSE 是默认串行化建议；CSER 是放宽策略，非零时取代 CSE |
| Scope vs Capability Effects | Scope 指"影响哪类资源"；Capability Effects 指"资源发生什么变化"（能力/数据） |
| CC.CSS vs CDW14.CSI | `CC.CSS=110b` 时由 CDW14 动态指定；其他值由 `CC.CSS` 选定 |

## 进阶细节

- **描述符字段（Figure 210 概括）**：
  - `CSUPP`（bit 0）：命令支持位；=0 时其他位全 0。
  - `Scope`（多位可设）：Namespace / Controller / NVM Set / Endurance Group / Domain / NVM Subsystem。
  - `USS`（UUID Selection Support）：是否支持按 UUID 选择。
  - `CSE` / `CSER`：命令提交与执行建议（串行化级别、放宽策略）。
  - Capability Effects 标志：控制器能力变更、命名空间清单变更、单个 NS 能力变更、用户数据变更。
- **Feature Identifiers Supported and Effects Log**（接口特定）：
  - 同样长度，每条目一个 FID 或 NVMe-MI Opcode。
  - Scope 非零时**恰好一位**为 1（额外含 CDQ）。
  - 区分接口实例：Admin Queue / PCIe VDM Endpoint / 2-Wire Endpoint。
  - 还报告 Get/Set Features 是否接受 UUID 选择。
- **主机协调流程模板**：
  1. 解析描述符；
  2. 若涉及 NS 能力/清单变更，暂停相关 NS 的 I/O；
  3. 提交命令并等待 CQE；
  4. 重新 Identify / 重新枚举；
  5. 恢复业务。
- **NVMe-MI 操作码效果**：结构与 FID 效果日志类似，区别在于条目索引是 MI Opcode 而非 FID。

## 规范依据

- [Commands Supported and Effects Log Page 布局（Figure 209），PDF 第 236 页](../_source/pages/page-236.md)
- [描述符字段定义 Scope/CSE（Figure 210），PDF 第 237 页](../_source/pages/page-237.md)
- [放宽策略与能力/数据变更标志，PDF 第 238 页](../_source/pages/page-238.md)
- [Feature Identifiers Supported and Effects（接口特定），PDF 第 281 页](../_source/pages/page-281.md)
- [NVMe-MI 操作码效果日志，PDF 第 283 页](../_source/pages/page-283.md)

## 相关阅读

- [admin-command-model.md](admin-command-model.md) - 命令 opcode 与命令集分类
- [command-and-feature-lockdown.md](command-and-feature-lockdown.md) - 动态权限层补充静态能力
- [format-nvm-lifecycle.md](format-nvm-lifecycle.md) - Format 描述符字段含义
- [command-abort-semantics.md](command-abort-semantics.md) - Abort 描述符字段含义
