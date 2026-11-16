# PCIe 数据包类型

**一句话总结**：PCIe 链路上流动的数据分为三大类——**TLP**（事务层数据包，负责搬运业务数据与读写请求）、**DLLP**（数据链路层包，负责相邻两跳的确认与流控）和 **Ordered Set**（物理层有序集，负责链路训练与时钟同步）。

> **直观类比**：如果把 PCIe 链路比作一条货运专线：
> - **TLP** 是装着货物和收发货地址的**集装箱**（端到端送达）；
> - **DLLP** 是相邻两个收费站之间的**回执单与通行证**（只管这一段路）；
> - **Ordered Set** 是路上的**交通信号灯与对表口令**（只管车道通畅和时钟对齐）。

---

## 三类数据包全景对比

| 数据包类型 | 所属层次 | 传输范围 | 长度特征 | 核心作用 |
|-----------|---------|---------|---------|---------|
| **TLP** (Transaction Layer Packet) | 事务层 | 端到端（可经多个 Switch） | 变长（Header 12/16B + Payload 0~4KB） | 内存/IO/配置读写、中断消息、完成响应 |
| **DLLP** (Data Link Layer Packet) | 数据链路层 | 仅相邻两端口（逐跳） | 固定 6 字节（含 16 位 CRC） | `Ack/Nak` 重传确认、流控信用更新、电源管理 |
| **Ordered Set** (有序集) | 物理层 | 仅相邻两端口（逐跳） | 特殊物理层符号序列 | `TS1/TS2` 链路训练、`SKP` 时钟补偿、`EIOS/EIEOS` 休眠唤醒 |

---

## 1. TLP（事务层数据包）

### TLP 整体封装结构

```
┌──────────────┬──────────────────┬────────────────────┬──────────────┐
│  TLP Prefix  │    TLP Header    │    Data Payload    │ Digest(ECRC) │
│ (可选, Gen3+)│   (3 或 4 DW)    │  (可选, 0-1024 DW) │  (可选, 1 DW)│
└──────────────┴──────────────────┴────────────────────┴──────────────┘
  * 注：经过数据链路层时，还会在头部前加 2B Sequence#，末尾加 4B LCRC。
  * 1 DW (Double Word) = 4 字节 = 32 位。
```

### 常见 TLP 类型表（由 DW0 的 `Fmt` 与 `Type` 共同决定）

| Fmt[2:0] | Type[4:0] | 事务名称 | 是否带数据 | 事务模型 | 典型场景 |
|----------|-----------|---------|-----------|---------|---------|
| `000` / `001` | `00000` | Memory Read (3DW / 4DW) | 无 | Non-Posted | CPU 读 BAR 寄存器；设备 DMA 读内存 |
| `010` / `011` | `00000` | Memory Write (3DW / 4DW) | 有 | Posted | CPU 写 BAR；设备 DMA 写内存；MSI-X 中断 |
| `000` / `010` | `00010` | I/O Read / Write | 无 / 有 | Non-Posted | 兼容传统 x86 I/O 端口 |
| `000` / `010` | `00100` | Config Read / Write Type 0 | 无 / 有 | Non-Posted | 访问当前总线上设备的配置空间 |
| `000` / `010` | `00101` | Config Read / Write Type 1 | 无 / 有 | Non-Posted | 经由 Bridge 转发访问下游总线配置空间 |
| `000` / `010` | `1xxxx` | Message (Msg / MsgD) | 无 / 有 | Posted | 错误报告（ERR_*）、电源管理、模拟 INTx |
| `000` / `010` | `01010` | Completion (Cpl / CplD) | 无 / 有 | Completion | 对 Non-Posted 请求的响应（返回状态或数据） |

> **3DW vs 4DW**：32 位地址（4GB 以下）使用 3DW Header（12 字节）；64 位地址（4GB 以上）使用 4DW Header（16 字节）。

### 关键 TLP Header 字段解读

1. **Memory Request Header（以 4DW 64 位地址为例）**：
   - **DW0**：`Fmt/Type`（包类型）、`TC`（3 位流量优先级）、`Attr`（如 Relaxed Ordering 宽松排序、No Snoop）、`TD`（是否含 ECRC）、`Length`（10 位负载长度，以 DW 为单位，`0` 代表 1024 DW = 4KB）。
   - **DW1**：`Requester ID`（发起方的 16 位 BDF）、`Tag`（8/10 位事务标签，用于将返回的 Completion 与原请求配对）、`Last DW BE` / `First DW BE`（首尾双字的字节使能掩码，支持非 4 字节对齐的局部读写）。
   - **DW2–DW3**：`Address[63:2]`（目标内存地址，最低 2 位隐含为 `00`）。

2. **Completion TLP（响应包）**：
   - **DW1**：`Completer ID`（响应方 BDF）、`Status`（3 位完成状态：`000` = `SC` 成功完成、`001` = `UR` 不支持的请求、`010` = `CRS` 配置重试、`100` = `CA` 中止）、`Byte Count`（剩余待传输字节数）。
   - **DW2**：`Requester ID` + `Tag`（原样带回请求方标识以便匹配）+ `Lower Address[6:0]`（数据起始地址低 7 位）。

---

## 2. DLLP（数据链路层包）

DLLP 产生于数据链路层并在相邻端口的链路层被直接消费（**不会上传到事务层，也不会跨 Switch 转发**），固定为 **6 字节（48 位）**：

```
┌──────────────────┬──────────────────┬──────────────────┐
│  DLLP Type (8b)  │  控制数据 (24b)  │   CRC-16 (16b)   │
└──────────────────┴──────────────────┴──────────────────┘
```

| DLLP 类型编码 | 名称 | 核心用途 |
|--------------|------|---------|
| `00h` | **Ack** | 确认已正确收到至 `AckSeq[11:0]` 为止的所有 TLP，通知对方释放 Retry Buffer |
| `10h` | **Nak** | 报告序列号为 `NakSeq[11:0]` 的 TLP 校验失败，触发对方立即重传 |
| `40h–4Fh` | **InitFC1 / InitFC2** | 链路初始化时通告各虚拟通道（VC）的 Posted/Non-Posted/Completion 初始信用 |
| `50h–5Fh` | **UpdateFC** | 运行时动态返还已释放的缓冲信用（`HdrFC` + `DataFC`） |
| `20h–23h` | **PM_Enter_L1 / L23 等** | 协商链路进入或退出低功耗电源状态 |

---

## 3. Ordered Set（物理层有序集）

Ordered Set 是物理层直接生成和识别的特殊符号序列，用于在数据链路层激活之前完成链路建立，以及在运行期间维持物理同步：

| 有序集名称 | 核心用途 |
|-----------|---------|
| **TS1 / TS2 (Training Sequence 1/2)** | 链路训练期间交换速率能力、Lane 编号、极性与宽度配置，完成比特锁定与符号锁定 |
| **SKP (Skip Ordered Set)** | 周期性插入数据流，供接收端弹性缓冲区（Elastic Buffer）增删，补偿两端独立时钟的微弱频差 |
| **FTS (Fast Training Sequence)** | 链路从浅度省电状态 `L0s` 快速唤醒回 `L0` 时用于重新锁定比特与符号边界 |
| **EIOS / EIEOS** | 通知对端进入电气空闲（`EIOS`，Electrical Idle）或退出电气空闲唤醒链路（`EIEOS`） |

---

## 实战视角：一次 Memory Read 经历了什么？

假设 CPU 读取 NVMe SSD 的 BAR0 寄存器（4 字节）：
1. **CPU 侧发送请求**：事务层构造 `MRd`（Memory Read）TLP，填入 `Requester ID = 00:00.0`、`Tag = 0x02`、`Length = 1 DW`、`Addr = 0xF8000400`。
2. **逐跳传递与确认**：每经过一条物理链路，数据链路层为其加上序列号和 LCRC；对端收到校验无误后回传 **Ack DLLP**。
3. **SSD 返回数据**：SSD 事务层读取寄存器值 `0xAABBCCDD`，构造 `CplD`（Completion with Data）TLP，填入 `Status = SC`、`Requester ID = 00:00.0`、`Tag = 0x02` 及 4 字节负载发回 CPU。

在 Linux 下，可通过内核调试接口或抓包追踪配置访问与 TLP 统计：

```bash
# 使用 trace-cmd 追踪内核 PCI 配置空间读写事件
sudo trace-cmd record -e pci_read_config -e pci_write_config
```

---

## 关联与引申概念

- **[三层协议模型](layering.md)**：理解 TLP、DLLP 与 Ordered Set 分别在协议栈哪一层产生与消费。
- **[时钟架构](clock-architecture.md)**：了解 `SKP` 有序集如何配合弹性缓冲区解决独立时钟频差问题。
- **[地址空间管理](address-space.md)**：理解 Memory Read/Write TLP 中的目标地址如何对应到设备的 BAR 寄存器或系统内存。
- **[设备枚举](enumeration.md)**：查看 `Config Read/Write Type 0` 与 `Type 1` TLP 在总线扫描中的实际分工。
