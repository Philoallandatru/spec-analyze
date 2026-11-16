# TLP 格式详解

**TLP（Transaction Layer Packet，事务层包）** 是 PCIe 总线上承载读写请求、完成响应和事件消息的基本数据包单元。理解 TLP 的头部字段，就相当于看懂了 PCIe 通信的"快递面单"。

> **直观类比**：如果把 PCIe 链路比作快递干线，**TLP Header** 就是贴在箱子上的标准快递单（写明了"寄件人工号 `Requester ID`"、"包裹类型 `Fmt/Type`"、"收货地址 `Address`"、"加急等级 `TC`"），**Data Payload** 是箱子里的货物，而可选的 **ECRC** 则是内层的原厂防伪封条。

---

## 1. TLP 整体结构

一个完整的事务层 TLP 由三部分组成（`DW = Double Word = 4 字节`）：

```
┌───────────────────────────────────────────────────────────┐
│  TLP Header (3 DW = 12 字节 或 4 DW = 16 字节)             │  必填：路由与控制信息
├───────────────────────────────────────────────────────────┤
│  Data Payload (0 ~ 1024 DW，最大 4096 字节)                │  可选：随请求/响应携带的数据
├───────────────────────────────────────────────────────────┤
│  ECRC / TLP Digest (1 DW = 4 字节)                        │  可选：端到端 CRC-32（TD=1 时存在）
└───────────────────────────────────────────────────────────┘
```

- **3 DW Header**：用于 32 位地址 Memory 事务、I/O 事务、Config 事务以及 Completion。
- **4 DW Header**：用于 **64 位地址** Memory 事务（大于 4 GB 的物理地址空间）及 Message 事务。

---

## 2. 通用 Header 比特布局（DW 0 ~ DW 3）

### 2.1 DW 0（所有 TLP 共享的公共头部，4 字节）

```
 Byte 0                Byte 1                  Byte 2                    Byte 3
 7 6 5 4 3 2 1 0       7 6 5 4 3 2 1 0         7 6 5 4 3 2 1 0           7 6 5 4 3 2 1 0
┌─────┬─────────┬─────┬───┬─────┬─┬─┬─┬─┬─────┬─┬─┬───┬───┬───┬─────────┬─────────────────┐
│ Fmt │  Type   │  R  │TC │  R  │I│R│T│T│  EP │A│N│ AT│Len│   Length[7:0]                 │
│ (3) │   (5)   │ (1) │(3)│ (1) │D│ │H│D│ (1) │t│S│(2)│[9:8]│           (8)               │
│     │         │     │   │     │O│ │ │ │     │t│ │   │(2)│                               │
└─────┴─────────┴─────┴───┴─────┴─┴─┴─┴─┴─────┴─┴─┴───┴───┴───────────────────────────────┘
(注：Byte 1 bit 2 为 Attr[2]/IDO；Byte 2 bit 5:4 为 Attr[1:0] 即 RO 与 No Snoop)
```

### 2.2 DW 1 ~ DW 3（以 Memory Request 为例）

- **DW 1（4 字节）**：`Requester ID (16 bits: Bus/Dev/Func)` + `Tag (8 bits)` + `Last DW BE (4 bits)` + `First DW BE (4 bits)`
- **DW 2（3 DW Header 时）**：`Address[31:2] (30 bits)` + `PH (2 bits，处理提示，默认 00b)`
- **DW 2 ~ DW 3（4 DW Header 时）**：DW 2 为 `Address[63:32]`，DW 3 为 `Address[31:2] + PH[1:0]`。

*(注：对于 **Completion**，DW 1 变为 `Completer ID + Status + BCM + Byte Count`，DW 2 变为 `Requester ID + Tag + R + Lower Address`；对于 **Configuration**，DW 2 为目标 `BDF + Ext Reg + Reg`；对于 **Message**，DW 1 低 8 位为 `Message Code`。)*

---

## 3. 核心字段速查

| 字段 | 位宽 | 含义与作用 |
|------|------|-----------|
| **Fmt / Type** | 3 + 5 bits | 共同决定 Header 长度（3/4 DW）、是否带数据以及事务类型（如 `000/00000b` = 32-bit MRd，`011/00000b` = 64-bit MWr） |
| **TC (Traffic Class)** | 3 bits | 流量类别（`TC0~TC7`），在 Switch 中映射到对应的虚拟通道（`VC0~VC7`）实现 QoS 隔离 |
| **Attr (IDO / RO / NS)** | 1 + 2 bits | 事务属性：`IDO`（基于 ID 乱序）、`RO`（宽松排序）、`No Snoop`（允许跳过 CPU Cache 一致性嗅探） |
| **TH (TLP Hints)** | 1 bit | 指示存在 **TPH（TLP Processing Hints）**，用于向主机提示 Cache 预取/目标 CPU |
| **TD (TLP Digest)** | 1 bit | `1` 表示 TLP 末尾附带 1 DW 的 **ECRC** |
| **EP (Poisoned)** | 1 bit | `1` 表示数据已"毒化"（发送端已知数据损坏但仍按协议转发） |
| **AT (Address Type)** | 2 bits | 地址类型（与 **ATS** 地址转换服务配合：`00b`=未转换，IOMMU 需翻译；`10b`=已转换物理地址） |
| **Length** | 10 bits | Data Payload 长度，单位为 **DW（4 字节）**；编码 `0` 代表最大长度 `1024 DW`（4096 字节） |
| **Requester ID** | 16 bits | 发起方的 `Bus(8):Device(5):Function(3)`，用于 Completion 反向路由回原设备 |
| **Tag** | 8 bits | 事务标签（PCIe 4.0+ 扩展支持 10-bit Tag），用于将返回的 Completion 与发出的 Non-Posted 请求一一匹配 |
| **First / Last DW BE** | 4 + 4 bits | 字节使能掩码（Byte Enable），指示首/尾 DW 中哪几个字节有效，从而支持非 4 字节对齐的读写 |

---

## 4. 常见 Fmt/Type 编码速查表

| Fmt[2:0] | Type[4:0] | TLP 类型 | 路由方式 | 流控类别 |
|----------|-----------|----------|----------|----------|
| `000b` / `001b` | `00000b` | Memory Read (MRd, 3DW/4DW) | 基于地址 | Non-Posted |
| `010b` / `011b` | `00000b` | Memory Write (MWr, 3DW/4DW) | 基于地址 | Posted |
| `000b` / `010b` | `00010b` | I/O Read / Write (IORd / IOWr) | 基于地址 | Non-Posted |
| `000b` / `010b` | `00100b` | Config Read / Write Type 0 | 基于 ID | Non-Posted |
| `000b` / `010b` | `00101b` | Config Read / Write Type 1 | 基于 ID | Non-Posted |
| `001b` / `011b` | `10xxx b` | Message / Message w/ Data (Msg/MsgD) | 隐式路由 (`xxx`) | Posted |
| `000b` / `010b` | `01010b` | Completion / Completion w/ Data (Cpl/CplD) | 基于 ID | Completion |

---

## 5. 关联与引申概念

- **事务类型（Transaction Types）**：详细了解 Memory、Config、Message、Completion 与 Split Transaction 机制，见 [PCIe 事务类型详解](transaction-types.md)。
- **路由机制（Routing）**：解析 Switch 如何利用 TLP 中的 `Address`、`BDF` 或 `r[2:0]` 转发数据包，见 [路由机制详解](routing.md)。
- **ECRC & LCRC**：`TD=1` 时的端到端校验与数据链路层的逐跳校验，见 [ECRC 详解](ecrc.md) 与 [LCRC 详解](../data-link-layer/lcrc.md)。
- **TPH / ATS**：TLP Header 中 `TH` 与 `AT` 字段所对应的现代性能优化与虚拟化 IOMMU 加速特性。
