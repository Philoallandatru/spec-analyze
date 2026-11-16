# ZNS（Zoned Namespaces）命令集与 Zone 状态机

传统块设备允许对任意 LBA 随机覆写，迫使 SSD 内部维护庞大的 FTL 映射表、预留大量 OP（Over-Provisioning）空间并忍受垃圾回收（GC）带来的写放大与尾部延迟抖动。**ZNS（Zoned Namespaces）** 命令集将命名空间切分为若干连续的 **Zone（分区）**，强制每个 Zone 内部只能**顺序追加写入**，把“什么时候擦除（Reset Zone）”的控制权交还给主机软件（如 RocksDB/F2FS/Ceph） `[ZNS CmdSet §1–2]`。

可以把 ZNS 想成**一排只能从头按顺序填满的空白笔记本**：
- **传统 SSD** = 允许你在笔记本任意一页涂改，秘书（FTL）在背后不停帮你把没作废的页抄到新本子上再撕掉旧本子（GC）。
- **ZNS SSD** = 每本笔记本（Zone）只能从第 1 页写到最后 1 页；想重写？必须先把整本笔记本**整本清空（Reset Zone）**。

---

## Zone 的 7 种状态与生命周期

每个 Zone 都由控制器维护一个**写指针（Write Pointer, `WP`）**，初始指向该 Zone 的起始地址 `ZSLBA`，最大可写容量为 `ZCAP`（注意 `ZCAP <= Zone Size`）。

```text
                    Write / Append (隐式打开)
        ┌────────────────────────────────────────────┐
        ▼                                            │
  [ Empty ] ──(Zone Mgmt: Open)──▶ [ Explicitly Open ] ──(写满至 ZCAP)──▶ [ Full ]
     ▲   ▲                               │                                   │
     │   │      (Zone Mgmt: Close)       ▼                                   │
     │   └─────── [ Closed ] ◀─────── [ Implicitly Open ] ───────────────────┘
     │                                                                       │
     └─────────────────────── (Zone Mgmt Send: Reset) ───────────────────────┘
  (硬件不可恢复异常) ──▶ [ Read Only ] ──(严重介质故障)──▶ [ Offline ]
```

| 状态 | 含义 | 允许的操作 |
| :--- | :--- | :--- |
| **`Empty`** | 空白状态，`WP == ZSLBA` | 读（返回未写入规则）、写/Append（自动转为 `Implicitly Open`）、显式 `Open` |
| **`Implicitly Open`** | 因主机直接写入而自动打开，占用控制器 Open 资源 | 继续顺序写、显式 `Open`/`Close`/`Finish`/`Reset` |
| **`Explicitly Open`** | 主机通过 `Zone Management Send (Open)` 显式打开 | 同上；即使暂时不写也不会被控制器自动关闭 |
| **`Closed`** | 已写入部分数据但暂时释放了控制器内部写缓冲 | 可读；再次写入时重新变为 Open；可 `Finish` 或 `Reset` |
| **`Full`** | `WP` 已到达 `ZCAP`，Zone 已满 | 只读，不可再写；必须 **`Reset Zone`** 后才能重新写入 |
| **`Read Only`** | 介质异常降级为只读 | 仅可读，无法写或 Reset |
| **`Offline`** | 彻底不可用 | 不可读不可写 |

---

## 核心 ZNS 专属命令

1. **`Zone Management Send`（Opcode `79h`）**：控制 Zone 状态转换（`CDW13` 指定动作 `ZSA`）：
   - **`Open Zone` (`03h`)** / **`Close Zone` (`01h`)**：显式管理活跃缓冲区。
   - **`Finish Zone` (`02h`)**：直接将未写满的 Zone 标记为 `Full`。
   - **`Reset Zone` (`04h`)**：擦除整个 Zone 的数据并将 `WP` 重置回 `ZSLBA`（可置 `Select All` 一键重置全盘）。
2. **`Zone Management Receive`（Opcode `7Ah`）**：等同于 `Report Zones`，返回每个 Zone 的 `ZSLBA`、`ZCAP`、`WP`、当前状态与属性。
3. **`Zone Append`（Opcode `7Dh`）**：**解决高并发写入锁竞争的杀手级命令** `[ZNS CmdSet §3.4]`：
   - 普通 `Write` 必须精确命中当前 `WP`，多线程并发写同一个 Zone 时主机必须加全局锁串行发包。
   - 使用 `Zone Append` 时，主机只需在 `SLBA` 填入该 Zone 的起始地址 **`ZSLBA`**，无需关心当前 `WP` 走到哪；**SSD 控制器自主排队分配具体 LBA 并推进 `WP`**，最后通过 **CQE `DW0/DW1`** 把实际写入的 LBA 告诉主机！

---

## `MAR` 与 `MOR` 资源上限（极易踩坑）

Open 与 Active 状态的 Zone 需要在控制器 SRAM 中保留映射与写缓冲，因此 `Identify Namespace (ZNS)` 明确规定了两个上限 `[ZNS CmdSet §4.1.5]`：
- **`MOR`（Maximum Open Resources）**：同时处于 `Implicitly Open` + `Explicitly Open` 的最大 Zone 数量（典型值如 14 或 32）。
- **`MAR`（Maximum Active Resources）**：同时处于 `Open` + `Closed` 的最大 Zone 数量（必须满足 $\text{MOR} \le \text{MAR}$；`FFFFFFFFh` 表示无限制）。
- **超限行为**：若主机同时打开的 Zone 超过 `MOR`，控制器会直接拒绝新写请求并返回 **`Too Many Open Zones`**（或 `Too Many Active Zones`），主机必须主动 `Close` 或 `Finish` 部分旧 Zone。

```bash
# 使用 nvme-cli 查看 ZNS 命名空间的 Zone 大小、MAR/MOR 与前 5 个 Zone 状态
nvme zns id-ns /dev/nvme0n1 -H | grep -E "mar|mor|zsze"
nvme zns report-zones /dev/nvme0n1 -d 5
```

---

## 关联与引申概念

- **命名空间管理**：ZNS 命名空间如何通过 `CSI = 02h` 标识与创建，详见 [命令集](../nvme/concepts/command-sets.md) 与 [命名空间管理生命周期](../nvme/concepts/namespace-management-lifecycle.md)。
- **FDP vs ZNS**：ZNS 强制顺序写（改造成本高），而 **FDP（Flexible Data Placement）** 允许随机写的同时按数据生命周期物理隔离，详见 [灵活数据放置配置](../nvme/concepts/flexible-data-placement-configurations.md)。
