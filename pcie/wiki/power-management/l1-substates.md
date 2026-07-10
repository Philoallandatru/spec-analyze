# L1 子状态详解

## 概述

L1 子状态是 PCIe 3.0+ 引入的更精细的电源管理机制，在 L1 状态的基础上进一步降低功耗。包括 L1.1（PLL 关闭）和 L1.2（PLL 和时钟关闭）两个子状态。

## L1 子状态架构

```
L1 状态的细分：

L1.0 (传统 L1)
├── PLL: 运行
├── 参考时钟: 运行
├── 功耗: ~10-20%
└── 恢复时间: < 4 μs

L1.1 (L1 with PLL Off)
├── PLL: 关闭
├── 参考时钟: 运行
├── 功耗: ~5-10%
└── 恢复时间: < 16 μs

L1.2 (L1 with PLL and Clock Off)
├── PLL: 关闭
├── 参考时钟: 关闭 (需要 CLKREQ#)
├── 功耗: ~1-5%
└── 恢复时间: < 100 μs
```

## 功耗和恢复时间对比

| 状态 | PLL | 时钟 | 功耗 | 恢复时间 | 典型应用 |
|------|-----|------|------|---------|---------|
| L0 | On | On | 100% | - | 活跃传输 |
| L0s | On | On | 60-80% | < 4 μs | 短暂空闲 |
| L1.0 | On | On | 10-20% | < 4 μs | 传统省电 |
| L1.1 | Off | On | 5-10% | < 16 μs | 中等省电 |
| L1.2 | Off | Off | 1-5% | < 100 μs | 深度省电 |

## L1 PM Substates Capability

### Extended Capability 结构

```
L1 PM Substates Extended Capability:
Capability ID = 0x001E
Offset 0x100+

+0x00: [15:0]  Extended Capability ID = 0x001E
       [19:16] Capability Version = 0x1
       [31:20] Next Capability Offset

+0x04: [31:0]  L1 PM Substates Capabilities
       [0]     PCI-PM L1.2 Supported
       [1]     PCI-PM L1.1 Supported
       [2]     ASPM L1.2 Supported
       [3]     ASPM L1.1 Supported
       [4]     L1 PM Substates Supported
       [7:5]   Reserved
       [15:8]  Port Common_Mode_Restore_Time
       [17:16] Port T_POWER_ON Scale
       [23:19] Port T_POWER_ON Value

+0x08: [31:0]  L1 PM Substates Control 1
       [0]     PCI-PM L1.2 Enable
       [1]     PCI-PM L1.1 Enable
       [2]     ASPM L1.2 Enable
       [3]     ASPM L1.1 Enable
       [7:4]   Reserved
       [15:8]  Common_Mode_Restore_Time
       [17:16] LTR L1.2 Threshold Scale
       [31:18] LTR L1.2 Threshold Value

+0x0C: [31:0]  L1 PM Substates Control 2
       [7:0]   T_POWER_ON Scale
       [12:8]  T_POWER_ON Value
```

## 配置和使用

### 查询和启用 L1 子状态

```c
// 查询 L1 子状态支持
static int query_l1_substates(struct pci_dev *pdev)
{
    int pos;
    u32 cap;
    
    // 查找 L1 PM Substates Capability
    pos = pci_find_ext_capability(pdev, PCI_EXT_CAP_ID_L1SS);
    if (!pos) {
        dev_info(&pdev->dev, "L1 PM Substates not supported\n");
        return -ENOTSUPP;
    }
    
    // 读取 Capabilities
    pci_read_config_dword(pdev, pos + PCI_L1SS_CAP, &cap);
    
    dev_info(&pdev->dev, "L1 PM Substates:\n");
    if (cap & PCI_L1SS_CAP_ASPM_L1_2)
        dev_info(&pdev->dev, "  - ASPM L1.2 supported\n");
    if (cap & PCI_L1SS_CAP_ASPM_L1_1)
        dev_info(&pdev->dev, "  - ASPM L1.1 supported\n");
    if (cap & PCI_L1SS_CAP_PCIPM_L1_2)
        dev_info(&pdev->dev, "  - PCI-PM L1.2 supported\n");
    if (cap & PCI_L1SS_CAP_PCIPM_L1_1)
        dev_info(&pdev->dev, "  - PCI-PM L1.1 supported\n");
    
    return 0;
}

// 启用 L1 子状态
static int enable_l1_substates(struct pci_dev *pdev)
{
    int pos;
    u32 cap, ctl1, ctl2;
    
    pos = pdev->l1ss_cap;
    if (!pos)
        return -EINVAL;
    
    // 读取能力
    pci_read_config_dword(pdev, pos + PCI_L1SS_CAP, &cap);
    
    // 配置 Control 1
    ctl1 = 0;
    
    if (cap & PCI_L1SS_CAP_ASPM_L1_2) {
        // 启用 ASPM L1.2
        ctl1 |= PCI_L1SS_CTL1_ASPM_L1_2;
        
        // 设置 LTR L1.2 阈值
        ctl1 |= (100 << 16) | (3 << 29);  // 100 μs, scale = 1 μs
    }
    
    if (cap & PCI_L1SS_CAP_ASPM_L1_1) {
        // 启用 ASPM L1.1
        ctl1 |= PCI_L1SS_CTL1_ASPM_L1_1;
    }
    
    if (cap & PCI_L1SS_CAP_PCIPM_L1_2) {
        ctl1 |= PCI_L1SS_CTL1_PCIPM_L1_2;
    }
    
    if (cap & PCI_L1SS_CAP_PCIPM_L1_1) {
        ctl1 |= PCI_L1SS_CTL1_PCIPM_L1_1;
    }
    
    // 设置 Common Mode Restore Time
    ctl1 |= ((cap >> 8) & 0xFF);
    
    pci_write_config_dword(pdev, pos + PCI_L1SS_CTL1, ctl1);
    
    // 配置 Control 2
    ctl2 = 0;
    
    // 设置 T_POWER_ON
    u8 t_power_on_scale = (cap >> 16) & 0x3;
    u8 t_power_on_value = (cap >> 19) & 0x1F;
    
    ctl2 = t_power_on_value | (t_power_on_scale << 8);
    
    pci_write_config_dword(pdev, pos + PCI_L1SS_CTL2, ctl2);
    
    dev_info(&pdev->dev, "L1 PM Substates enabled\n");
    return 0;
}
```

## LTR 集成

### Latency Tolerance Reporting

```
L1.2 需要 LTR 支持：

LTR Capability:
- Max Snoop Latency
- Max No-Snoop Latency

设备通过 LTR 消息报告延迟容忍度
系统根据 LTR 值决定是否进入 L1.2

LTR 阈值配置：
if (device_ltr > threshold) {
    // 不进入 L1.2
} else {
    // 可以进入 L1.2
}
```

### 配置 LTR

```c
// 配置 LTR
static int configure_ltr(struct pci_dev *pdev, u32 snoop_lat, u32 nosnoop_lat)
{
    int pos;
    u16 ctl;
    
    // 查找 LTR Capability
    pos = pci_find_ext_capability(pdev, PCI_EXT_CAP_ID_LTR);
    if (!pos)
        return -ENOTSUPP;
    
    // 设置 Snoop Latency
    pci_write_config_dword(pdev, pos + PCI_LTR_MAX_SNOOP_LAT, snoop_lat);
    
    // 设置 No-Snoop Latency
    pci_write_config_dword(pdev, pos + PCI_LTR_MAX_NOSNOOP_LAT, nosnoop_lat);
    
    // 启用 LTR
    pos = pci_pcie_cap(pdev);
    pci_read_config_word(pdev, pos + PCI_EXP_DEVCTL2, &ctl);
    ctl |= PCI_EXP_DEVCTL2_LTR_EN;
    pci_write_config_word(pdev, pos + PCI_EXP_DEVCTL2, ctl);
    
    return 0;
}
```

## T_POWER_ON 参数

```
T_POWER_ON: 从 L1.2 恢复到 L0 所需的时间

计算公式：
T_POWER_ON = Value × 10^Scale μs

Scale:
00 = 2 μs
01 = 10 μs
10 = 100 μs

示例：
Value = 50, Scale = 10
T_POWER_ON = 50 × 100 = 5000 μs = 5 ms

配置时需要：
1. 读取设备的 T_POWER_ON 能力
2. 设置到 Control 2 寄存器
3. 确保系统可以容忍该恢复时间
```

## CLKREQ# 支持

```
L1.2 需要 CLKREQ# 信号：

设备通过 CLKREQ# 请求参考时钟：
- CLKREQ# = Low: 需要时钟
- CLKREQ# = High: 不需要时钟

L1.2 进入流程：
1. 链路空闲
2. 进入 L1
3. 关闭 PLL
4. CLKREQ# 置高
5. 系统关闭参考时钟
6. 进入 L1.2

L1.2 退出流程：
1. 需要唤醒
2. CLKREQ# 置低
3. 系统提供参考时钟
4. PLL 锁定
5. 链路训练
6. 返回 L0
```

## 调试和优化

### 查看 L1 子状态

```bash
# lspci 显示 L1 子状态配置
lspci -vvv -s 01:00.0 | grep -A 5 "L1 PM Substates"

# 输出示例：
# Capabilities: [150] L1 PM Substates
#   L1SubCap: PCI-PM_L1.2+ PCI-PM_L1.1+ ASPM_L1.2+ ASPM_L1.1+ L1_PM_Substates+
#   L1SubCtl1: PCI-PM_L1.2+ PCI-PM_L1.1+ ASPM_L1.2+ ASPM_L1.1+
#   T_CommonMode=0us LTR_L1.2_Threshold=100us
#   L1SubCtl2: T_PwrOn=50us
```

### 功耗测量

```bash
# 使用 powertop 监控
powertop --auto-tune

# 测量功耗差异
# L1.0 vs L1.1 vs L1.2
```

### 性能影响

```c
// 测量 L1.2 恢复延迟
static void measure_l1_2_latency(struct pci_dev *pdev)
{
    u64 start, end, latency;
    
    // 触发进入 L1.2
    enter_l1_2(pdev);
    
    // 等待稳定
    msleep(100);
    
    // 测量恢复时间
    start = ktime_get_ns();
    
    // 触发唤醒
    trigger_wakeup(pdev);
    
    // 等待链路恢复
    wait_for_link_active(pdev);
    
    end = ktime_get_ns();
    latency = (end - start) / 1000;  // 转换为 μs
    
    dev_info(&pdev->dev, "L1.2 recovery latency: %llu μs\n", latency);
}
```

## 最佳实践

1. **评估延迟需求**：确保应用可以容忍 L1.2 的恢复时间
2. **配置 LTR**：合理设置延迟容忍度
3. **测试功耗**：实际测量功耗节省效果
4. **兼容性验证**：某些设备可能有 L1.2 问题
5. **性能权衡**：在功耗和性能之间找平衡

## 总结

L1 子状态的关键点：

1. **L1.1**：关闭 PLL，保持时钟，中等功耗节省
2. **L1.2**：关闭 PLL 和时钟，最大功耗节省
3. **LTR 集成**：通过延迟容忍度控制进入条件
4. **CLKREQ#**：L1.2 需要动态时钟请求支持
5. **恢复时间**：L1.2 恢复时间较长，需要权衡

参见：
- [电源状态](power-states.md)
- [ASPM](aspm.md)
- [CLKREQ](clkreq.md)
- [设备电源状态](device-states.md)
