# Sanitize 操作状态（Sanitize Operation Status）

Sanitize Status 日志（LID `81h`）是 NVMe 控制器持久化保留的**只读观察口**，用于向主机报告最近一次 Sanitize 操作的时间估计、当前进度（`SPROG`）与最终结果（`SOS`）。只要控制器在 `SANICAP` 中报告支持 Sanitize，就必须实现该日志；日志内容**跨控制器重置和电源循环保留**，且在 `CSTS.RDY=1` 时有效。[PDF pp. 302-305](../_source/pages/page-302.md)

可以把 Sanitize Status 日志想成**医院的体检检验报告面板**：
- 下达 Sanitize 命令相当于挂号开单，后台擦除相当于做检查；
- 查报告 панели（读取 LID `81h`）是完全**只读**的：`SPROG` 是当前阶段进度，`SOS` 是总状态（未开始 / 进行中 / 成功 / 失败 / 意外释放），`GDE` 表示"自上次清空后是否再未写入过数据"。

---

## 状态观察模型与关键字段

```text
  主机下发 Get Log Page (LID 81h) ──> 返回 512 字节持久快照（Figure 291）
  +--------+--------+---------+-----------------------------------+
  | SPROG  | SSTAT  |   SSI   | ETO / ETBE / ETCE / ETxNMM / ETPVDS|
  | (2B)   | (2B)   |  (4B)   | (各 4B，单位：毫秒)                |
  +--------+--------+---------+-----------------------------------+
```

### 1. 进度与核心状态字段（`SPROG` 与 `SSTAT`）

| 字段 | 含义与编码规则 |
|------|----------------|
| **`SPROG`**（Bytes `01:00`） | 16 位进度分子，分母固定为 **65,536**（进度 = `SPROG / 65536`）。**仅在**处于"清理处理中"或"验证后释放"阶段时有效；在"介质验证"或非进行中状态时固定为 `FFFFh`（不适用，非 100%） [PDF p. 302](../_source/pages/page-302.md) |
| **`SOS`**（`SSTAT` 中 3 bit） | 操作状态：`000b`=从未启动（启动过一次后永不回此值）；`001b`=已清理（空闲）；`010b`=清理中；`011b`=失败；`100b`=已清理但意外释放空间（`NDAS=1` 在 Warning Mode 下实际释放了空间）；`101b–111b`=保留 [PDF pp. 303-304](../_source/pages/page-303.md) |
| **`MVCNCLD`** | 介质验证被取消标志：因验证期间子系统组成变化或发生特定重置（如传输层重置、NVM 子系统重置）而置位 |
| **`GDE`**（Global Data Erased） | 全局数据擦除标志：当自出厂后首次 Sanitize 前（或自最近一次成功 Sanitize 后）**未写入过任何用户数据**，且期间**未启用过 PMR** 时置 1 |
| **`OPC`** | 已完成的 Overwrite 遍数（仅 Overwrite 操作有效，其他操作为 0） |

### 2. 详细状态信息（`SSI`，Bytes `07:04`）

- **`SSI.SANS`**：仅在版本有效性位允许时有效，细化暴露规范 8.1.24.3 状态机的 7 种状态：空闲（Idle）、受限处理中/受限失败（Restricted Processing/Failure）、非受限处理中/非受限失败（Unrestricted Processing/Failure）、介质验证中（Media Verification）、验证后释放中（Post-Verification Deallocation）。[PDF p. 305](../_source/pages/page-305.md)
- **`SSI.FAILS`**：仅在 `SOS=011b`（失败）时记录具体失败状态码；其他状态下固定为 `0`。

---

## 时间估计字段（单位：毫秒）

日志提供各类擦除方式的后台预估耗时，帮助主机规划运维窗口：[PDF pp. 304-305](../_source/pages/page-304.md)

| 类别 | 字段名 | 估计的操作类型 | 哨兵值含义 |
|------|--------|----------------|------------|
| **标准操作** | `ETO` / `ETBE` / `ETCE` | 覆写 / 块擦除 / 加密擦除 | `0h` = 预计随命令完成同步结束；`FFFFFFFFh` = 无法提供估计 |
| **No-Deallocate 介质修改** | `ETODMM` / `ETBENMM` / `ETCENMM` | 启用 `NDAS=1` 且包含额外介质修改的覆写 / 块擦除 / 加密擦除 | `0h` = 预计同步结束；`FFFFFFFFh` = 无法提供估计 |
| **验证后释放** | `ETPVDS` | 验证后释放（Post-Verification Deallocation）耗时 | `FFFFFFFFh` = 无法提供估计（无 `0h` 哨兵） |

---

## 常见易混淆点

| 容易混淆的概念 | 实际区别 |
|----------------|----------|
| **`SPROG=FFFFh` vs `100%`** | `FFFFh` 表示"不适用"（如当前在介质验证阶段或已空闲），并非 100% 进度 |
| **`GDE=1` vs 数据安全** | `GDE=1` 仅表示"自上次成功 Sanitize（或出厂）后没有写过数据且未开 PMR"，**不代表**盘上一定执行过 Sanitize |
| **`SOS=001b` vs `SOS=100b`** | `001b` 是正常完成；`100b` 是请求了 `NDAS=1`（不释放空间）但控制器最终仍释放了用户数据空间，审计时需特别关注 |
| **Sanitize vs Format NVM** | 本日志仅跟踪 Sanitize 状态机，不反映 Format NVM 的 Secure Erase 操作 |

---

## 关联与引申概念

- **Sanitize Operation State Machine（规范 8.1.24.3）**：Sanitize 后台执行的权威状态机，LID `81h` 只是该状态机的持久化只读投影。
- **CQE 与日志更新顺序**：控制器在返回 Sanitize 命令的 CQE 之前必须先更新 LID `81h`，因此主机收到成功 CQE 后立即查询日志必定能看到更新后的状态。
- **相关文档**：
  - [Sanitize 操作生命周期](sanitize-operation-lifecycle.md) — 触发 Sanitize 的命令、门控条件与配置特性
  - [日志页获取](log-page-retrieval.md) — `Get Log Page` 通用读取机制
  - [Format NVM 生命周期](format-nvm-lifecycle.md) — 命名空间级格式化对比
  - [持久事件日志](persistent-event-log.md) — 记录 Sanitize 历史事件的持久日志

## 规范依据

- [Sanitize Status 支持性、持久性与 SPROG 进度语义，PDF p. 302](../_source/pages/page-302.md)
- [Sanitize Status 位字段（SOS/MVCNCLD/GDE/OPC），PDF pp. 303-304](../_source/pages/page-303.md)
- [时间估计字段（ETO/ETBE/ETCE/ETxNMM/ETPVDS）与 SSI 编码，PDF pp. 304-305](../_source/pages/page-304.md)
- [Sanitize 操作状态机（8.1.24.3），PDF p. 387](../_source/pages/page-387.md)
