# 排序规则 (Ordering Rules)

**PCIe 排序规则**定义了同一虚拟通道（VC）/ 流量类别（TC）内，后到达的 TLP 是否允许**超越（Pass）**先到达的 TLP。它既要保证经典的 **Producer-Consumer（生产者-消费者）** 数据一致性，又要通过允许特定场景下的乱序来防止死锁并提升并发性能。

> **直观类比**：就像医院的急诊分诊规则——如果病人先挂号写好了病历（Posted Write 1），再按下呼叫铃（Posted Write 2），呼叫铃**绝对不能**插队跑到病历前面，否则医生（Consumer）听到铃声去翻病历时看到的还是空白页；但如果前面有一张化验单（Non-Posted Read）卡在化验室等结果，后面来的新写操作（Posted Write）**必须允许**绕过去先执行，否则整个走廊都会被堵死（死锁）。

---

## 1. 核心问题：Producer-Consumer 模型

理解 PCIe 排序规则，只需记住下面这个最经典的 DMA / Doorbell 场景：

```
CPU / 设备 (Producer)                    对端设备 / 内存 (Consumer)
   │                                           │
   │ 1. Posted Write: 把有效数据写入内存 Buffer  │
   ├──────────────────────────────────────────▶│
   │ 2. Posted Write: 写 Doorbell / Flag 通知就绪│
   ├──────────────────────────────────────────▶│  ① 规则保证：Write 2 绝不会超越 Write 1
   │                                           │
   │                                           │  ② Consumer 看到 Flag 变化后，
   │ 3. Non-Posted Read: 读数据                 │     发起 Memory Read 读取数据
   │◀──────────────────────────────────────────┤  ③ 由于 Read 在 Write 1 之后到达，
   │ 4. Completion: 返回最新数据                │     保证能读到最新写入的内容
   ├──────────────────────────────────────────▶│
```

---

## 2. PCIe 基准排序规则表（同一 TC 内）

下表遵循 PCIe Base Spec 标准定义：**行**代表**后到达的事务**（尝试超越者），**列**代表**先到达的事务**（前方排队者）：

| 后到达 (Row) ╲ 先到达 (Col) | Posted Request<br>*(MWr, Msg)* | Non-Posted Request (无数据)<br>*(MRd, IORd, CfgRd)* | Non-Posted Request (带数据)<br>*(IOWr, CfgWr)* | Completion<br>*(Cpl, CplD)* |
| :--- | :---: | :---: | :---: | :---: |
| **Posted Request** | **No** *(严禁超越)* | **Yes** *(必须可超越)* | **Yes** *(必须可超越)* | **Yes/No** *(可选超越)* |
| **Non-Posted Request (无数据)** | **No** *(严禁超越)* | **Yes/No** | **Yes/No** | **Yes/No** |
| **Non-Posted Request (带数据)** | **No** *(严禁超越)* | **Yes/No** | **Yes/No** | **Yes/No** |
| **Completion** | **No** *(严禁超越)* | **Yes** *(必须可超越)* | **Yes** *(必须可超越)* | **Yes/No** |

- **No**：**严禁超越**（Strongly Ordered），必须严格按先来后到顺序推进。
- **Yes**：**必须允许超越**（Mandatory Pass），中间设备（如 Switch/RC）不能让前面的包永久阻塞后面的包，否则会导致协议死锁。
- **Yes/No**：**允许但不强制**超越，由硬件实现自行决定。

### 关键规则解读

1. **Posted 不能超越 Posted（`No`）**：保障 Producer-Consumer 模型——数据写入与 Doorbell 通知的顺序绝不会颠倒。
2. **Non-Posted 不能超越 Posted（`No`）**：防止"写后读"（RAW）读到旧数据——先发的写请求未落盘前，后发的读请求不能抢先执行。
3. **Posted 与 Completion 必须能超越 Non-Posted（`Yes`）**：**防死锁核心规则**——如果一个 Non-Posted 读请求因对端等待某个 Posted 写或 Completion 而卡住，后者必须能够绕过该读请求前进。
4. **Completion 不能超越 Posted（`No`）**：确保在设备发起 DMA 写之后再读一次（或通过读 Completion 确认写已刷出）时，Completion 不会抢在前面的 Posted 写之前到达目标域。

---

## 3. 性能优化：放宽排序约束

默认严格排序在多核、多队列高并发下可能引发**队头阻塞（Head-of-Line Blocking）**。PCIe 提供了两种在 TLP Header 中声明"互不依赖、可以插队"的机制：

### 3.1 Relaxed Ordering（RO，宽松排序）

通过 TLP Header 中的 `Attr[1]`（Relaxed Ordering 位）置 `1`，向 Switch 和 Root Complex 声明"本事务与之前的某些事务无依赖关系"：
- 当 **Posted Request** 或 **Completion** 的 `RO = 1` 时，它**被允许超越**先前的 Posted Request（原本为 `No` → 变为 `Yes/No`）。
- 当 **Completion** 的 `RO = 1` 时，还允许**不同地址/无依赖**的读请求响应乱序返回（提升多通道内存控制器效率）。
- **注意**：仅当软件/驱动确认数据流彼此独立（如大块帧缓冲、独立 DMA 数据块）时才可开启，Producer-Consumer 的 Doorbell 通知位绝不能开 `RO`。

### 3.2 ID-Based Ordering（IDO，基于 ID 的排序）

PCIe 3.0 引入的机制（`Attr[2]` 为 IDO 位，需在配置空间 `Device Control 2` 中开启 `IDO Request Enable` / `IDO Completion Enable`）：
- **核心思想**：只有**相同 Requester ID**（来自同一设备/功能）的事务之间才需要严格遵守基准排序规则；**不同 Requester ID** 之间的 Posted 或 Completion 事务可以相互超越。
- **价值**：在虚拟化（SR-IOV）或多设备挂载同一 Switch 的场景下，彻底消除不同设备/VF 之间的队头阻塞。

---

## 4. 关联与引申概念

- **事务类型（Posted / Non-Posted / Completion）**：排序规则分类的基础，详见 [PCIe 事务类型详解](transaction-types.md)。
- **虚拟通道（VC）与流量类别（TC）**：PCIe 的排序规则**仅在同一 TC 内有效**；不同 TC/VC 之间完全没有排序保证，详见 [虚拟通道](virtual-channels.md)。
- **TPH（TLP Processing Hints）与 ATS**：与 RO/IDO 类似，均为 TLP Header 中的高级优化属性，分别用于提示 CPU Cache 预取策略与地址转换缓存。
- **NVMe Doorbell 机制**：Producer-Consumer 模型在存储领域的典型应用——主机先写 Submission Queue (SQ) 内存，再通过 Posted Write 写控制器 Doorbell 寄存器，依赖 PCIe 排序规则保证控制器 DMA 读 SQ 时能看到最新命令。
