# 断电信号机制（Power Loss Signaling, PLS）

**断电信号机制（Power Loss Signaling，PLS）** 是 NVMe 在 PCIe 上提供的 Domain（域）级掉电预警机制：当系统检测到电源即将断开时，传输层通过断言 **PLN（Power Loss Notification）** 信号通知控制器，使其在真正断电前按 Domain 统一配置的 **强制静默（FQ）** 或 **紧急断电（EPF）** 模式完成数据保护（Vault）。[PDF pp. 615-616](../_source/pages/page-615.md)

> **类比**：把同一 Domain 内的控制器想成**商场里的一群收银员**——**PLN 信号**是商场广播“即将停电”；**FQ 模式**是收银员结完手头这几单、暂停接新单（收银机还亮着）；**EPF 模式**是收银员直接把账本锁进保险箱（丢弃未完成订单），甚至可以直接关掉收银机电源（端口禁用）。

## 状态机与信号流转

```text
上电 / 复位 / 关机 (非零 SHST)
         │
         ▼
  [PLS Not Ready]  (屏蔽所有 PLN 转换)
         │  SHST=00b (正常工作)
         ▼
   [PLS Ready] <─────────────────────────────────┐
         │  检测到 PLN 断言 + 依据 Domain 配置     │ 解除 PLN
         ├───────────────────┬───────────────────┤
         ▼                   ▼                   ▼
   [FQ 处理中]         [EPF 处理中]        [EPF 处理中]
   (端口保持启用)      (端口保持启用)      (端口已禁用)
         │                   │                   │
         ▼                   ▼                   ▼
   [FQ Complete]       [EPF Complete]      [EPF Complete]
         │                   │                   │
    解除 PLN ────────────────┘                   │
                    (复位/关机返回 Not Ready)     └── 只能重新上电恢复
```

- **信号定义**：[PDF pp. 615-616](../_source/pages/page-615.md)
  - **PLN（传输层断言，强制支持）**：告知电源即将断开。在控制器级复位（CLR）或关机（Shutdown）进行中/已完成时，PLN 转换会被忽略；CLR 后 PLN 与 PLA 均重置为未断言。
  - **PLA（控制器断言，可选支持）**：向传输层应答处理状态，有三种态：`Deasserted`（未在 EPF 或处理已完成）、`Asserted-EPF-Enabled`（EPF 激活且 PCIe 端口保持通信）、`Asserted-EPF-Disabled`（EPF 激活且 PCIe 端口已停止）。
- **Domain 级一致性**：同一 Domain 内所有控制器必须披露相同的支持模式、使用相同的选定模式，且实现相同的 EPF 端口行为（每个控制器只能实现“端口启用”或“端口禁用”之一）。[PDF pp. 615-620](../_source/pages/page-615.md)

## 两种处理模式对比

| 维度 | 强制静默模式（Forced Quiescence, FQ） | 紧急断电模式（Emergency Power Fail, EPF） |
|---|---|---|
| **已取命令** | 停止取新命令，**完成或中止**所有已取出的命令 | 停止取新命令，**直接丢弃**所有已取 I/O 及管理端点带外命令 |
| **后台操作** | **挂起**（GC、磨损均衡等暂停，直至 PLN 解除） | 直接停止/丢弃以全力进行数据保护（Vault） |
| **PCIe 端口通信** | **保持启用**（可继续接收/管理，但已取状态锁定配置） | 可实现为 **端口保持启用** 或 **端口禁用（不处理任何命令）** |
| **退出与恢复** | 若电源未断且解除 PLN，自动恢复正常取指；复位/关机回 Not Ready | 端口启用分支可通过复位/关机回 Not Ready；**端口禁用分支只能重新上电恢复** |
| **断电后恢复期** | 正常启动 | 首次初始化时在 Identify Controller 披露恢复间隔（`EPF Recovery`）；期间可能暴露未就绪或降级的命名空间，实际恢复可在 `CC.EN=1` 前后开始且可能长于 `CSTS.RDY=1`。[PDF pp. 616-617, 621-622](../_source/pages/page-616.md) |

- **时序值语义**：支持 PLS 时需实现 Domain 范围配置、命名空间 I/O 影响报告及至少一个非零适用电源状态时序字段；若 Vault 或 Recovery 时长配置为 `0`，表示实际时长为厂商自定义。[PDF pp. 615-617](../_source/pages/page-615.md)

## 关联与引申概念

- **PLP（Power Loss Protection，掉电保护）**：SSD 硬件层（如电容阵列）与固件协作在意外断电时将缓存数据刷入闪存的统称；PLS 则是主机/传输层向控制器发出提前预警的协议级信号接口。
- **SHN / SHST（Shutdown Notification / Status）**：控制器配置/状态寄存器中的关机通知与关机状态字段；`SHST=00b` 表示正常运行时才能进入 `PLS Ready`。
- **相关文档**：
  - [power-state-descriptors.md](power-state-descriptors.md) — 报告 FQ/EPF vault 与 recovery 时长参数
  - [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) — 导致 PLS 返回 Not Ready 的复位与关机条件
  - [common-controller-features.md](common-controller-features.md) — Domain 范围的 PLS 模式选择器配置
  - [domains-and-divisions.md](domains-and-divisions.md) — Domain 边界与多域独立性

## 规范依据

- [PLS 模式、作用域、PLA 表与被忽略的转换，PDF pp. 615-616](../_source/pages/page-615.md)
- [Figure 660 状态机与端口行为矩阵，PDF pp. 617-620](../_source/pages/page-617.md)
- [FQ 与 EPF 处理序列及恢复间隔，PDF pp. 621-622](../_source/pages/page-621.md)
