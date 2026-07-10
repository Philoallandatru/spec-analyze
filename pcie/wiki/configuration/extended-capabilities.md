# PCIe 扩展能力结构（Extended Capabilities）

## 概述

**PCIe Extended Capabilities**（扩展能力结构）是 PCIe 配置空间的高级扩展机制，位于 **扩展配置空间**（Extended Configuration Space）的 0x100-0xFFF 区域。与传统 PCI Capabilities（0x40-0xFF）不同，Extended Capabilities 使用增强的链表格式，支持更大的地址空间和更多的功能特性。

## 扩展配置空间访问

### 地址空间对比

```
传统配置空间：0x000-0x0FF (256 字节)
├── 标准头部 (0x00-0x3F)
└── PCI Capabilities (0x40-0xFF)

扩展配置空间：0x100-0xFFF (4096 字节)
└── PCIe Extended Capabilities (0x100-0xFFF)
```

### ECAM 访问方法

Extended Capabilities 只能通过 **ECAM**（Enhanced Configuration Access Mechanism）访问：

```c
// ECAM 地址计算
物理地址 = ECAM_Base + (Bus << 20) + (Device << 15) + 
           (Function << 12) + Offset

示例：访问 Extended Capability at 0x100
ECAM_Base = 0xE0000000
Bus = 1, Device = 0, Function = 0
物理地址 = 0xE0000000 + (1 << 20) + 0x100 = 0xE0100100
```

**访问代码示例：**
```c
void __iomem *ecam_base = ioremap(0xE0000000, 256 * 1024 * 1024);

u32 read_ext_cap(u8 bus, u8 dev, u8 func, u16 offset) {
    void __iomem *addr = ecam_base + 
                         (bus << 20) + (dev << 15) + (func << 12) + offset;
    return readl(addr);
}
```

## Extended Capability 链表结构

### 头部格式

每个 Extended Capability 以 4 字节头部开始：

```
+-------------------+-------------------+
| Extended Cap ID   | Cap Version (4b)  | DW 0 (Offset 0x00)
|    (16 bits)      | Next Offset (12b) |
+-------------------+-------------------+
|  Capability-specific registers        | DW 1+
|  (长度由 Cap ID 决定)                  |
+---------------------------------------+
```

**字段说明：**
- **Extended Capability ID** (Bits 15:0)：标识能力类型（16 位，支持 65536 种）
- **Capability Version** (Bits 19:16)：版本号（4 位）
- **Next Capability Offset** (Bits 31:20)：下一个 Extended Capability 的偏移（12 位，4 字节对齐）
  - 0x000 表示链表结束
  - 实际偏移 = Next Offset × 4

### 链表遍历

从偏移 0x100 开始遍历 Extended Capability 链表：

```c
// 查找特定 Extended Capability
u16 find_ext_capability(struct pci_dev *dev, u16 cap_id) {
    u16 pos = 0x100;  // 起始偏移
    u32 header;
    
    while (pos) {
        // 读取 Extended Capability Header
        pci_read_config_dword(dev, pos, &header);
        
        u16 id = header & 0xFFFF;           // Bits 15:0
        u16 next = (header >> 20) & 0xFFF;  // Bits 31:20
        
        if (id == cap_id)
            return pos;
        
        if (next == 0)
            break;  // 链表结束
        
        pos = next;
    }
    
    return 0;  // 未找到
}
```

**链表遍历示例：**
```
[0x100] AER (ID=0x0001) → Next=0x140
    ↓
[0x140] VC (ID=0x0002) → Next=0x200
    ↓
[0x200] Serial Number (ID=0x0003) → Next=0x000 (结束)
```

## 常见 Extended Capability 类型

### Extended Capability ID 列表

```
ID     名称                           版本  长度      描述
-----  ---------------------------    ----  ------    --------------------------
0x0000 Null Capability                -     4B        保留/链表结束
0x0001 Advanced Error Reporting       1     48B       高级错误报告（AER）
0x0002 Virtual Channel (VC)           1     Var       虚拟通道
0x0003 Device Serial Number           1     12B       64位设备序列号
0x0004 Power Budgeting                1     16B       功率预算
0x0005 Root Complex Link Declaration 1     Var       RC链路声明
0x0007 Root Complex Event Collector   1     12B       RC事件收集器
0x0009 Multi-Function VC              1     Var       多功能虚拟通道
0x000B Multicast                      1     Var       多播
0x000D Access Control Services (ACS)  1     8B        访问控制服务
0x000E Alternative Routing-ID (ARI)   1     8B        备选路由ID解释
0x000F Address Translation Services   1     Var       地址翻译服务（ATS）
0x0010 SR-IOV                         1     64B       单根IO虚拟化
0x0011 MR-IOV                         1     Var       多根IO虚拟化
0x0013 Resizable BAR                  1     Var       可调整BAR
0x0015 Dynamic Power Allocation       1     16B       动态功率分配
0x0016 TPH Requester                  1     12B       TLP处理提示
0x0017 Latency Tolerance Reporting    1     8B        延迟容忍度报告
0x0018 Secondary PCI Express          1     Var       辅助PCIe
0x0019 Protocol Multiplexing          1     Var       协议复用
0x001A Process Address Space ID       1     8B        进程地址空间ID
0x001B LN Requester (LNR)             1     Var       轻量级通知请求者
0x001D Downstream Port Containment    1     16B       下游端口遏制（DPC）
0x001E L1 PM Substates                1     12B       L1电源管理子状态
0x001F Precision Time Measurement     1     12B       精确时间测量（PTM）
0x0020 PCI Express over M-PHY         1     Var       M-PHY上的PCIe
0x0023 Data Object Exchange           1     24B       数据对象交换（DOE）
0x0026 Physical Layer 16.0 GT/s       1     Var       16.0 GT/s物理层
0x0027 Lane Margining at Receiver     1     8B        接收端通道边界
0x0028 Hierarchy ID                   1     8B        层次结构ID
0x0029 Native PCIe Enclosure Mgmt     1     Var       原生PCIe机箱管理
0x002A Physical Layer 32.0 GT/s       1     Var       32.0 GT/s物理层
0x002B Alternate Protocol             1     Var       备用协议
0x002C System Firmware Intermediary   1     16B       系统固件中介（SFI）
0x002E Integrity and Data Encryption  1     48B       完整性和数据加密（IDE）
```

## 重要 Extended Capability 详解

### 1. Advanced Error Reporting (AER, 0x0001)

AER 提供增强的错误检测、报告和日志记录功能。

**结构布局（48 字节）：**
```
Offset  字段                              大小
------  -------------------------------   ----
0x00    Extended Capability Header        4B
0x04    Uncorrectable Error Status        4B
0x08    Uncorrectable Error Mask          4B
0x0C    Uncorrectable Error Severity      4B
0x10    Correctable Error Status          4B
0x14    Correctable Error Mask            4B
0x18    Advanced Error Capabilities       4B
0x1C    Header Log Register (4 DWords)    16B
0x2C    Root Error Command (仅Root Port)  4B
0x30    Root Error Status                 4B
0x34    Error Source Identification       4B
```

**错误类型：**
```
不可纠正错误（Uncorrectable Errors）：
- Data Link Protocol Error
- Surprise Down Error
- Poisoned TLP
- Flow Control Protocol Error
- Completion Timeout
- Completer Abort
- Unexpected Completion
- Receiver Overflow
- Malformed TLP
- ECRC Error
- Unsupported Request

可纠正错误（Correctable Errors）：
- Receiver Error
- Bad TLP
- Bad DLLP
- REPLAY_NUM Rollover
- Replay Timer Timeout
- Advisory Non-Fatal Error
```

详见 **../error-handling/aer.md**。

### 2. Device Serial Number (0x0003)

提供设备唯一的 64 位序列号。

**结构布局（12 字节）：**
```
Offset  字段                          大小
------  ---------------------------   ----
0x00    Extended Capability Header    4B
0x04    Serial Number (Lower 32b)     4B
0x08    Serial Number (Upper 32b)     4B
```

**读取示例：**
```c
u16 dsn_cap = find_ext_capability(dev, 0x0003);
if (dsn_cap) {
    u32 lower, upper;
    pci_read_config_dword(dev, dsn_cap + 4, &lower);
    pci_read_config_dword(dev, dsn_cap + 8, &upper);
    u64 serial = ((u64)upper << 32) | lower;
    printk("Device Serial Number: %016llx\n", serial);
}
```

### 3. Power Budgeting (0x0004)

声明设备的功率需求和预算。

**结构布局（16 字节）：**
```
Offset  字段                          大小
------  ---------------------------   ----
0x00    Extended Capability Header    4B
0x04    Data Select                   4B
0x08    Data Register                 4B
0x0C    Power Budget Capability       4B
```

**Data Register 字段：**
```
Bits 7:0   - Base Power (0.1W 单位)
Bits 9:8   - Data Scale (乘数因子)
             00 = 1.0x
             01 = 0.1x
             10 = 0.01x
             11 = 0.001x
Bits 12:10 - PM Sub-State
Bits 14:13 - PM State
Bits 17:15 - Type (Thermal, Chassis, System)
Bits 20:18 - Power Rail
```

### 4. Virtual Channel (VC, 0x0002)

配置虚拟通道以支持流量分类和 QoS。

**结构布局（可变长度）：**
```
Offset  字段                          大小
------  ---------------------------   ----
0x00    Extended Capability Header    4B
0x04    Port VC Capability 1          4B
0x08    Port VC Capability 2          4B
0x0C    Port VC Control               2B
0x0E    Port VC Status                2B
0x10    VC Resource Control/Status    每个VC 12字节
```

详见 **../transaction-layer/virtual-channels.md**。

### 5. SR-IOV (0x0010)

单根 IO 虚拟化，允许单个物理设备呈现为多个虚拟设备。

**结构布局（64 字节）：**
```
Offset  字段                          大小
------  ---------------------------   ----
0x00    Extended Capability Header    4B
0x04    SR-IOV Capabilities           4B
0x08    SR-IOV Control                2B
0x0A    SR-IOV Status                 2B
0x0C    InitialVFs                    2B
0x0E    TotalVFs                      2B
0x10    NumVFs                        2B
0x12    Function Dependency Link      1B
0x14    First VF Offset               2B
0x16    VF Stride                     2B
0x18    VF Device ID                  2B
0x1C    Supported Page Sizes          4B
0x20    System Page Size              4B
0x24    VF BAR0-5                     24B (6×4B)
0x3C    VF Migration State Array Offset 4B
```

详见 **../advanced-features/sriov.md**。

### 6. Access Control Services (ACS, 0x000D)

控制 P2P（点对点）流量路由和隔离。

**结构布局（8 字节）：**
```
Offset  字段                          大小
------  ---------------------------   ----
0x00    Extended Capability Header    4B
0x04    ACS Capability                2B
0x06    ACS Control                   2B
```

**ACS Capability/Control 位：**
```
Bit 0  - ACS Source Validation
Bit 1  - ACS Translation Blocking
Bit 2  - ACS P2P Request Redirect
Bit 3  - ACS P2P Completion Redirect
Bit 4  - ACS Upstream Forwarding
Bit 5  - ACS P2P Egress Control
Bit 6  - ACS Direct Translated P2P
```

详见 **../advanced-features/acs.md**。

### 7. Alternative Routing-ID Interpretation (ARI, 0x000E)

扩展单个设备支持的功能数从 8 个（3 位）到 256 个（8 位）。

**结构布局（8 字节）：**
```
Offset  字段                          大小
------  ---------------------------   ----
0x00    Extended Capability Header    4B
0x04    ARI Capability                2B
0x06    ARI Control                   2B
```

**关键字段：**
```
ARI Capability:
  Bit 0     - MFVC Function Groups Capability
  Bits 15:8 - Next Function Number

ARI Control:
  Bit 0     - MFVC Function Groups Enable
  Bit 1     - ACS Function Groups Enable
  Bits 6:4  - Function Group
```

详见 **../advanced-features/ari.md**。

### 8. Precision Time Measurement (PTM, 0x001F)

提供纳秒级精度的时间同步。

**结构布局（12 字节）：**
```
Offset  字段                          大小
------  ---------------------------   ----
0x00    Extended Capability Header    4B
0x04    PTM Capability                4B
0x08    PTM Control                   4B
```

**PTM Capability：**
```
Bit 0     - PTM Requester Capable
Bit 1     - PTM Responder Capable
Bit 2     - PTM Root Capable
Bits 15:8 - Local Clock Granularity (纳秒)
```

详见 **../advanced-features/ptm.md**。

### 9. L1 PM Substates (0x001E)

L1 电源管理子状态，提供更细粒度的低功耗模式。

**结构布局（12 字节）：**
```
Offset  字段                          大小
------  ---------------------------   ----
0x00    Extended Capability Header    4B
0x04    L1 PM Substates Capabilities  4B
0x08    L1 PM Substates Control 1     4B
0x0C    L1 PM Substates Control 2     4B (可选)
```

**L1 子状态：**
```
L1.0  - 传统 L1 (ASPM L1)
L1.1  - L1 with PCI-PM L1.1 (Clock Gating)
L1.2  - L1 with PCI-PM L1.2 (Power Gating)
```

详见 **../power-management/aspm.md**。

### 10. Data Object Exchange (DOE, 0x0023)

提供安全的数据对象交换机制，支持 CMA/SPDM 等协议。

**结构布局（24 字节）：**
```
Offset  字段                          大小
------  ---------------------------   ----
0x00    Extended Capability Header    4B
0x04    DOE Capability                4B
0x08    DOE Control                   4B
0x0C    DOE Status                    4B
0x10    DOE Write Data Mailbox        4B
0x14    DOE Read Data Mailbox         4B
```

详见 **../advanced-features/doe.md**。

### 11. Downstream Port Containment (DPC, 0x001D)

下游端口遏制，在错误发生时快速隔离故障设备。

**结构布局（16 字节）：**
```
Offset  字段                          大小
------  ---------------------------   ----
0x00    Extended Capability Header    4B
0x04    DPC Capability                2B
0x06    DPC Control                   2B
0x08    DPC Status                    2B
0x0A    DPC Error Source ID           2B
0x0C    RP PIO Status (可选)          4B
0x10    RP PIO Mask (可选)            4B
```

**DPC 触发条件：**
- 不可纠正错误（Uncorrectable Error）
- ERR_FATAL 消息
- ERR_NONFATAL 消息（可配置）

### 12. Resizable BAR (0x0013)

允许动态调整 BAR 大小。

**结构布局（可变长度）：**
```
每个 Resizable BAR 占用 8 字节：

Offset  字段                          大小
------  ---------------------------   ----
0x00    Extended Capability Header    4B
0x04    Resizable BAR Capability      4B (BAR 0)
0x08    Resizable BAR Control         4B (BAR 0)
0x0C    Resizable BAR Capability      4B (BAR 1)
0x10    Resizable BAR Control         4B (BAR 1)
...
```

**Capability Register：**
```
Bits 23:0  - Sizes Supported (每位表示 2^(n+20) 字节)
             Bit 0 = 1MB, Bit 1 = 2MB, ..., Bit 23 = 8TB
```

**Control Register：**
```
Bits 4:0   - BAR Size (编码为 2^(n+20) 字节)
Bits 12:8  - BAR Index
```

### 13. Integrity and Data Encryption (IDE, 0x002E)

提供链路层加密和完整性保护。

**结构布局（48 字节）：**
```
Offset  字段                          大小
------  ---------------------------   ----
0x00    Extended Capability Header    4B
0x04    IDE Capability                4B
0x08    IDE Control                   4B
0x0C    Link IDE Stream Status        4B
0x10    Selective IDE Stream Cap      8B
0x18    Selective IDE Stream Control  8B
0x20    IDE RID Association Reg 1     4B
0x24    IDE RID Association Reg 2     4B
0x28    IDE Address Association Reg 1 4B
0x2C    IDE Address Association Reg 2 4B
```

详见 **../advanced-features/ide.md**。

## Extended Capability 编程实践

### 初始化流程

```c
// 1. 枚举所有 Extended Capabilities
void enumerate_ext_caps(struct pci_dev *dev) {
    u16 pos = 0x100;
    u32 header;
    
    printk("Extended Capabilities:\n");
    
    while (pos) {
        pci_read_config_dword(dev, pos, &header);
        
        u16 id = header & 0xFFFF;
        u16 version = (header >> 16) & 0xF;
        u16 next = (header >> 20) & 0xFFF;
        
        if (id == 0)
            break;
        
        printk("  [0x%03x] ID=0x%04x Ver=%d\n", pos, id, version);
        
        pos = next;
    }
}

// 2. 配置 AER
void configure_aer(struct pci_dev *dev) {
    u16 aer_cap = find_ext_capability(dev, 0x0001);
    if (!aer_cap)
        return;
    
    // 使能所有不可纠正错误报告
    pci_write_config_dword(dev, aer_cap + 0x08, 0x00000000);  // Unmask
    
    // 配置严重性
    pci_write_config_dword(dev, aer_cap + 0x0C, 0x00462030);
    
    // 使能可纠正错误报告
    pci_write_config_dword(dev, aer_cap + 0x14, 0x00000000);  // Unmask
}

// 3. 读取设备序列号
u64 get_device_serial_number(struct pci_dev *dev) {
    u16 dsn_cap = find_ext_capability(dev, 0x0003);
    if (!dsn_cap)
        return 0;
    
    u32 lower, upper;
    pci_read_config_dword(dev, dsn_cap + 4, &lower);
    pci_read_config_dword(dev, dsn_cap + 8, &upper);
    
    return ((u64)upper << 32) | lower;
}

// 4. 配置 SR-IOV
int enable_sriov(struct pci_dev *dev, int num_vfs) {
    u16 sriov_cap = find_ext_capability(dev, 0x0010);
    if (!sriov_cap)
        return -ENODEV;
    
    u16 total_vfs;
    pci_read_config_word(dev, sriov_cap + 0x0E, &total_vfs);
    
    if (num_vfs > total_vfs)
        return -EINVAL;
    
    // 写入要启用的 VF 数量
    pci_write_config_word(dev, sriov_cap + 0x10, num_vfs);
    
    // 启用 SR-IOV
    u16 ctrl;
    pci_read_config_word(dev, sriov_cap + 0x08, &ctrl);
    ctrl |= 0x0001;  // VF Enable
    pci_write_config_word(dev, sriov_cap + 0x08, ctrl);
    
    return 0;
}
```

### 最佳实践

```c
// 驱动初始化检查清单
void driver_init_ext_caps(struct pci_dev *dev) {
    // 1. 检查 AER 支持
    if (find_ext_capability(dev, 0x0001)) {
        configure_aer(dev);
        dev_info(&dev->dev, "AER enabled\n");
    }
    
    // 2. 读取设备序列号
    u64 sn = get_device_serial_number(dev);
    if (sn)
        dev_info(&dev->dev, "S/N: %016llx\n", sn);
    
    // 3. 检查 ACS（IOMMU 隔离需要）
    if (find_ext_capability(dev, 0x000D)) {
        configure_acs(dev);
    }
    
    // 4. 检查 PTM（时间同步）
    if (find_ext_capability(dev, 0x001F)) {
        enable_ptm(dev);
    }
    
    // 5. 检查 SR-IOV（虚拟化）
    if (find_ext_capability(dev, 0x0010)) {
        dev->is_physfn = 1;
        init_sriov(dev);
    }
}
```

## 调试与诊断

### lspci 输出示例

```bash
$ lspci -vvv -s 01:00.0 | grep -A 20 "Extended"

Capabilities: [100 v1] Advanced Error Reporting
    UESta:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    UEMsk:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    UESvrt: DLP+ SDES- TLP+ FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
    CESta:  RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
    CEMsk:  RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
    AERCap: First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
            MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap-

Capabilities: [140 v1] Device Serial Number 00-00-00-00-12-34-56-78

Capabilities: [1c0 v1] Latency Tolerance Reporting
    Max snoop latency: 0ns
    Max no snoop latency: 0ns

Capabilities: [220 v1] Secondary PCI Express
    LnkCtl3: LnkEquIntrruptEn- PerformEqu-

Capabilities: [300 v1] L1 PM Substates
    L1SubCap: PCI-PM_L1.2+ PCI-PM_L1.1+ ASPM_L1.2+ ASPM_L1.1+ L1_PM_Substates+
    L1SubCtl1: PCI-PM_L1.2- PCI-PM_L1.1- ASPM_L1.2- ASPM_L1.1-
```

### 常见问题

```
问题：Extended Capabilities 链表遍历失败
检查：
1. 是否通过 ECAM 访问？（传统 IO 端口方法无法访问）
2. Next Offset 是否有效？（0x000 表示结束，需要 4 字节对齐）
3. 是否形成环路？（记录已访问的偏移）

问题：AER 错误不上报
检查：
1. AER Capability 的 Mask 寄存器是否正确配置？
2. PCIe Capability 的 Device Control 是否使能错误报告？
3. Root Port 的 Root Control 是否使能错误中断？

问题：SR-IOV VF 不出现
检查：
1. SR-IOV Control 的 VF Enable 位是否置位？
2. NumVFs 是否设置？
3. VF BARs 是否有足够的 MMIO 空间？
4. 系统是否支持 IOMMU？
```

## 相关资源

- **capabilities.md**: 传统 PCI Capabilities（0x40-0xFF）
- **config-space.md**: 配置空间总览
- **../error-handling/aer.md**: AER 详细指南
- **../advanced-features/sriov.md**: SR-IOV 实现
- **../advanced-features/acs.md**: ACS 访问控制
- **../advanced-features/ptm.md**: PTM 时间同步

## 总结

PCIe Extended Capabilities 是现代 PCIe 设备的核心特性，提供了：

1. **错误管理**：AER 提供详细的错误检测和日志
2. **虚拟化**：SR-IOV 支持硬件级虚拟化
3. **安全性**：ACS、IDE 提供隔离和加密
4. **性能优化**：VC、TPH 提供 QoS 和缓存优化
5. **电源管理**：L1 PM Substates 提供细粒度功耗控制
6. **时间同步**：PTM 提供纳秒级精度
7. **可扩展性**：Resizable BAR、ARI 提供灵活配置

掌握 Extended Capabilities 的结构和编程方法是开发高性能、高可靠性 PCIe 驱动和系统的基础。
