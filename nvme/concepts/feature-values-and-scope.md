# 特性值与作用域（Feature Values and Scope）

## 一句话说明

NVMe 把控制器可调的运行参数统称为"特性"（Feature），用特性标识符（FID）寻址；每个特性有"默认值 / 当前值 /（可选）保存值"三层状态，并绑定到"控制器 / 命名空间 / 子系统 / Endurance Group / NVM Set / Domain"等不同作用域。

## 生活化类比

把 Feature 想成**手机的"设置项"**：

- **默认值（default）** = 出厂设置，不可改。
- **当前值（current）** = 你现在实际在用的设置。
- **保存值（saved）** = "记忆"开关打开后，掉电重启也保留。
- **作用域（scope）** = 这个设置是只影响本机（控制器）、某只 App（命名空间），还是整个账号云（子系统）？
- **Get/Set Features** = "查看/修改设置"两个按钮；`SEL` 字段决定读的是当前/默认/保存值。
- **SSFS 开关** = 你的手机型号是否支持"设置项云端同步"——支持才有"保存值"。

## 工作流程

```text
  特性状态机（规范 4.4）：

  manufacturer default ── reset/power ──> current
       (不可改)                            │
                                            │  Set Features (Save=0)
                                            v
                                       current (生效)
                                            │
                                            │  Set Features (Save=1)
                                            │  且特性可保存 (SVBL=1)
                                            v
                                       saved (跨掉电/重置保留)
                                            │
                                            └── 读 saved 时若不存在 ──> default

  作用域 → 资源层级（从大到小）：
  Subsystem → Domain → Endurance Group → NVM Set → Namespace → Controller
  也存在：Reclaim Unit Handle / Controller Data Queue

  Get Features `SEL`（Figure 192）：
    000b → current
    001b → default
    010b → saved（若不存在则回 default）
    011b → supported capabilities（返回 CQE DW0）
```

## 初学者案例

**场景：`nvme set-feature -f 0x01` 改仲裁突发失败。**

1. 你想改仲裁突发（Arbitration Burst），特性 ID `01h`。
2. 改之前先 `nvme get-feature -f 0x01 -s 3` 查能力（`SEL=011b`）。
3. CQE DW0 返回 `0x07`，二进制 `111`：bit0 `SVBL=1`（可保存），bit1 `NSSPEC=0`（不是 NS 级），bit2 `CHANG=1`（可改）。
4. 改用 `nvme set-feature -f 0x01 -v 4`（`SV=0` 改当前值）成功。
5. 想让重置后保留，再 `nvme set-feature -f 0x01 -v 4 -s 1`（`Save=1`）持久化。
6. 若 `CHANG=0`，Set 任何不同值都会返回 `Feature Not Changeable`；同值可能成功也可能不成功。

> 排错提示：永远先 `SEL=011b` 查能力再改；`SVBL=0` 时不要设 `Save=1`，会回 `Feature Identifier Not Saveable`。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 特性数量上限 | 256 个特性 ID（含 00h 与 FFh） |
| 值的种类 | default（不可改）/ current（生效）/ saved（持久化，仅 SSFS=1 时存在） |
| SSFS 位 | `ONCS` 字段内；`SSFS=1` 支持三值；`SSFS=0` 只支持 default+current |
| Get SEL 字段 | `000b`=current；`001b`=default；`010b`=saved；`011b`=capabilities |
| Capabilities DW0 | bit0=SVBL；bit1=NSSPEC；bit2=CHANG；其余保留 |
| 作用域分类 | controller / namespace / subsystem / domain / endurance group / NVM set / reclaim unit handle / controller data queue |
| 控制器作用域 | `NSID=0` 或 `FFFFFFFFh`；用有效 NSID Set 失败 `Feature Not Namespace Specific` |
| 命名空间作用域 | 用活动 NSID 选一个；`FFFFFFFFh` 可设全部仅在控制器允许时 |
| 资源作用域（子系统/EG/NVM Set） | `NSID` 应为 `0h`；非零按 Common Command Format 拒绝 |
| Get `FFFFFFFFh` | 对 NS 作用域 **不是** 聚合读，常失败 `Invalid Namespace or Format` |
| 未知 FID | 失败 `Invalid Field in Command` |
| 多主机协调 | 作用域大于一个控制器时，多主机并发改需协调，Base 规范不规定协议 |
| 主机行为支持 | FID `16h` 反方向——主机声明自己能力，控制器依赖；非保存型，"替换而非合并" |
| Set 后的可见性 | 后续命令用新值；正在执行的命令可能用旧值——要"清边界"先排空相关工作 |
| 不可改特性 | 不同值失败；同值可能成功也可能 `Feature Not Changeable` |
| Set 缓冲 | PRP 缓冲至多跨 1 个页边界；**不得**以 PRP List 开头 |
| Vendor Feature | 用 CDW14 `UIDX` 选 UUID 定义的命名空间时，命令和特性都需支持 UUID 选择 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| current vs saved | current 是当前生效；saved 是下次重置/上电要恢复的值 |
| Get vs Set `Save` | Get `SEL=010b` 读 saved；Set `Save=1` 写 saved |
| `011b` Get vs Set | 仅 Get 支持 `011b`；Set 时 CDW10 `SV` 是另一回事 |
| NSID=0 vs FFFFFFFFh | 0=未指定/资源作用域；FFFFFFFFh=广播/全部（按命令语义） |
| CHANG=0 vs 当前值不可变 | CHANG=0 是"全局不可变"；具体字段可能仍可变（value-dependent） |
| CHANG=1 vs 任意值合法 | CHANG=1 仅说"能改"，并不保证任何值转换合法 |
| SSFS vs SVBL | SSFS 是控制器能力位；SVBL 是具体特性可保存位 |
| Host Behavior Support vs 普通 Feature | 普通特性"控制器告诉主机"；HBS"主机告诉控制器" |

## 进阶细节

- **特性 ID 段**（Figure 194，部分）：
  - `01h` Arbitration
  - `02h` Power Management
  - `03h` LBA Range Type
  - `04h` Temperature Threshold
  - `05h` Error Recovery
  - `06h` Volatile Write Cache
  - `07h` Number of Queues
  - `08h` Interrupt Coalescing
  - `09h` Interrupt Vector Configuration
  - `0Bh` Asynchronous Event Configuration
  - `0Ch` Autonomous Power State Transition
  - `0Dh` Host Memory Buffer
  - `0Eh` Timestamp
  - `0Fh` Keep Alive Timer
  - `10h` Host Controlled Thermal Management
  - `11h` Non-Operational Power State Config
  - `12h` Read Recovery Level Config
  - `13h/14h` Predictable Latency Mode
  - `16h` Host Behavior Support
  - `17h` Sanitize Config
  - `18h` Endurance Group Event Configuration
  - `19h` I/O Command Set Profile
  - `1Ah` Spinup Control
  - `1Bh` Power Loss Signaling Config
  - `1Dh/1Eh` Flexible Data Placement
  - `1Fh` Namespace Admin Label
  - `21h` Controller Data Queue
  - `78h` Embedded Management Controller Address
  - `79h` Host Management Agent Address
  - `7Dh/7Eh/7Fh` Enhanced Controller / Controller / Namespace Metadata
  - `80h` Software Progress Marker
  - `81h` Host Identifier
  - `82h` Reservation Notification Mask
  - `83h` Reservation Persistence
  - `84h` Namespace Write Protection Config
  - `85h` Boot Partition Write Protection Config
- **重置作用**（规范 4.4）：Controller Level Reset 时控制器作用域特性的 current 值恢复按 Figure 124 规则；其他作用域"宽于控制器的"通常不受单控制器重置影响。
- **不可保存但持久**（Figure 385）：某些 Feature 即使 SVBL=0，current 仍跨掉电/重置保留，由特性自身定义。
- **Capabilities 字段限制**：CHANG/NSSPEC/SVBL 三位；bit 3-31 保留。
- **Set Features 命令字段**（规范 5.1.25）：
  - CDW10：FID[7:0] + SV[31]
  - CDW11：Feature-specific
  - CDW14：UIDX[6:0]（Vendor Feature 用）
  - DPTR：可选数据缓冲（按特性是否需要）
- **错误码**（常见）：
  - `Invalid Field in Command`：未知 FID / 字段非法
  - `Feature Not Changeable`：特性不可改
  - `Feature Not Namespace Specific`：用 NSID Set 控制器作用域特性
  - `Feature Not Saveable`：对不可保存特性设 Save=1
  - `Invalid Namespace or Format`：NS 作用域 Get FFFFFFFFh
- **Host Behavior Support `16h`** 行为方向反转：主机声明 `ACRE`（高级命令重试）、Telemetry Data Area 4、LBA 格式扩展等能力；不声明控制器不得使用依赖能力。
- **执行顺序**（规范 5.1.25）：Set 完成只保证"后续命令用新值"；正在执行的命令可能仍用旧值。需清边界先排空。

## 规范依据

- [Feature 值与状态机 Figure 124-125，PDF 第 181-182 页](../_source/pages/page-181.md)
- [作用域与 NSID 选择规则，PDF 第 182-183 页](../_source/pages/page-182.md)
- [Get Features 命令与 SEL 字段，PDF 第 220-223 页](../_source/pages/page-220.md)
- [Capabilities 结果 Figure 195，PDF 第 223 页](../_source/pages/page-223.md)
- [Set Features 命令包络与 Feature 目录，PDF 第 392-395 页](../_source/pages/page-392.md)
- [Host Behavior Support Feature 16h，PDF 第 407-409 页](../_source/pages/page-407.md)

## 相关阅读

- [通用控制器特性](common-controller-features.md) - 通用 FID 列表详解
- [识别命令模型](identify-command-model.md) - Get Features Capabilities 返回能力
- [命令与功能锁定](command-and-feature-lockdown.md) - 特性锁定场景
- [主机内存缓冲区](host-memory-buffer.md) - Set Features 启用 HMB
