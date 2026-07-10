# PCIe 事务类型详解

PCIe 定义了多种事务类型（Transaction Types），用于实现不同的数据传输和控制操作。每种事务类型都有特定的用途、格式和处理规则。

---

## 概述

PCIe 事务通过 TLP (Transaction Layer Packet) 承载，主要分为以下几大类：

- **Memory Transactions**：内存读写操作
- **I/O Transactions**：I/O 空间读写操作
- **Configuration Transactions**：配置空间访问
- **Message Transactions**：事件通知和带外信号
- **Completion Transactions**：对 Non-Posted 请求的响应

### 事务分类方式

```
按响应机制分类：
├─ Posted Transactions（已发布事务）
│  └─ 无需 Completion 响应，高效但无确认
│     例：Memory Write, Message
│
└─ Non-Posted Transactions（未发布事务）
   └─ 需要 Completion 响应，确保可靠性
      例：Memory Read, I/O Read/Write, Config Read/Write

按功能分类：
├─ Memory Transactions（内存事务）
├─ I/O Transactions（I/O 事务）
├─ Configuration Transactions（配置事务）
├─ Message Transactions（消息事务）
└─ Completion Transactions（完成事务）
```

---

## 1. Memory Transactions（内存事务）

Memory 事务用于访问系统内存地址空间，是 PCIe 中最常用的事务类型。

### 1.1 Memory Read Request (MRd)

**用途**：从指定内存地址读取数据

**格式**：
```
Header Only (No Data)
┌────────────────────────────────────────┐
│ Fmt=000/001 (3DW/4DW), Type=00000     │
│ Length: 请求的数据长度（DW）            │
│ Requester ID: 请求者的 BDF             │
│ Tag: 事务标识                          │
│ Address: 32-bit 或 64-bit 地址         │
└────────────────────────────────────────┘
```

**特点**：
- **Non-Posted**：需要 Completion 返回数据
- 支持 32-bit 和 64-bit 地址
- 最大读取长度受 Max_Read_Request_Size 限制（典型 512 字节或 4096 字节）

**使用场景**：
- DMA 设备读取系统内存中的命令队列
- CPU 读取设备的内存映射寄存器（MMIO）
- NVMe 控制器读取 SQ/CQ 数据

**FEMU 代码示例**：
```c
// hw/femu/nvme.c - NVMe 控制器通过 DMA 读取主机内存
static void nvme_rw_cb(void *opaque, int ret)
{
    NvmeRequest *req = opaque;
    NvmeSQueue *sq = req->sq;
    
    // 使用 PCIe Memory Read 从主机内存读取数据
    pci_dma_read(&n->parent_obj, addr, ptr, len);
}
```

**性能优化**：
```
Max_Read_Request_Size 影响性能：
- 128 字节：TLP 数量多，头部开销大
- 512 字节：平衡的选择（默认）
- 4096 字节：最大化带宽，但需要全路径支持

查看设备配置：
$ lspci -vvv -s 01:00.0 | grep MaxReadReq
  MaxReadReq 512 bytes
```

### 1.2 Memory Write Request (MWr)

**用途**：向指定内存地址写入数据

**格式**：
```
Header + Data Payload
┌────────────────────────────────────────┐
│ Fmt=010/011 (3DW/4DW), Type=00000     │
│ Length: 数据长度（DW）                  │
│ Requester ID: 请求者的 BDF             │
│ Address: 32-bit 或 64-bit 地址         │
├────────────────────────────────────────┤
│ Data Payload (Length × 4 bytes)       │
│ ...                                    │
└────────────────────────────────────────┘
```

**特点**：
- **Posted**：无需 Completion，高效
- 不保证写入完成的时刻，只保证顺序
- 最大写入长度受 Max_Payload_Size 限制（典型 128-512 字节）

**使用场景**：
- DMA 设备写入数据到系统内存
- CPU 写入设备的 MMIO 寄存器
- NVMe 控制器更新 CQ 头指针

**FEMU 代码示例**：
```c
// hw/femu/nvme.c - NVMe 通过 DMA 写入主机内存
static uint16_t nvme_write(NvmeCtrl *n, NvmeNamespace *ns, NvmeCmd *cmd,
                          NvmeRequest *req)
{
    // 使用 PCIe Memory Write 向主机内存写入数据
    pci_dma_write(&n->parent_obj, prp_list_addr, 
                  prp_ents, prp_trans);
}
```

**Posted vs Non-Posted 对比**：

| 特性 | Posted (MWr) | Non-Posted (MRd) |
|------|--------------|------------------|
| 需要 Completion | 否 | 是 |
| 延迟 | 低 | 高（需等待响应）|
| 确认机制 | 无 | 有 |
| 适用场景 | 高吞吐量写入 | 需要读取数据 |
| 示例 | DMA 写数据 | DMA 读描述符 |

### 1.3 Memory Read Lock (MRdLk) - 已废弃

**用途**：原子读-修改-写操作（PCIe 3.0+ 已废弃）

**替代方案**：使用 **AtomicOp**（原子操作）

---

## 2. I/O Transactions（I/O 事务）

I/O 事务用于访问 x86 架构的 I/O 地址空间（0-64K），现代设备较少使用。

### 2.1 I/O Read Request (IORd)

**格式**：
```
┌────────────────────────────────────────┐
│ Fmt=000, Type=00010                    │
│ Length: 1 (仅支持 1 DW = 4 字节)       │
│ Address: 32-bit I/O 地址（低 16 位有效）│
└────────────────────────────────────────┘
```

**限制**：
- 仅支持 32-bit 地址（实际只用低 16 位）
- 单次最大 4 字节
- **Non-Posted**

### 2.2 I/O Write Request (IOWr)

**格式**：
```
┌────────────────────────────────────────┐
│ Fmt=010, Type=00010                    │
│ Length: 1                              │
│ Address: 32-bit I/O 地址               │
│ Data: 1-4 字节                         │
└────────────────────────────────────────┘
```

**使用场景**：
- 传统 x86 设备（如串口 0x3F8）
- Legacy 模式兼容

**现代替代方案**：
- 使用 MMIO（Memory-Mapped I/O）替代 Port I/O
- 更高效，支持更大传输，跨架构

---

## 3. Configuration Transactions（配置事务）

配置事务用于访问 PCIe 设备的配置空间。

### 3.1 Configuration Read/Write Type 0

**用途**：访问**直接连接**的设备配置空间

**格式**：
```
┌─────────────────────────────────────────┐
│ Fmt=000/010, Type=00100 (Read/Write)   │
│ Bus Number: 目标总线号                  │
│ Device Number: 目标设备号 (5 bits)      │
│ Function Number: 目标功能号 (3 bits)    │
│ Register Number: 寄存器偏移 (12 bits)   │
└─────────────────────────────────────────┘
```

**路由方式**：
- 不检查 Bus 号（隐式认为是当前总线）
- 使用 IDSEL 信号或设备号匹配

### 3.2 Configuration Read/Write Type 1

**用途**：通过 Bridge 访问**下游设备**的配置空间

**格式**：
```
┌─────────────────────────────────────────┐
│ Fmt=000/010, Type=00101                │
│ Bus Number: 目标总线号                  │
│ Device Number: 目标设备号               │
│ Function Number: 目标功能号             │
│ Register Number: 寄存器偏移             │
└─────────────────────────────────────────┘
```

**路由过程**：
```
1. Root Complex 生成 Type 1 Config TLP
   Target: Bus 5, Dev 0, Func 0
   
2. Bridge 1 检查：
   Secondary Bus = 1, Subordinate Bus = 3
   5 > 3 → 不转发
   
3. Bridge 2 检查：
   Secondary Bus = 4, Subordinate Bus = 10
   4 ≤ 5 ≤ 10 → 转发
   
4. Bridge 2 到达 Bus 5 → 转换为 Type 0
   
5. Bus 5 上的设备接收并响应
```

**使用场景**：
- 系统枚举期间扫描所有设备
- BIOS/操作系统读取 Vendor ID, Device ID
- 配置 BARs, 中断等

**FEMU 代码示例**：
```c
// hw/pci/pci.c - 配置空间读取
uint32_t pci_default_read_config(PCIDevice *d, uint32_t address, int len)
{
    uint32_t val = 0;
    
    // 从配置空间数组读取
    memcpy(&val, d->config + address, len);
    return le32_to_cpu(val);
}
```

---

## 4. Message Transactions（消息事务）

Message 事务用于传递事件通知和带外控制信号，不直接传输用户数据。

### 4.1 Message 分类

**按路由方式**：

| 类型 | Fmt/Type | 路由方式 |
|------|----------|----------|
| **Routed to Root Complex** | 000/10xxx | 显式路由到 RC |
| **Routed by Address** | 001/10xxx | 基于地址路由 |
| **Routed by ID** | 010/10xxx | 基于 Requester ID |
| **Broadcast from Root** | 011/10xxx | RC 广播到所有端点 |
| **Local (Same Hierarchy)** | 100/10xxx | 本地消息 |
| **Gathered to Root** | 101/10xxx | 收集到 RC |

### 4.2 常见 Message 类型

#### (1) INTx Interrupt Assert/Deassert

**用途**：模拟传统 PCI INTx 中断（INTA#-INTD#）

**格式**：
```
┌─────────────────────────────────────────┐
│ Fmt=001, Type=10100 (Assert)           │
│      or Type=10101 (Deassert)          │
│ Message Code: 0x00-0x03 (INTA-INTD)    │
└─────────────────────────────────────────┘
```

**路由**：发送到 Root Complex

**使用场景**：Legacy 设备兼容

#### (2) MSI/MSI-X Interrupt

**用途**：消息信号中断（高性能中断机制）

**实现**：MSI 中断实际上是**特殊的 Memory Write**
```
Memory Write Request:
  Address: 0xFEE0_0000 + offset (MSI 地址空间)
  Data: Interrupt Vector + Trigger Info
```

**特点**：
- Posted 事务，无需等待
- 无共享中断问题
- 支持多个中断向量

#### (3) Power Management Messages

**用途**：电源状态变更通知

| Message Code | 名称 | 用途 |
|--------------|------|------|
| 0x18 | PM_PME | 电源管理事件通知 |
| 0x19 | PME_Turn_Off | 请求进入低功耗 |
| 0x1B | PME_TO_Ack | 确认 PME_Turn_Off |

#### (4) Error Messages

**用途**：错误报告（AER 机制）

| Message Code | 名称 | 用途 |
|--------------|------|------|
| 0x20 | ERR_COR | 可纠正错误 |
| 0x21 | ERR_NONFATAL | 非致命错误 |
| 0x22 | ERR_FATAL | 致命错误 |

**FEMU 代码示例**：
```c
// hw/pci/pcie_aer.c - 发送错误消息
static void pcie_aer_msg(PCIDevice *dev, const PCIEAERMsg *msg)
{
    // 生成 ERR_COR, ERR_NONFATAL, 或 ERR_FATAL 消息
    if (msg->severity == AER_ERR_COR) {
        // 发送可纠正错误消息到 Root Complex
    }
}
```

#### (5) Vendor-Defined Messages

**用途**：厂商自定义控制

**Type**: 0x7E (Type 0) / 0x7F (Type 1)

---

## 5. Completion Transactions（完成事务）

Completion 是对 Non-Posted 请求的响应。

### 5.1 Completion Without Data (Cpl)

**用途**：确认 I/O Write 或 Config Write 完成

**格式**：
```
┌─────────────────────────────────────────┐
│ Fmt=000, Type=01010                    │
│ Completer ID: 响应者 BDF               │
│ Completion Status: 成功/失败状态        │
│ Byte Count: 剩余字节数                  │
│ Requester ID: 原请求者 BDF             │
│ Tag: 匹配原请求的 Tag                   │
│ Lower Address: 数据在请求中的偏移       │
└─────────────────────────────────────────┘
```

### 5.2 Completion With Data (CplD)

**用途**：返回 Read 请求的数据

**格式**：
```
┌─────────────────────────────────────────┐
│ Fmt=010, Type=01010                    │
│ (Header 同 Cpl)                        │
├─────────────────────────────────────────┤
│ Data Payload                           │
│ (Length × 4 bytes)                     │
└─────────────────────────────────────────┘
```

**Completion Status 字段**：

| Status | 值 | 含义 | 场景 |
|--------|---|------|------|
| **SC (Successful Completion)** | 000 | 成功完成 | 正常情况 |
| **UR (Unsupported Request)** | 001 | 不支持的请求 | 访问不存在的地址 |
| **CRS (Configuration Retry Status)** | 010 | 配置重试 | 设备未准备好 |
| **CA (Completer Abort)** | 100 | 完成者中止 | 设备内部错误 |

### 5.3 Split Completion（拆分完成）

对于大的 Read 请求，Completer 可以返回多个 Completion：

```
Memory Read Request: 256 bytes (64 DW)
  Tag = 0x05
  
Completer 响应：
  Completion 1: 128 bytes, Byte Count = 256, Lower Addr = 0x00
  Completion 2: 128 bytes, Byte Count = 128, Lower Addr = 0x80
  
Requester 重组：
  使用 Lower Address 将数据拼接到正确位置
```

**FEMU 代码示例**：
```c
// hw/pci/pci.c - 处理配置读取的 Completion
static void pci_req_completion_handler(PCIDevice *dev, uint8_t tag)
{
    // 接收 Completion，提取数据
    // 检查 Status 字段
    if (status == PCI_CPL_STATUS_UR) {
        // Unsupported Request - 处理错误
    }
}
```

---

## 6. 事务排序规则

PCIe 定义了严格的事务排序规则，确保数据一致性。

### 6.1 排序矩阵（简化版）

**规则**：表格中的 "Yes" 表示行事务必须等待列事务完成

|  | Posted Req | Non-Posted Req | Completion |
|--|------------|----------------|------------|
| **Posted Req** | Yes | No | No |
| **Non-Posted Req** | Yes | Yes | No |
| **Completion** | Yes | Yes | No |

**解读**：
- **Posted 必须按序**：多个 Memory Write 保持顺序
- **Non-Posted 不能超越 Posted**：Read 不能超过之前的 Write
- **Completion 可以超越**：允许乱序返回

### 6.2 Relaxed Ordering (RO)

**用途**：允许某些事务乱序，提高性能

**启用方式**：
- TLP Header 中的 Attr[2] 位设置为 1
- 需要设备和系统都支持

**效果**：
- RO Posted 可以超越非 RO Posted
- 提高并发度和带宽利用率

**适用场景**：
- 大块数据传输（如视频流）
- 对顺序不敏感的操作

---

## 7. 流控制与事务类型

不同事务类型消耗不同的流控制信用。

### 7.1 流控制分类

PCIe 流控制按事务类型分为三类：

| 流控制类型 | 事务类型 | 特点 |
|-----------|---------|------|
| **Posted (P)** | Memory Write, Message | 无 Completion |
| **Non-Posted (NP)** | Memory Read, I/O, Config | 需要 Completion |
| **Completion (Cpl)** | Completion | 响应 NP 请求 |

### 7.2 信用消耗

**Header 信用**：每个 TLP 消耗 1 个
**Data 信用**：按数据长度消耗（单位：DW）

**示例**：
```
发送 Memory Write (64 bytes = 16 DW):
  Posted Header Credit: -1
  Posted Data Credit: -16
  
收到 UpdateFC DLLP:
  Posted Header Credit: +5
  Posted Data Credit: +128
```

---

## 8. 实用技巧

### 8.1 选择合适的事务类型

| 需求 | 推荐事务类型 | 原因 |
|------|-------------|------|
| 高吞吐量数据传输 | Memory Write (Posted) | 无需等待响应 |
| 读取状态/数据 | Memory Read (Non-Posted) | 需要返回数据 |
| 设备初始化 | Config Read/Write | 访问配置空间 |
| 中断通知 | MSI (Memory Write) | 高效，无共享 |
| 错误报告 | ERR Message | 标准化错误处理 |

### 8.2 性能优化建议

**1. 批量传输**：
```
差：10 × Memory Write (4 bytes) = 10 TLPs
好：1 × Memory Write (40 bytes) = 1 TLP
优势：减少头部开销，提高带宽利用率
```

**2. 调整 Max Payload Size**：
```bash
# 查看当前配置
$ lspci -vvv -s 01:00.0 | grep MaxPayload
  MaxPayload 256 bytes

# 增大 MPS（需要全路径支持）
$ setpci -s 01:00.0 ECAP_EXP+8.w=5000
# DevCtl[7:5] = 010 → MPS = 512 bytes
```

**3. 使用 Relaxed Ordering**（适用于流式数据）

**4. 平衡 Read Request Size**：
- 太小：TLP 数量多
- 太大：延迟高，占用缓冲区

### 8.3 调试方法

**1. 使用 lspci 查看能力**：
```bash
lspci -vvv -s 01:00.0 | grep -E "MaxPayload|MaxReadReq|RelaxedOrd"
```

**2. 启用 AER 日志**：
```bash
# 查看 PCIe 错误
dmesg | grep -i "aer\|pcie"
```

**3. 使用 PCIe 分析仪**（硬件）：
- 实时捕获 TLP
- 分析事务类型和时序
- 检测协议违规

---

## 9. 常见问题

### Q1: 为什么 Memory Write 是 Posted？

**答**：Posted 机制允许发送方立即继续，无需等待响应，大幅提高吞吐量。对于写操作，通常不需要立即知道是否成功，顺序保证即可。

### Q2: 如何确保 Memory Write 已完成？

**答**：发送一个 Memory Read 到相同地址或设备，由于排序规则，Read 必须等待之前的 Write 完成。

### Q3: Configuration 事务为何是 Non-Posted？

**答**：配置操作通常需要读取设备信息或确认写入成功，Non-Posted 提供了必要的反馈。

### Q4: MSI 中断为何比 INTx 高效？

**答**：
- MSI 是 Posted Memory Write，无需等待
- INTx 需要 Message 往返，延迟高
- MSI 无共享，避免中断风暴

---

## 10. 总结

### 关键要点

1. ✅ **5 大事务类型**：Memory, I/O, Config, Message, Completion
2. ✅ **Posted vs Non-Posted**：平衡性能与可靠性
3. ✅ **排序规则**：保证数据一致性
4. ✅ **流控制分类**：P, NP, Cpl 独立管理
5. ✅ **Message 机制**：高效的事件通知
6. ✅ **Completion 匹配**：使用 Tag 关联请求和响应

### 事务类型速查表

| 事务类型 | Fmt | Type | Posted? | 用途 |
|---------|-----|------|---------|------|
| Memory Read | 00x | 00000 | No | 读内存 |
| Memory Write | 01x | 00000 | Yes | 写内存 |
| I/O Read | 000 | 00010 | No | 读 I/O |
| I/O Write | 010 | 00010 | No | 写 I/O |
| Config Read Type 0 | 000 | 00100 | No | 配置读 |
| Config Write Type 0 | 010 | 00100 | No | 配置写 |
| Config Read Type 1 | 000 | 00101 | No | 配置读（桥接）|
| Message | 00x | 1xxxx | Yes | 事件通知 |
| Completion | 00x | 01010 | N/A | 响应 |

---

## 下一步学习

- [TLP 格式详解](tlp-format.md) - 事务层包的详细结构
- [流控制机制](flow-control.md) - 信用管理和缓冲区
- [排序规则](ordering.md) - 事务排序的详细规则
- [路由机制](routing.md) - TLP 如何在拓扑中转发

---

## 参考资料

- **规范**：PCIe Base Spec Rev 5.0, Chapter 2 (Transaction Layer Specification)
- **表格**：Table 2-3 (Fmt and Type Encoding)
- **表格**：Table 2-40 (Ordering Rules)
- **实现**：`/hw/pci/pcie_aer.c`, `/hw/pci/pci.c` (FEMU)
- **寄存器**：`/include/standard-headers/linux/pci_regs.h`

---

**相关页面**：
- [← TLP 格式](tlp-format.md)
- [流控制 →](flow-control.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
