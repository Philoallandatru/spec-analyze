# Format NVM 生命周期（Format NVM Lifecycle）

## 一句话说明

Format NVM 是 NVMe 的**命名空间级**格式化命令：它修改介质的 LBA 格式（Format Index）并可选择性地执行用户数据擦除（User Data Erase）或加密擦除（Cryptographic Erase）；它**不是**后台异步操作，命令完成时格式化与擦除（在控制器能力范围内）都已生效。

## 生活化类比

把命名空间想成**一个仓库的标准货架**：

- **Format NVM** = "把货架按新规格重新画线"
- **LBA Format** = 货架上每个格子的尺寸（512 字节、4 KiB、…）
- **User Data Erase** = 把格子里的货全清掉
- **Cryptographic Erase** = 锁换新的——格子里的货看着还在，但密码变了打不开
- **FNS / SENS 作用域** = "只画这间办公室"还是"画所有办公室"
- **NSID = FFFFFFFFh** = "广播：一键全清"——但老板可能禁用（`FNVMBS=1`）

> 与 Sanitize 的关键区别：Format NVM 是**单间办公室**的操作，Sanitize 是**整栋楼**的后台操作。

## 工作流程

```text
                  SES = 000b (无擦除)            SES ≠ 000b (安全擦除)
                       |                                 |
                       v                                 v
                FNS 控制格式作用域                SENS 控制擦除作用域
                       |                                 |
                       +--------------+------------------+
                                      |
                                      v
                       NSID 选择：单 NS / 控制器已挂载集合 / 子系统集合
                                      |
                                      v
                       [格式化介质 + 可选擦除] -- 同步/近同步完成
                                      |
                                      v
                                  操作完成
```

**端到端流程**：

1. 主机发 Format NVM 命令；`NSID` 选择目标（单 NS / `FFFFFFFFh` 广播）。
2. `SES` 字段决定"是否擦除"以及"擦除类型"；`FNS`/`SENS` 选择"作用域"。
3. 控制器检查作用域是否合法（多域、PMR、写保护、广播禁用等）。
4. 控制器执行格式化；若 `SES ≠ 000b` 且能力允许，执行擦除。
5. CQE 返回成功；新格式由 Identify Namespace 报告。
6. 期间发到受影响命名空间的新 I/O 可能返回 `Format in Progress`。

## 初学者案例

**场景：新装的 SSD 之前用 4 KiB 物理扇区，现在想换成 512 字节模拟怎么办？**

1. 工程师确认该 NS 上没有正在进行的 I/O（否则命令会 `Command Sequence Error`）。
2. 工具用 `nvme format /dev/nvme0n1 -n 1 -l 0 -s 0`：
   - `-n 1` = NSID 1
   - `-l 0` = LBA Format Index 0（512 字节）
   - `-s 0` = SES 0（无擦除）
3. 控制器在命令返回时已经把 LBA 切到 512 字节。
4. 工具用 `nvme id-ctrl /dev/nvme0n1` 看 `LBAFD0` 字段确认是 512。
5. 上层重新 `mkfs`，I/O 恢复正常。

> 注意：若想"格式化的同时把数据彻底擦掉"，用 `nvme format /dev/nvme0n1 -n 1 -s 2`（`SES=2` = Cryptographic Erase）；加密擦除通常比块擦除快得多。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 范围是命名空间级 | Format NVM 影响**所选 NS**；不存在"只擦一个 Endurance Group"的格式 |
| `SES=000b` 看 `FNS` | 不擦除时由 `FNS` 决定格式作用域；`SENS` 字段不适用 |
| `SES≠000b` 看 `SENS` | 擦除时由 `SENS` 决定擦除作用域；`FNS` 不适用 |
| `NSID=FFFFFFFFh` 广播 | 当控制器能力支持（`FNVMBS=0`）时，广播作用域内的所有 NS 都被格式化 |
| `FNVMBS=1` 禁广播 | 若控制器报告 `FNVMBS=1`，`NSID=FFFFFFFFh` 会被 `Invalid Field in Command` 拒绝 |
| 空作用域算成功 | 广播作用域内**无** NS 时，命令仍成功完成（不报错） |
| 多域要谨慎 | 多域子系统分割时，若控制器无法访问目标 NS，命令返回 `Asymmetric Access Inaccessible` 或 `Asymmetric Access Persistent Loss` |
| 写保护阻断 | 任何受影响 NS 处于写保护 → `Namespace Is Write Protected` |
| 冲突 I/O 阻断 | 目标 NS 上有进行中 I/O → 命令返回 `Command Sequence Error` |
| 期间新 I/O | 格式化期间，新到受影响 NS 的 I/O 可能返回 `Format in Progress` |
| 期间 Admin 白名单缩减 | 格式化期间允许的 Admin 命令集合被临时缩小（参考 5.1.x） |
| `LBAFU` 受 `LBAFEE` 门控 | 主机未启用 `LBAFEE`（Host Behavior Support）时，`LBAFU` 被忽略 |
| `SES=001b` vs `010b` | `001b`=User Data Erase（控制器在所有数据已加密时可走加密擦除路径）；`010b`=Cryptographic Erase（必须用密钥擦除） |
| 完成后 Identify 反映 | 新格式由 Identify Namespace 数据结构（PI/MSET/LBAF）报告 |
| `LBAFU` 编码 | bits `13:12` 是 LBA Format Upper，与 LBAFL 共同组成 Format Index |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Format NVM vs Sanitize | Format = 命名空间级，同步；Sanitize = 子系统级，后台异步 |
| Secure Erase vs Sanitize | "Secure Erase" 是 Format 的一个**选项**；"Sanitize" 是独立命令 |
| `FNS` vs `SENS` | `FNS` 是格式作用域（无擦除时）；`SENS` 是擦除作用域（有擦除时） |
| `FNS=0` vs `FNS=1` | `FNS=0` 时 `NSID=FFFFFFFFh` 仅覆盖**本控制器**已挂载 NS；`FNS=1` 时覆盖**整个子系统**所有 NS |
| `SES=001b` User Data Erase | 控制器可"借加密擦除"完成；但规范上仍归类为"User Data Erase" |
| `SES=010b` Cryptographic Erase | 必须通过删除加密密钥完成；不能只是覆盖 |
| 广播 vs 单 NS | 广播**只能**用 `FFFFFFFFh`；传其他值是单 NS，不要混淆 |
| `FNVMBS=1` 含义 | 控制器"禁止广播"——并不代表控制器不支持 Format NVM 本身 |
| `Format in Progress` vs `Command Sequence Error` | 前者：I/O 在格式化**进行中**到达；后者：Format 命令发现**已有 I/O** 冲突 |
| `Invalid Format` vs `Invalid Field` | `Invalid Format`=LBA Format Index 不支持；`Invalid Field`=命令字段非法（如广播禁用） |
| `LBAFU` vs `LBAFL` | `LBAFU` 是 Format Index 的高 2 位（CDW10 bits 13:12）；`LBAFL` 是低 8 位（CDW10 bits 7:0） |
| 格式化粒度 | 命名空间级 → 可能影响所有挂载它的控制器；但不会改 NVM Set 边界 |

## 进阶细节

- **Figure 188 作用域矩阵**（5.1.x）：
  - `SES=000b` + `FNS=0` + `NSID=FFFFFFFFh`（仅当 `FNVMBS=0`）→ 本控制器已挂载的所有 NS
  - `SES=000b` + `FNS=0` + 其他 NSID → 指定单 NS
  - `SES=000b` + `FNS=1` + 任意 NSID 或 `FFFFFFFFh` → 子系统所有 NS
  - `SES=001b/010b` + `SENS=0` + `NSID=FFFFFFFFh`（仅当 `FNVMBS=0`）→ 本控制器已挂载的所有 NS（带擦除）
  - `SES=001b/010b` + `SENS=0` + 其他 NSID → 指定单 NS（带擦除）
  - `SES=001b/010b` + `SENS=1` + 任意 NSID 或 `FFFFFFFFh` → 子系统所有 NS（带擦除）
  - 其他组合 → `Invalid Field in Command`
- **`FNVMBS` / `FNS` / `SENS` 来源**（Figure 312）：均在 Identify Controller 的 `FNA` 字段。
- **Format NVM CDW10（Figure 189）**：Bits `13:12` = `LBAFU`（2 位）；Bits `07:00` = `LBAFL`（8 位；与 `LBAFU` 共同组成 Format Index）；Bit `09` = `SEST`；其余位是 MSET/PI 等（依 I/O 命令集而定）。
- **`LBAFEE` 字段**（Host Behavior Support, 5.1.25.1.14）：主机未启用 LBA Format Extension 时 `LBAFU` 被忽略。
- **LBA Format Index 选取**：Format Index 决定 `LBAFDx`（LBA Format Data Size）、`MSET`/`PI`；具体支持的 Format 由 Identify Namespace 报告。
- **空作用域处理**（4 种成功组合）：`NSID=FFFFFFFFh` + 任意 `SES`/`FNS`/`SENS` 组合 + 作用域内无 NS → 命令成功完成。
- **写保护交互**：作用域含任何写保护 NS（8.1.16）→ abort 为 `Namespace Is Write Protected`。
- **多域交互**：多域子系统分割（3.2.5）+ 控制器不能访问目标 NS → abort 为 `Asymmetric Access Inaccessible` 或 `Asymmetric Access Persistent Loss`。
- **I/O 命令交互**：
  - 目标 NS 有 I/O 在执行 → Format NVM 可能被 abort 为 `Command Sequence Error`
  - Format NVM 在执行时 → 新到受影响 NS 的 I/O 可能被 abort 为 `Format in Progress`
- **安全规范交互**（如 TCG）：某些安全状态下可能被 abort。
- **与 Sanitize 的根本区别**：
  - 范围：Format = NS；Sanitize = Subsystem
  - 时序：Format = 同步/近同步；Sanitize = 显式后台异步
  - 监控：Format 无独立进度日志；Sanitize 有 LID `81h`
- **Cryptographic Erase 可逆性**：用 SES `010b` 删除密钥后，**理论**上密钥恢复即可恢复数据（取决于密钥是否在控制器外部保存）；这是与块擦除的关键差异。

## 规范依据

- [Format 操作边界与固件下载完成的关系，PDF 第 217 页](../_source/pages/page-217.md)
- [Figure 188：Format 与安全擦除作用域矩阵，PDF 第 218 页](../_source/pages/page-218.md)
- [Figure 189 起始部分：作用域边界与 LBAFEE 门控，PDF 第 219 页](../_source/pages/page-219.md)
- [Figure 189 续与 Figure 190：状态描述与 SES 编码，PDF 第 220 页](../_source/pages/page-220.md)
- [Admin 命令白名单在 Format 期间的变化，PDF 第 192 页](../_source/pages/page-192.md)

## 相关阅读

- [admin-command-model.md](admin-command-model.md) - Format opcode 在命令表中的位置
- [capacity-management-operations.md](capacity-management-operations.md) - NS 所属 NVM Set 的容量层级
- [firmware-update-lifecycle.md](firmware-update-lifecycle.md) - 固件激活与 Format 冲突
- [command-effects-and-support.md](command-effects-and-support.md) - Format 效果描述符字段
