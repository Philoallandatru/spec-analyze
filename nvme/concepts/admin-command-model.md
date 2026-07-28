# Admin 命令集模型（Admin Command Model）

## 一句话说明

Admin Command Set 规定提交到 **Admin Submission Queue**（管理提交队列）的全部命令；它由所有传输模型通用的命令与 Memory-based、Message-based 传输模型特有的命令三组拼成，套用统一的 SQE/CQE 信封，再用命令专属的 dword 区分。

## 生活化类比

把 Admin Submission Queue 想成一家**酒店的前台**：

- **前台** = Admin SQ（管理提交队列），所有"管理类业务"都从这里发起
- **常驻服务台** = 5.1 通用命令（Identify、Get Log Page、Set Features、Abort 等），无论客人坐高铁还是飞机来都要走这里
- **本地柜台** = 5.2 Memory-based 命令（Create/Delete I/O Queue、Doorbell Buffer Config），只有 PCIe 这类"本地客"才用
- **远程柜员** = 5.3 Message-based 命令（Discovery、Zoning、Exported Resource 管理），只有 Fabrics 这种"远程客"才用
- **客房钥匙牌** = SQE/CQE 共用信封；房号 = opcode；行李方向牌 = Data Transfer Direction（DTD）

客人（命令）只能在前台登记，前台再决定把它转给哪个柜员办理。

## 工作流程

```text
                    Admin Submission Queue
                    +-------------------------+
                    |  SQE 共用部分           |  (opcode, NSID, 数据指针)
                    |  CDW10..CDW15 命令专属  |  (由各命令定义)
                    +-----------+-------------+
                                |
                                v
                       +--------+--------+
                       |  opcode 选择命令 |
                       |  DTD 选择方向   |
                       +--------+--------+
                                |
        +-------------+----------+-----------+--------------+
        |                     |                          |
        v                     v                          v
  5.1 通用命令          5.2 Memory-based            5.3 Message-based
  (所有传输)            (仅 PCIe 等)                (仅 Fabrics)
  Identify              Create/Delete I/O Queue     Discovery
  Features              Virtualization Mgmt         Fabric Zoning
  Logs / Firmware       Doorbell Buffer Config      Exported NVM Resource
  Format / Sanitize
  Fabrics(opcode 7Fh)
```

简化说明：上图只展示"三组命令 + 统一信封"的拓扑，省略具体的命令支持/禁止矩阵。Fabrics 命令统一用 `7Fh`，具体操作由 Fabrics 命令格式内部字段决定。

## 初学者案例

**场景：为什么我提交了一个"自定义"的 Admin 命令，控制器直接报错？**

1. 主机想要一个厂商自定义的管理操作，于是挑了个"没在表里"的 opcode 塞进 Admin SQ。
2. 控制器看到 SQE，按图 Figure 141 查表——opcode 不在表内。
3. 控制器以 **Invalid Opcode** 状态码返回 CQE，命令不执行。
4. 主机改用厂商保留区 `C0h..FFh` 重试——这才是规范允许的厂商自定义空间。
5. 再次提交，若厂商实现了该命令，控制器正常处理；否则返回 `Invalid Opcode` 或 `Command Not Supported`。
6. 若用 `00h`（Delete I/O SQ）这种合法通用命令，控制器就能正常完成。

> 故障速查："我提交的命令控制器没反应" 90% 是 **opcode 不在 Figure 141 中** 或 **DTD 方向填错**。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 信封统一 | 所有 Admin 命令复用 SQE/CQE 共用结构（opcode、NSID、数据指针、CID、PRT 等） |
| opcode 构成 | combined opcode = `Function[5:0]` + `Data Transfer Direction[1:0]`，同一功能可有 4 种方向 |
| NSID 处理 | 命令不使用 NSID 时字段清零；使用 NSID 时一般支持 `FFFFFFFFh`（除非 Figure 141 脚注例外） |
| 三段分类 | 5.1 通用（所有传输）、5.2 Memory-based、5.3 Message-based，按控制器传输类型取交集 |
| 未列即保留 | Figure 141 未列出的 opcode 全部保留，提交会被 `Invalid Opcode` 拒收 |
| 厂商范围 | combined opcode `C0h..FFh` 为厂商自定义（11xx xx b） |
| Fabrics 共享 | 全部 Fabrics 命令共用 opcode `7Fh`，由 Fabrics 命令格式内字段区分 |
| 队列独立 | Admin 命令处理不应被 I/O 队列状态影响（如 I/O CQ 满不应阻塞 Delete I/O SQ） |
| 导出资源管理命令 | 在仅使用 Memory-based 传输的子系统中**禁止**支持（参见 Figure 141 脚注 10） |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 通用 vs 传输专属 | 5.1 命令任何传输都可用；5.2/5.3 只在对应控制器上合法 |
| opcode vs Function | opcode 是 8 位 combined 值，Function 只是其中高 6 位 |
| DTD vs NSID | DTD 决定数据流向，NSID 决定命令作用于哪个命名空间，互不相干 |
| Format 限制 vs Sanitize 限制 | 两者共用 Figure 142 的允许表，但额外限制各自不同（Sanitize 期间 Vendor/Persistent 日志禁止，Format 期间 Set Features 中 NWP 被禁） |
| 厂商命令 vs 保留 | 厂商区 `C0h..FFh` 是"被允许的厂商自定义"；其余未列 opcode 是"禁止使用" |
| Abort vs AER | Abort 是同步命令、提交即处理；AER 是占位命令、控制器有事件时才完成 |

## 进阶细节

- **Figure 141 opcode 分段**：
  - `00h..3Fh` 为标准管理命令通用区（含 Identify、Features、Format、Sanitize、Namespace Management 等）
  - `40h..7Eh` 为 Fabrics 之外保留区
  - `7Fh` 专属 Fabrics 命令
  - `80h..BFh` 为 I/O Command Set 专属或保留（`80h` Format NVM、`84h` Sanitize 落在这一段）
  - `C0h..FFh` 厂商区
- **Figure 141 脚注 10 排除**：导出资源管理类命令（Clear/Create/Manage Exported NVM Subsystem、Manage Exported Namespace、Manage Exported Port）仅在 Message-based 子系统内允许。
- **Format NVM 与 Sanitize 的允许表 Figure 142**：
  - Format 进行中：被 Format 影响的命名空间上的非允许表命令可能被中止，返回 `Format In Progress`
  - Format 请求时：若该命名空间上还有非允许表命令在执行，Format 本身可能被中止，返回 `Command Sequence Error`
  - Sanitize 进行中：Vendor Specific 日志与 Persistent Event Log 禁止；Sanitize 命令本身在 `SANACT≠101b` 时被禁
  - 任何时刻下列命令均可用：Abort、AER、Create/Delete I/O Queue、Identify、Keep Alive、Set Features 中除 NWP 外的多数、Get Features、Get Log Page（仅特定日志）、Selected Fabrics 命令
  - 厂商命令在"既不影响也不读取用户数据"时允许
- **opcode 与 controller type**：是否支持某 opcode 还要查 Figure 28 的控制器类型支持矩阵，不能只凭 Figure 141 判定。
- **Format/Sanitize 期间影响范围**：中止/拒绝判定都仅针对"被该 Format 影响的命名空间"，其他命名空间不受牵连。

## 规范依据

- [Admin Command Set 概述与 Figure 141 opcode 矩阵上半，PDF 第 191 页](../_source/pages/page-191.md)
- [Figure 141 opcode 矩阵下半与脚注，PDF 第 192 页](../_source/pages/page-192.md)
- [Figure 142 Format/Sanitize 期间允许的 Admin 命令，PDF 第 193 页](../_source/pages/page-193.md)

## 相关阅读

- [command-effects-and-support.md](command-effects-and-support.md) - 命令支持与效果日志
- [command-and-feature-lockdown.md](command-and-feature-lockdown.md) - 命令锁定的动态权限
- [asynchronous-event-reporting.md](asynchronous-event-reporting.md) - AER 是 Admin 命令
- [command-abort-semantics.md](command-abort-semantics.md) - Abort 是 Admin 命令
