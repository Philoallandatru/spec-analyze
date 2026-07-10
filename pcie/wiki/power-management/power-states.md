# PCIe 电源状态

## 概述

PCIe 定义了多种电源状态，允许系统和设备在不同的功耗级别之间切换，以平衡性能和能耗。本文详细介绍 PCIe 的各种电源状态及其转换机制。

## 电源状态层次

### 链路电源状态 (Link Power States)

```
L0 (Active)         全速运行
    ↓
L0s (Standby)       快速唤醒，部分省电
    ↓
L1 (Low Power)      深度睡眠，更多省电
    ↓
L2 (Auxiliary)      主电源关闭
    ↓
L3 (Off)            完全关闭
```

### 状态对比

| 状态 | 功耗 | 恢复时间 | CLKREQ# | 数据传输 | 配置空间访问 |
|------|------|---------|---------|---------|-------------|
| L0   | 100% | -       | Active  | ✅ 是   | ✅ 是       |
| L0s  | 60-80% | < 4 μs | Active  | ❌ 否   | ✅ 是       |
| L1   | 10-20% | < 10 μs | Inactive | ❌ 否  | ⚠️ 有限    |
| L2   | < 1% | > 50 ms | Off     | ❌ 否   | ❌ 否       |
| L3   | 0%   | > 100 ms | Off    | ❌ 否   | ❌ 否       |

## L0 - Active State

### 特性

```
L0 是正常工作状态：
- 全速数据传输
- LTSSM 处于 L0 状态
- 时钟活跃
- 所有功能可用
- 最高功耗

电源特性：
- Vcc: 全功率
- PHY: 全速运行
- PLL: 锁定
- 时钟: 持续运行
```

## L0s - Standby State

### 进入和退出

```
进入 L0s:
1. 链路空闲（无 TLP 传输）
2. ASPM L0s 启用
3. 发送 EIOS (Electrical Idle Ordered Set)
4. 进入电气空闲
5. 关闭发送器

退出 L0s:
1. 检测到需要传输数据
2. 发送 FTS (Fast Training Sequence)
3. 恢复位锁定和符号锁定
4. 进入 L0
5. 开始传输

时序：
L0 --> 空闲检测 --> EIOS --> 电气空闲 --> L0s
       |<-- 退出延迟 (< 4 μs) --|
L0s --> 唤醒 --> FTS --> 位锁定 --> L0
```

### 配置

```c
// 配置 L0s
static void configure_l0s(struct pci_dev *pdev, bool enable)
{
    u16 link_ctrl;
    
    // 读取 Link Control 寄存器
    pcie_capability_read_word(pdev, PCI_EXP_LNKCTL, &link_ctrl);
    
    if (enable) {
        // 启用 L0s
        link_ctrl |= PCI_EXP_LNKCTL_ASPM_L0S;
    } else {
        // 禁用 L0s
        link_ctrl &= ~PCI_EXP_LNKCTL_ASPM_L0S;
    }
    
    pcie_capability_write_word(pdev, PCI_EXP_LNKCTL, link_ctrl);
    
    dev_info(&pdev->dev, "L0s %s\n", enable ? "enabled" : "disabled");
}
```

### FTS (Fast Training Sequence)

```
FTS 数量配置：
- N_FTS: 链路训练时协商的 FTS 数量
- 典型值: 32-128
- 影响 L0s 退出延迟

LTSSM 中的 FTS:
Recovery.RcvrLock 状态
    ↓
接收 N_FTS 个 FTS
    ↓
恢复位锁定
    ↓
进入 Recovery.RcvrCfg

查看 N_FTS:
# lspci -vvv | grep "N_FTS"
```

## L1 - Low Power State

### 进入条件

```
进入 L1 需要满足：
1. ASPM L1 启用
2. 链路空闲
3. 数据链路层所有重试缓冲区为空
4. 所有待处理的 TLP 已确认
5. 流控制信用已更新

进入流程：
L0 --> 空闲 --> 检查条件 --> 发送 PM_Enter_L1 DLLP
                              ↓
                         对端确认 PM_Enter_L1
                              ↓
                         双方进入 L1
```

### DLLP 交换

```
发起端：
1. 发送 PM_Enter_L1 DLLP
2. 继续监听接收端的 PM_Enter_L1
3. 收到后双方进入 L1

PM_Enter_L1 DLLP 格式:
+--------+----------+----------+--------+
| Type   | Reserved | Reserved | CRC-16 |
| 0x20   | 0x00     | 0x0000   | ...    |
+--------+----------+----------+--------+

退出 L1:
1. 任一端检测到唤醒条件
2. 发送 EIEOS (退出电气空闲)
3. 发送 TS1 重新训练
4. 链路进入 Recovery
5. 返回 L0
```

### 配置和实现

```c
// 配置 L1
static int configure_l1(struct pci_dev *pdev, bool enable)
{
    int pos;
    u16 link_ctrl;
    u32 link_cap;
    
    pos = pci_pcie_cap(pdev);
    if (!pos)
        return -EINVAL;
    
    // 检查是否支持 L1
    pci_read_config_dword(pdev, pos + PCI_EXP_LNKCAP, &link_cap);
    if (!(link_cap & PCI_EXP_LNKCAP_ASPM_L1)) {
        dev_warn(&pdev->dev, "L1 not supported\n");
        return -ENOTSUPP;
    }
    
    // 读取 Link Control
    pci_read_config_word(pdev, pos + PCI_EXP_LNKCTL, &link_ctrl);
    
    if (enable) {
        // 启用 L1
        link_ctrl |= PCI_EXP_LNKCTL_ASPM_L1;
        
        // 配置 L1 退出延迟容忍度
        u32 dev_cap2;
        pci_read_config_dword(pdev, pos + PCI_EXP_DEVCAP2, &dev_cap2);
        if (dev_cap2 & PCI_EXP_DEVCAP2_LTR) {
            // 支持 LTR，设置延迟容忍度
            configure_ltr(pdev, 1000);  // 1 ms
        }
    } else {
        // 禁用 L1
        link_ctrl &= ~PCI_EXP_LNKCTL_ASPM_L1;
    }
    
    pci_write_config_word(pdev, pos + PCI_EXP_LNKCTL, link_ctrl);
    
    dev_info(&pdev->dev, "L1 %s\n", enable ? "enabled" : "disabled");
    return 0;
}

// L1 进入检查
static bool can_enter_l1(struct pcie_port *port)
{
    // 检查重试缓冲区
    if (!is_retry_buffer_empty(&port->dll_tx))
        return false;
    
    // 检查待处理的 TLP
    if (has_pending_tlp(port))
        return false;
    
    // 检查流控制信用
    if (fc_credits_in_flight(port))
        return false;
    
    return true;
}

// 发送 PM_Enter_L1
static void send_pm_enter_l1(struct pcie_port *port)
{
    struct dllp pm_dllp = {
        .type = DLLP_TYPE_PM_ENTER_L1,
        .reserved = 0,
        .data = 0,
    };
    
    // 计算 CRC
    pm_dllp.crc = calculate_dllp_crc(&pm_dllp);
    
    // 发送 DLLP
    send_dllp(port, &pm_dllp);
    
    // 等待对端的 PM_Enter_L1
    wait_for_pm_enter_l1(port);
    
    // 进入 L1
    ltssm_set_state(port, LTSSM_L1_IDLE);
}
```

## L1 子状态

### L1.1 和 L1.2

```
L1 子状态 (PCIe 3.0+):

L1.1 (L1 with PLL Off):
- 关闭 PLL
- 时钟停止
- 更低功耗
- 恢复时间稍长

L1.2 (L1 with PLL and Clock Off):
- 关闭 PLL 和参考时钟
- 最低功耗的活跃状态
- 恢复时间最长
- 需要 CLKREQ# 支持

功耗对比：
L1.0: ~10-20% (PLL on, Clock on)
L1.1: ~5-10% (PLL off, Clock on)
L1.2: ~1-5% (PLL off, Clock off)
```

### L1 子状态配置

```c
// 配置 L1 子状态
static int configure_l1_substates(struct pci_dev *pdev)
{
    int pos;
    u32 cap, ctl;
    
    // 查找 L1 PM Substates Extended Capability
    pos = pci_find_ext_capability(pdev, PCI_EXT_CAP_ID_L1SS);
    if (!pos) {
        dev_info(&pdev->dev, "L1 substates not supported\n");
        return -ENOTSUPP;
    }
    
    // 读取 Capabilities
    pci_read_config_dword(pdev, pos + PCI_L1SS_CAP, &cap);
    
    // 配置 Control
    ctl = 0;
    
    if (cap & PCI_L1SS_CAP_ASPM_L1_2) {
        // 启用 L1.2
        ctl |= PCI_L1SS_CTL1_ASPM_L1_2;
        
        // 配置 T_POWER_ON
        ctl |= (50 << 3);  // 50 μs
        
        dev_info(&pdev->dev, "L1.2 enabled\n");
    }
    
    if (cap & PCI_L1SS_CAP_ASPM_L1_1) {
        // 启用 L1.1
        ctl |= PCI_L1SS_CTL1_ASPM_L1_1;
        
        dev_info(&pdev->dev, "L1.1 enabled\n");
    }
    
    if (cap & PCI_L1SS_CAP_PCIPM_L1_2) {
        // 启用 PCI-PM L1.2
        ctl |= PCI_L1SS_CTL1_PCIPM_L1_2;
    }
    
    if (cap & PCI_L1SS_CAP_PCIPM_L1_1) {
        // 启用 PCI-PM L1.1
        ctl |= PCI_L1SS_CTL1_PCIPM_L1_1;
    }
    
    // 配置 LTR Threshold
    u32 ctl2 = (100 << 16) | (3 << 29);  // 100 μs, scale = 1 μs
    pci_write_config_dword(pdev, pos + PCI_L1SS_CTL2, ctl2);
    
    pci_write_config_dword(pdev, pos + PCI_L1SS_CTL1, ctl);
    
    return 0;
}
```

## L2 - Auxiliary Power State

### 特性

```
L2 状态：
- 主电源 (Vcc) 关闭
- 辅助电源 (Vaux) 维持
- 配置空间保持
- 需要完整链路训练才能恢复
- 用于系统睡眠（S3）

进入 L2:
1. 软件通过 ACPI 或 PM 寄存器请求
2. 发送 PM_Turn_Off Message
3. 对端响应 PM_Ack
4. 进入 L2
5. 关闭主电源

退出 L2:
1. 恢复主电源
2. 完整的链路训练
3. 恢复到 L0
```

### 进入 L2

```c
// 请求进入 L2
static int enter_l2(struct pci_dev *pdev)
{
    struct pci_bus *bus = pdev->bus;
    struct pci_dev *bridge;
    int pos;
    u32 pmcsr;
    
    // 找到上游 Bridge
    if (!bus->self) {
        dev_err(&pdev->dev, "No upstream bridge\n");
        return -EINVAL;
    }
    
    bridge = bus->self;
    
    // 发送 PM_Turn_Off Message
    pos = pci_find_capability(bridge, PCI_CAP_ID_EXP);
    if (!pos)
        return -EINVAL;
    
    // 设置 PME Turn Off
    pci_read_config_dword(bridge, pos + PCI_EXP_RTCTL, &pmcsr);
    pmcsr |= PCI_EXP_RTCTL_PMEIE;
    pci_write_config_dword(bridge, pos + PCI_EXP_RTCTL, pmcsr);
    
    // 等待 PM_Ack (通过 Data Link Layer Active 位检测)
    msleep(10);
    
    // 进入 D3hot
    pci_set_power_state(pdev, PCI_D3hot);
    
    dev_info(&pdev->dev, "Entered L2 state\n");
    return 0;
}

// 从 L2 唤醒
static int wake_from_l2(struct pci_dev *pdev)
{
    // 恢复到 D0
    pci_set_power_state(pdev, PCI_D0);
    
    // 等待链路训练完成
    msleep(100);
    
    // 恢复配置空间
    pci_restore_state(pdev);
    
    dev_info(&pdev->dev, "Woken from L2 state\n");
    return 0;
}
```

## L3 - Off State

```
L3 状态：
- 所有电源关闭（包括 Vaux）
- 配置空间丢失
- 需要完整的枚举和初始化
- 用于系统关机（S4/S5）

L2 vs L3:
┌─────────┬────────┬────────┐
│ 状态    │ L2     │ L3     │
├─────────┼────────┼────────┤
│ Vcc     │ Off    │ Off    │
│ Vaux    │ On     │ Off    │
│ 配置空间 │ 保持   │ 丢失   │
│ 恢复方式 │ 训练   │ 枚举   │
└─────────┴────────┴────────┘
```

## 电源状态转换

### 状态转换图

```
        ┌─────────────────────────┐
        │          L0             │ <──┐
        │     (Active)            │    │
        └──────┬────────┬─────────┘    │
               │        │               │
          EIOS │        │ PM_Enter_L1   │
               │        │               │
               v        v               │
        ┌──────┴────┐ ┌────────────┐   │
        │   L0s     │ │     L1     │   │ FTS/
        │(Standby)  │ │(Low Power) │   │ TS1
        └──────┬────┘ └─────┬──────┘   │
               │            │           │
          FTS  │            │ PM_Turn_  │
               │            │ Off       │
               └────────────┴───────────┘
                            │
                            v
                     ┌──────────────┐
                     │      L2      │
                     │ (Auxiliary)  │
                     └──────┬───────┘
                            │
                      电源关闭
                            │
                            v
                     ┌──────────────┐
                     │      L3      │
                     │    (Off)     │
                     └──────────────┘
```

### 转换延迟

| 转换 | 典型延迟 | 最大延迟 |
|------|---------|---------|
| L0 → L0s | < 1 μs | < 2 μs |
| L0s → L0 | 2-4 μs | < 7 μs |
| L0 → L1 | 5-10 μs | < 20 μs |
| L1 → L0 | 10-20 μs | < 64 μs |
| L0 → L2 | 50-100 ms | - |
| L2 → L0 | 50-100 ms | - |

## 软件接口

### 内核 API

```c
// 设置设备电源状态
int pci_set_power_state(struct pci_dev *dev, pci_power_t state)
{
    u16 pmcsr;
    int pos;
    
    pos = dev->pm_cap;
    if (!pos)
        return -EIO;
    
    // 读取 PM Control/Status 寄存器
    pci_read_config_word(dev, pos + PCI_PM_CTRL, &pmcsr);
    
    // 设置 Power State
    pmcsr &= ~PCI_PM_CTRL_STATE_MASK;
    pmcsr |= state;
    
    pci_write_config_word(dev, pos + PCI_PM_CTRL, pmcsr);
    
    // 等待状态稳定
    if (state == PCI_D3hot || state == PCI_D3cold)
        msleep(10);
    else
        udelay(200);
    
    return 0;
}

// 获取当前电源状态
pci_power_t pci_current_state(struct pci_dev *dev)
{
    u16 pmcsr;
    int pos;
    
    pos = dev->pm_cap;
    if (!pos)
        return PCI_UNKNOWN;
    
    pci_read_config_word(dev, pos + PCI_PM_CTRL, &pmcsr);
    
    return (pmcsr & PCI_PM_CTRL_STATE_MASK);
}
```

### ACPI 集成

```c
// ACPI 电源状态映射
static pci_power_t acpi_state_to_pci_power_state(int acpi_state)
{
    switch (acpi_state) {
    case ACPI_STATE_D0:
        return PCI_D0;
    case ACPI_STATE_D1:
        return PCI_D1;
    case ACPI_STATE_D2:
        return PCI_D2;
    case ACPI_STATE_D3_HOT:
        return PCI_D3hot;
    case ACPI_STATE_D3_COLD:
        return PCI_D3cold;
    default:
        return PCI_UNKNOWN;
    }
}
```

## 调试和监控

### 查看电源状态

```bash
# lspci 显示电源状态
lspci -vvv -s 01:00.0 | grep -i "power\|aspm"

# sysfs 接口
cat /sys/bus/pci/devices/0000:01:00.0/power_state
cat /sys/bus/pci/devices/0000:01:00.0/d3cold_allowed

# ASPM 状态
cat /sys/module/pcie_aspm/parameters/policy
```

### 内核参数

```bash
# 禁用 ASPM
pcie_aspm=off

# 强制 ASPM L0s + L1
pcie_aspm=force
```

## 最佳实践

1. **评估唤醒延迟**：根据应用选择合适的省电级别
2. **测试兼容性**：某些设备可能不支持所有状态
3. **监控功耗**：使用工具测量实际功耗
4. **考虑性能影响**：频繁的状态转换可能影响性能

## 总结

PCIe 电源状态的关键点：

1. **L0/L0s/L1**：活跃状态，可快速恢复
2. **L2/L3**：深度睡眠，需要长时间恢复
3. **ASPM**：自动状态转换机制
4. **L1 子状态**：更精细的功耗控制
5. **软件配置**：通过寄存器和 ACPI 控制

参见：
- [ASPM 详解](aspm.md)
- [L1 子状态](l1-substates.md)
- [设备电源状态](device-states.md)
- [CLKREQ 管理](clkreq.md)
