# 特性值与作用域（Feature Values and Scope）

**特性（Feature）**是 NVMe 控制器中可调节的运行参数（例如仲裁突发、电源管理、温度阈值、主机内存缓冲区等），通过 8 位 **特性标识符（Feature Identifier, FID）** 寻址，并由 **Get Features（获取特性）** 与 **Set Features（设置特性）** 两条 Admin 命令读写。每个特性都绑定到特定**作用域（Scope）**，并包含默认值（default）、当前值（current）以及可选的保存值（saved）三层状态。[PDF pp. 181-183](../_source/pages/page-181.md)

可以把 Feature 想象成**手机的"系统设置项"**：**默认值**是出厂设置（不可改）；**当前值**是此刻正在生效的设置；**保存值**是勾选"记忆设置"后跨重启/掉电依然保留的恢复值；而**作用域**则决定了这个设置是只影响本机（控制器）、某个 App（命名空间），还是整个子系统。

## 值的契约与状态模型

```text
manufacturer default (出厂默认，不可改)
        |
        +---- reset / power-cycle 回退 ----+
        v                                  |
    current value <--- Set Features --- saved value
        |                 `-- Save(SV)=1 ---^ (仅当特性可保存 SVBL=1 时)
        `---- 决定控制器当前实际行为
```

当控制器在 `ONCS` 中通过 `SSFS` 位声明支持保存值时，可保存的特性才同时拥有 `default / current / saved` 三类值；即便某个特性不可保存（`SVBL=0`），其自身定义也可能规定当前值跨重置或掉电保持（参见 Figure 385）。[PDF p. 181](../_source/pages/page-181.md)

| Get Features `SEL`（CDW10） | 返回的属性 |
|---|---|
| `000b` | **current（当前值）**：控制器当前实际生效的值 |
| `001b` | **default（默认值）**：厂商设定，除非特性另行规定否则不可改 |
| `010b` | **saved（保存值）**：持久化恢复值；若不支持保存或无保存值，则回退返回 default |
| `011b` | **supported capabilities（支持的能力）**：通过 CQE Dword 0 返回该特性的能力位 |

当 `SEL=011b` 时，CQE Dword 0 返回三个关键能力位（bit 3–31 保留）：
- **bit 0 `SVBL`（Saveable）**：该特性是否可保存。`SVBL=0` 时 Set Features 若设 `SV=1` 会返回 `Feature Identifier Not Saveable`。
- **bit 1 `NSSPEC`（Namespace Specific）**：该特性是否为命名空间作用域。
- **bit 2 `CHANG`（Changeable）**：该特性是否有任何值可改。注意 `CHANG=1` 仅说明允许修改，并不保证所有取值转换都合法；`CHANG=0` 时写入不同值会以 `Feature Not Changeable` 失败，写入相同值可能成功也可能失败。[PDF pp. 220-223](../_source/pages/page-220.md)

## 重置与作用域交互

```text
重置覆盖整个 NVM 子系统：
  `-- 所有受影响作用域：若有 saved 则恢复 saved，否则恢复 default

重置仅覆盖子系统的一部分（如单控制器 CLR）：
  |-- 控制器 / 每控制器命名空间作用域：恢复 saved 或 default
  `-- 更宽的资源作用域（Subsystem / Domain / Endurance Group / NVM Set）：保持不变
```

一次局部重置（如多控制器子系统中的单控制器重置）不会自动覆盖子系统、Domain（域）、Endurance Group（耐久度组）或 NVM Set 作用域的当前值。[PDF pp. 181-182](../_source/pages/page-181.md)

| 特性作用域 | 常规 `NSID` 选择规则 |
|---|---|
| **控制器（Controller）** | `0h` 或 `FFFFFFFFh`；传入有效 NSID 去 Set 会以 `Feature Not Namespace Specific` 失败 |
| **命名空间（Namespace）** | 传入活动 NSID 选中单个命名空间；仅当控制器允许时，`FFFFFFFFh` 才可对所有已挂载命名空间执行 Set |
| **子系统 / Domain / Endurance Group / NVM Set** | `NSID` 应为 `0h`；非零值按通用命令格式（Common Command Format）规则被拒绝 |
| **其他作用域** | 还包括 Reclaim Unit Handle（FDP 回收单元句柄）与 Controller Data Queue（控制器数据队列） |

对于命名空间作用域的 Get Features，`NSID=FFFFFFFFh` **不是**聚合读取，通常会以 `Invalid Namespace or Format` 失败；不支持的 FID 会返回 `Invalid Field in Command`。[PDF pp. 182-183](../_source/pages/page-182.md)

## Set Features 执行契约与常见用法

Set Features（CDW10 `FID[7:0]` + `SV[31]`，CDW11 特性参数，CDW14 `UIDX[6:0]`）可选配内存缓冲区。若使用 PRP（Physical Region Page，物理区域页）传递缓冲，最多只能跨越一个页边界且不得以 PRP List 开头。厂商自定义特性（Vendor-specific Feature）只有在命令和目标特性都支持 UUID 选择时，才能通过 `CDW14.UIDX` 选定由 UUID 定义的命名空间。[PDF pp. 392-395](../_source/pages/page-392.md)

- **排错建议（先查能力再修改）**：修改特性前先用 `nvme get-feature -f <FID> -s 3`（`SEL=011b`）确认 `CHANG` 和 `SVBL` 位，避免误设 `Save=1` 触发错误。
- **可见性与切换边界**：Set 完成后，后续提交的命令使用新值，正在执行的命令可能仍使用旧值；如需干净切换边界，主机应先排空相关工作，并在完成后重新发现受影响的能力。[PDF p. 395](../_source/pages/page-395.md)
- **多主机协调**：当多个主机并发修改作用域大于单个控制器的特性时，需由主机侧自行协调，Base 规范不规定协调协议。[PDF p. 182](../_source/pages/page-182.md)
- **反向能力声明（Host Behavior Support, FID `16h`）**：与普通特性"控制器告知/接受主机配置"不同，FID `16h` 是主机向控制器声明自己支持的行为（如高级命令重试 ACRE、Telemetry Data Area 4、LBA 格式扩展等）。该特性不可保存且采用"替换而非合并"语义；主机未声明前控制器不得启用依赖功能。[PDF pp. 407-409](../_source/pages/page-407.md)

## 规范依据

- [特性 default/current/saved 契约与 Figure 124-125 重置恢复，PDF pp. 181-182](../_source/pages/page-181.md)
- [控制器、命名空间与资源作用域的 NSID 选择，PDF pp. 182-183](../_source/pages/page-182.md)
- [Get Features 选择器与支持能力返回结果（Figure 191-195），PDF pp. 220-223](../_source/pages/page-220.md)
- [Set Features 包络、Save 位、UUID 选择与特性目录，PDF pp. 392-395](../_source/pages/page-392.md)
- [Host Behavior Support（FID 16h），PDF pp. 407-409](../_source/pages/page-407.md)

## 关联与引申概念

- **PRP（Physical Region Page）**：NVMe 用于描述主机物理内存页的指针机制，单次传输最多跨一个页边界时可直接用两个 PRP 条目而无需 PRP List。详见 [数据指针布局](data-pointer-layouts.md)。
- **通用控制器特性**：将 Arbitraton、Power Management、Volatile Write Cache 等基础 FID 组织在一起的策略集合。详见 [通用控制器特性](common-controller-features.md)。
- **命令与功能锁定（Command and Feature Lockdown）**：可禁止未授权路径修改特定 Feature。详见 [命令与功能锁定](command-and-feature-lockdown.md)。
- **Identify 命令模型**：控制器通过 Identify Controller 报告全局能力，配合 Get Features Capabilities 共同决定特性是否可用。详见 [识别命令模型](identify-command-model.md)。
- **HMB（Host Memory Buffer）**：通过 Set Features FID `0Dh` 启停并移交内存所有权的典型特性。详见 [主机内存缓冲区](host-memory-buffer.md)。
