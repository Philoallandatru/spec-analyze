# DLLP 格式详解

Data Link Layer Packet (DLLP) 是数据链路层使用的控制包，用于链路管理、流控制和可靠性保证。

---

## 概述

DLLP 是 **PCIe 数据链路层**专用的控制包，与 TLP 不同：

- **TLP**：承载用户数据和事务（由事务层生成）
- **DLLP**：承载链路控制信息（由数据链路层生成）

**DLLP 的主要功能**：
```
┌─────────────────────────────────────┐
│  1. ACK/NAK 机制                    │  确认 TLP 接收状态
├─────────────────────────────────────┤
│  2. 流控制初始化 (InitFC)           │  交换缓冲区信息
├─────────────────────────────────────┤
│  3. 流控制更新 (UpdateFC)           │  实时更新可用信用
├─────────────────────────────────────┤
│  4. 电源管理 (PM)                   │  协商电源状态
├─────────────────────────────────────┤
│  5. 厂商自定义 (Vendor)             │  厂商特定控制
└─────────────────────────────────────┘
```

**DLLP 在协议栈中的位置**：
```
事务层 (Transaction Layer)
    ↓ 生成 TLP
数据链路层 (Data Link Layer)
    ↓ 添加 Seq#, LCRC
    ↓ 生成 DLLP (ACK/NAK/FC)
    ├─→ TLP + LCRC ─→ 物理层
    └─→ DLLP ───────→ 物理层
物理层 (Physical Layer)
    ↓ 编码和传输
```

---

## DLLP 基本结构

### 通用格式

所有 DLLP 都是固定长度：**8 字节（64 位）**

```
┌────────────────────────────────────────────────────┐
│  Byte 0-1: DLLP Type + Reserved                    │  2 字节
├────────────────────────────────────────────────────┤
│  Byte 2-5: DLLP Content (Type-Specific)            │  4 字节
├────────────────────────────────────────────────────┤
│  Byte 6-7: CRC-16                                  │  2 字节
└────────────────────────────────────────────────────┘

总长度：8 字节（固定）
```

**与 TLP 的对比**：

| 特性 | TLP | DLLP |
|------|-----|------|
| 长度 | 可变（12 字节 - 4108 字节）| 固定（8 字节）|
| 生成层 | 事务层 | 数据链路层 |
| 用途 | 数据传输 | 链路控制 |
| CRC | LCRC (32-bit) + 可选 ECRC | CRC-16 |
| 序列号 | 有（12-bit Seq#）| 无 |
| 需要 ACK | 是 | 否（DLLP 不被确认）|

---

## DLLP 类型

### DLLP Type 编码

DLLP Type 字段占据第一个字节，定义了 DLLP 的类型和子类型。

**完整 DLLP 类型表**：

| Type Value | 名称 | 用途 | 方向 |
|------------|------|------|------|
| **0x00** | Ack | 确认 TLP 接收 | 双向 |
| **0x10** | Nak | 请求重传 TLP | 双向 |
| **0x20** | PM_Enter_L1 | 请求进入 L1 状态 | 双向 |
| **0x21** | PM_Enter_L23 | 请求进入 L2/L3 状态 | 下游→上游 |
| **0x24** | PM_Active_State_Request_L1 | 请求从 L1 退出到 L0 | 双向 |
| **0x30** | PM_Request_Ack | 电源管理请求确认 | 双向 |
| **0x40** | Vendor-Specific | 厂商自定义 | 双向 |
| **0x50** | InitFC1-P | 流控制初始化（Posted）| 双向 |
| **0x60** | InitFC1-NP | 流控制初始化（Non-Posted）| 双向 |
| **0x70** | InitFC1-Cpl | 流控制初始化（Completion）| 双向 |
| **0xC0** | InitFC2-P | 流控制初始化确认（Posted）| 双向 |
| **0xD0** | InitFC2-NP | 流控制初始化确认（Non-Posted）| 双向 |
| **0xE0** | InitFC2-Cpl | 流控制初始化确认（Completion）| 双向 |
| **0x80** | UpdateFC-P | 流控制更新（Posted）| 双向 |
| **0x90** | UpdateFC-NP | 流控制更新（Non-Posted）| 双向 |
| **0xA0** | UpdateFC-Cpl | 流控制更新（Completion）| 双向 |

---

## 1. Ack DLLP（确认）

### 用途

确认成功接收一个或多个 TLP。

### 格式

```
Byte 0-1:
┌─────────────┬──────────────────┐
│ Type = 0x00 │ Reserved (0x00)  │
└─────────────┴──────────────────┘

Byte 2-5:
┌──────────────────────────────────┐
│  Reserved  │  AckNak Seq Number  │
│  (20 bits) │    (12 bits)        │
└──────────────────────────────────┘

Byte 6-7:
┌──────────────────────────────────┐
│           CRC-16                 │
└──────────────────────────────────┘
```

### 字段说明

**AckNak Seq Number (12 bits)**：
- 表示已成功接收到序列号 ≤ 此值的所有 TLP
- 范围：0-4095（循环计数）

**工作机制**：
```
发送方：发送 TLP with Seq# = 100
接收方：成功接收 → 发送 Ack DLLP (AckNak Seq# = 100)
发送方：收到 Ack → 清除重传缓冲区中 Seq# ≤ 100 的 TLP
```

**累积确认**：
```
发送方连续发送：Seq# 10, 11, 12, 13
接收方只需发送：Ack (Seq# = 13)
  → 表示 10-13 全部成功接收
```

---

## 2. Nak DLLP（否认）

### 用途

请求重传从指定序列号开始的所有 TLP。

### 格式

```
Byte 0-1:
┌─────────────┬──────────────────┐
│ Type = 0x10 │ Reserved (0x00)  │
└─────────────┴──────────────────┘

Byte 2-5:
┌──────────────────────────────────┐
│  Reserved  │  AckNak Seq Number  │
│  (20 bits) │    (12 bits)        │
└──────────────────────────────────┘

Byte 6-7:
┌──────────────────────────────────┐
│           CRC-16                 │
└──────────────────────────────────┘
```

### 工作机制

**Nak 触发条件**：
1. LCRC 校验失败
2. TLP 序列号不连续（检测到丢包）
3. 接收缓冲区溢出

**重传流程**：
```
发送方：TLP Seq# 20, 21, 22, 23
接收方：收到 20, 21, [22 LCRC 错误], 23
        ↓
        发送 Nak (Seq# = 22)
        ↓
发送方：收到 Nak → 重传 Seq# ≥ 22 的所有 TLP
        重传 22, 23
        ↓
接收方：收到 22, 23 → 发送 Ack (Seq# = 23)
```

**超时重传**：
```
如果发送方在超时时间内未收到 Ack：
  → 自动重传所有未确认的 TLP
  → 不需要 Nak
```

---

## 3. 流控制 DLLP

流控制 DLLP 用于管理接收缓冲区，防止溢出。

### 3.1 InitFC1 / InitFC2（初始化）

**用途**：链路初始化时交换缓冲区容量信息

**发送时机**：
- **InitFC1**：链路训练完成后，每个端口发送
- **InitFC2**：收到对方 InitFC1 后，回复确认

**格式**（以 InitFC1-P 为例）：
```
Byte 0-1:
┌─────────────┬──────────────────┐
│ Type = 0x50 │ Reserved         │
│ (InitFC1-P) │                  │
└─────────────┴──────────────────┘

Byte 2-5:
┌──────────────┬───────────────────┐
│  HdrFC       │     DataFC        │
│  (8 bits)    │    (12 bits)      │
└──────────────┴───────────────────┘
  Header Credit  Data Credit (DW)

Byte 6-7:
┌──────────────────────────────────┐
│           CRC-16                 │
└──────────────────────────────────┘
```

**字段说明**：

| 字段 | 位宽 | 说明 |
|------|------|------|
| **HdrFC** | 8 bits | Header 信用数量（可缓冲的 TLP 个数）|
| **DataFC** | 12 bits | Data 信用数量（可缓冲的数据 DW 数）|

**InitFC 类型**：
- **InitFC1-P / InitFC2-P**：Posted 流控制
- **InitFC1-NP / InitFC2-NP**：Non-Posted 流控制
- **InitFC1-Cpl / InitFC2-Cpl**：Completion 流控制

**初始化流程**：
```
Port A                          Port B
  │                               │
  ├─ InitFC1-P (HdrFC=32, DataFC=512) →│
  ├─ InitFC1-NP (HdrFC=8, DataFC=128) →│
  ├─ InitFC1-Cpl (HdrFC=64, DataFC=2048)→│
  │                               │
  │←─ InitFC1-P (HdrFC=16, DataFC=256) ─┤
  │←─ InitFC1-NP (HdrFC=4, DataFC=64) ──┤
  │←─ InitFC1-Cpl (HdrFC=32, DataFC=1024)┤
  │                               │
  ├─ InitFC2-P  ─────────────────→│ (确认)
  ├─ InitFC2-NP ─────────────────→│
  ├─ InitFC2-Cpl ────────────────→│
  │                               │
  │←─ InitFC2-P  ───────────────┤
  │←─ InitFC2-NP ───────────────┤
  │←─ InitFC2-Cpl ──────────────┤
  │                               │
  └─ 流控制初始化完成，可以发送 TLP
```

### 3.2 UpdateFC（运行时更新）

**用途**：运行时通知对方已释放的缓冲区空间

**发送时机**：
- 处理完 TLP 后释放缓冲区
- 定期批量更新（减少 DLLP 数量）

**格式**（以 UpdateFC-P 为例）：
```
Byte 0-1:
┌─────────────┬──────────────────┐
│ Type = 0x80 │ Reserved         │
│(UpdateFC-P) │                  │
└─────────────┴──────────────────┘

Byte 2-5:
┌──────────────┬───────────────────┐
│  HdrFC       │     DataFC        │
│  (8 bits)    │    (12 bits)      │
└──────────────┴───────────────────┘
  释放的 Header   释放的 Data (DW)

Byte 6-7:
┌──────────────────────────────────┐
│           CRC-16                 │
└──────────────────────────────────┘
```

**更新策略**：
```
接收方缓冲区状态：
  初始：Posted Header = 32, Posted Data = 512 DW
  
收到 Memory Write (1 Header, 16 DW Data)：
  Posted Header: 32 → 31
  Posted Data: 512 → 496
  
处理完毕，释放缓冲区：
  发送 UpdateFC-P (HdrFC = 1, DataFC = 16)
  
发送方接收 UpdateFC：
  Posted Header Credit: +1
  Posted Data Credit: +16
```

**批量更新优化**：
```
不推荐：每处理一个 TLP 就发送 UpdateFC
推荐：积累释放一定数量后再发送
  例如：每释放 8 个 Header 或 128 DW Data 发送一次
```

---

## 4. 电源管理 DLLP

电源管理 DLLP 用于协商链路电源状态。

### 4.1 PM_Enter_L1

**用途**：请求进入 L1 低功耗状态

**格式**：
```
Byte 0-1:
┌─────────────┬──────────────────┐
│ Type = 0x20 │ Reserved         │
└─────────────┴──────────────────┘

Byte 2-5: Reserved (全 0)

Byte 6-7: CRC-16
```

**进入 L1 流程**：
```
Port A 空闲检测：无 TLP 传输超过 T(idle)
  ↓
Port A: 发送 PM_Enter_L1 DLLP
  ↓
Port B: 收到 PM_Enter_L1
  ↓
Port B: 发送 PM_Request_Ack DLLP
  ↓
两端进入 L1 状态：
  - 关闭 TX/RX 高速电路
  - 保持参考时钟（快速恢复）
  - 延迟 < 10 μs
```

### 4.2 PM_Enter_L23

**用途**：请求进入 L2/L3 深度睡眠状态

**方向**：仅从下游（设备）到上游（RC/Switch）

**进入 L2/L3 流程**：
```
系统电源管理事件（如休眠）
  ↓
Root Complex: 广播 PME_Turn_Off Message
  ↓
设备: 停止发送 TLP
  ↓
设备: 发送 PM_Enter_L23 DLLP（如果是 Active 状态）
  ↓
Root Complex: 收到所有设备的 PM_Enter_L23
  ↓
进入 L2/L3 状态：
  - 关闭 TX/RX
  - 可选关闭参考时钟（L3）
  - 恢复时间：L2 < 100 μs, L3 > 1 ms
```

### 4.3 PM_Request_Ack

**用途**：确认电源状态变更请求

---

## 5. Vendor-Specific DLLP

**用途**：厂商自定义的链路控制

**格式**：
```
Byte 0-1:
┌─────────────┬──────────────────┐
│ Type = 0x40 │ Vendor Defined   │
└─────────────┴──────────────────┘

Byte 2-5: Vendor-Specific Content

Byte 6-7: CRC-16
```

**使用场景**：
- 专有的调试和测试功能
- 厂商特定的性能优化
- 私有协议扩展

---

## 6. DLLP CRC-16

### CRC 计算

**多项式**：CRC-16-IBM (x¹⁶ + x¹⁵ + x² + 1)

**计算范围**：DLLP 的前 6 字节（Type + Content）

**初始值**：0xFFFF

**算法**（伪代码）：
```c
uint16_t calculate_dllp_crc(uint8_t *dllp_data) {
    uint16_t crc = 0xFFFF;  // 初始值
    
    for (int i = 0; i < 6; i++) {  // 前 6 字节
        crc = crc ^ (dllp_data[i] << 8);
        for (int j = 0; j < 8; j++) {
            if (crc & 0x8000) {
                crc = (crc << 1) ^ 0x8005;  // 多项式
            } else {
                crc = crc << 1;
            }
        }
    }
    
    return crc;
}
```

**错误处理**：
- **CRC 正确**：正常处理 DLLP
- **CRC 错误**：丢弃 DLLP（DLLP 不会被重传）

---

## 7. DLLP vs TLP 对比

### 完整对比表

| 特性 | DLLP | TLP |
|------|------|-----|
| **生成层** | 数据链路层 | 事务层 |
| **长度** | 固定 8 字节 | 可变（12-4108 字节）|
| **用途** | 链路控制（ACK/FC/PM）| 数据传输（Memory/IO/Config）|
| **序列号** | 无 | 有（12-bit）|
| **需要确认** | 否 | 是（通过 Ack DLLP）|
| **CRC** | CRC-16 (2 字节) | LCRC (4 字节) |
| **重传** | 不重传 | 支持重传 |
| **流控制** | 不消耗信用 | 消耗信用 |
| **物理层编码** | 与 TLP 相同 | 与 DLLP 相同 |

### 混合传输

DLLP 和 TLP 在物理层混合传输：
```
时间轴 →
┌─────┬──────┬─────┬──────┬─────┬──────┐
│ TLP │ DLLP │ TLP │ DLLP │ TLP │ DLLP │
│Seq#1│ Ack  │Seq#2│ UpdFC│Seq#3│ Ack  │
└─────┴──────┴─────┴──────┴─────┴──────┘

物理层可以插入 DLLP 而不影响 TLP 序列
```

---

## 8. FEMU 代码中的 DLLP

虽然 FEMU 是软件模拟器，数据链路层大部分功能被抽象，但仍包含相关概念。

### 流控制初始化

```c
// hw/pci/pcie.c - PCIe 设备初始化
int pcie_cap_init(PCIDevice *dev, uint8_t offset,
                  uint8_t type, uint8_t version)
{
    // 设置 Link Capabilities
    // 实际硬件会在此阶段交换 InitFC DLLP
    pci_set_long(dev->config + offset + PCI_EXP_LNKCAP,
                 link_cap);
}
```

### 数据链路层状态

```c
// include/hw/pci/pcie_regs.h
// Link Status Register 反映数据链路层状态
#define PCI_EXP_LNKSTA_DLLLA    0x2000  // Data Link Layer Link Active
// 该位表示数据链路层已完成 InitFC 交换，链路可用
```

---

## 9. 实用技巧

### 9.1 调试 DLLP 问题

**1. 检查链路状态**：
```bash
# 查看 Data Link Layer Link Active 状态
lspci -vvv -s 01:00.0 | grep "LnkSta:"
  LnkSta: Speed 8GT/s, Width x4, DLLLA+
         # DLLLA+ 表示数据链路层活跃（InitFC 已完成）
```

**2. 监控重传统计**：
```bash
# 查看链路错误和重传
lspci -vvv -s 01:00.0 | grep -A 10 "Advanced Error Reporting"
  UncorrErr: DLP+ SDES- TLP- FCP+ CmpltTO- ...
  # DLP (Data Link Protocol Error) 可能与 DLLP 相关
```

**3. 使用 PCIe 分析仪**：
- 实时捕获 DLLP
- 验证 Ack/Nak 时序
- 分析流控制信用变化

### 9.2 流控制优化

**增大接收缓冲区**：
```
更大的 InitFC 值：
  优点：支持更多未完成事务，提高吞吐量
  缺点：消耗更多硬件资源
  
典型配置：
  Posted: HdrFC=32-128, DataFC=512-2048 DW
  Non-Posted: HdrFC=8-32, DataFC=128-512 DW
  Completion: HdrFC=32-256, DataFC=1024-8192 DW
```

**UpdateFC 策略**：
```c
// 伪代码：批量更新策略
if (released_hdr_credits >= 8 || released_data_credits >= 128) {
    send_updatefc_dllp(released_hdr_credits, released_data_credits);
    released_hdr_credits = 0;
    released_data_credits = 0;
}
```

### 9.3 电源管理注意事项

**ASPM L1 延迟**：
```bash
# 查看 L1 退出延迟
lspci -vvv -s 01:00.0 | grep "L1 Exit Latency"
  L1 Exit Latency: <4us
  
# 延迟敏感应用可能需要禁用 ASPM
echo performance > /sys/module/pcie_aspm/parameters/policy
```

---

## 10. 常见问题

### Q1: DLLP 丢失会怎样？

**答**：
- **Ack DLLP 丢失**：发送方超时后自动重传 TLP，无严重影响
- **UpdateFC DLLP 丢失**：发送方信用不更新，可能导致流控制阻塞，但不会丢数据
- **InitFC DLLP 丢失**：链路训练失败，需要重新训练

### Q2: 为什么 DLLP 不需要确认？

**答**：DLLP 是控制信息，丢失后：
- Ack/Nak：发送方有超时重传机制
- UpdateFC：接收方会继续发送新的 UpdateFC
- PM：电源状态变更有超时和重试机制

设计为不确认 DLLP 避免了递归确认问题（确认的确认...）。

### Q3: 如何区分 DLLP 和 TLP？

**答**：物理层通过**帧头**区分：
- TLP 以 `STP`（Start of TLP）开始
- DLLP 以 `SDP`（Start of DLLP）开始

### Q4: 流控制信用用完会怎样？

**答**：
- 事务层停止发送该类型的 TLP
- 等待 UpdateFC DLLP 补充信用
- 不会丢包，只是延迟增加

---

## 11. 总结

### 关键要点

1. ✅ **DLLP 固定 8 字节**：Type (2B) + Content (4B) + CRC-16 (2B)
2. ✅ **5 大类 DLLP**：Ack/Nak, InitFC, UpdateFC, PM, Vendor
3. ✅ **Ack/Nak 机制**：保证 TLP 可靠传输
4. ✅ **流控制三步**：InitFC1 → InitFC2 → UpdateFC（运行时）
5. ✅ **电源管理**：PM_Enter_L1/L23 协商低功耗状态
6. ✅ **DLLP 不被确认**：丢失后依赖上层机制恢复

### DLLP 类型速查表

| Type | 名称 | 用途 | 关键字段 |
|------|------|------|----------|
| 0x00 | Ack | 确认 TLP | Seq Number |
| 0x10 | Nak | 请求重传 | Seq Number |
| 0x50 | InitFC1-P | 初始化 Posted FC | HdrFC, DataFC |
| 0x60 | InitFC1-NP | 初始化 Non-Posted FC | HdrFC, DataFC |
| 0x70 | InitFC1-Cpl | 初始化 Completion FC | HdrFC, DataFC |
| 0x80 | UpdateFC-P | 更新 Posted FC | HdrFC, DataFC |
| 0x90 | UpdateFC-NP | 更新 Non-Posted FC | HdrFC, DataFC |
| 0xA0 | UpdateFC-Cpl | 更新 Completion FC | HdrFC, DataFC |
| 0x20 | PM_Enter_L1 | 进入 L1 状态 | - |
| 0x21 | PM_Enter_L23 | 进入 L2/L3 状态 | - |

---

## 下一步学习

- [ACK/NAK 机制详解](ack-nak.md) - 可靠传输的实现
- [TLP 格式](../transaction-layer/tlp-format.md) - 事务层包
- [流控制机制](../transaction-layer/flow-control.md) - 信用管理
- [LTSSM 状态机](../physical-layer/ltssm.md) - 链路训练

---

## 参考资料

- **规范**：PCIe Base Spec Rev 5.0, Chapter 3 (Data Link Layer Specification)
- **表格**：Table 3-3 (DLLP Types)
- **图表**：Figure 3-4 (DLLP Format)
- **图表**：Figure 3-6 (Ack/Nak Protocol)
- **实现**：`/hw/pci/pcie.c` (FEMU)

---

**相关页面**：
- [← 数据链路层概述](overview.md)
- [ACK/NAK 机制 →](ack-nak.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
