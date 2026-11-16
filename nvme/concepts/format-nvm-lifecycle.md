# Format NVM 生命周期（Format NVM Lifecycle）

**Format NVM** 是 NVMe 的**命名空间（Namespace）级**格式化命令，用于更改存储介质的 LBA（逻辑块地址）格式（如 512 字节与 4 KiB 切换、保护信息 PI 与元数据设置），并可选择性地执行用户数据擦除或加密擦除。与后台异步执行的子系统级操作 **Sanitize** 不同，Format NVM 在命令完成返回时即生效。[PDF pp. 217-220](../_source/pages/page-217.md)

可以把命名空间想象成**一间仓库**：`Format NVM` 是重新规划这间仓库的货架格子尺寸（LBA Format）；`User Data Erase` 是把格子里的货全部搬空；`Cryptographic Erase` 则是直接销毁门锁密钥让旧货无法被解密读取；而 `Sanitize` 则是对整栋大楼（整个 NVM 子系统）发起的后台彻底清场。

## 操作流程与作用域矩阵

```text
                    SES = 000b (无擦除)           SES ≠ 000b (安全擦除)
                           |                                |
                           v                                v
                   FNS 控制格式作用域               SENS 控制擦除作用域
                           |                                |
                           +---------------+----------------+
                                           |
                                           v
                     NSID 选择：单 NS / 本控制器已挂载 NS / 子系统所有 NS
                                           |
                                           v
                        [格式化介质 + 可选安全擦除] (命令完成即生效)
```

Identify Controller 的 `FNA` 字段定义了 `FNVMBS`、`FNS` 与 `SENS` 三个能力位，与命令中的 `NSID` 共同决定受影响的范围（Figure 188）：[PDF pp. 218-219](../_source/pages/page-218.md)

| `SES`（安全擦除设置） | 适用作用域位 | `NSID` 参数 | 受影响的命名空间范围 |
|---|---|---|---|
| `000b`（无擦除） | `FNS=0` | 已分配的 NSID | 仅指定的单个命名空间 |
| `000b`（无擦除） | `FNS=0` | `FFFFFFFFh`（需 `FNVMBS=0`） | 当前控制器已挂载的所有命名空间 |
| `000b`（无擦除） | `FNS=1` | 已分配的 NSID 或 `FFFFFFFFh` | 整个 NVM 子系统中的所有命名空间 |
| `001b`（用户数据擦除）或 `010b`（加密擦除） | `SENS=0` | 已分配的 NSID / `FFFFFFFFh` | 仅单个 NS / 当前控制器已挂载的所有 NS（带擦除） |
| `001b` 或 `010b` | `SENS=1` | 已分配的 NSID 或 `FFFFFFFFh` | 整个 NVM 子系统中的所有命名空间（带擦除） |

- **广播限制（`FNVMBS=1`）**：若控制器报告 `FNVMBS=1`，则禁止使用广播 `NSID=FFFFFFFFh`，否则返回 `Invalid Field in Command`。
- **空作用域处理**：当合法的广播作用域内没有任何命名空间时，命令仍会成功完成（不报错）。

## 安全擦除与 LBA 格式字段

| `SES` 值 | 擦除方式 | 关键规则 |
|---|---|---|
| `000b` | 无安全擦除 | 仅切换 LBA 格式，不承诺擦除旧数据 |
| `001b` | **User Data Erase（用户数据擦除）** | 彻底擦除用户数据；若所有受影响数据均已加密，控制器**允许**通过加密擦除实现 |
| `010b` | **Cryptographic Erase（加密擦除）** | 必须通过删除/更换内部加密密钥使原密文无法解密，通常比物理块擦除快得多 |

Format NVM 的格式索引由 CDW10 的 `LBAFU`（bits 13:12，高 2 位）与 `LBAFL`（bits 7:0，低 8 位）组合而成；当 Host Behavior Support（FID `16h`）未启用 `LBAFEE`（LBA Format Extension Enable）时，`LBAFU` 会被忽略。成功完成后，新格式由 **Identify Namespace** 数据结构（如 `LBAF`、保护信息 `PI`、元数据设置 `MSET`）反映。[PDF pp. 219-220](../_source/pages/page-219.md)

**常用 CLI 示例**（将 NS 1 切换为 LBA Format 0 且不擦除 / 执行加密擦除）：
```bash
nvme format /dev/nvme0n1 -n 1 -l 0 -s 0   # 仅切换格式 (SES=0)
nvme format /dev/nvme0n1 -n 1 -s 2        # 格式化并执行 Cryptographic Erase (SES=2)
```

## 并发控制、错误处理与 Format vs Sanitize

- **I/O 冲突时序**：
  - 下发 Format NVM 时若目标命名空间仍有正在进行的 I/O，Format NVM 可能被中止并返回 `Command Sequence Error`。
  - Format NVM 执行期间，发往受影响命名空间的新 I/O 可能返回 `Format in Progress`。
  - 格式化期间，控制器允许执行的 Admin 命令白名单会临时缩减。[PDF pp. 192, 218-219](../_source/pages/page-192.md)
- **写保护与多域访问**：若作用域内任一命名空间处于写保护状态，命令返回 `Namespace Is Write Protected`；在多域子系统中若控制器无法访问目标命名空间，返回 `Asymmetric Access Inaccessible` 或 `Asymmetric Access Persistent Loss`。[PDF pp. 218-219](../_source/pages/page-218.md)
- **常见参数错误**：不支持的 LBA Format 索引返回 `Invalid Format`；禁用或非法的字段（如 `FNVMBS=1` 时使用广播 NSID）返回 `Invalid Field in Command`。[PDF pp. 219-220](../_source/pages/page-219.md)

| 对比维度 | Format NVM | Sanitize |
|---|---|---|
| **默认作用域** | 命名空间级（受 `FNS`/`SENS` 与 `NSID` 控制） | 整个 NVM 子系统（包含缓存与未分配区域） |
| **执行方式** | 命令完成时即生效（同步/近同步） | 后台异步执行，独立状态机 |
| **进度监控** | 可选由 Identify Namespace 的 `FPI` 字段报告 | 专用的 Sanitize Status 日志（LID `81h`） |

## 规范依据

- [Format 操作边界与固件下载完成关系，PDF p. 217](../_source/pages/page-217.md)
- [Figure 188：Format 与安全擦除作用域矩阵，PDF p. 218](../_source/pages/page-218.md)
- [Figure 189–190：作用域边界、LBAFEE 门控与状态描述，PDF pp. 219-220](../_source/pages/page-219.md)
- [Format 期间 Admin 命令白名单变化，PDF p. 192](../_source/pages/page-192.md)

## 关联与引申概念

- **Sanitize**：子系统级不可中断的后台数据清理操作，与待生效的固件更新互斥。详见 [Sanitize 操作状态](sanitize-operation-status.md) 与 [固件更新生命周期](firmware-update-lifecycle.md)。
- **ANA（Asymmetric Namespace Access，非对称命名空间访问）**：多域/多路径子系统中决定控制器对命名空间可达性的状态机制。详见 [命名空间访问模型](namespace-access-models.md)。
- **Admin 命令模型与效果**：Format NVM 在命令支持与效果日志中具有专门的效果描述符。详见 [Admin 命令模型](admin-command-model.md) 与 [命令效果与支持](command-effects-and-support.md)。
