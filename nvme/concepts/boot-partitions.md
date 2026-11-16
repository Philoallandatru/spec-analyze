# 启动分区（Boot Partitions）

**启动分区（Boot Partitions）** 是 NVMe 控制器提供的一项可选特性，由**两个大小完全相同的专用分区（分区 0 与分区 1）**组成，用于存放 BIOS/UEFI 启动代码或平台固件。它最大的特点是**无需初始化队列或启用控制器即可通过寄存器读取**，非常适合系统启动极早期阶段。[PDF pp. 88-89](../_source/pages/page-088.md)

> **类比理解**：把启动分区想成汽车仪表盘下的小抽屉，里面放着两张一模一样的应急纸质地图（分区 0 和分区 1 互为冗余）。司机（BIOS/UEFI）刚上车、还没启动整套车机导航（NVMe 队列未初始化）时，就能直接拉开抽屉拿地图上路，抽屉旁边还有个小锁扣（写保护）防止误涂改。

司机不会一上车就把整套导航系统初始化好；他先打开小抽屉拿一张地图就能出发，等导航系统启动后再做精细操作。

## 寄存器读取流程（早期启动路径）

主机无需创建 Admin/I/O 队列，只需通过三步控制器属性寄存器操作即可读取启动分区：[PDF pp. 88-89](../_source/pages/page-088.md)

```text
第 1 步：配置目标主机内存地址
   BPMBL.BMBBA ← 4 KiB 页对齐的主机内存地址（低 12 位保留为 0）

第 2 步：选择分区与读取范围（写 BPRSEL.BPID 会触发读取）
   BPRSEL ← { BPID (0 或 1), BPROF (偏移 × 4 KiB), BPRSZ (大小 × 4 KiB) }
              |
              v
第 3 步：轮询 BPINFO.BRS（Boot Read Status）
   00b → 无活动请求
   01b → 数据传输进行中
   10b → 传输成功完成
   11b → 传输出错（不传输数据，目标缓冲区内容未定义）
```

### 关键寄存器与对齐规则

| 寄存器（偏移） | 关键字段 | 单位与约束 |
|---|---|---|
| **BPINFO** (`40h`) | `ABPID`[31] 活动分区 ID（0/1）；`BRS`[25:24] 读状态；`BPSZ`[14:0] 单分区大小 | `BPSZ` 以 **128 KiB** 为单位；两分区大小永远相同；若不支持启动分区则全 0 |
| **BPRSEL** (`44h`) | `BPID`[31] 目标分区；`BPROF`[29:10] 起始偏移；`BPRSZ`[9:0] 读取大小 | 偏移与大小均以 **4 KiB** 为单位；**偏移 + 大小 <= 分区总大小**，越界则不传输数据且 `BRS = 11b` |
| **BPMBL** (`48h`) | `BMBBA` 目标主机内存基址 | 必须 **4 KiB 页对齐**（低 12 位为 0） |

---

## 日志页读取方式（标准化路径）

在队列已初始化（如 OS 阶段）后，主机还可以通过标准 Admin 命令 `Get Log Page`（`LID = 15h`）只读访问完整启动分区：[PDF pp. 286-287](../_source/pages/page-286.md)

```text
Get Log Page (LID=15h, CDW10.BPID=0/1)
   ↓
+---------------------- 16 字节头部 ----------------------+--------------------+
| LID | BPINFO 镜像（活动分区 ID + 分区大小 128 KiB 单位） | 完整分区数据       |
+--------------------------------------------------------+--------------------+
|<--------------------- 固定头部 ------------------------>|<-- BPSZ × 128 KiB -->|
```

| 对比维度 | 寄存器方式（`BPRSEL`） | 日志页方式（`Get Log Page, LID=15h`） |
|---|---|---|
| **前置条件** | 无需队列或控制器启用（极早期可用） | 需要 Admin 队列已初始化 |
| **读取粒度** | 任意 4 KiB 对齐的偏移与大小 | 仅能读取完整分区 |
| **寄存器副作用** | 会更新 `BPINFO.BRS` | 不修改 `BPINFO`、`BPRSEL` 或 `BPMBL` |

## 规范依据

## 写保护机制与 Feature ID 85h

两个启动分区均可独立配置写保护，上电后默认处于 **Write Locked（写锁定）** 状态，以防止启动代码被意外或恶意篡改：[PDF pp. 426-427](../_source/pages/page-426.md)

```text
系统上电
   ↓
[Write Locked] <--- Set Features (85h) ---> [Write Unlocked]（控制器级复位后回退为 Locked）
   ↑                                                |
   |                  Set Features (011b)           |
   +---- 下次完整断电 <-- [Locked Until Power Cycle] <--+（不可通过 Feature 解除，可穿越 CLR）
```

通过 `Set Features / Get Features`（**Feature ID = 85h**, Boot Partition Write Protection Config）配置动作码：

| 动作码 | 名称 | 说明 |
|---|---|---|
| `000b` | No Change | 保持当前状态（**仅用于 Set**，Get 永不返回） |
| `001b` | Write Unlock | 写解锁，允许写入，控制器级复位后恢复为写锁定 |
| `010b` | Write Lock | 写锁定，禁止写入 |
| `011b` | Lock Until Power Cycle | 强制锁定直到下次完整断电，期间 `Set Features` 无法更改 |
| `100b` | RPMB Controlled | **仅用于 Get**，表示由 RPMB 机制控制，无法通过 Feature 修改 |

---

## 易混概念对比

| 易混概念 | 实际区别 |
|---|---|
| **BPMBL vs BPRSEL** | `BPMBL` 指定数据写到主机内存的哪里；`BPRSEL` 选择读哪个分区、哪一段（写入触发读） |
| **BPSZ (128 KiB) vs BPRSZ (4 KiB)** | `BPSZ` 是整个分区的总容量单位（128 KiB）；`BPRSZ` 是单次读取的大小单位（4 KiB） |
| **Write Unlocked vs Locked Until Power Cycle** | 前者在控制器级复位后恢复写锁定；后者可穿越复位，必须完整断电才解除 |

---

## 关联与引申概念

- **MMIO 寄存器（Controller Properties）**：控制器映射到主机内存空间的寄存器组，启动早期无需队列即可直接读写 `BPINFO`/`BPRSEL`/`BPMBL`。
- **Get Log Page**：标准 Admin 命令，通过指定 `LID`（Log Identifier）读取控制器日志，`LID=15h` 对应启动分区数据。
- **Feature ID**：`Get/Set Features` 命令用来标识不同可配置特性的编号，`85h` 专用于启动分区写保护。
- **RPMB（Replay Protected Memory Block）**：带重放保护的安全存储分区，可接管启动分区的写保护控制权。
- **延伸阅读**：
  - [admin-command-model.md](admin-command-model.md) — `Get Log Page` 与 `Set Features` 所属的 Admin 命令模型
  - [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) — 控制器使能、关机与复位对早期访问及写锁定的影响
  - [firmware-update-lifecycle.md](firmware-update-lifecycle.md) — 启动分区固件写入与激活流程
  - [format-nvm-lifecycle.md](format-nvm-lifecycle.md) — 启动分区独立于命名空间，不参与 Format NVM

---

## 规范依据

- [启动分区信息与读取选择寄存器（BPINFO / BPRSEL，Figures 49-50），PDF p. 88](../_source/pages/page-088.md)
- [启动分区内存缓冲区寄存器（BPMBL），PDF p. 89](../_source/pages/page-089.md)
- [启动分区日志页选择器与布局（Figures 267-268，LID=15h），PDF pp. 286-287](../_source/pages/page-286.md)
- [Feature 85h 写保护配置（Figure 413），PDF pp. 426-427](../_source/pages/page-426.md)
