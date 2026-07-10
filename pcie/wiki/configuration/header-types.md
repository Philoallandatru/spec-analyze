# PCIe 配置空间头部类型

## 概述

PCIe 配置空间的前 64 字节称为配置空间头部（Configuration Space Header），定义了设备的基本标识和控制信息。根据设备类型不同，头部分为两种格式：

- **Type 0 Header**：用于 PCIe 端点设备（Endpoint）
- **Type 1 Header**：用于 PCIe 桥接设备（Bridge/Switch）

头部类型由配置空间偏移 0x0E 处的 Header Type 字段标识。

## 通用头部字段（前 16 字节）

无论 Type 0 还是 Type 1，配置空间的前 16 字节（0x00-0x0F）格式相同：

```
Offset  Bits    字段名称              描述
------  ----    ----------------      ------------------------------------
0x00    15:0    Vendor ID             厂商标识（由 PCI-SIG 分配）
0x02    15:0    Device ID             设备标识（厂商自定义）
0x04    15:0    Command               命令寄存器
0x06    15:0    Status                状态寄存器
0x08    7:0     Revision ID           版本号
0x09    23:0    Class Code            类代码（基类/子类/接口）
0x0C    7:0     Cache Line Size       缓存行大小
0x0D    7:0     Latency Timer         延迟计时器（PCIe 中保留）
0x0E    7:0     Header Type           头部类型
0x0F    7:0     BIST                  内建自检
```

### Header Type 字段解析

```
Bit 7: Multi-Function Device
       0 = 单功能设备
       1 = 多功能设备

Bits 6:0: Header Layout
       0x00 = Type 0 (Endpoint)
       0x01 = Type 1 (Bridge)
       0x02 = CardBus Bridge（已废弃）
```

## Type 0 Header（端点设备）

Type 0 头部用于所有端点设备，包括 NVMe SSD、网卡、显卡等。完整布局如下：

```
+--------+--------+--------+--------+
|   Device ID     |   Vendor ID     | 0x00
+--------+--------+--------+--------+
|     Status      |     Command     | 0x04
+--------+--------+--------+--------+
|        Class Code        | Rev ID | 0x08
+--------+--------+--------+--------+
| BIST   |Header T|Lat Tmr |CchLine | 0x0C
+--------+--------+--------+--------+
|       Base Address 0 (BAR0)       | 0x10
+-----------------------------------+
|       Base Address 1 (BAR1)       | 0x14
+-----------------------------------+
|       Base Address 2 (BAR2)       | 0x18
+-----------------------------------+
|       Base Address 3 (BAR3)       | 0x1C
+-----------------------------------+
|       Base Address 4 (BAR4)       | 0x20
+-----------------------------------+
|       Base Address 5 (BAR5)       | 0x24
+-----------------------------------+
|       CardBus CIS Pointer         | 0x28
+--------+--------+--------+--------+
| Subsystem Dev ID| Subsys Vend ID  | 0x2C
+--------+--------+--------+--------+
|   Expansion ROM Base Address      | 0x30
+--------+--------+--------+--------+
|     Reserved    |Capab Ptr|Reserv | 0x34
+--------+--------+--------+--------+
|            Reserved               | 0x38
+--------+--------+--------+--------+
|Max Lat |Min Gnt |Int Pin |Int Line| 0x3C
+--------+--------+--------+--------+
```

### Type 0 关键字段详解

#### Base Address Registers (BARs)

Type 0 提供 6 个 BAR（0x10-0x27），用于映射设备内存和 I/O 空间：

**BAR 格式 - 内存空间：**
```
Bit 0:     0 (表示内存空间)
Bit 2:1:   类型
           00 = 32-bit 地址
           10 = 64-bit 地址
Bit 3:     Prefetchable
           0 = Non-prefetchable
           1 = Prefetchable
Bits 31:4: Base Address（16-byte 对齐）
```

**BAR 格式 - I/O 空间：**
```
Bit 0:     1 (表示 I/O 空间)
Bit 1:     保留
Bits 31:2: Base Address（4-byte 对齐）
```

**64-bit BAR 示例：**
```
BAR0: 0xFFFFFFF0  ← 低 32 位（bit 2:1 = 10 表示 64-bit）
BAR1: 0x00000001  ← 高 32 位
完整地址: 0x1_FFFFFFF0
```

#### Subsystem IDs（0x2C-0x2F）

子系统标识允许板卡厂商标识其产品：
- **Subsystem Vendor ID**：板卡制造商（如 ASUS、MSI）
- **Subsystem Device ID**：具体板卡型号

例如：NVIDIA GPU 芯片相同，但不同厂商的显卡有不同的子系统 ID。

#### Capabilities Pointer（0x34）

指向配置空间中第一个 Capability 结构的偏移地址（见 capabilities.md）。

#### Interrupt Pin/Line（0x3C-0x3D）

- **Interrupt Line**：传统 PCI 使用，PCIe 中无意义
- **Interrupt Pin**：
  - 0x00 = 不使用 INTx 中断
  - 0x01-0x04 = INTA/INTB/INTC/INTD

## Type 1 Header（桥接设备）

Type 1 头部用于 PCIe Switch、Root Port 等桥接设备，负责连接多个 PCIe 总线：

```
+--------+--------+--------+--------+
|   Device ID     |   Vendor ID     | 0x00
+--------+--------+--------+--------+
|     Status      |     Command     | 0x04
+--------+--------+--------+--------+
|        Class Code        | Rev ID | 0x08
+--------+--------+--------+--------+
| BIST   |Header T|Lat Tmr |CchLine | 0x0C
+--------+--------+--------+--------+
|       Base Address 0 (BAR0)       | 0x10
+-----------------------------------+
|       Base Address 1 (BAR1)       | 0x14
+-----------------------------------+
|Sec Lat |Subord# |Second# |Primary#| 0x18
+--------+--------+--------+--------+
|  Sec Status     |IO Limit|IO Base | 0x1C
+--------+--------+--------+--------+
|  Memory Limit   |   Memory Base   | 0x20
+--------+--------+--------+--------+
|Prefetch Mem Lim |Prefetch Mem Base| 0x24
+--------+--------+--------+--------+
|  Prefetch Base Upper 32 bits      | 0x28
+-----------------------------------+
|  Prefetch Limit Upper 32 bits     | 0x2C
+-----------------------------------+
| IO Limit Upper  | IO Base Upper   | 0x30
+--------+--------+--------+--------+
|     Reserved    |Capab Ptr|Reserv | 0x34
+-----------------------------------+
|   Expansion ROM Base Address      | 0x38
+--------+--------+--------+--------+
|  Bridge Control | Int Pin|Int Line| 0x3C
+--------+--------+--------+--------+
```

### Type 1 关键字段详解

#### 总线编号（0x18-0x1A）

```
Primary Bus Number (0x18):
    桥接器主端总线号（上游总线）

Secondary Bus Number (0x19):
    桥接器次端总线号（下游总线起始）

Subordinate Bus Number (0x1A):
    下游总线范围的最大编号
```

**总线拓扑示例：**
```
                  Bus 0
                    |
              [Root Port]
         Primary=0, Secondary=1, Subordinate=10
                    |
                  Bus 1
                    |
               [Switch]
         Primary=1, Secondary=2, Subordinate=10
                /       \
            Bus 2       Bus 5
             |           |
        [Endpoint]   [Endpoint]
```

#### 内存/IO 窗口

桥接器使用 Base/Limit 寄存器定义地址转发范围：

**内存窗口（0x20-0x23）：**
```
Memory Base (0x20):    起始地址 [31:20]，低 20 位为 0
Memory Limit (0x22):   结束地址 [31:20]，低 20 位为 0xFFFFF

示例：
  Memory Base  = 0xF000  → 0xF0000000
  Memory Limit = 0xF7FF  → 0xF7FFFFFF
  转发范围：0xF0000000 - 0xF7FFFFFF（128MB）
```

**Prefetchable 内存窗口（0x24-0x2F）：**

支持 64-bit 地址，由 4 个寄存器组成：
- Prefetchable Memory Base (0x24)
- Prefetchable Memory Limit (0x26)
- Prefetchable Base Upper 32 bits (0x28)
- Prefetchable Limit Upper 32 bits (0x2C)

**I/O 窗口（0x1C, 0x30-0x33）：**

PCIe 中通常不使用，但为兼容性保留。

#### Bridge Control（0x3E-0x3F）

桥接器控制寄存器：

```
Bit 0:  Parity Error Response Enable
Bit 1:  SERR# Enable
Bit 2:  ISA Enable
Bit 3:  VGA Enable - VGA 地址转发
Bit 5:  Master Abort Mode
Bit 6:  Secondary Bus Reset - 复位下游总线
Bit 7:  Fast Back-to-Back Enable
Bit 8:  Primary Discard Timer
Bit 9:  Secondary Discard Timer
Bit 10: Discard Timer Status
Bit 11: Discard Timer SERR# Enable
```

## 使用场景与最佳实践

### Type 0 设备初始化流程

```
1. 读取 Vendor/Device ID 识别设备
   ├─ 0xFFFF/0xFFFF 表示无设备
   └─ 匹配驱动程序

2. 检查 Class Code 确认设备类型
   例如：0x010802 = NVMe SSD

3. 扫描 BARs 确定资源需求
   ├─ 写入全 1 读回判断大小
   ├─ 系统分配地址空间
   └─ 写入分配的基地址

4. 配置 Command 寄存器
   ├─ Bit 0: I/O Space Enable
   ├─ Bit 1: Memory Space Enable
   ├─ Bit 2: Bus Master Enable
   └─ Bit 10: Interrupt Disable

5. 设置中断机制
   ├─ 读取 Capabilities Pointer
   ├─ 遍历 Capability 链表
   └─ 配置 MSI/MSI-X

6. 启动设备驱动程序
```

### Type 1 桥接器枚举流程

```
1. 初始化桥接器
   ├─ 分配次端总线号
   ├─ 设置 Subordinate = 0xFF（临时最大值）
   └─ 启用 Memory/IO 转发

2. 扫描次端总线（深度优先）
   ├─ 探测设备 0-31
   ├─ 对每个桥接器递归扫描
   └─ 更新 Subordinate Bus Number

3. 资源分配
   ├─ 收集下游设备资源需求
   ├─ 合并成连续地址范围
   └─ 配置 Base/Limit 寄存器

4. 最终化配置
   ├─ 回写正确的 Subordinate Number
   ├─ 启用桥接器控制位
   └─ 清除 Secondary Bus Reset
```

### 多功能设备处理

当 Header Type 的 Bit 7 为 1 时，设备有多个功能（Function）：

```
扫描逻辑：
  if (Header_Type & 0x80):
      # 多功能设备，扫描功能 0-7
      for func in range(8):
          check_function(bus, dev, func)
  else:
      # 单功能设备，只检查功能 0
      check_function(bus, dev, 0)
```

## 寄存器编程示例

### 读取设备信息（Type 0）

```c
// 读取 Vendor/Device ID
uint16_t vendor = pci_read_config16(bus, dev, func, 0x00);
uint16_t device = pci_read_config16(bus, dev, func, 0x02);

// 读取 Class Code
uint32_t class_rev = pci_read_config32(bus, dev, func, 0x08);
uint8_t revision = class_rev & 0xFF;
uint32_t class_code = class_rev >> 8;

// 检查 Header Type
uint8_t header = pci_read_config8(bus, dev, func, 0x0E);
bool is_multifunction = (header & 0x80) != 0;
uint8_t type = header & 0x7F;
```

### 配置 BAR（Type 0）

```c
// 探测 BAR 大小
pci_write_config32(bus, dev, func, 0x10, 0xFFFFFFFF);
uint32_t size_mask = pci_read_config32(bus, dev, func, 0x10);

// 判断 BAR 类型
if (size_mask & 0x01) {
    // I/O BAR
    uint32_t size = ~(size_mask & 0xFFFFFFFC) + 1;
} else {
    // Memory BAR
    bool is_64bit = ((size_mask & 0x06) == 0x04);
    bool prefetchable = (size_mask & 0x08) != 0;
    
    if (is_64bit) {
        // 64-bit BAR 占用两个寄存器
        uint32_t size_low = ~(size_mask & 0xFFFFFFF0) + 1;
    }
}

// 写入分配的地址
pci_write_config32(bus, dev, func, 0x10, allocated_address);
```

### 配置桥接器（Type 1）

```c
// 设置总线编号
pci_write_config8(bus, dev, func, 0x18, primary_bus);
pci_write_config8(bus, dev, func, 0x19, secondary_bus);
pci_write_config8(bus, dev, func, 0x1A, subordinate_bus);

// 配置内存窗口（128MB @ 0xF0000000）
pci_write_config16(bus, dev, func, 0x20, 0xF000); // Base
pci_write_config16(bus, dev, func, 0x22, 0xF7FF); // Limit

// 禁用 Prefetchable 窗口
pci_write_config16(bus, dev, func, 0x24, 0xFFF0);
pci_write_config16(bus, dev, func, 0x26, 0x0000);

// 启用内存转发
uint16_t cmd = pci_read_config16(bus, dev, func, 0x04);
cmd |= 0x06; // Memory Space + Bus Master
pci_write_config16(bus, dev, func, 0x04, cmd);
```

## 相关资源

- **PCI Express Base Specification**: 第 7.5 节详述配置空间布局
- **PCI Local Bus Specification**: Type 0/1 头部的原始定义
- **capabilities.md**: Capability 链表机制
- **../enumeration/bus-scanning.md**: 设备枚举详细流程

## 总结

Type 0 和 Type 1 头部是 PCIe 配置空间的基础，分别适用于端点设备和桥接设备。理解这两种头部格式是 PCIe 系统编程和驱动开发的第一步。Type 0 侧重资源映射（BARs），Type 1 侧重拓扑管理（总线编号、地址窗口），两者协作构建完整的 PCIe 层次结构。
