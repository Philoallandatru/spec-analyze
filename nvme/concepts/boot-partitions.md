# 启动分区（Boot Partitions）

## 一句话说明

启动分区（Boot Partitions）是 NVMe 控制器上两块**大小相同**的专用存储区域，用于存放 BIOS/UEFI 启动代码或平台固件；它无需初始化队列即可访问，因此主机能在极早期把启动镜像读进内存。

## 生活化类比

把启动分区想成**汽车仪表盘里藏着的两张"应急地图"**：

- **两张完全相同的纸质地图** = 分区 0 与分区 1（大小相同，互为冗余）
- **仪表盘下的小抽屉** = 控制器内部的启动分区存储区
- **司机（主机/BIOS）伸手进抽屉** = 通过 BPRSEL 寄存器发起读操作
- **抽屉旁边的小屏幕** = BPINFO 状态字段（显示读完成没、现在用的是哪张）
- **锁扣** = 写保护机制（默认上锁，避免意外覆盖启动代码）

司机不会一上车就把整套导航系统初始化好；他先打开小抽屉拿一张地图就能出发，等导航系统启动后再做精细操作。

## 工作流程

### 三步寄存器读

```text
第 1 步：把"小抽屉"地址告诉控制器
   BPMBL.BMBBA ← 页对齐（4 KiB）的主机内存地址

第 2 步：选哪张地图 + 读哪段
   BPRSEL ← { BPID (0/1), 偏移 × 4 KiB, 大小 × 4 KiB }

第 3 步：轮询小屏幕确认
   BPINFO.BRS 状态：
     00b = 无活动
     01b = 传输中
     10b = 成功完成
     11b = 出错（目标缓冲内容未定义）
```

### 越界保护

```text
分区起始 |--- 偏移 ---|--- 大小 ---| 分区结束
                     ↑             ↑
                读取起点        读取终点

  规则：偏移 + 大小 ≤ 分区大小，否则 BRS=11b 且不传输任何数据
```

## 初学者案例

**场景：UEFI 平台从 NVMe 启动**

1. 主板上电，CPU 复位向量指向 SPI Flash 上的最小引导。
2. 引导想要拉一段更大的 UEFI payload，但它只知道 NVMe 控制器，还没初始化队列。
3. 引导先写 `BPMBL.BMBBA = 0x10000000`（4 KiB 对齐的目的缓冲）。
4. 引导写 `BPRSEL = {BPID=0, 偏移=0, 大小=64}`（即 64×4 KiB = 256 KiB）。
5. 引导轮询 `BPINFO.BRS`：`01b → 10b`，传输完成。
6. 引导从 0x10000000 跳转到 UEFI payload，正常初始化 NVMe 队列。
7. 后续 OS 用标准日志页（`Get Log Page, LID=15h`）读取完整启动分区做完整性校验。

> 速查：BPINFO 还提供 `ABPID`（活动分区 ID，0 或 1）和 `BPSZ`（分区大小，单位 128 KiB）。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 两个分区大小相同 | BPINFO.BPSZ 以 128 KiB 为单位；分区 0 和分区 1 永远同大小 |
| 早期可访问 | 寄存器方式无需队列初始化、Controller 启用或命名空间 |
| 偏移/大小粒度 | 4 KiB 单位（BPROF/BPRSZ 字段定义） |
| 目的地址页对齐 | BPMBL.BMBBA 低 12 位为 0（保留） |
| 越界不传输 | 偏移+大小超过分区大小 → 不传输 + BRS=11b + 目标缓冲未定义 |
| 默认写锁定 | 上电后两个分区默认处于 Write Locked 状态 |
| 三种保护态 | Write Locked / Write Unlocked / Locked Until Power Cycle（+ RPMB Controlled 只读态） |
| 复位回默认 | 控制器级复位后 Write Unlocked 自动回退到 Write Locked |
| 电源周期锁不可改 | 处于 Locked Until Power Cycle 时，Set Features 不可改，必须等完整断电 |
| Feature ID 85h | Boot Partition Write Protection Config，每个分区独立配置 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| BPMBL vs BPRSEL | BPMBL 是"目标地址"（写到主机哪里）；BPRSEL 是"选择 + 发起"（读分区几、哪段、多大） |
| 寄存器方式 vs 日志页方式 | 寄存器方式：可指定偏移/大小、无需队列；日志页方式（LID=15h）：只能读完整分区、需要队列 |
| BPID 写入触发的读 vs 读 BPRSEL | 写 BPRSEL.BPID 字段**会**触发读；其他字段写入只更新选择参数 |
| BPSZ 单位 128 KiB vs BPRSZ 单位 4 KiB | 一个是分区总大小，一个是单次读大小 |
| Write Unlocked vs Locked Until Power Cycle | 前者控制器级复位即可解锁；后者必须完整断电 |
| ABPID vs 控制器活动分区 | ABPID 字段报告"活动"分区（0/1），但写入 BPRSEL 才会触发实际读取 |

## 进阶细节

- **寄存器组**（PCIe 配置空间）：
  - `BPINFO`（offset 40h）：`ABPID` (bit 31)、`BRS` (bit 25:24)、`BPSZ` (bit 14:0)。
  - `BPRSEL`（offset 44h）：`BPID` (bit 31)、`BPROF` (bit 29:10, 4 KiB 单位)、`BPRSZ` (bit 9:0, 4 KiB 单位)。
  - `BPMBL`（offset 48h）：`BMBBA` (低 12 位保留，须 4 KiB 对齐)。
- **Get Log Page 方式**（LID=15h）：
  - 返回 16 字节头部（LID + BPINFO 镜像） + 完整分区数据（BPSZ × 128 KiB）。
  - `CDW10.BPID` 选择分区 0/1；不修改 BPINFO/BPRSEL/BPMBL。
- **写保护配置（Feature 85h）动作码**：
  - `000b` No Change（仅 Set 使用，Get 永不返回）
  - `001b` Write Unlock
  - `010b` Write Lock
  - `011b` Lock Until Power Cycle
  - `100b` RPMB Controlled（Get 报告 RPMB 控制，Set 不能改）
- **典型应用**：UEFI/BIOS 镜像、平台初始化代码、安全启动信任根、固件备份/回滚。
- **特性可选**：Boot Partitions 整体是可选特性；若不支持，BPINFO/BPRSEL/BPMBL 全为 0。

## 规范依据

- [BPINFO 寄存器定义（Figure 49），PDF 第 88 页](../_source/pages/page-088.md)
- [BPRSEL 寄存器定义（Figure 50），PDF 第 88 页](../_source/pages/page-088.md)
- [BPMBL 寄存器（offset 48h），PDF 第 89 页](../_source/pages/page-089.md)
- [启动分区日志页（Figures 267-268，LID=15h），PDF 第 286 页](../_source/pages/page-286.md)
- [Feature 85h 写保护配置（Figure 413），PDF 第 426 页](../_source/pages/page-426.md)

## 相关阅读

- [admin-command-model.md](admin-command-model.md) - Get Log Page 是 Admin 命令
- [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) - 早期寄存器访问无需队列
- [firmware-update-lifecycle.md](firmware-update-lifecycle.md) - Boot Partition 固件写入命令
- [format-nvm-lifecycle.md](format-nvm-lifecycle.md) - 启动分区不参与 Format 范围
