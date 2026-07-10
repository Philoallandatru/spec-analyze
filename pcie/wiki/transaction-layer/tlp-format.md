# TLP 格式详解

Transaction Layer Packet (TLP) 是 PCIe 通信的基本单元，携带所有的读写事务、消息和完成响应。

---

## 概述

TLP 是**事务层**生成的数据包，包含：
- **Header**（头部）：控制信息和路由信息
- **Data Payload**（数据负载）：实际数据（可选）
- **ECRC**（端到端 CRC）：数据完整性校验（可选）

**TLP 在协议栈中的位置**：
```
应用层数据
    ↓
┌─────────────────────────────────┐
│  Transaction Layer              │
│  生成 TLP：Header + Data        │
└─────────────┬───────────────────┘
              │ TLP
┌─────────────▼───────────────────┐
│  Data Link Layer                │
│  添加 Seq# + LCRC               │
└─────────────┬───────────────────┘
              │ TLP + LCRC
┌─────────────▼───────────────────┐
│  Physical Layer                 │
│  Framing + Encoding             │
└─────────────────────────────────┘
```

---

## TLP 基本结构

### 通用格式

```
┌────────────────────────────────────────────────────┐
│           TLP Header (3 or 4 DW)                   │
│  Byte 0-11 (3 DW) or 0-15 (4 DW)                   │
├────────────────────────────────────────────────────┤
│           Data Payload (0-1024 DW)                 │  可选
│  最大 4096 字节                                     │
├────────────────────────────────────────────────────┤
│           ECRC (1 DW = 4 bytes)                    │  可选
│  端到端 CRC 校验                                    │
└────────────────────────────────────────────────────┘

DW = Double Word = 32 bits = 4 bytes
```

### Header 格式分类

PCIe 定义了两种 Header 长度：

| Header 类型 | 大小 | 用途 |
|------------|------|------|
| **3 DW Header** | 12 字节 | 32 位地址事务、消息、完成 |
| **4 DW Header** | 16 字节 | 64 位地址事务 |

---

## TLP Header 字段详解

### 3 DW Header 格式（通用）

```
Byte 0-3 (DW 0):
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
┌───┬───┬─────┬─┬─┬─┬─┬─┬───┬─────────────┬─────────┬─────────┐
│Fmt│Type │ R │T│T│A│ R │Len(H)│   Length[7:0]   │         │
│   │     │   │C│E│t│   │      │                  │         │
│(3)│ (5) │(1)│ │ │t│(2)│ (2)  │      (8)         │   (8)   │
└───┴─────┴───┴─┴─┴─┴───┴──────┴──────────────────┴─────────┘
 Fmt: Format (3 bits)                     Length: 数据长度（DW）
 Type: Type (5 bits)                      
 R: Reserved                              
 TC: Traffic Class (3 bits)               
 Attr: Attributes (2 bits)                
 TH: TLP Processing Hints (1 bit)        
 TD: TLP Digest (ECRC) Present (1 bit)   
 EP: Poisoned (1 bit)                     
 Attr: Attributes (1 bit)                 
 AT: Address Type (2 bits)                

Byte 4-7 (DW 1):
┌─────────────────┬───────┬─────────────────────────┐
│  Requester ID   │  Tag  │  Last DW BE  │ 1st DW BE│
│   (16 bits)     │ (8)   │     (4)      │   (4)    │
└─────────────────┴───────┴──────────────┴──────────┘
 Requester ID: Bus:Device.Function (8:5:3)
 Tag: 用于匹配 Request 和 Completion
 BE: Byte Enable（首/末 DW 的字节使能）

Byte 8-11 (DW 2):
┌───────────────────────────────────────────────────┐
│           Address[31:2] / Other Fields            │
│                   (30 bits)                       │
└───────────────────────────────────────────────────┘
 对于 Memory Request: 32 位地址（DW 对齐，低 2 位为 0）
 对于 Message: Message Code
 对于 Completion: Completer ID + Status
```

### 4 DW Header 格式（64 位地址）

```
Byte 0-7 (DW 0-1): 与 3 DW 相同

Byte 8-11 (DW 2):
┌───────────────────────────────────────────────────┐
│              Address[63:32]                       │
│                 (32 bits)                         │
└───────────────────────────────────────────────────┘

Byte 12-15 (DW 3):
┌───────────────────────────────────────────────────┐
│              Address[31:2]                        │
│                 (30 bits)                         │
└───────────────────────────────────────────────────┘
```

---

## 关键字段说明

### Fmt (Format) - 3 bits

指示 TLP 的基本格式：

| Fmt | 含义 |
|-----|------|
| 000b | 3 DW Header, No Data |
| 001b | 4 DW Header, No Data |
| 010b | 3 DW Header, With Data |
| 011b | 4 DW Header, With Data |
| 100b | TLP Prefix (PCIe 3.0+) |

### Type - 5 bits

结合 Fmt，完整定义 TLP 类型：

| Fmt[2:0] | Type[4:0] | TLP 类型 |
|----------|-----------|---------|
| 000b | 00000b | Memory Read Request (MRd) |
| 010b | 00000b | Memory Write Request (MWr) - 32 bit |
| 001b | 00000b | Memory Read Request (MRd) - 64 bit |
| 011b | 00000b | Memory Write Request (MWr) - 64 bit |
| 000b | 00010b | IO Read Request (IORd) |
| 010b | 00010b | IO Write Request (IOWr) |
| 000b | 00100b | Config Read Type 0 (CfgRd0) |
| 010b | 00100b | Config Write Type 0 (CfgWr0) |
| 000b | 00101b | Config Read Type 1 (CfgRd1) |
| 010b | 00101b | Config Write Type 1 (CfgWr1) |
| 000b | 1****b | Message (Msg) |
| 010b | 1****b | Message with Data (MsgD) |
| 000b | 01010b | Completion (Cpl) |
| 010b | 01010b | Completion with Data (CplD) |

### TC (Traffic Class) - 3 bits

流量类别，用于虚拟通道映射和 QoS：
- **0-7**：8 个流量类别
- 默认：TC0

### Length - 10 bits

数据负载长度，单位为 **DW (4 字节)**：
- **0**：表示 1024 DW (4096 字节)
- **1-1023**：实际 DW 数量
- **最大**：4096 字节

### Requester ID - 16 bits

请求者的 BDF 地址：
```
┌────────────┬──────────────┬───────────────┐
│   Bus      │   Device     │   Function    │
│  (8 bits)  │   (5 bits)   │   (3 bits)    │
└────────────┴──────────────┴───────────────┘

示例：Bus 3, Device 0, Function 0 → 0x0300
```

### Tag - 8 bits

用于匹配 Non-Posted Request 和其 Completion：
- 发起方生成唯一 Tag
- Completion 必须携带相同的 Tag
- 允许 256 个并发未完成请求

### Byte Enable (BE)

指示 DW 中哪些字节有效：

```
First DW BE (4 bits): 第一个 DW 的字节使能
Last DW BE (4 bits):  最后一个 DW 的字节使能

示例：读取 4 字节（1 DW）对齐数据
  First DW BE = 1111b (全部字节)
  Last DW BE  = 0000b (无最后 DW)

示例：读取非对齐的 3 字节（从偏移 1 开始）
  First DW BE = 1110b (字节 1, 2, 3)
  Last DW BE  = 0000b
```

### EP (Error/Poisoned) - 1 bit

指示 TLP 数据可能不可靠：
- **0**：数据正常
- **1**：数据被"毒化"（检测到错误但仍转发）

### TD (TLP Digest) - 1 bit

指示是否包含 ECRC：
- **0**：无 ECRC
- **1**：TLP 末尾包含 ECRC

---

## 常见 TLP 类型详解

### 1. Memory Read Request (MRd)

**用途**：从内存地址读取数据

**格式**：
```
3 DW Header (32-bit 地址):
┌────────────────────────────────────────────┐
│ Fmt=000 | Type=00000 | ... | Length       │  DW 0
├────────────────────────────────────────────┤
│ Requester ID | Tag | Last BE | First BE   │  DW 1
├────────────────────────────────────────────┤
│ Address[31:2] | 00                         │  DW 2
└────────────────────────────────────────────┘

4 DW Header (64-bit 地址):
+ DW 2: Address[63:32]
+ DW 3: Address[31:2]

无 Data Payload（读请求不携带数据）
```

**示例**：
```c
// 读取地址 0x1000，长度 64 字节（16 DW）
Memory Read Request:
  Fmt/Type: 0x00 (3DW, no data)
  Length: 16 (0x10)
  Requester ID: 0x0300 (Bus 3, Dev 0, Func 0)
  Tag: 0x05
  First DW BE: 0xF (全部字节)
  Last DW BE: 0xF
  Address: 0x1000
```

### 2. Memory Write Request (MWr)

**用途**：向内存地址写入数据（Posted）

**格式**：
```
3 DW Header + Data:
┌────────────────────────────────────────────┐
│ Fmt=010 | Type=00000 | ... | Length       │  DW 0
├────────────────────────────────────────────┤
│ Requester ID | Tag | Last BE | First BE   │  DW 1
├────────────────────────────────────────────┤
│ Address[31:2] | 00                         │  DW 2
├────────────────────────────────────────────┤
│ Data Payload (Length DW)                   │
│ ...                                        │
└────────────────────────────────────────────┘
```

**特点**：
- **Posted**：无需 Completion 响应
- 高效，但无法确认写入成功

**示例**：
```c
// 写入 4 字节到地址 0x2000
Memory Write Request:
  Fmt/Type: 0x40 (3DW, with data)
  Length: 1 (1 DW)
  Requester ID: 0x0300
  Address: 0x2000
  Data: 0x12345678
```

### 3. IO Read/Write Request

**用途**：访问 x86 架构的 IO 地址空间（已较少使用）

**格式**：与 Memory Request 类似，但 Type 不同

**限制**：
- 仅 32 位地址
- 最大 4 字节传输
- Non-Posted（需要 Completion）

### 4. Configuration Read/Write Request

**用途**：访问设备的配置空间

**Type 0**：访问直连设备
**Type 1**：通过 Bridge 访问下游设备

**格式**：
```
┌────────────────────────────────────────────┐
│ Fmt=000/010 | Type=00100 | ... | Length   │  DW 0
├────────────────────────────────────────────┤
│ Requester ID | Tag | Last BE | First BE   │  DW 1
├────────────────────────────────────────────┤
│ Bus | Dev | Func | Ext Reg | Reg | 00     │  DW 2
└────────────────────────────────────────────┘

Bus: 8 bits (目标总线)
Device: 5 bits (目标设备)
Function: 3 bits (目标功能)
Ext Reg: 4 bits (扩展寄存器号，PCIe)
Reg: 6 bits (寄存器号)
```

**用途示例**：
```c
// 读取 Bus 1, Device 0, Function 0 的 Vendor ID (偏移 0x00)
Config Read Type 0:
  Fmt/Type: 0x04 (3DW, no data, CfgRd0)
  Length: 1
  Requester ID: 0x0000 (Root Complex)
  Tag: 0x01
  Bus: 1, Device: 0, Function: 0
  Register: 0x00
  
响应 Completion with Data:
  Data: 0x80861234 (Intel Vendor ID = 0x8086)
```

### 5. Message Request (Msg/MsgD)

**用途**：事件通知（中断、电源管理、错误报告等）

**格式**：
```
┌────────────────────────────────────────────┐
│ Fmt=000/010 | Type=1xxxx | ... | Length   │  DW 0
├────────────────────────────────────────────┤
│ Requester ID | Tag | Message Code         │  DW 1
├────────────────────────────────────────────┤
│ Lower Address / Message-Specific          │  DW 2
├────────────────────────────────────────────┤
│ Upper Address (if 4 DW)                    │  DW 3 (可选)
├────────────────────────────────────────────┤
│ Message-Specific Data (if MsgD)            │  Data (可选)
└────────────────────────────────────────────┘
```

**常见消息类型**：

| Message Code | 类型 | 用途 |
|--------------|------|------|
| 0x00 | Unlock | 解锁（已废弃）|
| 0x10 | INTx Interrupt | 传统中断断言 |
| 0x11 | INTx Interrupt | 传统中断取消 |
| 0x14 | PM_Active_State_Nak | 电源管理拒绝 |
| 0x18 | PM_PME | 电源管理事件 |
| 0x20 | ERR_COR | 可纠正错误 |
| 0x21 | ERR_NONFATAL | 非致命错误 |
| 0x22 | ERR_FATAL | 致命错误 |
| 0x30 | MSI/MSI-X | 中断消息 |

**MSI 中断示例**：
```c
// 发送 MSI 中断
Message Request (MWr format, 实际上是 Memory Write):
  Address: 0xFEE00000 (MSI 地址)
  Data: 0x4000 (中断向量)
```

### 6. Completion (Cpl/CplD)

**用途**：响应 Non-Posted 请求

**格式**：
```
┌────────────────────────────────────────────┐
│ Fmt=000/010 | Type=01010 | ... | Length   │  DW 0
├────────────────────────────────────────────┤
│ Completer ID | Status | BCM | Byte Count  │  DW 1
├────────────────────────────────────────────┤
│ Requester ID | Tag | R | Lower Address    │  DW 2
├────────────────────────────────────────────┤
│ Data (if CplD)                             │  Data (可选)
└────────────────────────────────────────────┘

Completer ID: 完成者的 BDF
Status: 完成状态 (3 bits)
BCM: Byte Count Modified
Byte Count: 剩余字节数
Lower Address: 数据在读请求中的偏移
```

**Status 字段**：

| Status | 含义 |
|--------|------|
| 000b | Successful Completion (SC) |
| 001b | Unsupported Request (UR) |
| 010b | Configuration Request Retry Status (CRS) |
| 100b | Completer Abort (CA) |

**Completion 示例**：
```c
// 响应之前的 Memory Read (Tag=0x05, 64 字节)
Completion with Data:
  Fmt/Type: 0x4A (3DW, with data, CplD)
  Length: 16 (64 字节 = 16 DW)
  Completer ID: 0x0100 (设备地址)
  Status: 000 (成功)
  Byte Count: 64
  Requester ID: 0x0300 (原请求者)
  Tag: 0x05 (匹配原请求)
  Data: [64 字节数据]
```

---

## TLP 路由机制

PCIe 定义了三种路由方式：

### 1. Address-Based Routing (基于地址)

**用于**：Memory Request

**机制**：
- Switch/Bridge 比较 TLP 地址与自己管理的地址窗口
- 如果匹配，转发到相应端口
- 如果不匹配，转发到上游

**示例**：
```
Memory Write to 0xF000_0000
  ↓
Root Complex 检查: 不在本地 → 转发到 Port 1
  ↓
Switch 检查: 在 Port 3 的窗口 (0xF000_0000-0xF0FF_FFFF)
  ↓
转发到 Port 3 → 到达目标设备
```

### 2. ID-Based Routing (基于 ID)

**用于**：Configuration Request, Completion

**机制**：
- 使用 Bus:Device.Function 路由
- Bridge 比较目标 Bus 号与自己管理的 Bus 范围
- Completion 使用 Requester ID 反向路由

**示例**：
```
Config Read to Bus 5, Device 0, Function 0
  ↓
Root Complex (管理 Bus 0)
  ↓
Bridge 1 (管理 Bus 1-3): Bus 5 不在范围 → 不转发
  ↓
Bridge 2 (管理 Bus 4-6): Bus 5 在范围 → 转发
  ↓
到达 Bus 5 的设备
```

### 3. Implicit Routing (隐式路由)

**用于**：某些 Message (如 INTx 中断)

**机制**：
- 根据消息类型和端口类型决定
- 例如：INTx 中断总是路由到 Root Complex

---

## TLP 处理流程

### 发送方（Transmitter）

```
1. 应用层发起事务请求
   ↓
2. 事务层检查流控制信用
   ├─> 无信用 → 等待
   └─> 有信用 → 继续
   ↓
3. 构建 TLP Header
   - 设置 Fmt/Type
   - 填充地址/ID
   - 设置 Length, Tag, BE
   ↓
4. 添加 Data Payload (如果有)
   ↓
5. 计算并添加 ECRC (如果启用)
   ↓
6. 消耗流控制信用
   ↓
7. 传递给数据链路层
```

### 接收方（Receiver）

```
1. 从数据链路层接收 TLP
   ↓
2. 验证 ECRC (如果存在)
   ├─> 错误 → 标记为 Poisoned 或丢弃
   └─> 正确 → 继续
   ↓
3. 解析 Header
   - 提取 Fmt/Type
   - 提取地址/ID
   - 提取 Length
   ↓
4. 路由判断
   ├─> 本设备 → 处理
   └─> 其他设备 → 转发
   ↓
5. 处理事务
   - Memory: 访问内存
   - Config: 访问配置空间
   - Message: 处理事件
   ↓
6. 生成 Completion (如果是 Non-Posted)
   ↓
7. 更新流控制信用
```

---

## FEMU 代码中的 TLP 处理

### TLP 格式定义

```c
// include/hw/pci/pcie_regs.h
// TLP Fmt 和 Type 定义
#define PCI_EXP_TYPE_ENDPOINT     0x0
#define PCI_EXP_TYPE_LEG_END      0x1
#define PCI_EXP_TYPE_ROOT_PORT    0x4
// ...
```

### TLP 错误注入（用于测试）

```c
// hw/pci/pcie_aer.c
void pcie_aer_inject_error(PCIDevice *dev, const PCIEAERErr *err)
{
    // 注入 TLP 错误（如 Poisoned TLP）
    if (err->flags & PCIE_AER_ERR_TLP_PREFIX_BLOCKED) {
        // 处理 TLP 前缀错误
    }
    // ...
}
```

---

## 实用技巧

### 调试 TLP

**使用 lspci 查看设备能力**：
```bash
lspci -vvv -s 01:00.0 | grep -i "maxpayload\|maxreadreq"
  MaxPayload 256 bytes, MaxReadReq 512 bytes
```

**使用协议分析仪**：
- 硬件：Lecroy, Teledyne, Keysight 的 PCIe 分析仪
- 软件：Wireshark (部分支持)

### 性能优化

1. **增加 Max Payload Size**：
   - 更大的 TLP 减少头部开销
   - 需要整个路径支持

2. **使用 Relaxed Ordering**：
   - 允许某些 TLP 乱序
   - 提高带宽利用率

3. **批量传输**：
   - 合并多个小请求为大请求
   - 减少 TLP 数量

---

## 总结

### 关键要点

1. ✅ **TLP 是 PCIe 的数据单元**，携带所有事务
2. ✅ **Header 分 3 DW 和 4 DW**，支持 32/64 位地址
3. ✅ **Fmt/Type 组合定义 TLP 类型**（Memory、IO、Config、Message、Completion）
4. ✅ **三种路由方式**：地址、ID、隐式
5. ✅ **Tag 用于匹配**Request 和 Completion
6. ✅ **流控制基于信用**，防止缓冲区溢出

---

## 下一步学习

- [事务类型详解](transaction-types.md) - 各种 TLP 的使用场景
- [流控制机制](flow-control.md) - 信用管理的详细机制
- [排序规则](ordering.md) - TLP 的排序约束
- [DLLP 格式](../data-link-layer/dllp-format.md) - 数据链路层包

---

## 参考资料

- **规范**：PCIe Base Spec Chapter 2 (Transaction Layer)
- **图表**：Figure 2-3 to 2-10 (TLP 格式)
- **表格**：Table 2-3 (Fmt/Type 编码)
- **实现**：`/hw/pci/pcie_aer.c` (FEMU)

---

**相关页面**：
- [← 三层协议模型](../architecture/layering.md)
- [流控制 →](flow-control.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
