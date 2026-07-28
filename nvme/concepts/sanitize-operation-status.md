# Sanitize 操作状态（Sanitize Operation Status）

## 一句话说明

Sanitize Status 日志（LID `81h`）是 NVMe 控制器持久化保留的**只读观察口**，用于报告最近一次 Sanitize 操作的时间估计、当前进度与最终结果；它本身不下发任何命令，只反映后台 Sanitize Operation State Machine（规范 8.1.24.3）的当前状态。

## 生活化类比

把 Sanitize 操作想成**医院的体检流程**：

- **Sanitize 命令** = 病人挂号，护士登记到 HIS 系统
- **后台擦除** = 病人去做 CT/抽血
- **Sanitize Status 日志** = HIS 系统的"检验报告面板"
  - **SPROG** = "检查完成百分比"
  - **SOS** = "总状态"（还没来 / 已完成 / 在检查 / 失败 / 已完成但异常）
  - **MVCNCLD** = "这次检查是否被中途打断"
  - **GDE** = "自上次清空后病人没碰过医院任何东西吗"
  - **OPC** = "覆写做了几轮"
  - **ETO/ETBE/ETCE** = "下次同类检查预计要多久"
- 病人查报告时系统是**只读**的——查报告不会改变报告内容

## 工作流程

```text
   主机发 Get Log Page LID 81h
              |
              v
   +---------------------------+
   |   Sanitize Status Log     |  (控制器返回 512 字节)
   +---------------------------+
   | SPROG  | SSTAT  | SSI ... |  ETO/ETBE/ETCE/ETPVDS | ...
   +---------------------------+
              |
              v
   主机按字段解析：
   - SOS (3 bit)         -> 当前/最近状态
   - SPROG (16 bit)      -> 仅在 Sanitizing/PVD 时是 0..65535 进度
   - MVCNCLD / GDE / OPC -> 状态细节
   - ETO/ETBE/ETCE/...   -> 时间估计（0h = 同步完成；FFFFFFFFh = 不知道）
```

**端到端流程**：

1. 主机定期 `Get Log Page LID 81h`。
2. 控制器返回当前 Sanitize Status 日志的快照（512 字节）。
3. 主机解析 `SOS`：
   - `000b` = 从未启动过
   - `001b` = 上次 Sanitize 成功，当前 Idle
   - `010b` = 正在 Sanitize
   - `011b` = 上次 Sanitize 失败
   - `100b` = 已 Sanitize 但中途"意外释放了空间"（NDAS=1 时的异常结果）
4. 若 `SOS=010b`，主机看 `SPROG`（0..65535）作为当前阶段的"分子"。
5. 主机可对照 `SSI.SANS`/`SSI.FAILS` 得到更细的状态（受限/非受限/失败/验证中/PVD）。
6. 主机读 `ETO/ETBE/ETCE/ETODMM/ETBENMM/ETCENMM/ETPVDS` 得到时间估计。

## 初学者案例

**场景：怎么知道"我两小时前发起的 Sanitize 走到哪了？"**

1. 主机两小时前发 `nvme sanitize /dev/nvme0n1 -a 0x02`（Crypto Erase）。
2. 工程师现在 `nvme sanitize-log /dev/nvme0n1` 查日志。
3. 工具解析：`SOS=010b`（Sanitizing），`SPROG=0x8000`（即 32768/65536 = 50%）。
4. 工程师对照 `ETCE`（Estimated Time for Crypto Erase）：如果 `ETCE=1800000` 表示还剩 30 分钟。
5. 一段时间后再次查询，`SOS=001b`（Sanitized），`GDE=1`（自上次 Sanitize 后没写过数据）。
6. 工程师放心地把这块盘退役。

> 边角案例：如果 `SOS=100b`（Sanitized Unexpected Deallocate），那说明当时请求 `NDAS=1`（不释放空间），但控制器**实际释放了**用户数据所占的全部空间——这种异常结果在审计上要专门标注。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 持久性 | Sanitize Status 日志在**电源循环和重置**后仍保留 |
| 有效性 | `CSTS.RDY=1` 时日志包含有效数据；控制器未就绪时不应解析 |
| 必需性 | 控制器报告 Sanitize 支持（`SANICAP ≠ 0`）就**必须**实现 LID `81h`；否则保留 |
| `SOS` 是状态机的快照 | 它反映 8.1.24.3 状态机的当前状态；不是单独维护的"业务状态" |
| `SPROG` 含义随状态变 | 仅在 `SOS=010b`（Sanitizing）的处理阶段或 Post-Verification Deallocation 阶段才返回 0..65535；其他阶段固定 `FFFFh` |
| `SPROG` 分母固定 65536 | 进度 = `SPROG / 65536`，不要被 `0x10000` 迷惑 |
| `SOS=100b` 是异常结果 | 表示"已 Sanitize 但意外释放了空间"，仅在 `NDAS=1` 异常路径下出现 |
| `SOS` 保留值 | `101b` 到 `111b` 保留；规范未定义含义，不应解释为有效状态 |
| `MVCNCLD=1` 表示验证被取消 | 原因可能是子系统组成变化或特定重置发生 |
| `GDE=1` 表示"自上次成功后没写过数据" | 配合"自出厂后首次 Sanitize 前没写过数据"或"自上次成功 Sanitize 后没写过数据"两个条件 |
| `OPC` 仅 Overwrite 时非零 | 其他擦除方式下 `OPC=0` |
| `ETx` 哨兵值 | `0h` = 估计命令完成时同步完成；`FFFFFFFFh` = 控制器无法提供估计 |
| `SSI.SANS` 仅在 Version Validity 允许时有效 | 否则不要解析这个字段 |
| `SSI.FAILS` 仅在 `SOS=011b` 时非零 | 其他 `SOS` 下 `FAILS=0` |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| `SOS=001b` vs `SOS=100b` | `001b` = 上次 Sanitize 干净结束；`100b` = 干净结束但**用户空间被意外释放了**（异常） |
| `SPROG` vs "实时百分比" | `SPROG` 仅在处理/PVD 阶段有效；其他阶段是 `FFFFh`（不是 100%！） |
| `GDE=1` vs "数据安全" | `GDE=1` 只代表"自上次成功后没写过数据"；不代表"现在盘上没数据" |
| `ETx` 估计 vs 实际时间 | 估计是控制器**开始 Sanitize 时的预算**；实际可能更短或更长 |
| `ETx` vs `ETxNMM` | `ETx`（ETO/ETBE/ETCE）是"标准操作时间"；`ETxNMM`（ETODMM/ETBENMM/ETCENMM）专用于 `NDAS=1` 启用时的"额外介质修改时间" |
| `MVCNCLD=1` vs `SOS=011b` | 验证被取消不一定导致 Sanitize 失败；要看是否进入 Failed 状态 |
| `SOS=010b` 子状态 | `SOS=010b` 不区分 Restricted/Unrestricted 处理、Media Verification、PVD；要靠 `SSI.SANS` 细分 |
| `SSI.FAILS=0` vs 成功 | `SOS ≠ 011b` 时 `FAILS=0`；不是"成功"，只是"没失败" |
| `SOS=000b` 含义 | "从未启动过"；一旦 Sanitize 启动过就再也不会回到 `000b` |
| 估计字段不写明单位 | 所有 `ETx` 单位都是"毫秒"；不要误以为是秒或微秒 |

## 进阶细节

- **Sanitize Status Log Page 字段布局**（Figure 291, 规范 5.1.12.1.33）：
  - Bytes `01:00` = SPROG（Sanitize Progress, 16 位）
  - Bytes `03:02` = SSTAT（Sanitize Status, 16 位，含 MVCNCLD/GDE/OPC/SOS 等子字段）
  - Bytes `07:04` = SSI（Sanitize Status Information, 含 SANS/FAILS）
  - Bytes `11:08` = ETO（Estimated Time for Overwrite, 32 位毫秒）
  - Bytes `15:12` = ETBE（Estimated Time for Block Erase, 32 位毫秒）
  - Bytes `19:16` = ETCE（Estimated Time for Crypto Erase, 32 位毫秒）
  - Bytes `23:20` = ETODMM（Overwrite with Deallocate Media Modification 时间）
  - Bytes `27:24` = ETBENMM（Block Erase with No-Deallocate Media Modification 时间）
  - Bytes `31:28` = ETCENMM（Crypto Erase with No-Deallocate Media Modification 时间）
  - Bytes `35:32` = ETPVDS（Estimated Time for Post-Verification Deallocation）
- **`SPROG` 边界行为**（规范 5.1.12.1.33）：
  - 当 `SOS ≠ 010b`（Sanitizing）→ 固定 `FFFFh`
  - 当处于 Media Verification 状态 → 固定 `FFFFh`（不适用）
  - 当 `SOS=010b` 且处于处理/PVD 阶段 → 0..65535 的实际进度
- **`SOS=100b` 触发条件**（规范 5.1.12.1.33）：控制器报告 Sanitize Capabilities 中"No-Deallocate After Sanitize"被禁止（Inhibit），但 Sanitize 命令又用 `NDAS=1` 成功完成时，可能被配置为"接受并记 Unexpected Deallocate"（Warning Mode）；此时 `SOS=100b`、状态机进入 Idle。
- **`MVCNCLD=1` 触发条件**（规范 5.1.12.1.33）：
  - 验证期间子系统组成变化
  - 验证期间发生特定 Controller Level Reset（传输层特定 reset 或 NVM Subsystem Reset）
- **`GDE=1` 双重条件**（规范 5.1.12.1.33）：
  - 自制造起 + 首次 Sanitize 之前未写过数据，**且** 自制造起未启用过 PMR
  - 或 自上次成功 Sanitize 起未写过数据，**且** 期间未启用过 PMR
- **`SSI.SANS` 7 种状态**（规范 5.1.12.1.33 + 8.1.24.3）：Idle / Restricted Processing / Restricted Failure / Unrestricted Processing / Unrestricted Failure / Media Verification / Post-Verification Deallocation
- **`SSI.FAILS` 失败码**：仅在 `SOS=011b` 时给出具体失败码；规范 8.1.24.3 定义详细编码。
- **时间估计哨兵值**：
  - `0h` = 控制器估计操作将在 Sanitize 命令返回 CQE 时同步完成（无后台）
  - `FFFFFFFFh` = 控制器无法提供估计（不一定意味着不会后台运行）
- **`ETPVDS` 哨兵值**：只有 `FFFFFFFFh`（不能提供估计），没有 `0h` 哨兵。
- **与命令序列**：Sanitize 命令**先**更新 Sanitize Status 日志，再返回 CQE；这个顺序保证了"看到 CQE 成功就一定能看到 `SOS=010b`"。
- **LID `81h` 存在性**：`SANICAP=0` → LID `81h` reserved；Get Log Page 会返回零页或按规范 5.1.12 通用行为处理。

## 规范依据

- [Sanitize Status 日志的必需性、持久性、就绪状态与进度语义，PDF 第 302 页](../_source/pages/page-302.md)
- [Sanitize Status 位字段（SOS/MVCNCLD/GDE/OPC）定义，PDF 第 303 页](../_source/pages/page-303.md)
- [时间估计字段（ETO/ETBE/ETCE/ETPVDS）与异常结果，PDF 第 304 页](../_source/pages/page-304.md)
- [No-Deallocate 时间估计与 SSI 编码，PDF 第 305 页](../_source/pages/page-305.md)
- [Sanitize Operation State Machine 完整状态机，PDF 第 387 页](../_source/pages/page-387.md)

## 相关阅读

- [sanitize-operation-lifecycle.md](sanitize-operation-lifecycle.md) - 触发与状态机源头
- [log-page-retrieval.md](log-page-retrieval.md) - Get Log Page 通用机制
- [persistent-event-log.md](persistent-event-log.md) - 关联事件流
- [format-nvm-lifecycle.md](format-nvm-lifecycle.md) - 类似异步操作对照
