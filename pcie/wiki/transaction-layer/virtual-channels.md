# PCIe 虚拟通道

## 概述

虚拟通道 (Virtual Channel, VC) 是 PCIe 提供的流量分类和 QoS (Quality of Service) 机制。通过虚拟通道，不同类型的流量可以在同一物理链路上获得不同的服务质量保证。

## 虚拟通道架构

### 基本概念

```
物理链路（单一）
    |
    +-- 虚拟通道 VC0 (必需，默认)
    |   - 所有流量的后备通道
    |   - 最低优先级
    |
    +-- 虚拟通道 VC1 (可选)
    |   - 高优先级流量
    |
    +-- 虚拟通道 VC2 (可选)
    |   - 中等优先级流量
    |
    +-- ...
    |
    +-- 虚拟通道 VC7 (可选)
        - 最高优先级流量

每个虚拟通道：
- 独立的流控制信用
- 独立的仲裁
- 独立的 QoS 设置
```

### 流量类别映射

```
Traffic Class (TC) --> Virtual Channel (VC)

TC0 --> VC0 (默认映射)
TC1 --> VC1
TC2 --> VC1
TC3 --> VC2
TC4 --> VC3
TC5 --> VC4
TC6 --> VC5
TC7 --> VC7

映射规则：
- TC 在 TLP Header 中指定（3 位）
- 一个 VC 可以承载多个 TC
- 一个 TC 只能映射到一个 VC
- VC0 必须支持 TC0
```

## TLP Header 中的 TC 字段

### TLP Header 格式（TC 字段）

```
3DW/4DW Header:
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|31|30|29|28|27|26|25|24|23|22|21|20|19|18|17|16|
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|  Fmt  |  Type     | R |TC[2:0]| R |  Attr ...
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

TC[2:0]: Traffic Class
- 000: TC0 (Default)
- 001: TC1
- 010: TC2
- 011: TC3
- 100: TC4
- 101: TC5
- 110: TC6
- 111: TC7
```

### 设置 TC

```c
// 构造 TLP 时指定 TC
struct tlp_header {
    uint8_t fmt_type;
    uint8_t tc : 3;      // Traffic Class
    uint8_t attr : 2;
    // ... 其他字段
};

// 示例：发送 TC2 的 Memory Write
void send_memory_write_tc2(uint64_t addr, uint32_t data)
{
    struct tlp_header hdr = {
        .fmt_type = 0x40,  // MWr32
        .tc = 2,           // TC2
        .attr = 0,
        // ... 填充其他字段
    };
    
    send_tlp(&hdr, &data, sizeof(data));
}
```

## 虚拟通道 Capability

### VC Capability 结构

```
VC Extended Capability (Offset 0x100+):
+0x00: [15:0]  Extended Capability ID = 0x0002
       [19:16] Capability Version = 0x1
       [31:20] Next Capability Offset

+0x04: [31:0]  Port VC Capability 1
       [2:0]   Extended VC Count (支持的 VC 数量 - 1)
       [3]     Reserved
       [6:4]   Low Priority Extended VC Count
       [7]     Reserved
       [11:8]  Reference Clock
       [13:12] Port Arbitration Table Entry Size
       
+0x08: [31:0]  Port VC Capability 2
       [7:0]   VC Arbitration Capability
       [31:8]  VC Arbitration Table Offset

+0x0C: [31:0]  Port VC Control
       [3:1]   VC Arbitration Select
       
+0x10: [31:0]  Port VC Status
       [0]     VC Arbitration Table Status

+0x14: [31:0]  VC Resource Capability (VC0)
       [7:0]   Port Arbitration Capability
       [14]    Advanced Packet Switching
       [15]    Reject Snoop Transactions
       [23:16] Maximum Time Slots
       [31:24] Port Arbitration Table Offset

+0x18: [31:0]  VC Resource Control (VC0)
       [7:0]   TC/VC Map
       [16]    Load VC Arbitration Table
       [17]    VC Enable
       [23:20] Port Arbitration Select

+0x1C: [31:0]  VC Resource Status (VC0)
       [0]     VC Negotiation Pending
       [1]     VC Arbitration Table Status

重复 VC Resource 寄存器组（每个 VC 12 字节）
+0x20-0x2B: VC1
+0x2C-0x37: VC2
...
```

### 软件配置

```c
// 配置虚拟通道
static int configure_virtual_channels(struct pci_dev *pdev)
{
    int vc_cap, vc_count, i;
    u32 cap1, cap2, ctrl;
    
    // 查找 VC Capability
    vc_cap = pci_find_ext_capability(pdev, PCI_EXT_CAP_ID_VC);
    if (!vc_cap) {
        dev_info(&pdev->dev, "No VC capability\n");
        return 0;
    }
    
    // 读取 VC 数量
    pci_read_config_dword(pdev, vc_cap + PCI_VC_PORT_CAP1, &cap1);
    vc_count = (cap1 & PCI_VC_CAP1_EVCC) + 1;
    
    dev_info(&pdev->dev, "Found %d virtual channels\n", vc_count);
    
    // 配置每个 VC
    for (i = 0; i < vc_count; i++) {
        int vc_offset = vc_cap + PCI_VC_RES_CAP + i * PCI_VC_RES_SIZE;
        
        // 读取 VC Resource Control
        pci_read_config_dword(pdev, vc_offset + PCI_VC_RES_CTRL, &ctrl);
        
        // 设置 TC/VC 映射
        if (i == 0) {
            // VC0: 映射 TC0
            ctrl = (ctrl & ~0xFF) | 0x01;
        } else if (i == 1) {
            // VC1: 映射 TC1, TC2
            ctrl = (ctrl & ~0xFF) | 0x06;  // Bit 1 | Bit 2
        } else if (i == 2) {
            // VC2: 映射 TC3
            ctrl = (ctrl & ~0xFF) | 0x08;  // Bit 3
        }
        
        // 启用 VC
        ctrl |= PCI_VC_RES_CTRL_ENABLE;
        
        pci_write_config_dword(pdev, vc_offset + PCI_VC_RES_CTRL, ctrl);
        
        dev_info(&pdev->dev, "VC%d configured, TC map: 0x%02x\n",
                 i, ctrl & 0xFF);
    }
    
    return 0;
}
```

## 流控制与虚拟通道

### 独立流控制信用

```
每个虚拟通道维护独立的流控制信用：

VC0:
- Posted Header Credits: 8
- Posted Data Credits: 64
- Non-Posted Header Credits: 8
- Non-Posted Data Credits: 64
- Completion Header Credits: 8
- Completion Data Credits: 64

VC1:
- Posted Header Credits: 4
- Posted Data Credits: 32
- ...

流控制 DLLP 包含 VC ID：
+----------------+
| DLLP Type      |
+----------------+
| VC ID[2:0]     |  指定虚拟通道
+----------------+
| Header Credits |
+----------------+
| Data Credits   |
+----------------+
```

### 流控制初始化

```c
// 虚拟通道流控制初始化
struct vc_flow_control {
    uint8_t vc_id;
    struct {
        uint8_t  hdr_credits;
        uint16_t data_credits;
    } posted, non_posted, completion;
};

void init_vc_flow_control(struct pcie_port *port)
{
    struct vc_flow_control vc_fc[8];
    
    // VC0 (默认，大信用)
    vc_fc[0].vc_id = 0;
    vc_fc[0].posted.hdr_credits = 8;
    vc_fc[0].posted.data_credits = 64;
    vc_fc[0].non_posted.hdr_credits = 8;
    vc_fc[0].non_posted.data_credits = 64;
    vc_fc[0].completion.hdr_credits = 8;
    vc_fc[0].completion.data_credits = 64;
    
    // VC1 (高优先级，中等信用)
    vc_fc[1].vc_id = 1;
    vc_fc[1].posted.hdr_credits = 4;
    vc_fc[1].posted.data_credits = 32;
    // ...
    
    // 发送 InitFC DLLP
    for (int i = 0; i < num_vcs; i++) {
        send_init_fc_dllp(&vc_fc[i]);
    }
}
```

## 虚拟通道仲裁

### 仲裁机制

**1. 固定优先级 (Fixed Priority)**：
```
VC7 > VC6 > VC5 > ... > VC1 > VC0

高优先级的 VC 总是先于低优先级被服务
```

**2. 加权轮询 (Weighted Round Robin, WRR)**：
```
VC0: Weight = 1  (分配 1 个时间片)
VC1: Weight = 2  (分配 2 个时间片)
VC2: Weight = 4  (分配 4 个时间片)

仲裁顺序：VC2, VC2, VC2, VC2, VC1, VC1, VC0
```

**3. 时间片仲裁 (Time-based WRR)**：
```
VC0: 10% 带宽
VC1: 30% 带宽
VC2: 60% 带宽

每个 VC 获得配置的带宽比例
```

### 仲裁配置

```c
// 配置仲裁
static int configure_vc_arbitration(struct pci_dev *pdev, int vc_cap)
{
    u32 cap2, ctrl;
    u8 arb_cap;
    
    // 读取仲裁能力
    pci_read_config_dword(pdev, vc_cap + PCI_VC_PORT_CAP2, &cap2);
    arb_cap = cap2 & 0xFF;
    
    dev_info(&pdev->dev, "VC Arbitration capability: 0x%02x\n", arb_cap);
    
    // 选择仲裁方式
    pci_read_config_dword(pdev, vc_cap + PCI_VC_PORT_CTRL, &ctrl);
    
    if (arb_cap & PCI_VC_ARB_CAP_WRR_128) {
        // 使用 128-phase WRR
        ctrl = (ctrl & ~PCI_VC_PORT_CTRL_ARB_SELECT) | (4 << 1);
        dev_info(&pdev->dev, "Using WRR-128 arbitration\n");
    } else if (arb_cap & PCI_VC_ARB_CAP_WRR_64) {
        // 使用 64-phase WRR
        ctrl = (ctrl & ~PCI_VC_PORT_CTRL_ARB_SELECT) | (3 << 1);
        dev_info(&pdev->dev, "Using WRR-64 arbitration\n");
    } else {
        // 使用固定优先级
        ctrl = (ctrl & ~PCI_VC_PORT_CTRL_ARB_SELECT) | (1 << 1);
        dev_info(&pdev->dev, "Using fixed priority arbitration\n");
    }
    
    pci_write_config_dword(pdev, vc_cap + PCI_VC_PORT_CTRL, ctrl);
    
    return 0;
}
```

## 应用场景

### 场景 1：存储网络融合

```
+------------------+
| 服务器            |
+------------------+
        |
    PCIe 链路
        |
        +-- VC0 (TC0): 控制流量
        |   - 配置访问
        |   - 管理流量
        |
        +-- VC1 (TC1-TC3): 网络流量
        |   - TCP/IP 数据
        |   - 以太网帧
        |
        +-- VC2 (TC4-TC7): 存储流量
            - NVMe 命令
            - 块数据传输
```

### 场景 2：实时系统

```
+------------------+
| 工业控制系统      |
+------------------+
        |
    PCIe 链路
        |
        +-- VC0 (TC0): 后台流量
        |   - 日志记录
        |   - 配置更新
        |
        +-- VC1 (TC1-TC2): 控制流量
        |   - 传感器数据
        |   - 执行器命令
        |   - 延迟 < 1ms
        |
        +-- VC2 (TC7): 安全关键流量
            - 紧急停止
            - 安全互锁
            - 延迟 < 100us
```

### 场景 3：多租户虚拟化

```
+------------------+
| 虚拟化主机        |
+------------------+
        |
    PCIe 链路 (SR-IOV)
        |
        +-- VC0: Hypervisor 流量
        |
        +-- VC1: 租户 A (VF 0-7)
        |
        +-- VC2: 租户 B (VF 8-15)
        |
        +-- VC3: 租户 C (VF 16-23)
```

## FEMU 实现

### VC Capability 添加

```c
// hw/pci/pcie.c
int pcie_add_vc_capability(PCIDevice *dev, uint8_t num_vcs)
{
    uint8_t *config = dev->config;
    int offset;
    
    offset = pcie_add_capability(dev, PCI_EXT_CAP_ID_VC,
                                  PCI_VC_VER, PCI_VC_SIZEOF);
    if (offset < 0) {
        return offset;
    }
    
    // Port VC Capability 1
    pci_set_long(config + offset + PCI_VC_PORT_CAP1,
                 ((num_vcs - 1) & 0x7));  // Extended VC Count
    
    // Port VC Capability 2
    pci_set_long(config + offset + PCI_VC_PORT_CAP2,
                 PCI_VC_ARB_CAP_HW_FIXED |  // 支持固定优先级
                 PCI_VC_ARB_CAP_WRR_64);    // 支持 WRR-64
    
    // VC0 Resource Capability
    pci_set_long(config + offset + PCI_VC_RES_CAP,
                 PCI_VC_RES_CAP_ARB_HW_FIXED);
    
    // VC0 Resource Control (默认映射 TC0)
    pci_set_long(config + offset + PCI_VC_RES_CTRL,
                 0x01 |                      // TC0/VC0 map
                 PCI_VC_RES_CTRL_ENABLE);    // VC Enable
    
    // 添加其他 VC 资源
    for (int i = 1; i < num_vcs; i++) {
        int vc_offset = offset + PCI_VC_RES_CAP + i * PCI_VC_RES_SIZE;
        
        pci_set_long(config + vc_offset,
                     PCI_VC_RES_CAP_ARB_HW_FIXED);
        
        // 默认禁用
        pci_set_long(config + vc_offset + 4, 0x00);
    }
    
    return offset;
}
```

### VC 感知的流控制

```c
// 每个 VC 独立的流控制
struct vc_flow_control {
    uint8_t vc_id;
    uint8_t p_hdr_fc;
    uint16_t p_data_fc;
    uint8_t np_hdr_fc;
    uint16_t np_data_fc;
    uint8_t cpl_hdr_fc;
    uint16_t cpl_data_fc;
};

// 检查流控制信用
bool check_vc_credit(struct pcie_port *port, uint8_t vc_id,
                     enum tlp_type type, uint16_t length)
{
    struct vc_flow_control *fc = &port->vc_fc[vc_id];
    
    switch (type) {
    case TLP_TYPE_POSTED:
        if (fc->p_hdr_fc == 0)
            return false;
        if (length > 0 && fc->p_data_fc < DWORDS_TO_CREDITS(length))
            return false;
        break;
        
    case TLP_TYPE_NON_POSTED:
        if (fc->np_hdr_fc == 0)
            return false;
        if (length > 0 && fc->np_data_fc < DWORDS_TO_CREDITS(length))
            return false;
        break;
        
    case TLP_TYPE_COMPLETION:
        if (fc->cpl_hdr_fc == 0)
            return false;
        if (length > 0 && fc->cpl_data_fc < DWORDS_TO_CREDITS(length))
            return false;
        break;
    }
    
    return true;
}
```

## 调试和监控

### 查看 VC 配置

```bash
# lspci 显示 VC 信息
lspci -vvv -s 01:00.0 | grep -A 20 "Virtual Channel"

# sysfs 查看
cat /sys/bus/pci/devices/0000:01:00.0/vc_arb_select
```

### 性能监控

```c
// 监控每个 VC 的流量
struct vc_stats {
    uint64_t tlp_count;
    uint64_t byte_count;
    uint64_t stall_cycles;
};

void print_vc_stats(struct vc_stats *stats, int num_vcs)
{
    for (int i = 0; i < num_vcs; i++) {
        printf("VC%d: %lu TLPs, %lu bytes, %lu stalls\n",
               i, stats[i].tlp_count, stats[i].byte_count,
               stats[i].stall_cycles);
    }
}
```

## 最佳实践

1. **默认使用 VC0**：大多数系统只需要 VC0
2. **明确需求**：启用多 VC 前确认有 QoS 需求
3. **合理分配**：避免过度分割带宽
4. **测试验证**：确保延迟敏感流量获得保证
5. **监控性能**：跟踪各 VC 的利用率和延迟

## 总结

虚拟通道的关键点：

1. **流量分类**：通过 TC 字段指定流量类别
2. **独立资源**：每个 VC 有独立的流控制信用
3. **QoS 保证**：通过仲裁机制提供带宽和延迟保证
4. **灵活映射**：TC 到 VC 的映射可配置
5. **向后兼容**：VC0 是必需的，支持所有流量

参见：
- [流控制机制](flow-control.md)
- [TLP 格式](tlp-format.md)
- [排序规则](ordering.md)
