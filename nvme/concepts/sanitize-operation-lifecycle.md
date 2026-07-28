# Sanitize 操作生命周期（Sanitize Operation Lifecycle）

## 一句话说明

Sanitize（数据清除）命令用于**启动**或**恢复**一次覆盖整个 NVM 子系统（Subsystem）范围的后台数据擦除；它本身只是触发器，真正的擦除在后台异步进行，进度通过 Sanitize Status 日志（LID `81h`）观察。

## 生活化类比

把 NVM 子系统想成**一座仓库**：

- **Sanitize 命令** = 仓库经理下达"清仓"指令
- **块擦除（Block Erase）** = 工人用钢丝球磨掉货架上的所有标签
- **加密擦除（Crypto Erase）** = 直接换锁——把仓库门锁换了，旧的标签虽然还贴在货架上但已经打不开
- **覆写（Overwrite）** = 在货架上用油漆反复盖掉旧标签若干遍
- **介质验证（Media Verification）** = 经理再派一个人逐个货架检查"真的清干净了吗？"
- **验证后空间释放（Post-Verification Deallocation）** = 检查通过后把货架也拆了

> 关键：经理下达"清仓"指令后**不等清完就回办公室**——Sanitize 命令成功完成**不代表**擦除完成。

## 工作流程

```text
                       EMVS=1 + 启用验证
   +-------+   Sanitize 命令 +------------------+ Exit Media Verify
   | Idle  | ----------------> | Media Verify    | ----------------+
   | 空闲  |                   | 介质验证        |                 |
   +-------+                   +-----------------+                 |
       ^                                |                          v
       |                                | 验证后释放                 |
       |                                v                          |
       |                       +------------------+                |
       |                       | Post-Verify      |                |
       +---------------------- | Dealloc 释放     |<---------------+
        Sanitize 完成/          +------------------+
        失败清除状态
```

**端到端流程**：

1. 主机发 Sanitize 命令（`SANACT` 选择 Block Erase / Crypto Erase / Overwrite / Exit Media Verify）。
2. 控制器在返回 CQE 前**先更新** Sanitize Status 日志；任何失败都不动日志、不动用户数据。
3. 成功后进入对应阶段；主机通过 LID `81h` 轮询进度（`SPROG` / `SOS`）。
4. 处理完成后回到 Idle 状态；下次 Sanitize 必须由新命令触发。
5. Sanitize 期间**禁止**新固件激活；固件已"待激活"也会拒绝 Sanitize。

## 初学者案例

**场景：服务器淘汰前的安全擦除，怎么做得"专业可信"？**

1. IT 工程师确认目标 SSD **不是** Exported NVM Subsystem（导出型子系统不支持 Sanitize）。
2. 检查是否有 `pmr0` 等 Persistent Memory Region 启用——若有，先停掉。
3. 检查是否有命名空间处于写保护——若有，先解除。
4. 确认没有"待固件激活"状态——若有固件未激活，需先完成激活或重置。
5. 检查控制器是否被挂起——若有，恢复控制器。
6. 用 `nvme sanitize /dev/nvme0n1 -a 0x02`（Crypto Erase）发起后台擦除。
7. 命令立即返回（同步部分仅"启动"），但状态日志已变 `SOS=010b`（Sanitizing）。
8. 工程师轮询 `nvme sanitize-log /dev/nvme0n1` 看 `SPROG` 进度。
9. 完成时 `SOS=001b`（Sanitized），且 `GDE=1` 表示"自上次成功 Sanitize 后没写过用户数据"。

> 错误示范：直接对一台多域（Multi-Domain）SSD 发 Sanitize 会被拒（Asymmetric Access Inaccessible）；必须先合并域。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 命令成功 ≠ 擦除完成 | Sanitize 命令成功只意味"已启动后台处理"；完成要看 LID `81h` 的 `SOS=001b` |
| 失败不动状态 | 命令失败（CQE status ≠ 0）不启动任何后台操作，**不**修改 Sanitize Status 日志，**不**改用户数据 |
| CQE 与日志顺序 | CQE 返回前，Sanitize Status 日志已被更新；读日志比看 CQE 更准 |
| 范围 = 整个 NVM 子系统 | 不存在"只擦一个 NS"——若想"只擦一个 NS"，请用 Format NVM（带 Secure Erase 选项） |
| 三种擦除方式 | Block Erase / Crypto Erase / Overwrite；不支持的方式在 `SANICAP` 中清零，必须 Invalid Field 失败 |
| 介质验证可选 | 需 `EMVS=1` + `VERS=1`（`SANICAP` 报告支持） + `NDAS=0` + 擦除方式不是 Overwrite |
| 退出验证独立动作 | `SANACT=101b`（Exit Media Verify）只在目标处于 Media Verification 状态时合法；否则 Invalid Field |
| NDAS 行为 | `NDAS=1` 要求控制器不释放空间；若控制器禁止该行为，行为由 `Sanitize Config` 决定 |
| 不可用于 Exported Subsystem | 导出型 NVM 子系统**不支持** Sanitize；命令直接被拒 |
| 拒绝条件 | 下列任一条件成立即拒绝：PMR 启用 / 命名空间写保护 / 固件激活待定 / 控制器挂起 / 多域分割 / CMB 队列不合规 |
| 固件互斥 | Sanitize 运行期间禁止新固件激活；固件激活待定也拒绝 Sanitize |
| Overwrite 参数 | Overwrite 时可设 Pass 计数（`0`=16 次）、模式反转位、32 位数据模式 |
| 不支持要"显式失败" | 不支持的方式/非法组合不能"静默降级"——必须 Invalid Field 失败，保证可审计 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Sanitize vs Format NVM | Sanitize = 子系统级后台擦除；Format NVM = 命名空间级格式化（可附 Secure Erase） |
| Sanitize vs Secure Erase | "Secure Erase" 是 Format NVM 的一个选项；"Sanitize" 是独立命令+状态机 |
| 块擦除 vs 加密擦除 vs 覆写 | 三种擦除的"清干净"机制完全不同；同一台 SSD 通常只支持其中一两种 |
| 介质验证 vs 数据擦除 | 验证 ≠ 擦除：擦除清掉数据，验证**只检查**"是否真的清干净"；可能要花更久 |
| NDAS=1 vs Sanitize Config | NDAS 是"这次请求不要释放空间"；Sanitize Config 是"控制器对 NDAS 请求的总策略" |
| 受限失败 vs 非受限失败 | 失败模式下"受限"（Restricted）= 旧数据可能仍在；"非受限"（Unrestricted）= 必须完整擦除后才能退出 |
| 错误模式 vs 警告模式 | Sanitize Config 选错误模式 → NDAS=1 直接拒；选警告模式 → 接受但记"Sanitized Unexpected Deallocate" |
| AUSE=0 vs AUSE=1 | AUSE=0 = Restricted 完成（更严格）；AUSE=1 = Unrestricted（更快但允许中间状态退出） |
| 控制器挂起 vs 域分割 | 挂起是控制器状态；域分割是 NVM 子系统物理/逻辑划分；两者都阻断 Sanitize 但原因不同 |
| "范围"语义 | 子系统范围 ≠ 命名空间范围 ≠ Endurance Group 范围；Sanitize 是子系统范围 |

## 进阶细节

- **SANICAP 字段**（规范 5.1.22，Identify Controller）：编码支持的擦除类型、No-Deallocate 行为、是否禁止 NDAS=1、是否支持 Media Verification（VERS 位）。
- **AUSE 行为**：选择 Unrestricted 完成允许控制器在未完成完整 Sanitize 处理时进入 Idle；Restricted 模式必须等 Sanitize 处理完成才能退出。AUSE 体现在 Sanitize Status 的 `SSI.SANS` 字段。
- **EMVS 限制细节**（规范 5.1.22）：
  - `EMVS=1` + `SANACT=011b`（Overwrite）→ Invalid Field
  - `EMVS=1` + `NDAS=1` → Invalid Field
  - `EMVS=1` + `SANACT ∈ {010b, 100b}` + `NDAS=0` → 成功后进入 Media Verification 状态
- **Overwrite 模式反转**（规范 5.1.22）：Pass 计数为 `0` 时表示 16 次；模式反转在多 Pass 场景下用于"奇数次写 0xAAAAAAAA、偶数次写 0x55555555"以掩盖残影。
- **Sanitize Config**（Feature ID `17h`，规范 5.1.25.1.15）：只在 `SANICAP` 报告"控制器禁止 NDAS=1"时才有意义；决定此时收到 NDAS=1 的请求是"拒绝"还是"接受并记 Unexpected Deallocate"。
- **多域限制**：NVM 子系统被划分为多个 Domain 时，若控制器无法访问所有命名空间，Sanitize 命令会返回 `Asymmetric Access Inaccessible` 或 `Asymmetric Access Persistent Loss`。
- **与 Format NVM 的边界**（规范 5.1.22）：Format NVM 是命名空间级、**通常在控制器侧同步或近同步**完成；Sanitize 是子系统级、**显式后台异步**——监控接口（LID `81h`）也是 Sanitize 独有的。
- **与 Sanitize Status 日志的关系**：本概念定义"命令与启动条件"；Sanitize Status 日志定义"如何观察后台进度和最近结果"。
- **失败状态机**（规范 8.1.24.3）：Restricted Failure 和 Unrestricted Failure 是 Sanitize Operation State Machine 中的失败节点；AUSE=0 进入 Restricted Failure，AUSE=1 进入 Unrestricted Failure；两者都对应 `SOS=011b`（Failed）。
- **恢复方式**：从 Restricted Failure 恢复用 Restricted Exit Media Verify 或重新 Sanitize；从 Unrestricted Failure 恢复直接重发 Sanitize（具体步骤见 8.1.24.3）。
- **MTFA 与激活时间**：固件激活在 `MTFA`（Max Time for Firmware Activation）内必须完成；超时会返回 Firmware Activation Requires Maximum Time Violation；此时 Sanitize 不能启动（因有"待激活"条件）。

## 规范依据

- [Sanitize 命令范围、操作类型、能力门控（SANICAP）与多域限制，PDF 第 387 页](../_source/pages/page-387.md)
- [介质验证转换、序列化门控与拒绝条件细节，PDF 第 388 页](../_source/pages/page-388.md)
- [命令启动原子性、字段、Overwrite 参数与 CQE/日志顺序，PDF 第 389 页](../_source/pages/page-389.md)
- [操作值、SANICAP 字段、Sanitize Config 与多域、固件互斥，PDF 第 390 页](../_source/pages/page-390.md)
- [Sanitize Config Feature（Feature `17h`）的错误/警告模式，PDF 第 409 页](../_source/pages/page-409.md)

## 相关阅读

- [sanitize-operation-status.md](sanitize-operation-status.md) - 进度与结果的观察口
- [log-page-retrieval.md](log-page-retrieval.md) - LID 81h 的读取接口
- [format-nvm-lifecycle.md](format-nvm-lifecycle.md) - 同类的破坏性操作对照
- [persistent-event-log.md](persistent-event-log.md) - 配对的开始/完成事件流
