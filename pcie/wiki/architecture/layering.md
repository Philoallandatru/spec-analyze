# PCIe 三层协议模型

## 概述

PCIe 采用**分层架构**，将复杂的通信协议分解为三个独立的层次。每层负责特定的功能，层与层之间通过定义良好的接口通信。

```
┌─────────────────────────────────────────────────────────┐
│          Software Layer (软件层)                        │
│        操作系统、驱动程序                                │
└────────────────┬────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────┐
│     Transaction Layer (事务层 - TL)                     │
│  • 生成和处理 TLP (Transaction Layer Packet)            │
│  • 流控制（Flow Control）                               │
│  • 排序规则（Ordering Rules）                           │
│  • 虚拟通道（Virtual Channels）                         │
└────────────────┬────────────────────────────────────────┘
                 │ TLP
┌────────────────▼────────────────────────────────────────┐
│     Data Link Layer (数据链路层 - DLL)                  │
│  • 可靠传输（ACK/NAK 协议）                             │
│  • 重试机制（Retry Buffer）                             │
│  • CRC 错误检测（LCRC）                                 │
│  • 流控制初始化（InitFC DLLPs）                         │
└────────────────┬────────────────────────────────────────┘
                 │ DLLP + TLP
┌────────────────▼────────────────────────────────────────┐
│     Physical Layer (物理层 - PHY)                       │
│  • 数据编码/解码（8b/10b, 128b/130b）                   │
│  • 串行化/解串行化                                      │
│  • 链路训练（LTSSM）                                    │
│  • 电气信号传输                                         │
└────────────────┬────────────────────────────────────────┘
                 │ Electrical Signals
                 ▼
              物理连接 (Lane)
```

---

## 1. Transaction Layer (事务层)

### 职责

事务层是 PCIe 协议栈的**最高层**，负责：
- 构建和解析 **TLP (Transaction Layer Packet)**
- 管理**内存、IO、配置空间**的读写事务
- 实现**排序规则**保证数据一致性
- 管理**流控制**防止缓冲区溢出
- 支持**虚拟通道**实现 QoS

### TLP 类型

PCIe 定义了多种 TLP 类型：

#### 1. Memory Request (内存请求)
```
Memory Read Request (MRd)
Memory Write Request (MWr)
  └─> 64位地址支持，支持大数据传输
```

#### 2. IO Request (IO 请求)
```
IO Read Request (IORd)
IO Write Request (IOWr)
  └─> 32位地址，传统兼容性
```

#### 3. Configuration Request (配置请求)
```
Configuration Read (CfgRd0/CfgRd1)
Configuration Write (CfgWr0/CfgWr1)
  └─> 用于设备枚举和配置
```

#### 4. Message Request (消息请求)
```
Message (Msg/MsgD)
  └─> 中断、电源管理、错误报告
  └─> 例如：MSI/MSI-X 中断
```

#### 5. Completion (完成)
```
Completion (Cpl)
Completion with Data (CplD)
  └─> 对 Non-Posted 请求的响应
```

### TLP 结构

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
┌───────────────────────────────────────────────────────────────┐
│                     TLP Header (3 or 4 DW)                    │  DW = Double Word (32 bits)
│  • Fmt/Type (格式/类型)                                        │
│  • Length (长度)                                               │
│  • Requester ID (请求者 ID)                                    │
│  • Tag (标签)                                                  │
│  • Address (地址) or Completer ID                             │
├───────────────────────────────────────────────────────────────┤
│                     Data Payload (0-1024 DW)                  │  可选
│  • 实际数据内容                                                 │
│  • 最大 4096 字节                                              │
├───────────────────────────────────────────────────────────────┤
│                     ECRC (4 bytes)                            │  可选
│  • End-to-End CRC                                             │
│  • 端到端数据完整性校验                                         │
└───────────────────────────────────────────────────────────────┘
```

### 流控制 (Flow Control)

**目的**：防止发送方发送超过接收方缓冲区容量的数据。

**机制**：基于**信用 (Credit)** 的流控制
- 接收方告诉发送方可用的缓冲区空间
- 发送方消耗信用，接收方返还信用
- 无信用 = 不能发送

**三种流控制类型**：

| 类型 | 说明 | 示例 |
|------|------|------|
| **Posted (P)** | 无需响应的请求 | Memory Write, Message |
| **Non-Posted (NP)** | 需要 Completion 的请求 | Memory Read, IO R/W, Config R/W |
| **Completion (Cpl)** | 对 NP 请求的响应 | Completion, Completion with Data |

**信用单位**：
- **Header Credits (HDR)**：TLP 头部数量
- **Data Credits (DATA)**：数据负载大小（以 DW 为单位）

**流控制示例**：
```
接收方 → 发送方: "我有 10 个 Posted Header Credits, 256 DW Data Credits"

发送方发送 Memory Write (1 HDR + 64 DW Data):
  - 剩余: 9 HDR, 192 DATA

发送方发送 Memory Write (1 HDR + 128 DW Data):
  - 剩余: 8 HDR, 64 DATA

发送方发送 Memory Write (1 HDR + 64 DW Data):
  - 剩余: 7 HDR, 0 DATA  ← 数据信用耗尽，必须等待

接收方处理完数据后:
  → 发送方: "返还 256 DW Data Credits"
  - 现在: 7 HDR, 256 DATA  ← 可以继续发送
```

### 排序规则 (Ordering Rules)

PCIe 定义了严格的排序规则，保证数据一致性。

**基本原则**：
- **同类型 TLP** 通常保持顺序
- **不同类型 TLP** 可能允许乱序
- **Relaxed Ordering** 属性可放宽约束

**关键规则**（简化版）：

| 先发送 | 后发送 | 是否必须保序？ |
|--------|--------|---------------|
| Posted Write | Posted Write | ✅ 是 |
| Posted Write | Read Request | ✅ 是 |
| Read Request | Posted Write | ❌ 否（可乱序）|
| Read Request | Read Request | ✅ 是 |
| Write | Completion | ❌ 否 |

**为什么需要排序规则？**
```c
// 驱动程序代码
*device_reg = 0x1;  // Posted Write 1
*device_reg = 0x2;  // Posted Write 2
value = *device_reg;  // Read Request

如果 Write 2 先于 Write 1 到达 → 设备状态错误！
排序规则保证 Write 1 → Write 2 → Read 的顺序
```

### 虚拟通道 (Virtual Channels)

**目的**：在单个物理链路上实现**流量分类**和 **QoS (服务质量)**。

**机制**：
- 每个 TLP 携带 **TC (Traffic Class)** 标签
- TC 映射到 **VC (Virtual Channel)**
- 不同 VC 有独立的流控制缓冲区

**默认配置**：
- **VC0**：默认虚拟通道，所有设备必须支持
- **VC1-VC7**：可选，高级流量管理

**应用场景**：
- **VC0**：普通数据流量
- **VC1**：时间敏感的实时流量（音频/视频）
- **VC2**：低优先级后台流量

---

## 2. Data Link Layer (数据链路层)

### 职责

数据链路层负责**可靠传输**，确保 TLP 正确到达目的地：
- 添加 **Sequence Number (序列号)**
- 添加 **LCRC (Link CRC)** 错误检测
- 实现 **ACK/NAK 协议**
- 管理 **Retry Buffer (重试缓冲区)**
- 发送 **DLLP (Data Link Layer Packet)** 控制包

### DLLP 类型

DLLP 是数据链路层的控制包，**不携带用户数据**：

| DLLP 类型 | 功能 |
|-----------|------|
| **Ack** | 确认收到 TLP |
| **Nak** | 请求重传 TLP |
| **InitFC1/InitFC2** | 流控制初始化 |
| **UpdateFC** | 运行时更新流控制信用 |
| **PM_Enter_L1/L23** | 电源管理状态转换 |
| **Vendor Specific** | 厂商自定义 |

### ACK/NAK 协议

**工作流程**：

```
发送方                                      接收方
  │                                            │
  ├──> TLP #100 (Seq=100, Data, LCRC) ───────>│
  │                                            ├─> 检查 LCRC
  │                                            ├─> LCRC OK
  │    <────────── Ack DLLP (Seq=100) ────────┤
  │                                            │
  ├─> TLP 从 Retry Buffer 中删除              │
  │                                            │
  ├──> TLP #101 (Seq=101, Data, LCRC) ───────>│
  │                                            ├─> 检查 LCRC
  │                                            ├─> LCRC Error!
  │    <────────── Nak DLLP (Seq=101) ────────┤
  │                                            │
  ├─> 从 Retry Buffer 重传 TLP #101           │
  ├──> TLP #101 (重传) ──────────────────────>│
  │                                            ├─> LCRC OK
  │    <────────── Ack DLLP (Seq=101) ────────┤
  │                                            │
```

**关键特性**：
- **12 位序列号**：0-4095 循环使用
- **超时重传**：如果没收到 ACK，Replay Timer 超时后重传
- **Retry Buffer**：发送方保存已发送但未确认的 TLP

### LCRC (Link CRC)

**定义**：32 位 CRC-32 校验和，覆盖整个 TLP（包括头部和数据）。

**作用**：
- 检测链路层的**位错误**
- 由数据链路层自动添加和验证
- 软件不可见，硬件自动处理

**与 ECRC 的区别**：

| 特性 | LCRC | ECRC |
|------|------|------|
| **覆盖范围** | 每一跳（逐跳） | 端到端（整个路径） |
| **添加位置** | 数据链路层 | 事务层 |
| **保护对象** | 链路传输错误 | Switch 内部错误、内存错误 |
| **强制性** | ✅ 必须 | ❌ 可选 |

---

## 3. Physical Layer (物理层)

### 职责

物理层是 PCIe 协议栈的**最底层**，负责：
- **数据编码/解码**：8b/10b 或 128b/130b
- **串行化/解串行化**：并行数据 ↔ 串行比特流
- **链路训练**：速度和宽度协商（LTSSM）
- **时钟恢复**：从数据流中提取时钟
- **电气信号传输**：差分信号驱动

### 物理层结构

```
┌─────────────────────────────────────────────────────┐
│            Physical Layer Logical Block              │
│  • Framing (成帧)                                    │
│  • Scrambling (加扰)                                 │
│  • Encoding (编码: 8b/10b or 128b/130b)             │
│  • LTSSM (Link Training State Machine)              │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│          Physical Layer Electrical Block             │
│  • TX Driver (发送驱动器)                            │
│  • RX Equalizer (接收均衡器)                         │
│  • Clock Recovery (时钟恢复)                         │
│  • De-emphasis (去加重)                              │
└─────────────────┬───────────────────────────────────┘
                  │
                  ▼
           Differential Pairs (差分对)
            TX+/TX-, RX+/RX-
```

### 编码方案

#### 8b/10b 编码 (PCIe 1.0/2.0)

**特点**：
- 每 8 位数据编码为 10 位符号
- **效率**：80% (8/10)
- **DC 平衡**：保证 0 和 1 数量接近
- **时钟嵌入**：足够的跳变用于时钟恢复

**特殊符号**：
- **COM (K28.5)**：链路训练时的对齐符号
- **SKP**：时钟补偿符号
- **EDB (End Bad)**：错误结束标记

#### 128b/130b 编码 (PCIe 3.0+)

**特点**：
- 每 128 位数据添加 2 位同步头
- **效率**：98.5% (128/130)
- **更高吞吐量**：减少编码开销

**同步头**：
- `00b`：数据块
- `01b`：Ordered Set 块
- `10b`：保留
- `11b`：保留

### 链路训练 (Link Training)

**LTSSM (Link Training and Status State Machine)** 管理链路的整个生命周期：

```
         Power-On
            │
            ▼
      ┌──────────┐
      │  Detect  │  检测是否有设备连接
      └────┬─────┘
           │
           ▼
      ┌──────────┐
      │ Polling  │  交换训练序列，速度协商
      └────┬─────┘
           │
           ▼
      ┌──────────┐
      │  Config  │  配置 Lane，宽度协商
      └────┬─────┘
           │
           ▼
      ┌──────────┐
      │    L0    │  正常工作状态（数据传输）
      └────┬─────┘
           │
           ├──> L0s (快速省电)
           ├──> L1  (深度省电)
           └──> L2  (超深度省电)
```

**关键状态**：
- **Detect**：检测链路对端是否存在
- **Polling**：比特锁定和符号锁定，速度协商
- **Configuration**：Lane 宽度协商，分配 Lane 编号
- **L0**：数据传输状态
- **Recovery**：链路恢复和重新训练

### Lane (通道)

**定义**：PCIe 的最小传输单元，由 **4 根信号线**组成：
- **TX+ / TX-**：发送差分对
- **RX+ / RX-**：接收差分对

**全双工通信**：
```
设备 A                           设备 B
  ├───> TX+/TX- ────────────> RX+/RX- ┤
  │                                    │
  ├───< RX+/RX- <──────────── TX+/TX- ┤
```

**多 Lane 配置**：
- **x1**：1 Lane = 4 根线
- **x4**：4 Lane = 16 根线
- **x16**：16 Lane = 64 根线

**Lane 反转和极性翻转**：
- 物理层自动检测并纠正连接错误
- 支持 TX/RX 交换、正负极性反转

### 电气特性

**信号电压**：
- **差分电压**：800-1200 mV (peak-to-peak)
- **共模电压**：0 V (接地参考)

**阻抗**：
- **差分阻抗**：100Ω ± 15%
- **单端阻抗**：50Ω

**信号速率与数据率**：

| PCIe Gen | 信号速率 | 编码 | 每 Lane 数据率 |
|----------|---------|------|---------------|
| Gen 1 | 2.5 GHz | 8b/10b | 2.0 Gbps (250 MB/s) |
| Gen 2 | 5.0 GHz | 8b/10b | 4.0 Gbps (500 MB/s) |
| Gen 3 | 8.0 GHz | 128b/130b | 7.877 Gbps (985 MB/s) |
| Gen 4 | 16.0 GHz | 128b/130b | 15.754 Gbps (1969 MB/s) |
| Gen 5 | 32.0 GHz | 128b/130b | 31.508 Gbps (3938 MB/s) |

---

## 层间交互示例

### 发送路径（TX）

```
应用程序: 写入 4KB 数据到设备内存

    ↓

驱动程序: 调用 PCIe 内存写函数

    ↓

┌─────────────────────────────────────────┐
│ Transaction Layer (事务层)              │
├─────────────────────────────────────────┤
│ 1. 创建 Memory Write TLP                │
│    - Header: 目标地址, 长度, Requester │
│    - Data: 4KB 负载                     │
│ 2. 检查流控制信用                        │
│ 3. 添加 ECRC (可选)                     │
└──────────────┬──────────────────────────┘
               │ TLP
               ▼
┌─────────────────────────────────────────┐
│ Data Link Layer (数据链路层)            │
├─────────────────────────────────────────┤
│ 1. 添加 Sequence Number                 │
│ 2. 计算并添加 LCRC                       │
│ 3. 存入 Retry Buffer                    │
└──────────────┬──────────────────────────┘
               │ TLP + LCRC
               ▼
┌─────────────────────────────────────────┐
│ Physical Layer (物理层)                 │
├─────────────────────────────────────────┤
│ 1. 添加 Framing (STP/END)               │
│ 2. Scrambling (加扰)                    │
│ 3. Encoding (8b/10b or 128b/130b)      │
│ 4. Serialization (串行化)               │
└──────────────┬──────────────────────────┘
               │ Serial Bit Stream
               ▼
            物理链路 (Lane)
```

### 接收路径（RX）

```
            物理链路 (Lane)
               │ Serial Bit Stream
               ▼
┌─────────────────────────────────────────┐
│ Physical Layer (物理层)                 │
├─────────────────────────────────────────┤
│ 1. Deserialization (解串行化)           │
│ 2. Decoding (解码)                      │
│ 3. Descrambling (解扰)                  │
│ 4. Framing 检测                         │
└──────────────┬──────────────────────────┘
               │ TLP + LCRC
               ▼
┌─────────────────────────────────────────┐
│ Data Link Layer (数据链路层)            │
├─────────────────────────────────────────┤
│ 1. 验证 LCRC                            │
│ 2. LCRC OK → 发送 Ack DLLP             │
│ 3. LCRC Error → 发送 Nak DLLP          │
│ 4. 检查 Sequence Number                │
└──────────────┬──────────────────────────┘
               │ TLP
               ▼
┌─────────────────────────────────────────┐
│ Transaction Layer (事务层)              │
├─────────────────────────────────────────┤
│ 1. 解析 TLP Header                      │
│ 2. 验证 ECRC (如果存在)                 │
│ 3. 路由到目标地址                        │
│ 4. 更新流控制信用                        │
└──────────────┬──────────────────────────┘
               │
               ▼

驱动程序: 中断处理，通知应用数据到达
```

---

## FEMU 代码中的层次实现

### Transaction Layer

```c
// hw/pci/pcie.c - PCIe 能力初始化
int pcie_cap_init(PCIDevice *dev, uint8_t offset,
                  uint8_t type, uint8_t version)
{
    // 设置事务层能力
    pci_set_word(dev->config + offset + PCI_EXP_DEVCTL, 0);
    // ...
}
```

### Data Link Layer

```c
// hw/pci/pcie_aer.c - 错误处理（数据链路层功能）
void pcie_aer_inject_error(PCIDevice *dev, const PCIEAERErr *err)
{
    // 注入链路层错误
    // 触发重试机制
}
```

### Physical Layer

```c
// hw/pci/pcie.c - 链路状态管理
void pcie_cap_fill_link_ep_usp(PCIDevice *dev, uint8_t speed, uint8_t width)
{
    // 配置链路速度和宽度
    uint32_t lnkcap = (speed << PCI_EXP_LNKCAP_SLS_SHIFT) |
                      (width << PCI_EXP_LNKCAP_MLW_SHIFT);
    pci_set_long(dev->config + offset + PCI_EXP_LNKCAP, lnkcap);
}
```

---

## 总结

### 关键要点

1. ✅ **三层分离**：事务层、数据链路层、物理层各司其职
2. ✅ **TLP**：事务层的数据包，携带实际事务
3. ✅ **DLLP**：数据链路层的控制包，用于 ACK/NAK 和流控制
4. ✅ **可靠传输**：通过序列号、LCRC、重试机制保证
5. ✅ **流控制**：基于信用，防止缓冲区溢出
6. ✅ **链路训练**：LTSSM 管理链路状态和参数协商
7. ✅ **编码**：8b/10b (Gen1/2) 或 128b/130b (Gen3+)

---

## 下一步学习

### 深入各层

- [TLP 格式详解](../transaction-layer/tlp-format.md)
- [DLLP 详解](../data-link-layer/dllp-format.md)
- [LTSSM 状态机](../physical-layer/ltssm.md)

### 相关主题

- [流控制机制](../transaction-layer/flow-control.md)
- [排序规则](../transaction-layer/ordering.md)
- [ACK/NAK 协议](../data-link-layer/ack-nak.md)

---

## 参考资料

- **规范**：PCIe Base Spec Chapter 1-4
- **实现**：FEMU `/hw/pci/pcie.c`, `/hw/pci/pcie_aer.c`

---

**相关页面**：
- [← 拓扑结构](../introduction/topology.md)
- [TLP 格式 →](../transaction-layer/tlp-format.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
