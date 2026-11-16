# BAR0 控制器寄存器与门铃步长（BAR0 & Doorbells）

在 PCIe 总线上，NVMe 控制器把所有控制面板与门铃开关都映射到 PCIe 配置空间的 **64 位 BAR0/BAR1**（分别对应规范中的 `MLBAR` 与 `MUBAR`）所指向的物理内存区间中 `[PCIe Transport §3.1]`。

可以把 `BAR0` 想成**写字楼的整套前台总控台**：
- **`0000h–0FFFh`（前 4KB）**：是总控面板（看设备能力、开关机、配置 Admin 队列地址）。
- **`1000h` 之后**：是一排排按固定间隔排列的**队列门铃（Doorbells）**，每对队列（SQ + CQ）占两个相邻的 32 位门铃槽位。

---

## BAR0/BAR1 寄存器偏移全景

| 偏移量 (Hex) | 符号 | 宽度 | 必选/可选 | 核心作用 |
| :--- | :--- | :--- | :--- | :--- |
| `00h – 07h` | **`CAP`** | 64-bit | 必选 | **控制器能力**：报告最大队列深度 `MQES`、门铃步长 `DSTRD`、超时 `TO`、支持页大小 `MPSMIN`/`MPSMAX` |
| `08h – 0Bh` | **`VS`** | 32-bit | 必选 | **规范版本**：如 `00020100h` 表示 NVMe 2.1 |
| `0Ch – 0Fh` | **`INTMS`** | 32-bit | 必选 | **中断屏蔽置位**：写 `1` 屏蔽对应向量（仅用于 Pin/单消息 MSI，**MSI-X 下禁止使用**） |
| `10h – 13h` | **`INTMC`** | 32-bit | 必选 | **中断屏蔽清除**：写 `1` 解除屏蔽（MSI-X 下同样禁止使用） |
| `14h – 17h` | **`CC`** | 32-bit | 必选 | **控制器配置**：`EN`（使能）、`MPS`（页大小）、`CSS`（命令集）、`SHN`（关机通知）、`IOSQES`/`IOCQES` |
| `1Ch – 1Fh` | **`CSTS`** | 32-bit | 必选 | **控制器状态**：`RDY`（就绪）、`CFS`（致命错误）、`SHST`（关机状态）、`NSSRO`（子系统复位发生） |
| `20h – 23h` | **`NSSR`** | 32-bit | 可选 | **NVM 子系统复位**：写入魔数 `4E564D65h`（ASCII `"NVMe"`）触发子系统级硬复位 |
| `24h – 27h` | **`AQA`** | 32-bit | 必选 | **Admin 队列属性**：配置 `ASQS`（Admin SQ 大小）与 `ACQS`（Admin CQ 大小），均为 0-based |
| `28h – 2Fh` | **`ASQ`** | 64-bit | 必选 | **Admin SQ 基址**：4KB（`CC.MPS`）对齐的主机物理/IOVA 地址 |
| `30h – 37h` | **`ACQ`** | 64-bit | 必选 | **Admin CQ 基址**：4KB（`CC.MPS`）对齐的主机物理/IOVA 地址 |
| `38h – 3Fh` | **`CMBLOC` / `CMBSZ`** | 32-bit×2 | 可选 | **控制器内存缓冲区（CMB）** 的 BAR 索引、偏移与容量 |
| `40h – 4Ch` | **`BPINFO` / `BPRSEL` / `BPMBL`** | 32/64-bit | 可选 | **引导分区（Boot Partition）** 只读/访问控制寄存器 |
| `1000h + ...` | **`SQyTDBL` / `CQyHDBL`** | 32-bit | 必选 | 队列 `y` 的 SQ 尾部门铃与 CQ 头部门铃 |

> **关键规则**：主机必须按寄存器自然宽度（32 位或 64 位）访问 BAR0；若平台仅支持 32 位 MMIO 写，写 64 位寄存器（如 `ASQ`/`ACQ`）必须**先写低 32 位、再写高 32 位**，不能跨寄存器拆成字节写 `[Base §3.1.2]`。

---

## 门铃寄存器地址与 `CAP.DSTRD` 步长公式

每个队列 `y`（`y = 0` 为 Admin 队列，`y >= 1` 为 I/O 队列）都有一对 32 位门铃，但两个门铃之间隔多少字节并不是写死的，而是由 **`CAP.DSTRD`（Bits `[35:32]`）** 决定 `[PCIe Transport §3.1.15]`：

$$\text{Stride} = 4 \times 2^{\text{DSTRD}} = 2^{(2 + \text{DSTRD})} \text{ bytes}$$

- **SQ `y` Tail Doorbell（提交新命令）**：`1000h + (2y) * Stride`
- **CQ `y` Head Doorbell（确认消费完成条目）**：`1000h + (2y + 1) * Stride`

```text
BAR0 偏移 (当 DSTRD = 0, Stride = 4 bytes 时)
  1000h : SQ 0 Tail Doorbell  (Admin SQ)
  1004h : CQ 0 Head Doorbell  (Admin CQ)
  1008h : SQ 1 Tail Doorbell  (I/O SQ 1)
  100Ch : CQ 1 Head Doorbell  (I/O CQ 1)

当 DSTRD = 1, Stride = 8 bytes 时：
  1000h : SQ 0 Tail Doorbell (有效 4B) + 1004h (保留 4B)
  1008h : CQ 0 Head Doorbell (有效 4B) + 100Ch (保留 4B)
```

**为什么要留空隙（`DSTRD > 0`）？**
某些硬件或 SR-IOV/虚拟化控制器需要让每个门铃独占一个独立的 Cache Line 或 MMU 页表项，避免不同队列/VF 的门铃写操作互相干扰。

---

## 影子门铃（Shadow Doorbell Buffer）

在虚拟机（KVM/QEMU）或软件仿真控制器场景下，每次敲门铃都是一次 MMIO Write，会触发昂贵的 **VM-Exit**。为此 NVMe 定义了 Admin 命令 **`Doorbell Buffer Config`（Opcode `7Fh`）** `[Base §5.1.8]`：
- 主机在内存中分配两块 4KB 页：**Shadow Doorbell Buffer**（存最新指针）与 **EventIdx Buffer**（控制器告诉主机“指针走到哪才需要真敲一次硬件门铃”）。
- 仅当新指针越过 `EventIdx` 时，驱动才真正写 BAR0 MMIO 门铃，大幅削减虚拟化开销。

```bash
# 在 Linux 中查看 NVMe 控制器的 BAR0/BAR1 地址与能力寄存器
lspci -vvv -s 0000:01:00.0 | grep -E "Region 0|Memory at"
nvme show-regs /dev/nvme0 -H | head -n 25
```

---

## 关联与引申概念

- **控制器属性基础**：`CAP.TO`、`CC.EN` 与 `CSTS.RDY` 的完整上电握手时序，详见 [控制器属性](../nvme/concepts/controller-properties.md)。
- **队列对模型**：SQ/CQ 环形缓冲区指针与单槽未使用规则，详见 [队列对](../nvme/concepts/queue-pair.md)。
- **PCIe BAR 空间探测**：UEFI/BIOS 如何为 BAR0/BAR1 分配物理地址，详见 [BARs 详解](../pcie/wiki/configuration/bars.md)。
- **虚拟化直通开销**：VFIO 场景下透传 BAR0 与门铃直写的原理，详见 [中断重映射、VFIO 与 SR-IOV 虚拟化](../host-system/interrupt-remapping-and-vfio.md)。
