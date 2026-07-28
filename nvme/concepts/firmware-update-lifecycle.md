# 固件更新生命周期（Firmware Update Lifecycle）

## 一句话说明

NVMe 固件更新是一个**下载 + 提交 + 激活**的序列化序列，由一个控制器或管理端点（Management Endpoint）主导；激活在 Domain（域）范围内生效，可以是"立即生效"或"等下一次重置"，重置路径会触发控制器的重新初始化和 I/O 队列的重新分配。

## 生活化类比

把 NVMe Domain 想成**公交车的车队**：

- **Firmware Image Download** = 把新固件"分批"塞进每辆车的"维修槽"——可分多次、带不同偏移
- **Firmware Commit** = 班长把"已塞好的新固件"贴上封条、放入"待启用仓"
- **Commit Action** = 班长按"激活按钮"：
  - `000b` = 只贴封条，**不**激活
  - `001b` = 贴封条 + 下次车辆打火（CLR）时换装
  - `010b` = 已有待启用固件，下次打火换装
  - `011b` = 现在就换装（立即激活，命令挂起到换装完）
  - `110b` = 换 Boot Partition 的内容
  - `111b` = 标记某个 Boot Partition 为活跃
- **同一车队（Domain）的所有车辆** = 共享固件槽位，固件激活影响整个 Domain
- **立即激活超时** → 班长改口"打火时再换"（`010b` 救场）

## 工作流程

```text
   [下载连续的固件分片] ---> [提交 + 校验] ---> 选择激活方式
                                                     |
                                  +------------------+------------------+
                                  |                                     |
                                  v                                     v
                       立即激活 (Commit Action 011b)             重置激活 (001b/010b)
                                  |                                     |
                                  v                                     v
                       Firmware Activation                  Controller Level Reset
                       Starting AEN                        / NVM Subsystem Reset
                                  |                       / Conventional Reset
                                  v                                     v
                            [前台挂起的 Commit]               [控制器重新初始化 +
                                                            重新分配 I/O 队列]
```

**端到端流程**：

1. 主机对**单个**控制器（或单一 Management Endpoint）发起 `Firmware Image Download`，携带 0 基 dword offset 与 dword count；普通固件可乱序到达，Boot Partition 片段除外；每次必须从 offset=0 起且遵循 `FWUG` 粒度。
2. 镜像下载完成后发 `Firmware Commit`，选 Commit Action。
3. 重置激活：主机做所需级别重置（Controller Level / NVM Subsystem / Conventional）；重置后固件生效，主机重分配 I/O 队列。
4. 立即激活：Action `011b` 让命令保持 outstanding 直到激活成功或失败；超 `MTFA` → `Firmware Activation Requires Maximum Time Violation`，镜像仍 commit，可改用 `010b` 在 CLR 时再激活。
5. 主机读 `Firmware Slot Information` 日志（LID `1Bh`）观察当前活跃槽与下次重置待激活槽。

## 初学者案例

**场景：怎么给一块 NVMe SSD 升级固件，且不掉电？**

1. 厂商发布新固件，工程师下载到主机。
2. 工程师执行 `nvme fw-download /dev/nvme0n1 -f new_fw.bin -x 4096`，工具按 offset 切片下发。
3. 下载完成后 `nvme fw-commit /dev/nvme0n1 -a 3 -s 2`（Action `011b`=立即激活，Slot 2）。
4. 控制器在 CQE 返回前可能发 `Firmware Activation Starting` AEN；commit 命令保持 outstanding。
5. 几秒后 commit CQE 返回，固件生效。
6. 若失败且返回 `Firmware Activation Requires Controller Level Reset`，工具改为 `nvme fw-commit /dev/nvme0n1 -a 2 -s 2`（Action `010b`）。
7. 重置后主机 `nvme reset` + 重分配队列，I/O 恢复。

> 反例：`dd` 给 PCIe BAR 直接写？不行——镜像必须按 `FWUG` 粒度，Boot Partition 片段必须顺序到达。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 序列串行 | 同一次固件/启动分区更新的所有命令必须在**同一**控制器或 Management Endpoint 上；**不能重叠**两次更新 |
| 镜像必须从 offset=0 起 | 不从 0 起 = 镜像无效；中间留 gap = 无效；片段重叠 = 无效 |
| `FWUG` 粒度 | 数据大小与 offset 应满足 `FWUG`，否则可能更新失败 |
| 镜像可乱序（除 Boot Partition） | 普通固件片段可乱序；Boot Partition 片段必须**顺序**到达 |
| Commit 可能做额外校验 | Controller 可用 checksum/CRC/哈希/数字签名二次校验 |
| Slot `0h` 是"由控制器选" | Slot 字段 `0h` 表示控制器在 1..7 中选一个；不要传 `0h` 来"激活 slot 0" |
| 同 Domain 共享 slot | Domain 内所有控制器共享同一组固件槽位与同一份固件镜像 |
| 域是激活范围 | 固件激活影响**整个 Domain**，不是单个控制器 |
| 立即激活挂起 | Action `011b` 让 commit CQE 一直 outstanding 直到激活成功或失败 |
| `MTFA` 超时 | 超 `MTFA` → `Firmware Activation Requires Maximum Time Violation`；固件已 commit |
| 加载失败回退 | 回退到"最近成功激活 slot"或"baseline 只读镜像"，并以 `Firmware Image Load Error` AEN 报告 |
| Sanitize 中激活失败 | 控制器在 Sanitize 期间变更固件会**导致该 Sanitize 失败** |
| 待激活阻断 Sanitize | 存在"待激活且需重置"的固件时，后续 Sanitize 命令被 abort |
| D3 cold 期间可能回滚 | 立即激活未完成时进 D3 cold，重启后可能是旧固件或新固件 |
| Overlapping sequence 报标志 | 重叠的更新序列即便 Firmware Commit 被 abort 也会在 CQE Dword 0 标记 |
| 重置激活失败保留运行 | 失败时**当前镜像继续运行**——不是被破坏 |
| Host-incompatible 必须 Conventional Reset | 立即激活若会引入主机不兼容行为，控制器必须返回 `Firmware Activation Requires Conventional Reset` |
| Commit Action 编码 | `000b`=只 commit 不激活；`001b`=commit + 下次 CLR 激活；`010b`=下次 CLR 激活已 commit 的；`011b`=立即激活（前台挂起）；`110b`=替换 Boot Partition；`111b`=标记 Boot Partition 活跃 |
| Slot 数量 = 7 | Firmware Slot Information log 报告 slot 1..7；空槽 revision 字段全 0 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| `001b` vs `010b` | `001b`=先 commit + 下次 CLR 激活；`010b`=**已 commit 过**、下次 CLR 激活它 |
| `011b` 立即激活 vs `010b` 重置 | `011b` 是前台挂起的 commit；`010b` 是 commit 之后的下次重置 |
| CLR vs Conventional vs NVM Subsystem Reset | CLR 单控制器；Conventional 走 PCIe；NVM Subsystem Reset 影响整个子系统 |
| Slot 字段 `0h` vs 其他槽 | `0h`=控制器自选；非 `0h`=指定 slot |
| Domain 激活 vs Controller 提交 | 命令从一个 Controller 提交；激活影响整个 Domain |
| "待激活" vs "已 commit" | `000b` 后只是 commit；激活要再走 `001b`/`010b`/`011b` |
| 镜像失效条件 | 失败**仅**在加载阶段；commit 成功镜像即使未激活也保留 |
| 立即激活的"挂起"语义 | commit CQE 一直 outstanding 时命令未完成 |
| `MTFA` 超时后该怎么办 | 不是必须放弃；可改用 `010b` 在 CLR 时再激活 |
| AEN Firmware Activation Starting | 控制器**可以**发（不是必须）；要求 `Firmware Activation Notices` 启用 |
| Boot Partition 与普通固件 | Boot Partition 片段必须顺序；普通固件可乱序 |
| 写入活跃 slot 的副作用 | 可能让"旧激活镜像"丢失，导致回退不到旧固件 |
| 一致性范围 | Domain 内所有控制器**同**时共享激活 |

## 进阶细节

- **Firmware Update Process（规范 3.11）**：
  - *重置路径*：Download（offset 显式）→ Commit → 主机执行所需级别重置（CLR / NVM Subsystem Reset / Conventional Reset）→ 主机重初始化控制器并重分配 I/O 队列
  - *立即路径*：Download → Commit（Action `011b`，命令 outstanding）→ 控制器内部激活（可能发 AEN）→ CQE 返回
- **Commit Action（Figure 181）**：`000b`=放入 slot 不激活；`001b`=放入 slot + 下次 CLR 激活；`010b`=激活**已 commit 的** slot 于下次 CLR；`011b`=commit 同一 slot 并立即激活（挂起到完成）；`110b`=替换 Boot Partition；`111b`=标记 Boot Partition 活跃
- **失败状态码（3.11）**：`Invalid Firmware Image` / `Firmware Activation Requires Controller Level Reset` / `Firmware Activation Requires NVM Subsystem Reset` / `Firmware Activation Requires Conventional Reset` / `Firmware Activation Requires Maximum Time Violation`
- **Firmware Slot Information Log（Figure 208, LID `1Bh`）**：当前活跃 slot + 可选的下次重置待激活 slot + 7 个 slot 各 8 字节 ASCII revision
- **D3 cold 与回滚（3.11）**：立即激活后进 D3 cold，重启后可能恢复旧或新固件（控制器决定）
- **Overlapping update 报告（5.1.8.1）**：即便 Commit 被 abort，CQE Dword 0 仍标记重叠更新序列
- **"较窄的 reset 优先"（5.1.8.1）**：Commit 返回"需 Conventional Reset"而主机只做 CLR → 固件不激活
- **Boot Partition 写锁（5.1.8.1）**：写锁状态下 `110b`/`111b` 被 abort
- **MTFA 字段**：Identify Controller 报告（毫秒）；超时后镜像仍保留
- **Domain 共享副作用**：一控制器激活成功 → 整个 Domain 都获新固件；老镜像可回退（若仍在 slot）

## 规范依据

- [重置激活的更新流程与命令序列，PDF 第 152 页](../_source/pages/page-152.md)
- [立即激活、AEN 与失败状态码，PDF 第 153 页](../_source/pages/page-153.md)
- [序列串行化与未 commit 时的丢弃规则，PDF 第 154 页](../_source/pages/page-154.md)
- [Firmware Commit 命令字段与 Commit Action（Figure 181），PDF 第 214 页](../_source/pages/page-214.md)
- [Firmware Image Download 命令字段与状态（Figures 184-187），PDF 第 217 页](../_source/pages/page-217.md)
- [Firmware Slot Information Log（Figure 208），PDF 第 235 页](../_source/pages/page-235.md)

## 相关阅读

- [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) - 重置激活路径与 CLR 触发
- [format-nvm-lifecycle.md](format-nvm-lifecycle.md) - Sanitize 与固件激活的冲突
- [domains-and-divisions.md](domains-and-divisions.md) - Domain 内共享固件槽与激活范围
- [asynchronous-event-reporting.md](asynchronous-event-reporting.md) - Firmware Activation Starting AEN
