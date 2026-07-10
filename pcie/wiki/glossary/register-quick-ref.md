# PCIe 寄存器速查表

## 概述

本页面提供 PCIe 配置空间寄存器的快速参考，包括标准寄存器、Capability 结构和扩展 Capability。

## 标准配置空间 (0x00-0xFF)

### Type 0 Header (Endpoint)

| 偏移 | 名称 | 位 | 访问 | 描述 |
|------|------|-----|------|------|
| 0x00 | Vendor ID | 15:0 | RO | 厂商标识 |
| 0x02 | Device ID | 15:0 | RO | 设备标识 |
| 0x04 | Command | 15:0 | RW | 命令寄存器 |
| 0x06 | Status | 15:0 | RW1C | 状态寄存器 |
| 0x08 | Revision ID | 7:0 | RO | 修订版本 |
| 0x09 | Class Code | 23:8 | RO | 设备类别 |
| 0x0C | Cache Line Size | 7:0 | RW | 缓存行大小 |
| 0x0D | Latency Timer | 7:0 | RO | 延迟定时器(PCIe无效) |
| 0x0E | Header Type | 7:0 | RO | 头类型 (0x00=Type 0) |
| 0x0F | BIST | 7:0 | RW | 自检 |
| 0x10-0x24 | BAR 0-5 | 31:0 | RW | 基地址寄存器 |
| 0x28 | Cardbus CIS Pointer | 31:0 | RO | Cardbus 指针 |
| 0x2C | Subsystem Vendor ID | 15:0 | RO | 子系统厂商ID |
| 0x2E | Subsystem ID | 15:0 | RO | 子系统ID |
| 0x30 | Expansion ROM Base | 31:0 | RW | 扩展ROM基地址 |
| 0x34 | Capabilities Pointer | 7:0 | RO | Capability链表指针 |
| 0x3C | Interrupt Line | 7:0 | RW | 中断线 |
| 0x3D | Interrupt Pin | 7:0 | RO | 中断引脚 (1=A,2=B,3=C,4=D) |
| 0x3E | Min_Gnt | 7:0 | RO | 最小授权(PCIe无效) |
| 0x3F | Max_Lat | 7:0 | RO | 最大延迟(PCIe无效) |

### Command Register (0x04)

| 位 | 名称 | 描述 |
|----|------|------|
| 0 | I/O Space | 启用I/O空间访问 |
| 1 | Memory Space | 启用内存空间访问 |
| 2 | Bus Master | 启用总线主控 |
| 3 | Special Cycles | 特殊周期(PCIe忽略) |
| 4 | Memory Write and Invalidate | MWI(PCIe忽略) |
| 5 | VGA Palette Snoop | VGA调色板(PCIe忽略) |
| 6 | Parity Error Response | 奇偶校验错误响应 |
| 8 | SERR# Enable | SERR#使能(PCIe忽略) |
| 9 | Fast Back-to-Back | 快速背靠背(PCIe忽略) |
| 10 | Interrupt Disable | 禁用INTx中断 |

### Status Register (0x06)

| 位 | 名称 | 描述 |
|----|------|------|
| 3 | Interrupt Status | INTx中断状态 |
| 4 | Capabilities List | 支持Capability链表 |
| 5 | 66 MHz Capable | 66MHz能力(PCIe忽略) |
| 7 | Fast Back-to-Back | 快速背靠背(PCIe忽略) |
| 8 | Master Data Parity Error | 主控数据奇偶错误 |
| 11 | Signaled Target Abort | 目标中止 |
| 12 | Received Target Abort | 接收到目标中止 |
| 13 | Received Master Abort | 接收到主控中止 |
| 14 | Signaled System Error | 系统错误 |
| 15 | Detected Parity Error | 检测到奇偶错误 |

## PCI Capabilities (0x40+)

### Power Management Capability (0x01)

| 偏移 | 名称 | 大小 | 描述 |
|------|------|------|------|
| +0x00 | Cap ID / Next | 16-bit | ID=0x01, 下一个Capability |
| +0x02 | PM Capabilities | 16-bit | 电源管理能力 |
| +0x04 | PM Control/Status | 16-bit | 电源状态控制 |

### MSI Capability (0x05)

| 偏移 | 名称 | 大小 | 描述 |
|------|------|------|------|
| +0x00 | Cap ID / Next | 16-bit | ID=0x05 |
| +0x02 | Message Control | 16-bit | MSI控制 |
| +0x04 | Message Address Lo | 32-bit | 消息地址(低32位) |
| +0x08 | Message Address Hi | 32-bit | 消息地址(高32位,64位时) |
| +0x0C | Message Data | 16-bit | 消息数据 |
| +0x10 | Mask Bits | 32-bit | 掩码位(可选) |
| +0x14 | Pending Bits | 32-bit | 待处理位(可选) |

### MSI-X Capability (0x11)

| 偏移 | 名称 | 大小 | 描述 |
|------|------|------|------|
| +0x00 | Cap ID / Next | 16-bit | ID=0x11 |
| +0x02 | Message Control | 16-bit | MSI-X控制 |
| +0x04 | Table Offset | 32-bit | Table偏移和BIR |
| +0x08 | PBA Offset | 32-bit | PBA偏移和BIR |

## PCIe Capability (0x10)

### 基本结构

| 偏移 | 名称 | 大小 | 描述 |
|------|------|------|------|
| +0x00 | Cap ID / Next | 16-bit | ID=0x10 |
| +0x02 | PCIe Capabilities | 16-bit | PCIe能力 |
| +0x04 | Device Capabilities | 32-bit | 设备能力 |
| +0x08 | Device Control | 16-bit | 设备控制 |
| +0x0A | Device Status | 16-bit | 设备状态 |
| +0x0C | Link Capabilities | 32-bit | 链路能力 |
| +0x10 | Link Control | 16-bit | 链路控制 |
| +0x12 | Link Status | 16-bit | 链路状态 |
| +0x14 | Slot Capabilities | 32-bit | 槽位能力(可选) |
| +0x18 | Slot Control | 16-bit | 槽位控制(可选) |
| +0x1A | Slot Status | 16-bit | 槽位状态(可选) |
| +0x1C | Root Control | 16-bit | Root控制(Root Port) |
| +0x1E | Root Capabilities | 16-bit | Root能力 |
| +0x20 | Root Status | 32-bit | Root状态 |

### Device Control Register

| 位 | 名称 | 描述 |
|----|------|------|
| 0 | Correctable Error Reporting Enable | 可纠正错误报告 |
| 1 | Non-Fatal Error Reporting Enable | 非致命错误报告 |
| 2 | Fatal Error Reporting Enable | 致命错误报告 |
| 3 | Unsupported Request Reporting Enable | 不支持请求报告 |
| 4 | Enable Relaxed Ordering | 启用宽松排序 |
| 7:5 | Max Payload Size | 最大负载大小 |
| 8 | Extended Tag Field Enable | 扩展标签使能 |
| 9 | Phantom Functions Enable | 幻影功能使能 |
| 10 | Aux Power PM Enable | 辅助电源PM使能 |
| 11 | Enable No Snoop | 启用No Snoop |
| 14:12 | Max Read Request Size | 最大读请求大小 |
| 15 | Initiate Function Level Reset | FLR |

### Link Control Register

| 位 | 名称 | 描述 |
|----|------|------|
| 1:0 | ASPM Control | L0s/L1控制 |
| 3 | Read Completion Boundary | RCB (64/128字节) |
| 4 | Link Disable | 禁用链路 |
| 5 | Retrain Link | 重新训练链路 |
| 6 | Common Clock Configuration | 通用时钟配置 |
| 7 | Extended Synch | 扩展同步 |
| 8 | Enable Clock Power Management | 时钟PM使能 |
| 9 | Hardware Autonomous Width Disable | 禁用宽度自动协商 |
| 10 | Link Bandwidth Management Interrupt Enable | 带宽管理中断 |
| 11 | Link Autonomous Bandwidth Interrupt Enable | 自主带宽中断 |

## 扩展配置空间 (0x100-0xFFF)

### 扩展 Capability 头格式

```
所有扩展Capability的通用头：
+0x00: [15:0]  Capability ID
       [19:16] Capability Version
       [31:20] Next Capability Offset
```

### 常用扩展 Capability ID

| ID | 名称 | 描述 |
|----|------|------|
| 0x0001 | AER | 高级错误报告 |
| 0x0002 | VC | 虚拟通道 |
| 0x0003 | Device Serial Number | 设备序列号 |
| 0x0004 | Power Budgeting | 功率预算 |
| 0x0005 | Root Complex Link Declaration | RC链路声明 |
| 0x0007 | Root Complex Event Collector | RC事件收集器 |
| 0x000D | ACS | 访问控制服务 |
| 0x000E | ARI | 备选路由ID |
| 0x000F | ATS | 地址转换服务 |
| 0x0010 | SR-IOV | 单根I/O虚拟化 |
| 0x0013 | FPB | 扁平门户桥 |
| 0x0015 | Resizable BAR | 可调整大小BAR |
| 0x0018 | LTR | 延迟容忍报告 |
| 0x0019 | Secondary PCIe | 辅助PCIe |
| 0x001B | PASID | 进程地址空间ID |
| 0x001D | DPC | 下游端口包容 |
| 0x001E | L1 PM Substates | L1 PM子状态 |
| 0x001F | PTM | 精确时间测量 |
| 0x0023 | DOE | 数据对象交换 |
| 0x0026 | IDE | 完整性和数据加密 |

### AER Extended Capability (0x0001)

| 偏移 | 名称 | 大小 | 描述 |
|------|------|------|------|
| +0x00 | Header | 32-bit | ID=0x0001 |
| +0x04 | Uncorrectable Error Status | 32-bit | 不可纠正错误状态 |
| +0x08 | Uncorrectable Error Mask | 32-bit | 不可纠正错误掩码 |
| +0x0C | Uncorrectable Error Severity | 32-bit | 不可纠正错误严重性 |
| +0x10 | Correctable Error Status | 32-bit | 可纠正错误状态 |
| +0x14 | Correctable Error Mask | 32-bit | 可纠正错误掩码 |
| +0x18 | Advanced Error Cap and Control | 32-bit | 高级错误能力和控制 |
| +0x1C | Header Log | 128-bit | TLP头日志(4x32-bit) |
| +0x2C | Root Error Command | 32-bit | Root错误命令 |
| +0x30 | Root Error Status | 32-bit | Root错误状态 |
| +0x34 | Error Source ID | 32-bit | 错误源ID |

### SR-IOV Extended Capability (0x0010)

| 偏移 | 名称 | 大小 | 描述 |
|------|------|------|------|
| +0x00 | Header | 32-bit | ID=0x0010 |
| +0x04 | SR-IOV Capabilities | 32-bit | SR-IOV能力 |
| +0x08 | SR-IOV Control | 16-bit | SR-IOV控制 |
| +0x0A | SR-IOV Status | 16-bit | SR-IOV状态 |
| +0x0C | Initial VFs | 16-bit | 初始VF数量 |
| +0x0E | Total VFs | 16-bit | 总VF数量 |
| +0x10 | Num VFs | 16-bit | 当前VF数量 |
| +0x12 | Function Dependency Link | 8-bit | 功能依赖链接 |
| +0x14 | First VF Offset | 16-bit | 第一个VF偏移 |
| +0x16 | VF Stride | 16-bit | VF步长 |
| +0x1A | VF Device ID | 16-bit | VF设备ID |
| +0x1C | Supported Page Sizes | 32-bit | 支持的页大小 |
| +0x20 | System Page Size | 32-bit | 系统页大小 |
| +0x24 | VF BAR 0-5 | 192-bit | VF基地址寄存器 |

## 快速查找

### 常用偏移速查

```bash
# 标准寄存器
Vendor ID:       0x00
Device ID:       0x02
Command:         0x04
Status:          0x06
Class Code:      0x09
BAR0:            0x10
BAR1:            0x14
Interrupt Line:  0x3C
Interrupt Pin:   0x3D
Capability Ptr:  0x34

# PCIe Capability (找到后+偏移)
Device Control:  Cap+0x08
Device Status:   Cap+0x0A
Link Control:    Cap+0x10
Link Status:     Cap+0x12
Link Control 2:  Cap+0x30
Link Status 2:   Cap+0x32
```

### setpci 使用示例

```bash
# 读取 Vendor/Device ID
setpci -s 01:00.0 0.L

# 读取 Command 寄存器
setpci -s 01:00.0 4.W

# 读取 BAR0
setpci -s 01:00.0 10.L

# 读取 PCIe Device Control
setpci -s 01:00.0 CAP_EXP+8.W

# 读取 PCIe Link Status
setpci -s 01:00.0 CAP_EXP+12.W

# 读取 AER 状态
setpci -s 01:00.0 ECAP_AER+4.L
```

## 位域定义速查

### Max Payload Size (MPS)

| 值 | 大小 |
|----|------|
| 000 | 128 bytes |
| 001 | 256 bytes |
| 010 | 512 bytes |
| 011 | 1024 bytes |
| 100 | 2048 bytes |
| 101 | 4096 bytes |

### Link Speed

| 值 | 速度 | 代数 |
|----|------|------|
| 0x1 | 2.5 GT/s | Gen 1 |
| 0x2 | 5.0 GT/s | Gen 2 |
| 0x3 | 8.0 GT/s | Gen 3 |
| 0x4 | 16.0 GT/s | Gen 4 |
| 0x5 | 32.0 GT/s | Gen 5 |
| 0x6 | 64.0 GT/s | Gen 6 |

### Link Width

| 值 | 宽度 |
|----|------|
| 0x01 | x1 |
| 0x02 | x2 |
| 0x04 | x4 |
| 0x08 | x8 |
| 0x0C | x12 |
| 0x10 | x16 |
| 0x20 | x32 |

## 总结

本速查表提供：
- 标准配置空间布局
- PCI Capability 结构
- PCIe Capability 详细定义
- 扩展 Capability ID 列表
- 常用位域值定义
- setpci 命令示例

参见：
- [配置空间详解](../configuration/config-space.md)
- [PCIe Capability](../configuration/pcie-capability.md)
- [扩展 Capability](../configuration/extended-capabilities.md)
- [BAR 寄存器](../configuration/bars.md)
