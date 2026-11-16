# 遥测捕获日志（Telemetry Capture Logs）

遥测捕获日志（Telemetry Capture Logs）是 NVMe 控制器暴露内部运行状态快照的标准"信封"，由 **512 字节标准化头部**和若干 **512 字节数据块**组成（数据内容由厂商自定义）。它分为**主机发起（Host-Initiated，LID `07h`）**和**控制器发起（Controller-Initiated，LID `08h`）**两类，方便现场故障排查与固件诊断。[PDF pp. 241-246](../_source/pages/page-241.md)

可以把遥测日志想成**赛车上的行车记录仪**：
- **LID `07h`（主机发起）** = 车手感觉车况异常时主动按下录制键（`CTHID=1`），立刻拍下当前状态快照
- **LID `08h`（控制器发起）** = 行车记录仪检测到剧烈颠簸或故障时自动触发保存的现场录像
- **512 字节标准化头** = 录像文件的统一封面（包含厂商 OUI、各区域边界、代数编号和触发原因）

---

## 共享数据块模型与嵌套边界

```text
  byte 0                                           byte 511
  +---------------------- 标准化头 (512B) ------------------+
  block 1       block 2                            block n
  +-------------+-------------+ ... +----------------------+
  | 所有 Area 均从 Block 1 起算，逐层包含递增：              |
  | Area 1 ⊂ Area 2 ⊂ Area 3 ⊂ Area 4                      |
  +--------------------------------------------------------+
```

- **逐层包含而非分段**：Area 1 到 Area 4 的边界（`THDA1LB`–`THDA4LB`）表示各自结束的最后块号，**单调不递减**（Area 2 包含 Area 1 的全部内容）。
- **512 字节对齐**：所有读取偏移（Log Page Offset）必须是 **512 字节的整数倍**，否则返回 `Invalid Field in Command`；超出最终 Area 边界的数据规范不做定义。

---

## 主机发起 vs 控制器发起对比

| 对比维度 | 主机发起（LID `07h`） | 控制器发起（LID `08h`） |
|----------|-----------------------|-------------------------|
| **触发方式** | 主机在 `Get Log Page` 中置位 `CTHID=1` 触发捕获（`CTHID=0` 仅读已有快照） | 厂商特定的控制器内部条件自动触发 |
| **范围控制** | 若控制器在 Supported Log Pages 中报告 `MCDAS=1`，主机可用 `MCDA`（`001b`–`100b`）限制最大捕获到 Area 1–4；否则 `MCDA` 被忽略 | 由控制器自行决定捕获范围 |
| **稳定性与跨重置** | 保留至下一次 `CTHID=1` 捕获、**Firmware Commit** 或上电复位（POR） | **Area 1–3 跨所有重置保留**；Area 4 可能跨 Controller Level Reset 保留 |
| **可用性与确认** | 头部镜像了 LID `08h` 的 `TCDA`（可用性）和 `TCDGN`（代数），便于单次读取对齐；无清位规则 | `available=1` 时，需通过 **带内（in-band）`RAE=0`** 成功读取才清零标志；带外管理口读取不影响该标志 |

### 关键字段与 Area 4 协商
- **8 位 Generation Number（`THDGN` / `TCDGN`）**：每次新捕获自增 1，到达 `FFh` 后回绕到 `00h`，用于判断快照是否更新。主机发起日志的 Scope 字段（`THS`）在 NVMe 2.1+ 中为 Controller（`01h`）或 NVM Subsystem（`02h`）。[PDF p. 243](../_source/pages/page-243.md)
- **Area 4 大小协商**：仅当控制器支持 Area 4（`DA4S=1`）**且**主机通过 Host Behavior Support 特性启用了 `ETDAS=1` 时，日志大小才按 Area 4 计算；否则即使控制器内部存在 Area 4，主机可见大小也会按 Area 3 截断，以兼容旧主机驱动。[PDF pp. 242, 244-245](../_source/pages/page-242.md)

**常用命令**：
```bash
# 触发主机发起遥测捕获（CTHID=1）并保存到文件
nvme telemetry-log /dev/nvme0n1 --host-generate=1 --output-file=telemetry.bin
```
> **注意**：下发 `CTHID=1` 会立即覆盖旧的主机发起快照，因此如需保留旧快照，请先用 `CTHID=0` 读出后再触发新捕获。

---

## 关联与引申概念

- **AER（Asynchronous Event Reporting）**：控制器在自动生成 LID `08h` 遥测快照后，可通过异步事件通知主机前来读取，并以 `RAE=0` 的带内读取作为确认清除事件。[PDF pp. 198, 246](../_source/pages/page-198.md)
- **IEEE OUI（Organizationally Unique Identifier）**：遥测头中的 3 字节厂商唯一标识符，解码工具据此匹配对应 SSD 厂商的私有遥测解析器。
- **相关文档**：
  - [日志页获取](log-page-retrieval.md) — `Get Log Page` 的 LID、Offset 与 `RAE` 通用规则
  - [固件更新生命周期](firmware-update-lifecycle.md) — Firmware Commit 会使主机发起遥测快照失效
  - [异步事件报告](asynchronous-event-reporting.md) — 触发读取控制器发起遥测的事件通道
  - [错误与健康日志](error-and-health-logs.md) / [持久事件日志](persistent-event-log.md) — 与遥测互补的诊断日志

## 规范依据

- [主机发起遥测触发（CTHID/MCDA）、保留与 Area 4 协商，PDF pp. 241-242](../_source/pages/page-241.md)
- [主机发起遥测头布局（OUI/Scope/Generation/Reason），PDF pp. 243-244](../_source/pages/page-243.md)
- [控制器发起遥测保留、Availability 与 Generation，PDF pp. 244-246](../_source/pages/page-244.md)
- [AER 触发读取控制器发起遥测，PDF p. 198](../_source/pages/page-198.md)
