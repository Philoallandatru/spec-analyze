# PCIe 事务类型详解

**PCIe 事务（Transactions）** 是设备与主机之间通过 TLP 完成的各类读、写、配置与通知操作。从通信模式上看，所有事务都可归入两大阵营：**发后即忘的 Posted** 与 **必须等回执的 Non-Posted（Split Transaction）**；按功能则分为五大类：Memory、I/O、Configuration、Message 与 Completion。

> **直观类比**：**Posted 事务**（如 Memory Write）就像"发微信群通知"——发出去就继续干活，不用等对方回复；**Non-Posted 事务**（如 Memory Read）则像"去窗口办业务拿号"——递交申请（Request）后先离开窗口去干别的事，不阻塞总线，等后台办好后再拿着你的号（`Requester ID + Tag`）叫你回来取结果（`Completion`），这种"请求与响应分离"的机制就叫 **Split Transaction（分离事务）**。

---

## 1. Posted vs Non-Posted 与 Split Transaction

| 事务模式 | 包含的 TLP 类型 | 是否需要 Completion | 核心优缺点 |
|----------|----------------|-------------------|------------|
| **Posted** | Memory Write (MWr), Message (Msg/MsgD) | **否** | 吞吐高、延迟低，但发送方无法直接获知何时真正落盘 |
| **Non-Posted** | Memory Read (MRd), I/O R/W, Config R/W, AtomicOp | **是**（Cpl / CplD） | 占用 Tag 资源并需等待回包，但能严格确认完成与获取数据 |
| **Completion** | Completion (Cpl), Completion with Data (CplD) | —（本身就是响应） | 携带原始 `Requester ID + Tag`，允许大数据分片返回 |

### 为什么 PCIe 采用 Split Transaction？
传统并行 PCI 总线在读取慢速设备时，会一直占用总线等待数据返回，导致其他设备全部饿死。PCIe 改为 **Split Transaction** 后：
1. 设备 A 发出 `MRd`（带上自己的 `BDF` 和 `Tag=5`）后立即释放链路。
2. 链路在等待期间可以全速传输其他设备的读写包。
3. 目标设备准备好数据后，发送携带 `Requester ID=设备A, Tag=5` 的 `CplD` 返回，设备 A 据此匹配到原请求。

---

## 2. 五大事务类型详解

### 2.1 Memory Transactions（内存事务）

用于访问系统主存（DMA）或设备的内存映射寄存器（**MMIO / BAR**），是 PCIe 99% 以上流量的载体：
- **Memory Read (MRd, Non-Posted)**：请求长度受限于配置空间中的 `Max_Read_Request_Size`（`MaxReadReq`，常见 512 B ~ 4 KB）。若一次读取跨越多片数据，目标端可拆分成多个较小的 `CplD` 分片返回（每个分片不超过 `Max_Payload_Size`）。
- **Memory Write (MWr, Posted)**：单包数据负载受限于 `Max_Payload_Size`（`MPS`，常见 128 B ~ 512 B）。**MSI / MSI-X 中断**在底层本质上就是设备向特定系统中断地址发起的一次 4 字节 `Memory Write`。
- **AtomicOp（原子操作, Non-Posted）**：PCIe 3.0+ 引入（取代已废弃的 `MRdLk`），支持对内存执行原子的 FetchAdd、Swap、CAS 操作。

### 2.2 Configuration Transactions（配置事务）

用于操作系统或 BIOS 在枚举阶段读取 Vendor/Device ID、分配 BAR 地址与中断号，全部为 **Non-Posted**：
- **Type 0**：访问与当前总线**直连**的设备配置空间。
- **Type 1**：需要**穿越 Switch/Bridge** 访问下游更深层总线的设备；到达目标总线所在的最后一个 Bridge 时，由 Bridge 自动转换为 Type 0。

### 2.3 I/O Transactions（I/O 事务）

- **IORd / IOWr（Non-Posted）**：仅用于兼容传统 x86 Port I/O（16 位有效地址、单次最多 4 字节），现代设备（如 NVMe、GPU）已全面改用 **MMIO**。

### 2.4 Message Transactions（消息事务）

- **Msg / MsgD（Posted）**：取代了传统 PCI 上的大量边带物理引脚，把事件通知统一封成 TLP，包括：
  - 模拟传统电平中断：`Assert_INTx` / `Deassert_INTx`
  - 电源管理：`PM_PME`、`PME_Turn_Off`、`PME_TO_Ack`
  - 错误上报（AER）：`ERR_COR`、`ERR_NONFATAL`、`ERR_FATAL`

### 2.5 Completion Transactions（完成事务）

- **Cpl（无数据）**：用于确认 Config Write、I/O Write 已完成，或在读取失败时返回错误状态（如 `UR` Unsupported Request、`CA` Completer Abort）。
- **CplD（带数据）**：用于返回 Memory Read、Config Read、I/O Read 的数据。
- **Completion Timeout**：若 Requester 发出 Non-Posted 请求后在超时阈值（典型数十毫秒）内未收到 Completion，将触发 `Completion Timeout` 错误，防止系统永久挂死。

---

## 3. 实战参数查看（Linux `lspci`）

```bash
# 查看设备的 Max_Payload_Size (MPS) 与 Max_Read_Request_Size (MRRS)
lspci -vvv -s 01:00.0 | grep -E "MaxPayload|MaxReadReq"
# 示例：MaxPayload 256 bytes, MaxReadReq 512 bytes
```

---

## 4. 关联与引申概念

- **TLP 格式详解**：查看各事务类型的 `Fmt`/`Type` 编码与 Header 比特布局，详见 [TLP 格式详解](tlp-format.md)。
- **流控制（Flow Control）**：Posted、Non-Posted 与 Completion 拥有各自独立的信用池以防止死锁，详见 [流控制机制](flow-control.md)。
- **排序规则（Ordering）**：定义 Posted、Non-Posted 与 Completion 之间谁能超越谁，详见 [排序规则](ordering.md)。
- **MMIO & DMA**：CPU 通过 `MRd/MWr` 访问设备 BAR 空间称为 MMIO；设备主动发起 `MRd/MWr` 访问系统内存称为 DMA。
- **MSI / MSI-X**：基于 `Memory Write` 实现的高性能消息中断机制，彻底消除了传统 `INTx` 的共享中断与边带延迟。
