# 固件更新生命周期（Firmware Update Lifecycle）

NVMe 固件更新是一套由**单个控制器或管理端点（Management Endpoint）**主导的串行化"**下载（Download）→ 提交（Commit）→ 激活（Activation）**"流程。固件激活的作用范围是整个 **Domain（域）**（域内所有控制器共享固件槽位与同一份镜像），支持"立即激活"或"在指定重置边界激活"两种路径。[PDF pp. 152-154](../_source/pages/page-152.md)

可以把 Domain 想象成**一支公交车队**：`Firmware Image Download` 是把新固件分批搬进车队的维修槽；`Firmware Commit` 是班长贴上封条并指定启用方式（仅存不启、下次打火启用、或立刻热切换）；一旦车队启用新固件，所有车辆（同 Domain 控制器）同步切到同一版本。

## 端到端流程与激活路径

```text
[分片下载 Firmware Image Download] ---> [提交与校验 Firmware Commit] ---> 选择激活方式
                                                                              |
                                                           +------------------+------------------+
                                                           |                                     |
                                                           v                                     v
                                                立即激活 (Action 011b)                 重置激活 (001b / 010b)
                                                           |                                     |
                                                           v                                     v
                                                可能发出 Firmware Activation           Controller Level Reset /
                                                Starting 异步通知 (AEN)                NVM Subsystem Reset /
                                                           |                           Conventional Reset
                                                           v                                     |
                                                Commit 保持 outstanding 直到完成                  v
                                                                                       [重新初始化控制器并重分配 I/O 队列]
```

1. **下载分片（Firmware Image Download）**：每次携带 0 基的 dword 偏移和 dword 计数。除 Boot Partition（启动分区）分片必须按顺序到达外，普通固件分片可乱序到达；但完整镜像必须从偏移 0 开始、无空洞、无重叠，且对齐/大小符合 `FWUG`（Firmware Update Granularity，固件更新粒度）。下载本身不会激活镜像。[PDF pp. 152, 216-217](../_source/pages/page-152.md)
2. **提交（Firmware Commit）**：将已下载镜像放入指定槽位（1–7；槽位字段为 `0h` 时由控制器自选），并可执行额外校验（如签名/CRC）。一次 Commit 完成后再发起新下载会丢弃旧分片；Commit 完成前若发生重置，已下载分片全部丢弃。[PDF pp. 152, 154, 214-215](../_source/pages/page-154.md)
3. **查询槽位（Firmware Slot Information Log, LID `03h`）**：报告当前运行槽位（`CAFS`）、下一次 Controller Level Reset 待激活槽位（`NAFS`，可选）以及 7 个槽位各自的 8 字节 ASCII 版本号（空槽或不支持的槽位全 0）。[PDF p. 235](../_source/pages/page-235.md)

| Commit Action（Figure 181） | 命令级效果 |
|---|---|
| `000b` | 将已下载镜像放入指定槽位，**不**激活 |
| `001b` | 放入槽位，并在下一次 Controller Level Reset（CLR）时激活 |
| `010b` | 激活一个**已存在（已 commit）**的槽位，在下一次 CLR 时生效 |
| `011b` | 若已下载则放入槽位，并**立即激活**；命令在激活完成前保持 outstanding（未完成）状态 |
| `110b` | 替换所选的 Boot Partition |
| `111b` | 将所选的 Boot Partition 标记为活跃 |

**常用 CLI 示例**：
```bash
nvme fw-download /dev/nvme0n1 -f new_fw.bin -x 4096   # 分片下载固件
nvme fw-commit /dev/nvme0n1 -a 3 -s 2                 # Action 011b 立即激活 Slot 2
# 若返回需 CLR 或超时：nvme fw-commit /dev/nvme0n1 -a 2 -s 2 (Action 010b) 后执行 reset
```

## 必须遵守的规则与边界情况

- **串行与不重叠**：同一次固件或 Boot Partition 更新序列的所有命令必须发往同一控制器或管理端点，禁止两次更新序列重叠。即使 Firmware Commit 被中止，检测到的重叠更新序列也会在 CQE Dword 0 中标记。[PDF pp. 154, 214-215](../_source/pages/page-154.md)
- **激活超时（`MTFA`）与回退**：若立即激活耗时超过 Identify Controller 报告的 `MTFA`（Max Time for Firmware Activation），控制器返回 `Firmware Activation Requires Maximum Time Violation`，但镜像仍保持已 commit 状态，主机可改用 Action `010b` 在 CLR 时激活。若镜像加载失败，控制器回退到最近一次成功激活的槽位或只读基准镜像，并上报 `Firmware Image Load Error`。[PDF p. 153](../_source/pages/page-153.md)
- **重置类型匹配**：若立即激活会引入主机不兼容行为，控制器必须返回 `Firmware Activation Requires Conventional Reset`。当状态码要求"更宽的重置"（如 Conventional Reset）而实际只执行了"更窄的重置"（如 CLR）时，当前镜像继续运行，不会被替换。[PDF pp. 214-216](../_source/pages/page-214.md)
- **与 Sanitize 的冲突**：在 Sanitize（数据擦除）期间变更固件会导致 Sanitize 失败；而当存在"待激活的重置固件"时，后续 Sanitize 命令会被中止。[PDF p. 153](../_source/pages/page-153.md)
- **D3 cold 掉电风险**：立即激活仍在进行时若系统进入 D3 cold（深度断电休眠），重新上电后可能运行旧镜像也可能运行新镜像。Boot Partition 处于写锁状态时，`110b`/`111b` 会被中止。[PDF pp. 153, 216](../_source/pages/page-153.md)

## 规范依据

- [基于重置的更新流程、立即激活与序列串行化，PDF pp. 152-154](../_source/pages/page-152.md)
- [Firmware Commit 字段、重叠标志与状态码（Figure 181-183），PDF pp. 214-216](../_source/pages/page-214.md)
- [Firmware Image Download 字段与状态（Figure 184-187），PDF pp. 216-217](../_source/pages/page-216.md)
- [Firmware Slot Information 日志（LID 03h, Figure 208），PDF p. 235](../_source/pages/page-235.md)

## 关联与引申概念

- **Controller Level Reset（CLR）**：单控制器级的重置（如 `CC.EN` 清零、FLR），重置激活路径生效后主机需重新初始化控制器并重建 I/O 队列。详见 [控制器启用、关闭与重置](controller-enable-shutdown-reset.md)。
- **Domain（域）**：NVMe 资源隔离与共享的顶层单元，固件槽位与固件激活均为 Domain 级作用域。详见 [域与划分](domains-and-divisions.md)。
- **AEN（Asynchronous Event Notification，异步事件通知）**：控制器在立即激活前可通过 `Firmware Activation Starting` AEN 提前通知主机。详见 [异步事件上报](asynchronous-event-reporting.md)。
- **Sanitize vs Format NVM**：Sanitize 是子系统级彻底数据擦除操作，与固件激活状态互斥。详见 [Format NVM 生命周期](format-nvm-lifecycle.md)。
