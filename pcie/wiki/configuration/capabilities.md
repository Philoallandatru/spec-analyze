# PCIe 能力结构（Capabilities）

## 概述

PCIe 配置空间中的 **Capability Structure**（能力结构）是一种扩展机制，允许设备声明和配置各种可选特性。这些结构以链表形式组织在配置空间的 0x40-0xFF 区域，避免了固定布局的限制。

PCI-SIG 定义了两类能力：
- **PCI Capabilities**：位于 0x40-0xFF（256 字节配置空间）
- **PCIe Extended Capabilities**：位于 0x100-0xFFF（4KB 配置空间）

## Capability 链表机制

### 链表结构

Capability 结构通过单向链表连接，每个结构包含：

```
+--------+--------+
| Next   | Cap ID | Offset 0x00
+--------+--------+
|  Capability-specific data
|  (长度由 Cap ID 决定)
+------------------+
```

**通用头部：**
- **Capability ID** (Byte 0)：标识能力类型
- **Next Pointer** (Byte 1)：指向下一个 Capability 的偏移（0x00 表示链表结束）

### 遍历链表

从配置空间 Type 0/1 Header 的 **Capabilities Pointer**（偏移 0x34）开始：

```
步骤：
1. 读取 Capabilities Pointer (0x34) 获取第一个 Capability 偏移
2. 读取该偏移处的 Capability ID 和 Next Pointer
3. 处理当前 Capability
4. 若 Next Pointer != 0，跳转到下一个 Capability
5. 重复直到 Next Pointer = 0
```

**链表遍历示例：**
```
Header (0x34): Cap Ptr = 0x40
    ↓
[0x40] Power Management (ID=0x01) → Next=0x50
    ↓
[0x50] MSI (ID=0x05) → Next=0x70
    ↓
[0x70] PCIe Capability (ID=0x10) → Next=0x00 (结束)
```

### 代码实现

```c
uint8_t cap_ptr = pci_read_config8(bus, dev, func, 0x34);

while (cap_ptr != 0) {
    uint8_t cap_id = pci_read_config8(bus, dev, func, cap_ptr);
    uint8_t next_ptr = pci_read_config8(bus, dev, func, cap_ptr + 1);
    
    switch (cap_id) {
        case 0x01: handle_power_management(cap_ptr); break;
        case 0x05: handle_msi(cap_ptr); break;
        case 0x10: handle_pcie_capability(cap_ptr); break;
        // ...
    }
    
    cap_ptr = next_ptr;
}
```

## 常见 PCI Capability 类型

### Capability ID 列表

```
ID    名称                           长度    描述
--    --------------------------     ----    ---------------------------
0x00  Reserved                       -       保留
0x01  Power Management               8B      电源管理
0x02  AGP                            Var     加速图形端口（已废弃）
0x03  VPD                            8B      重要产品数据
0x04  Slot Identification            4B      插槽标识
0x05  MSI                            10-24B  消息信号中断
0x06  CompactPCI Hot Swap            -       热插拔
0x07  PCI-X                          Var     PCI-X
0x08  HyperTransport                 Var     AMD HyperTransport
0x09  Vendor Specific                Var     厂商自定义
0x0A  Debug Port                     Var     调试端口
0x0B  CompactPCI Central Resource    -       资源控制
0x0C  PCI Hot-Plug                   Var     热插拔控制
0x0D  Bridge Subsystem Vendor ID     4B      桥接器子系统 ID
0x0E  AGP 8x                         Var     AGP 8x
0x0F  Secure Device                  Var     安全设备
0x10  PCI Express                    Var     PCIe 能力（核心）
0x11  MSI-X                          12B     扩展 MSI
0x12  SATA Data/Index Config         Var     SATA 配置
0x13  Advanced Features              Var     高级特性
0x14  Enhanced Allocation            Var     增强分配
0x15  Flattening Portal Bridge       Var     扁平化门户
```

### 能力结构布局示例

**示例配置空间（0x40-0xFF）：**
```
Offset  +0  +1  +2  +3  +4  +5  +6  +7  +8  +9  +A  +B  +C  +D  +E  +F
------  --  --  --  --  --  --  --  --  --  --  --  --  --  --  --  --
0x40    01  50  03  C8  00  00  00  00  [Power Management - 8 bytes]
0x50    05  70  82  00  00  00  00  00  08  00  00  00  [MSI - 12 bytes]
0x60    11  00  03  00  00  20  00  00  00  00  00  00  [MSI-X - 12 bytes]
0x70    10  00  02  00  [PCIe Capability ...]
```

## Power Management Capability（0x01）

电源管理能力允许设备进入低功耗状态（D0-D3）。

### 结构布局

```
Offset  +0      +1      +2      +3
------  ------  ------  ------  ------
0x00    Cap ID  Next    PMC (Power Management Capabilities)
        (0x01)  Ptr     
0x04    PMCSR (Power Management Control/Status)
        
总长度：8 字节
```

### 关键字段

**PMC - Power Management Capabilities (Offset +2, 16 bits):**
```
Bits 2:0  - Version (通常为 0x3 表示 PCIe)
Bit 3     - PME Clock (PCIe 中为 0)
Bit 4     - Reserved
Bit 5     - DSI (Device Specific Initialization)
Bits 8:6  - Aux Current (辅助电流需求)
Bit 9     - D1 Support
Bit 10    - D2 Support
Bits 15:11 - PME Support (哪些 D 状态可以发 PME#)
```

**PMCSR - Control/Status Register (Offset +4, 16 bits):**
```
Bits 1:0  - Power State
            00 = D0 (全功耗)
            01 = D1 (低功耗 1)
            10 = D2 (低功耗 2)
            11 = D3hot (热复位保持)
Bit 8     - PME_En (PME 使能)
Bits 14:13 - Data Select
Bit 15    - PME Status
```

### 电源状态转换

```
D0 (Operating) ←→ D1 (Low Power) ←→ D2 (Lower Power) ←→ D3hot (Lowest)
                                                              ↓
                                                        D3cold (Off)
                                                              ↑
                                                          (电源移除)

转换时间：
D0 → D3hot: 微秒级
D3hot → D0: 毫秒级（需要重新初始化）
D3cold → D0: 秒级（需要完整初始化）
```

### 编程示例

```c
// 将设备设置为 D3hot
void enter_d3hot(uint8_t cap_offset) {
    uint16_t pmcsr = pci_read_config16(bus, dev, func, cap_offset + 4);
    pmcsr = (pmcsr & ~0x03) | 0x03;  // 设置 Power State = 11b
    pci_write_config16(bus, dev, func, cap_offset + 4, pmcsr);
}

// 唤醒设备到 D0
void enter_d0(uint8_t cap_offset) {
    uint16_t pmcsr = pci_read_config16(bus, dev, func, cap_offset + 4);
    pmcsr = (pmcsr & ~0x03) | 0x00;  // 设置 Power State = 00b
    pci_write_config16(bus, dev, func, cap_offset + 4, pmcsr);
    
    // 等待设备就绪
    usleep(10000);  // 10ms 恢复时间
}
```

## MSI Capability（0x05）

Message Signaled Interrupts 允许设备通过内存写事务发送中断，替代传统的 INTx 引脚。

### 结构布局

MSI Capability 有三种格式，取决于是否支持 64-bit 地址和 Per-Vector Masking：

**32-bit 地址格式（10 字节）：**
```
Offset  +0      +1      +2      +3
------  ------  ------  ------  ------
0x00    0x05    Next    Message Control
0x04    Message Address (32-bit)
0x08    Message Data (16-bit)
```

**64-bit 地址格式（14 字节）：**
```
Offset  +0      +1      +2      +3
------  ------  ------  ------  ------
0x00    0x05    Next    Message Control
0x04    Message Address Lower 32 bits
0x08    Message Address Upper 32 bits
0x0C    Message Data (16-bit)
```

**64-bit 带 Masking（24 字节）：**
```
Offset  +0      +1      +2      +3
------  ------  ------  ------  ------
0x00    0x05    Next    Message Control
0x04    Message Address Lower 32 bits
0x08    Message Address Upper 32 bits
0x0C    Message Data (16-bit)
0x10    Mask Bits (32-bit)
0x14    Pending Bits (32-bit)
```

### Message Control 字段

```
Bits 2:0  - Multiple Message Capable (设备支持的向量数)
            000 = 1 vector
            001 = 2 vectors
            010 = 4 vectors
            011 = 8 vectors
            100 = 16 vectors
            101 = 32 vectors
Bits 6:4  - Multiple Message Enable (系统分配的向量数)
Bit 7     - 64-bit Address Capable
Bit 8     - Per-Vector Masking Capable
Bits 15:9 - Reserved
```

### 中断向量分配

```
设备请求 8 个向量：
1. Multiple Message Capable = 011b (8 vectors)
2. 系统分配 4 个向量
3. Multiple Message Enable = 010b (4 vectors)
4. Message Data = 0x20
5. 设备可以发送：
   Vector 0: Data = 0x20
   Vector 1: Data = 0x21
   Vector 2: Data = 0x22
   Vector 3: Data = 0x23
```

详见 **interrupts/msi.md**。

## MSI-X Capability（0x11）

MSI-X 是 MSI 的增强版本，支持更多向量（最多 2048）和独立掩码。

### 结构布局

```
Offset  +0      +1      +2      +3
------  ------  ------  ------  ------
0x00    0x11    Next    Message Control
0x04    Table Offset / BIR (BAR Indicator)
0x08    PBA Offset / BIR
        
总长度：12 字节
```

### Message Control 字段

```
Bits 10:0  - Table Size (N-1，实际向量数 = N)
Bit 14     - Function Mask (全局掩码)
Bit 15     - MSI-X Enable
```

### MSI-X Table

MSI-X Table 存储在设备内存空间（由 BAR 指示）：

```
每个表项 16 字节：

Offset  字段
------  ---------------------
+0x00   Message Address Lower
+0x04   Message Address Upper
+0x08   Message Data
+0x0C   Vector Control
        Bit 0: Mask Bit
```

### 编程流程

```
1. 定位 MSI-X Capability (ID=0x11)
2. 读取 Table Offset/BIR → BAR + 偏移 = Table 地址
3. 映射 Table 内存区域
4. 为每个向量配置：
   table[i].addr_lo = 0xFEE00000
   table[i].addr_hi = 0x00000000
   table[i].data = vector_number
   table[i].control = 0 (unmask)
5. 设置 Message Control: Enable=1
```

详见 **interrupts/msi.md**。

## PCI Express Capability（0x10）

这是 PCIe 设备最重要的 Capability，包含链路、设备、插槽的能力和控制寄存器。

### 结构布局

```
Offset  字段                        大小
------  --------------------------  ----
0x00    Capability Header           2B
0x02    PCI Express Capabilities    2B
0x04    Device Capabilities         4B
0x08    Device Control              2B
0x0A    Device Status               2B
0x0C    Link Capabilities           4B
0x10    Link Control                2B
0x12    Link Status                 2B
0x14    Slot Capabilities           4B (仅 Root/Switch Downstream Port)
0x18    Slot Control                2B
0x1A    Slot Status                 2B
0x1C    Root Control                2B (仅 Root Port)
0x1E    Root Capabilities           2B
0x20    Root Status                 4B

总长度：36-60 字节（取决于设备类型）
```

### PCI Express Capabilities Register

```
Bits 3:0  - Capability Version (当前 0x2)
Bits 7:4  - Device/Port Type
            0000 = PCIe Endpoint
            0001 = Legacy PCIe Endpoint
            0100 = Root Port
            0101 = Upstream Switch Port
            0110 = Downstream Switch Port
            0111 = PCIe-to-PCI Bridge
            1000 = PCI-to-PCIe Bridge
            1001 = Root Complex Integrated Endpoint
Bit 8     - Slot Implemented
Bits 13:9 - Interrupt Message Number
```

### 关键寄存器组

**Device Capabilities（0x04）：**
- Max Payload Size Supported
- Phantom Functions
- Extended Tag Support
- Role-Based Error Reporting

**Link Capabilities（0x0C）：**
- Max Link Speed (2.5/5.0/8.0/16.0 GT/s)
- Max Link Width (x1/x2/x4/x8/x16/x32)
- ASPM Support (L0s/L1)
- Port Number

**Link Status（0x12）：**
- Current Link Speed
- Negotiated Link Width
- Link Training

详见 **pcie-capability.md**。

## Vendor Specific Capability（0x09）

厂商自定义能力，格式由厂商决定：

```
Offset  +0      +1      +2      +3
------  ------  ------  ------  ------
0x00    0x09    Next    Length  (厂商定义长度)
0x03    [Vendor-specific data ...]
```

常见用途：
- 硬件调试接口
- 性能监控
- 专有特性控制
- OEM 配置

## 能力发现与管理

### 系统初始化流程

```
PCIe 设备枚举时：

1. BIOS/UEFI 阶段
   ├─ 扫描所有设备 Capabilities
   ├─ 识别电源管理能力
   ├─ 配置基础中断（MSI）
   └─ 建立 Capability 映射表

2. 操作系统加载
   ├─ 读取 BIOS 留下的配置
   ├─ 重新枚举设备 Capabilities
   ├─ 驱动程序注册需要的能力
   └─ 配置高级特性（MSI-X、ASPM）

3. 运行时管理
   ├─ 电源状态转换
   ├─ 链路状态监控
   ├─ 错误报告
   └─ 热插拔事件处理
```

### 驱动程序最佳实践

```c
// 查找特定 Capability
uint8_t find_capability(uint8_t cap_id) {
    uint8_t cap_ptr = pci_read_config8(bus, dev, func, 0x34);
    
    while (cap_ptr != 0) {
        uint8_t id = pci_read_config8(bus, dev, func, cap_ptr);
        if (id == cap_id)
            return cap_ptr;
        cap_ptr = pci_read_config8(bus, dev, func, cap_ptr + 1);
    }
    
    return 0;  // 未找到
}

// 设备初始化
void init_device() {
    // 1. 配置电源管理
    uint8_t pm_cap = find_capability(0x01);
    if (pm_cap)
        configure_power_management(pm_cap);
    
    // 2. 配置中断
    uint8_t msix_cap = find_capability(0x11);
    if (msix_cap) {
        configure_msix(msix_cap);
    } else {
        uint8_t msi_cap = find_capability(0x05);
        if (msi_cap)
            configure_msi(msi_cap);
    }
    
    // 3. 读取 PCIe 能力
    uint8_t pcie_cap = find_capability(0x10);
    if (pcie_cap)
        parse_pcie_capability(pcie_cap);
}
```

### 能力冲突与优先级

某些能力互斥或有优先级：

```
中断机制优先级：
MSI-X > MSI > INTx

电源管理：
- PME# 信号与 MSI 冲突，优先使用 MSI
- ASPM 可能影响延迟，性能优先时禁用

链路控制：
- Hardware Autonomous Speed Disable
- Hardware Autonomous Width Disable
```

## 调试与诊断

### lspci 输出示例

```bash
$ lspci -vvv -s 01:00.0

01:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD
    Capabilities: [40] Power Management version 3
        Flags: PMEClk- DSI- D1- D2- AuxCurrent=0mA PME(D0-,D1-,D2-,D3hot-,D3cold-)
        Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=0 PME-
    Capabilities: [50] MSI: Enable- Count=1/32 Maskable- 64bit+
        Address: 0000000000000000  Data: 0000
    Capabilities: [70] Express Endpoint, MSI 00
        DevCap: MaxPayload 256 bytes, PhantFunc 0, Latency L0s unlimited, L1 unlimited
        DevCtl: Report errors: Correctable- Non-Fatal- Fatal- Unsupported-
        LnkCap: Port #0, Speed 8GT/s, Width x4
        LnkSta: Speed 8GT/s, Width x4
    Capabilities: [b0] MSI-X: Enable+ Count=8 Masked-
        Vector table: BAR=0 offset=00003000
        PBA: BAR=0 offset=00002000
```

### 常见问题诊断

```
问题：设备未响应
检查：
1. Capabilities Pointer (0x34) 是否有效？
2. Status Register (0x06) Bit 4 (Capabilities List) 是否置位？
3. Capability 链是否有环路？

问题：MSI 不工作
检查：
1. MSI Capability 是否存在？
2. Message Control 的 Enable 位是否设置？
3. Message Address/Data 是否正确配置？

问题：链路速度低于预期
检查：
1. PCIe Capability Link Capabilities 的 Max Link Speed
2. Link Control 的 Target Link Speed
3. 链路训练状态（Link Status）
```

## 相关资源

- **pcie-capability.md**: PCIe Capability 详细解析
- **extended-capabilities.md**: 扩展能力结构（0x100-0xFFF）
- **../interrupts/msi.md**: MSI/MSI-X 深入指南
- **header-types.md**: 配置空间头部（Capabilities Pointer）

## 总结

Capability 链表机制是 PCIe 配置空间的核心扩展方式，提供了灵活性和向后兼容性。掌握常见 Capability（电源管理、MSI/MSI-X、PCIe Capability）的结构和配置方法，是 PCIe 驱动开发和系统调试的关键技能。通过正确遍历和配置这些能力，系统可以充分发挥 PCIe 设备的性能和功能。
