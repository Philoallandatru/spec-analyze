# 路由机制详解

PCIe 路由机制决定了 TLP（Transaction Layer Packet）如何在复杂的 PCIe 拓扑中从源设备传递到目标设备。

---

## 概述

在 PCIe 拓扑中，设备通过 **Root Complex**、**Switch** 和 **Bridge** 相互连接。当一个设备发送 TLP 时，中间的路由设备必须决定将该数据包转发到哪个端口。PCIe 定义了三种路由机制来实现这一功能。

**为什么需要路由？**

```
         Root Complex (Bus 0)
              |
    ┌─────────┴─────────┐
    |                   |
  Port 1              Port 2
 (Bus 1-5)           (Bus 6-10)
    |                   |
  Switch              NVMe SSD
    |                 (Bus 6)
  ┌─┴─┐
  |   |
GPU  NIC
```

当 CPU 访问 NVMe SSD 时，Root Complex 必须判断该 TLP 应该转发到 Port 2（而不是 Port 1），这就是路由的作用。

**三种路由机制**：

| 路由类型 | 使用场景 | 路由依据 |
|---------|---------|---------|
| **Address-Based Routing** | Memory Request | 目标地址 |
| **ID-Based Routing** | Configuration Request, Completion | Bus:Device.Function |
| **Implicit Routing** | 某些 Message | 端口类型和消息类型 |

---

## Address-Based Routing（基于地址路由）

### 工作原理

用于 **Memory Read/Write Request**，根据 TLP 中的目标地址进行路由。

每个 Bridge/Switch 端口维护**地址窗口**（Address Window），记录其下游设备的地址范围。


### 地址窗口配置

Bridge 配置空间中的地址窗口寄存器：

```
偏移 0x20: Memory Base       - 下游内存起始地址
偏移 0x22: Memory Limit      - 下游内存结束地址
偏移 0x24: Prefetchable Base - 可预取内存起始地址
偏移 0x26: Prefetchable Limit- 可预取内存结束地址
偏移 0x28: Pref Base Upper   - 64 位地址高 32 位
偏移 0x2C: Pref Limit Upper  - 64 位地址高 32 位
```

### 路由决策流程

```
收到 Memory TLP
  ↓
检查：地址在本地 BAR 范围内？
  ├─ 是 → 本地处理
  └─ 否 → 是 Bridge/Switch？
           ├─ 否 → 丢弃或上报错误
           └─ 是 → 检查所有下游端口
                    ├─ 地址在某端口窗口内 → 转发到该端口
                    └─ 都不匹配 → 转发到上游端口
```

### 实际示例

**拓扑配置**：

```
Root Complex
  ├─ Port 1: 管理地址 0xF000_0000 - 0xF0FF_FFFF
  │    └─ GPU (BAR0: 0xF000_0000, 256MB)
  └─ Port 2: 管理地址 0xF100_0000 - 0xF1FF_FFFF
       └─ NVMe SSD (BAR0: 0xF100_0000, 16KB)
```

**场景 1：CPU 读取 NVMe 寄存器**

```
1. CPU 发起 Memory Read: 地址 0xF100_0000, 长度 4 字节
   ↓
2. Root Complex 收到 TLP:
   - Fmt/Type: 0x00 (Memory Read, 3DW)
   - Address: 0xF100_0000
   ↓
3. Root Complex 路由决策:
   - 检查 Port 1 窗口: 0xF000_0000 - 0xF0FF_FFFF ❌ 不匹配
   - 检查 Port 2 窗口: 0xF100_0000 - 0xF1FF_FFFF ✅ 匹配
   ↓
4. 转发到 Port 2 → 到达 NVMe SSD
```

**场景 2：GPU 直接访问系统内存（P2P DMA）**

```
1. GPU 发起 Memory Write: 地址 0x8000_0000 (系统 RAM)
   ↓
2. Root Complex Port 1 收到 TLP
   ↓
3. 检查所有下游端口窗口: 都不匹配
   ↓
4. 转发到上游（Root Complex 主接口）→ 到达内存控制器
```

### FEMU 代码实现

```c
// hw/pci/pci_bridge.c
// 获取 Bridge 的内存基地址
static pcibus_t pci_config_get_memory_base(const PCIDevice *d, uint32_t base)
{
    return ((pcibus_t)pci_get_word(d->config + base) & 0xfff0) << 16;
}

// 检查地址是否在 Bridge 的窗口内
static bool pci_bridge_has_region(PCIDevice *bridge, 
                                   pcibus_t addr, 
                                   pcibus_t size)
{
    pcibus_t base = pci_config_get_memory_base(bridge, PCI_MEMORY_BASE);
    pcibus_t limit = pci_config_get_memory_base(bridge, PCI_MEMORY_LIMIT);
    
    if (ranges_overlap(addr, size, base, limit - base + 1)) {
        return true;
    }
    return false;
}
```

**Linux Kernel 中的地址窗口配置**：

```c
// drivers/pci/setup-bus.c
// 配置 Bridge 的地址窗口
static void pci_setup_bridge_mmio(struct pci_dev *bridge)
{
    struct resource *res = &bridge->resource[PCI_BRIDGE_RESOURCES];
    u32 l;
    
    // 设置 Memory Base/Limit
    l = (res->start >> 16) & 0xfff0;
    l |= (res->end >> 16) & 0xfff00000;
    pci_write_config_dword(bridge, PCI_MEMORY_BASE, l);
}
```

---

## ID-Based Routing（基于 ID 路由）

### 工作原理

用于 **Configuration Request** 和 **Completion**，根据 **Bus:Device.Function (BDF)** 进行路由。

每个 Bridge 维护**总线号范围**：

```
Bridge 配置空间（偏移 0x18-0x1A）：

偏移 0x18: Primary Bus Number    - Bridge 所在的总线号
偏移 0x19: Secondary Bus Number  - Bridge 下游的直连总线号
偏移 0x1A: Subordinate Bus Number- Bridge 下游的最大总线号

示例：
  Primary: 0 (Bridge 在 Bus 0 上)
  Secondary: 1 (直接下游是 Bus 1)
  Subordinate: 5 (下游范围是 Bus 1-5)
```

### 路由决策流程

```
收到 ID-Based TLP (目标 Bus = N)
  ↓
是 Bridge/Switch?
  ├─ 否 → 检查是否是本设备 (Bus/Dev/Func 匹配)
  │        ├─ 是 → 本地处理
  │        └─ 否 → 丢弃
  │
  └─ 是 → 检查总线号范围
           ├─ N == Secondary Bus → Type 1 转为 Type 0，转发
           ├─ Secondary < N <= Subordinate → 保持 Type 1，转发
           └─ 其他 → 转发到上游
```

### Configuration Request 类型

**Type 0**：访问直连设备

```
TLP Header (DW 2):
┌─────────┬─────────┬──────────┬─────────┬────────┬──┐
│ Bus (8) │ Dev (5) │ Func (3) │ ExtReg  │ Reg(6) │00│
└─────────┴─────────┴──────────┴─────────┴────────┴──┘

用于访问当前总线上的设备（Bridge 直接下游）
```

**Type 1**：穿越 Bridge

```
TLP Header (DW 2):
┌─────────┬─────────┬──────────┬─────────┬────────┬──┐
│ Bus (8) │ Dev (5) │ Func (3) │ ExtReg  │ Reg(6) │00│
└─────────┴─────────┴──────────┴─────────┴────────┴──┘

用于访问下游 Bridge 后面的设备
```

### 实际示例

**拓扑配置**：

```
Bus 0: Root Complex
  └─ Bridge A (Dev 1, Func 0)
       Primary: 0, Secondary: 1, Subordinate: 3
       Bus 1:
         ├─ NVMe SSD (Dev 0, Func 0) → BDF: 01:00.0
         └─ Bridge B (Dev 1, Func 0)
              Primary: 1, Secondary: 2, Subordinate: 3
              Bus 2:
                └─ GPU (Dev 0, Func 0) → BDF: 02:00.0
```

**场景 1：读取 NVMe 的 Vendor ID**

```
1. Root Complex 生成 Config Read Type 1:
   - Bus: 1, Device: 0, Function: 0
   - Register: 0x00 (Vendor ID)
   ↓
2. Bridge A 收到 TLP:
   - 目标 Bus = 1
   - Secondary Bus = 1 ✅ 匹配
   ↓
3. Bridge A 转换为 Type 0（去掉 Bus 号层级）
   转发到下游总线
   ↓
4. NVMe SSD (01:00.0) 收到 Type 0 Config Read
   - Device: 0, Function: 0 ✅ 匹配
   ↓
5. NVMe 响应 Completion with Data: 0x144D (Samsung)
```

**场景 2：读取 GPU 的配置**

```
1. Root Complex 生成 Config Read Type 1:
   - Bus: 2, Device: 0, Function: 0
   ↓
2. Bridge A 收到 TLP:
   - 目标 Bus = 2
   - Secondary (1) < 2 <= Subordinate (3) ✅ 在范围内
   ↓
3. Bridge A 保持 Type 1，转发到下游
   ↓
4. Bridge B 收到 TLP:
   - 目标 Bus = 2
   - Secondary Bus = 2 ✅ 匹配
   ↓
5. Bridge B 转换为 Type 0，转发
   ↓
6. GPU 收到并响应
```


### Completion 的反向路由

Completion TLP 使用原始请求的 **Requester ID** 进行反向路由：

```
Memory Read Request:
  Requester ID: 01:00.0 (NVMe)
  Tag: 0x05
  ↓
Completion with Data:
  Requester ID: 01:00.0 ← 用于路由回 NVMe
  Tag: 0x05 ← 用于匹配请求
```

### FEMU 代码实现

```c
// hw/pci/pci.c
// 检查目标总线是否在 Bridge 的管理范围内
static bool pci_secondary_bus_in_range(PCIDevice *dev, int bus_num)
{
    return !(pci_get_word(dev->config + PCI_BRIDGE_CONTROL) &
             PCI_BRIDGE_CTL_BUS_RESET) &&
        pci_get_byte(dev->config + PCI_SECONDARY_BUS) <= bus_num &&
        pci_get_byte(dev->config + PCI_SUBORDINATE_BUS) >= bus_num;
}

// 根据总线号查找目标设备
PCIDevice *pci_find_device(PCIBus *bus, int bus_num, uint8_t devfn)
{
    bus = pci_find_bus_nr(bus, bus_num);
    if (!bus) {
        return NULL;
    }
    return bus->devices[devfn];
}
```

**Linux Kernel 总线号分配**：

```c
// drivers/pci/probe.c
// 扫描并分配总线号
unsigned int pci_scan_child_bus(struct pci_bus *bus)
{
    unsigned int max = bus->busn_res.start;
    
    // 扫描每个设备
    for (devfn = 0; devfn < 256; devfn += 8) {
        struct pci_dev *dev = pci_scan_single_device(bus, devfn);
        
        // 如果是 Bridge，递归扫描下游
        if (dev && pci_is_bridge(dev)) {
            max = pci_scan_bridge(bus, dev, max);
        }
    }
    return max;
}
```

---

## Implicit Routing（隐式路由）

### 工作原理

用于**某些特殊 Message**，路由规则由端口类型和消息类型隐式决定，无需地址或 ID。

**典型应用场景**：

| Message 类型 | 路由规则 |
|-------------|---------|
| **INTx Interrupt Assert/Deassert** | 向上游路由，直到 Root Complex |
| **PM_PME (电源管理事件)** | 向上游路由到 Root Complex |
| **ERR_COR / ERR_NONFATAL / ERR_FATAL** | 向上游路由到 Root Complex（错误报告）|
| **PME_TO_Ack** | 从 Root Complex 向下游广播 |

### INTx 中断路由示例

```
设备发送 INTx 中断消息：
┌────────────────────────────────────┐
│ Message Type: INTx Assert (0x10)   │
│ No Address, No Bus/Dev/Func       │
└────────────────────────────────────┘
  ↓
Switch/Bridge 收到：
  - 隐式规则：INTx 消息总是向上游转发
  ↓
Root Complex 收到：
  - 触发 CPU 的中断控制器 (IOAPIC/GIC)
```

### 错误报告路由

```
设备检测到错误：
  ↓
生成 ERR_NONFATAL Message:
  ┌─────────────────────────────────┐
  │ Message Code: 0x21              │
  │ 携带错误详情（AER 扩展能力）      │
  └─────────────────────────────────┘
  ↓
所有中间 Switch/Bridge:
  - 隐式向上游转发
  - 可选：记录到本地 AER 日志
  ↓
Root Complex:
  - 触发系统错误处理程序
  - Linux: AER 驱动捕获并处理
```

### 实际应用场景

**场景：NVMe 设备报告可纠正错误**

```
1. NVMe 检测到 Bad TLP (可纠正错误)
   ↓
2. NVMe 生成 ERR_COR Message:
   Fmt/Type: 0x30 (Message)
   Message Code: 0x20 (ERR_COR)
   ↓
3. Switch 收到 Message:
   - 端口类型：Downstream Port → 向上游转发
   ↓
4. Root Complex 收到:
   - 记录到 AER Root Error Status
   - 触发中断通知 OS
   ↓
5. Linux AER 驱动处理:
   - 读取错误详情
   - 记录到系统日志
   - 决定恢复策略
```

### FEMU 代码实现

```c
// hw/pci/pcie_aer.c
// 生成错误消息（隐式路由到 Root Complex）
void pcie_aer_msg(PCIDevice *dev, const PCIEAERMsg *msg)
{
    // 向上游发送错误消息
    if (pci_is_express(dev)) {
        pcie_aer_inject_error(dev, &msg->err);
    }
    
    // 隐式路由：消息自动传播到 Root Complex
}
```

---

## 路由机制对比表

| 特性 | Address-Based | ID-Based | Implicit |
|-----|--------------|----------|----------|
| **使用 TLP** | Memory Read/Write | Config Read/Write<br/>Completion | Message |
| **路由依据** | 64 位地址 | Bus:Device.Function | 端口类型 + 消息类型 |
| **配置方式** | BAR + Bridge Memory Base/Limit | Bus 号范围 | 硬编码规则 |
| **性能特点** | 高效，适合大数据传输 | 慢，仅用于配置 | 低开销，用于事件 |
| **典型场景** | DMA、MMIO 访问 | 设备枚举、配置 | 中断、电源管理、错误报告 |
| **路由复杂度** | 中等（地址比较） | 中等（范围检查）| 低（固定规则）|

---

## 路由优化和最佳实践

### 1. 地址窗口对齐

**问题**：未对齐的地址窗口导致性能下降

```
❌ 错误配置：
Bridge A: 0xF000_0000 - 0xF00F_FFFF (1MB)
Bridge B: 0xF010_0000 - 0xF01F_FFFF (1MB)
→ 窗口不连续，可能导致路由表碎片化

✅ 正确配置：
Bridge A: 0xF000_0000 - 0xF0FF_FFFF (16MB)
Bridge B: 0xF100_0000 - 0xF1FF_FFFF (16MB)
→ 地址空间连续，路由效率高
```

### 2. P2P（Peer-to-Peer）路由

当两个设备在同一 Switch 下游时，数据可以直接在 Switch 内部转发，无需经过 Root Complex：

```
标准路径（低效）：
GPU → Switch → Root Complex → Switch → NVMe

P2P 路径（高效）：
GPU → Switch → NVMe

条件：
1. ACS (Access Control Services) 允许 P2P
2. 地址窗口配置正确
3. 操作系统支持（Linux: CONFIG_PCI_P2PDMA）
```

**Linux 查看 P2P 能力**：

```bash
# 检查 ACS 配置
lspci -vvv -s 01:00.0 | grep -A 10 "Access Control Services"
  ACSCap: SrcValid+ TransBlk+ ReqRedir+ CmpltRedir+ UpstreamFwd+ 
  ACSCtl: SrcValid- TransBlk- ReqRedir- CmpltRedir- UpstreamFwd-

# P2P 启用表示允许 Switch 内部转发
```

### 3. 避免路由死锁

**死锁场景**：

```
设备 A 发送 Non-Posted Request → 设备 B
设备 B 同时发送 Non-Posted Request → 设备 A
双方都在等待 Completion → 死锁
```

**解决方案**：使用虚拟通道（Virtual Channel）分离不同类型的流量。


---

## 实用调试技巧

### 查看 Bridge 的地址窗口

```bash
# 使用 lspci 查看 Bridge 配置
lspci -vvv -s 00:01.0 | grep -A 5 "Region"
  Memory behind bridge: f0000000-f0ffffff [size=16M]
  Prefetchable memory behind bridge: e0000000-efffffff [size=256M]

# 查看总线号范围
lspci -vvv -s 00:01.0 | grep "Bus:"
  Bus: primary=00, secondary=01, subordinate=05
```

### 使用 setpci 读取路由配置

```bash
# 读取 Bridge 的 Memory Base (偏移 0x20)
setpci -s 00:01.0 0x20.w
f000

# 读取 Secondary Bus Number (偏移 0x19)
setpci -s 00:01.0 0x19.b
01
```

### 内核日志中的路由信息

```bash
# 查看 PCIe 总线扫描日志
dmesg | grep -i "pci.*bus"
[    0.234567] pci 0000:00:01.0: bridge window [mem 0xf0000000-0xf0ffffff]
[    0.234568] pci 0000:00:01.0: bridge window [mem 0xe0000000-0xefffffff 64bit pref]
```

### 使用 PCIe 分析工具

**lspci 高级用法**：

```bash
# 显示完整的拓扑树
lspci -tv
-[0000:00]-+-00.0  Intel Corporation Host Bridge
           +-01.0-[01-05]----00.0  Samsung NVMe SSD
           +-02.0-[06]----00.0  NVIDIA GPU

# 显示所有 Bridge 的路由信息
for bridge in $(lspci | grep Bridge | cut -d' ' -f1); do
    echo "=== Bridge $bridge ==="
    lspci -vvv -s $bridge | grep -E "Bus:|Memory behind"
done
```

---

## 总结

### 关键要点

1. ✅ **三种路由机制各司其职**：
   - Address-Based：高效数据传输（Memory 事务）
   - ID-Based：设备配置和响应（Config、Completion）
   - Implicit：系统事件和通知（Message）

2. ✅ **路由配置在枚举时确定**：
   - BIOS/UEFI 或 Linux 在启动时分配地址窗口和总线号
   - 配置后通常不再改变（除非热插拔）

3. ✅ **理解路由对性能至关重要**：
   - P2P 路由可大幅提升设备间通信效率
   - 地址窗口配置影响路由决策速度
   - 正确的拓扑设计减少路由跳数

4. ✅ **调试工具**：
   - `lspci`：查看配置和拓扑
   - `setpci`：读写寄存器
   - `dmesg`：内核日志
   - PCIe 协议分析仪：硬件级调试

### 路由决策总结表

| TLP 类型 | 路由方式 | 关键字段 | 转发规则 |
|---------|---------|---------|---------|
| Memory Read/Write | Address-Based | Address[63:0] | 匹配地址窗口 |
| Config Read/Write | ID-Based | Bus:Dev.Func | 匹配总线号范围 |
| Completion | ID-Based | Requester ID | 反向路由到请求者 |
| INTx Message | Implicit | Message Code | 向上游转发 |
| Error Message | Implicit | Message Code | 向上游转发 |
| PME Message | Implicit | Message Code | 向上游转发 |

### 常见问题排查

**问题 1：设备无法访问**

```
症状：Memory 读写超时或返回全 0xFF
排查：
1. 检查 BAR 是否正确配置：lspci -vvv -s <BDF>
2. 检查 Bridge 地址窗口是否包含 BAR：lspci -vvv -s <Bridge>
3. 检查设备是否启用：lspci -vvv | grep "Memory.*disabled"
```

**问题 2：配置空间访问失败**

```
症状：读取配置空间返回 0xFFFFFFFF
排查：
1. 检查总线号分配：lspci -tv
2. 检查 Bridge 的 Secondary/Subordinate Bus Number
3. 检查设备是否在线：ls /sys/bus/pci/devices/
```

**问题 3：P2P 传输失败**

```
症状：设备间 DMA 无法工作
排查：
1. 检查 ACS 是否阻止 P2P：lspci -vvv | grep ACS
2. 检查内核是否启用 P2P：cat /sys/module/pci_p2pdma/parameters/*
3. 检查拓扑：设备是否在同一 Switch 下
```

---

## 下一步学习

- [TLP 格式详解](tlp-format.md) - 了解 TLP Header 中的路由字段
- [配置空间](../configuration/config-space.md) - Bridge 路由寄存器详解
- [虚拟通道](virtual-channels.md) - 避免路由死锁的机制
- [错误处理](../error-handling/aer.md) - 错误消息的路由路径
- [排序规则](ordering.md) - 路由与事务排序的关系

---

## 参考资料

### 规范文档

- **PCIe Base Spec Chapter 2.2.6 - 2.2.8**：Routing Mechanisms
- **PCIe Base Spec Figure 2-17**：Address-Based Routing Example
- **PCIe Base Spec Figure 2-18**：ID-Based Routing Example
- **PCIe Base Spec Table 2-3**：TLP Types and Routing

### 代码实现

- **FEMU**：`/hw/pci/pci.c`, `/hw/pci/pci_bridge.c`
- **Linux Kernel**：`drivers/pci/probe.c`, `drivers/pci/setup-bus.c`
- **QEMU**：`hw/pci/pci_host.c`

### 工具和调试

- **lspci**：PCI utilities package
- **setpci**：Direct PCI configuration access
- **PCIe Protocol Analyzers**：Teledyne LeCroy, Keysight

---

**相关页面**：
- [← TLP 格式](tlp-format.md)
- [排序规则 →](ordering.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
