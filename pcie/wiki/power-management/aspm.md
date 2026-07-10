# ASPM (Active State Power Management) 电源管理

ASPM (Active State Power Management) 是 PCIe 的链路电源管理机制，通过在空闲时将链路切换到低功耗状态来节省能源，对笔记本和数据中心尤为重要。

---

## 概述

**ASPM 的目标**：
- **降低功耗**：在链路空闲时减少电源消耗
- **快速恢复**：从低功耗状态快速返回全速
- **透明操作**：对软件尽可能透明
- **灵活配置**：平衡性能和功耗

**ASPM 与传统电源管理的区别**：

| 特性 | ASPM | Device Power States (D0-D3) |
|------|------|----------------------------|
| 范围 | 链路级别 | 设备级别 |
| 触发 | 硬件自动 | 软件控制 |
| 延迟 | 微秒级 | 毫秒级 |
| 透明性 | 对软件透明 | 需要软件干预 |
| 节能 | 中等 | 高 |

**ASPM 在系统中的位置**：
```
┌─────────────────────────────────────────┐
│  应用层：无感知                         │
└─────────────────────────────────────────┘
                   │
┌─────────────────────────────────────────┐
│  OS/驱动：配置 ASPM 策略                │
│  - 启用/禁用 L0s/L1                     │
│  - 设置进入/退出延迟                    │
└──────────────────┬──────────────────────┘
                   │
┌─────────────────────────────────────────┐
│  PCIe 硬件：自动状态转换                │
│  ┌──────────────────────────────────┐   │
│  │ L0 (Active)                      │   │
│  │   ↕ 空闲检测                     │   │
│  │ L0s (Standby)                    │   │
│  │   ↕ 更长空闲                     │   │
│  │ L1 (Low Power)                   │   │
│  │   ↕ CLKREQ# 控制                 │   │
│  │ L1.1/L1.2 (Ultra Low Power)      │   │
│  └──────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

---

## ASPM 状态详解

### L0 - Active State (活动状态)

**特点**：
- 全速运行状态
- 链路时钟运行，可传输数据
- 功耗最高
- 无延迟

**L0 状态机制**：
```
发送器 (TX)               接收器 (RX)
  │                          │
  ├─> 发送 TLP ──────────────>│
  │                          │
  │<────────── 接收 TLP ──────┤
  │                          │
  ├─> 空闲：发送 IDL 字符 ───>│
  │   (8b/10b: K28.5)        │
  │   (128b/130b: Idle OS)   │
```

### L0s - Standby State (待机状态)

**特点**：
- 快速低功耗状态
- TX 或 RX 可独立进入
- 关闭发送器电路，保持接收器运行
- 退出延迟：< 1 μs (通常 200-500 ns)

**L0s 子状态**：
- **Tx L0s**：发送方向进入 L0s，接收方向保持 L0
- **Rx L0s**：接收方向进入 L0s，发送方向保持 L0

**状态转换**：
```
L0 → L0s:
┌────────────────────────────────────────┐
│ 1. 检测空闲（无 TLP 发送 N μs）        │
│ 2. 发送 EIOS (Electrical Idle OS)     │
│ 3. 进入 Electrical Idle (静电状态)    │
│ 4. 关闭发送器                          │
└────────────────────────────────────────┘

L0s → L0:
┌────────────────────────────────────────┐
│ 1. 检测到待发送 TLP                    │
│ 2. 启动发送器                          │
│ 3. 发送 FTS (Fast Training Sequence)  │
│ 4. 接收器锁定时钟                      │
│ 5. 恢复到 L0                           │
└────────────────────────────────────────┘

时序图:
L0            L0s              L0
 │  空闲检测   │   需要发送      │
 ├────────────>│                │
 │  发送 EIOS  │                │
 │  关闭 TX    │   启动 TX      │
 │             ├───────────────>│
 │             │  发送 FTS      │
 │             │  时钟恢复      │
               │                │
        延迟: 200-500 ns
```

**FTS (Fast Training Sequence)**：
```
用于快速时钟恢复的训练序列

8b/10b (Gen 1-2):
  发送 N 个 K28.1 (FTS) 符号
  N 由 Link Capabilities 定义
  
128b/130b (Gen 3+):
  发送 EIEOS (Electrical Idle Exit OS)
  更快的恢复机制
```

### L1 - Low Power State (低功耗状态)

**特点**：
- 更深度的低功耗状态
- TX 和 RX 都进入低功耗
- 关闭链路时钟（PLL 可选关闭）
- 退出延迟：2-4 μs（取决于配置）

**进入 L1 条件**：
- 两端都支持 L1
- 空闲时间超过阈值
- 双向握手完成

**状态转换**：
```
L0 → L1:
┌────────────────────────────────────────┐
│ 1. Downstream Port 发送 PM_Enter_L1    │
│    DLLP                                │
│ 2. Upstream Port 确认                  │
│ 3. 双方发送 EIOS                       │
│ 4. 进入 Electrical Idle                │
│ 5. 关闭 PLL (可选)                     │
│ 6. 关闭链路时钟                        │
└────────────────────────────────────────┘

L1 → L0:
┌────────────────────────────────────────┐
│ 1. 检测到唤醒事件                      │
│    - WAKE# 信号                        │
│    - Beacon (低频信号)                 │
│ 2. 启动 PLL                            │
│ 3. 时钟稳定                            │
│ 4. 执行链路训练 (TS1/TS2)              │
│ 5. 恢复到 L0                           │
└────────────────────────────────────────┘

时序图:
L0               L1                  L0
 │  空闲超时      │    唤醒事件        │
 │  PM_Enter_L1   │                   │
 ├───────────────>│                   │
 │  确认          │                   │
 │  EIOS          │   Beacon/WAKE#    │
 │  关闭时钟      │                   │
 │                ├──────────────────>│
 │                │   启动 PLL        │
 │                │   链路训练        │
                  │                   │
           延迟: 2-4 μs
```

### L1.1 - L1 Sub-State 1 (L1.1)

**特点** (PCIe 3.1+)：
- L1 的子状态，更低功耗
- 关闭 PLL，保持参考时钟
- 退出延迟：8-16 μs

**与 L1 的区别**：
```
L1:   PLL 可能运行，时钟分布关闭
L1.1: PLL 关闭，参考时钟保持
L1.2: PLL 和参考时钟都关闭
```

### L1.2 - L1 Sub-State 2 (L1.2)

**特点** (PCIe 3.1+)：
- 最低功耗状态
- 关闭 PLL 和参考时钟
- 退出延迟：> 16 μs (可达 100 μs)
- 需要 CLKREQ# 信号控制

**CLKREQ# 机制**：
```
CLKREQ# 信号流程:

L0 状态:
  Device                     Platform
    │                           │
    │  CLKREQ# = Low (请求时钟) │
    ├──────────────────────────>│
    │                           │
    │  提供 100 MHz 参考时钟    │
    │<──────────────────────────┤

进入 L1.2:
  Device                     Platform
    │                           │
    │  CLKREQ# = High (不需要)  │
    ├──────────────────────────>│
    │                           │
    │  停止参考时钟             │
    │<──────────────────────────┤

退出 L1.2:
  Device                     Platform
    │                           │
    │  CLKREQ# = Low (请求时钟) │
    ├──────────────────────────>│
    │                           │
    │  重新提供参考时钟 (等待)  │
    │<──────────────────────────┤
    │                           │
    │  时钟稳定后恢复           │
```

---

## 延迟与功耗对比

### 状态对比表

| 状态 | 功耗节省 | 退出延迟 | 应用场景 | PCIe Gen |
|------|---------|---------|---------|----------|
| L0 | 0% (基准) | 0 | 活动传输 | All |
| L0s | 10-20% | 200-500 ns | 短暂空闲 | All |
| L1 | 60-90% | 2-4 μs | 较长空闲 | All |
| L1.1 | 90-95% | 8-16 μs | 长时间空闲 | 3.1+ |
| L1.2 | 95-99% | 16-100 μs | 极长空闲 | 3.1+ |

### 功耗示例

**典型 PCIe Gen 3 x4 链路**：
```
L0 (Active):        ~1.5 W
L0s:                ~1.2 W (20% 节省)
L1:                 ~0.3 W (80% 节省)
L1.1:               ~0.1 W (93% 节省)
L1.2:               ~0.02 W (98% 节省)
```

### 延迟影响

**对性能的影响**：
```
场景 1: 高频率小请求 (如随机读)
  L0s: 每次恢复 500 ns → 累积延迟显著
  建议: 禁用 ASPM 或仅 L1
  
场景 2: 批量传输 + 空闲
  L0s: 短暂空闲快速恢复
  L1: 长时间空闲深度节能
  建议: 启用 L0s + L1
  
场景 3: 低频访问 (如系统盘)
  L1.2: 长时间空闲，高节能
  建议: 启用 L1.2 (如果支持)
```

---

## 配置与控制

### Link Capabilities (链路能力)

**配置空间 PCIe Capability + 0x0C**：
```
Link Capabilities Register:
31                                        0
┌────┬────┬────┬────────────┬────────────┐
│Port│ASPM│ L1 │ L0s Exit   │ L0s Exit   │
│Num │Sup │Exit│ Latency    │ Latency    │
│    │    │Lat │ (Up Port)  │ (Down Port)│
└────┴────┴────┴────────────┴────────────┘

Bit 11-10: ASPM Support
  00b = 不支持 ASPM
  01b = 仅支持 L0s
  10b = 仅支持 L1
  11b = 支持 L0s 和 L1

Bit 17-15: L1 Exit Latency
  000b = < 1 μs
  001b = 1-2 μs
  010b = 2-4 μs
  011b = 4-8 μs
  100b = 8-16 μs
  101b = 16-32 μs
  110b = 32-64 μs
  111b = > 64 μs

Bit 14-12: L0s Exit Latency (Upstream)
  000b = < 64 ns
  001b = 64-128 ns
  010b = 128-256 ns
  011b = 256-512 ns
  100b = 512 ns - 1 μs
  101b = 1-2 μs
  110b = 2-4 μs
  111b = > 4 μs
```

### Link Control (链路控制)

**配置空间 PCIe Capability + 0x10**：
```
Link Control Register:
15                                        0
┌────┬────┬────┬────────────┬────────────┐
│... │ASPM│... │   ...      │   ...      │
│    │Ctrl│    │            │            │
└────┴────┴────┴────────────┴────────────┘

Bit 1-0: ASPM Control
  00b = Disabled (禁用 ASPM)
  01b = L0s Enabled
  10b = L1 Enabled
  11b = L0s and L1 Enabled
```

### L1 PM Substates Capability (L1 子状态)

**Extended Capability (0x100+)**：
```
L1 PM Substates Capability (PCIe 3.1+):

Offset 0x04: L1 PM Substates Capabilities
┌────────────────────────────────────────┐
│ PCI-PM L1.2 Supported                  │
│ PCI-PM L1.1 Supported                  │
│ ASPM L1.2 Supported                    │
│ ASPM L1.1 Supported                    │
│ L1 PM Substates Support                │
│ ...                                    │
└────────────────────────────────────────┘

Offset 0x08: L1 PM Substates Control 1
┌────────────────────────────────────────┐
│ PCI-PM L1.2 Enable                     │
│ PCI-PM L1.1 Enable                     │
│ ASPM L1.2 Enable                       │
│ ASPM L1.1 Enable                       │
│ Common Mode Restore Time (8 bits)      │
│ LTR L1.2 Threshold Value (10 bits)     │
│ ...                                    │
└────────────────────────────────────────┘

Offset 0x0C: L1 PM Substates Control 2
┌────────────────────────────────────────┐
│ T_POWER_ON Scale (2 bits)              │
│ T_POWER_ON Value (5 bits)              │
│   上电延迟 = Value * Scale             │
└────────────────────────────────────────┘
```

---

## FEMU 实现

### ASPM Capability 初始化

```c
// hw/pci/pcie.c
void pcie_cap_lnk_init(PCIDevice *dev)
{
    uint32_t lnkcap = 0;
    uint8_t *exp_cap = dev->config + dev->exp.exp_cap;
    
    // 设置 ASPM Support (L0s and L1)
    lnkcap |= PCI_EXP_LNKCAP_ASPMS_0S;  // L0s
    lnkcap |= PCI_EXP_LNKCAP_ASPMS_L1;  // L1
    
    // 设置 L0s Exit Latency (< 1 μs)
    lnkcap |= (2 << 12);  // 128-256 ns
    
    // 设置 L1 Exit Latency (2-4 μs)
    lnkcap |= (2 << 15);  // 2-4 μs
    
    // 写入 Link Capabilities
    pci_set_long(exp_cap + PCI_EXP_LNKCAP, lnkcap);
    
    // Link Control: 默认禁用 ASPM (由 OS 启用)
    pci_set_word(exp_cap + PCI_EXP_LNKCTL, 0);
    
    // Link Control wmask: 允许 OS 写入
    pci_set_word(dev->wmask + dev->exp.exp_cap + PCI_EXP_LNKCTL,
                 PCI_EXP_LNKCTL_ASPMC);
}
```

### ASPM 状态模拟

```c
// hw/pci/pcie.c
// FEMU 作为功能模拟器，不真正进入低功耗状态
// 但可以模拟寄存器访问

static void pcie_aspm_state_change(PCIDevice *dev, uint16_t lnkctl)
{
    uint8_t aspm = lnkctl & PCI_EXP_LNKCTL_ASPMC;
    
    switch (aspm) {
    case 0:  // ASPM Disabled
        // 保持 L0
        break;
    case 1:  // L0s Enabled
        // 在实际硬件中，会在空闲时自动进入 L0s
        // FEMU 不模拟，但记录配置
        break;
    case 2:  // L1 Enabled
        // 实际硬件会进入 L1
        break;
    case 3:  // L0s and L1 Enabled
        // 优先使用 L0s (快速)，长时间空闲进入 L1
        break;
    }
    
    // 在实际系统中，这里会:
    // 1. 配置硬件状态机
    // 2. 设置空闲检测定时器
    // 3. 启用自动状态转换
}
```

### L1 子状态支持

```c
// hw/pci/pcie.c
void pcie_cap_l1pm_init(PCIDevice *dev, uint16_t offset)
{
    uint8_t *cfg = dev->config + offset;
    uint32_t cap = 0;
    
    // 添加 L1 PM Substates Extended Capability
    pcie_add_capability(dev, PCI_EXT_CAP_ID_L1PM, 0x1,
                       offset, PCI_EXT_CAP_L1PM_SIZEOF);
    
    // 设置支持的子状态
    cap |= PCI_L1PM_CAP_ASPM_L1_1;   // ASPM L1.1
    cap |= PCI_L1PM_CAP_ASPM_L1_2;   // ASPM L1.2
    cap |= PCI_L1PM_CAP_PCIPM_L1_1;  // PCI-PM L1.1
    cap |= PCI_L1PM_CAP_PCIPM_L1_2;  // PCI-PM L1.2
    
    // Common Mode Restore Time (典型 10 μs)
    cap |= (10 << 8);
    
    // T_POWER_ON (上电时间，典型 50 μs)
    cap |= (50 << 19);
    
    pci_set_long(cfg + PCI_L1PM_CAP, cap);
    
    // Control: 默认禁用，由 OS 启用
    pci_set_long(cfg + PCI_L1PM_CTRL1, 0);
    pci_set_long(cfg + PCI_L1PM_CTRL2, 0);
}
```

---

## 实用技巧

### 查看 ASPM 配置

**使用 lspci**：
```bash
# 查看设备 ASPM 配置
lspci -vvv -s 01:00.0 | grep -i aspm
  LnkCap: Port #0, Speed 8GT/s, Width x4, ASPM L0s L1, Exit Latency L0s <512ns, L1 <4us
  LnkCtl: ASPM L1 Enabled; RCB 64 bytes, Disabled- CommClk+

# 解读:
# - LnkCap: 设备支持 L0s 和 L1
# - LnkCtl: 当前启用 L1
# - Exit Latency: L0s < 512 ns, L1 < 4 μs
```

**使用 setpci 读取**：
```bash
# 读取 Link Capabilities
setpci -s 01:00.0 CAP_EXP+0c.l
  00042c11
  # Bit 11-10 = 11b → 支持 L0s 和 L1

# 读取 Link Control
setpci -s 01:00.0 CAP_EXP+10.w
  0002
  # Bit 1-0 = 10b → L1 Enabled
```

### 启用/禁用 ASPM

**内核参数**：
```bash
# 禁用所有 ASPM
pcie_aspm=off

# 强制启用 ASPM (覆盖 BIOS)
pcie_aspm=force

# 编辑 GRUB
vi /etc/default/grub
GRUB_CMDLINE_LINUX="... pcie_aspm=off"
update-grub
reboot
```

**运行时修改**（单个设备）：
```bash
# 禁用 ASPM
setpci -s 01:00.0 CAP_EXP+10.w=0000

# 启用 L0s
setpci -s 01:00.0 CAP_EXP+10.w=0001

# 启用 L1
setpci -s 01:00.0 CAP_EXP+10.w=0002

# 启用 L0s + L1
setpci -s 01:00.0 CAP_EXP+10.w=0003
```

**通过 sysfs**（推荐）：
```bash
# 查看当前策略
cat /sys/module/pcie_aspm/parameters/policy
  [default] performance powersave powersupersave

# 切换到性能模式（禁用 ASPM）
echo performance > /sys/module/pcie_aspm/parameters/policy

# 切换到省电模式（启用 ASPM）
echo powersave > /sys/module/pcie_aspm/parameters/policy
```

### 测试 ASPM 影响

**性能测试**：
```bash
# 禁用 ASPM
echo performance > /sys/module/pcie_aspm/parameters/policy

# 测试性能
fio --name=test --rw=randread --bs=4k --numjobs=4 \
    --iodepth=32 --filename=/dev/nvme0n1 --direct=1 \
    --runtime=30 --output=perf_no_aspm.txt

# 启用 ASPM
echo powersave > /sys/module/pcie_aspm/parameters/policy

# 再次测试
fio --name=test --rw=randread --bs=4k --numjobs=4 \
    --iodepth=32 --filename=/dev/nvme0n1 --direct=1 \
    --runtime=30 --output=perf_with_aspm.txt

# 对比结果
# 通常: L0s 对性能影响 < 5%
#       L1 对高频访问影响 5-15%
#       L1.2 对低频访问几乎无影响
```

**功耗测试**：
```bash
# 需要硬件功耗监控工具
# 如笔记本可以查看电池消耗

# 空闲功耗测试
echo powersave > /sys/module/pcie_aspm/parameters/policy
# 等待 10 秒，观察功耗
# 预期: 显著降低（尤其是 L1.2）

echo performance > /sys/module/pcie_aspm/parameters/policy
# 对比功耗差异
```

### 调试 ASPM 问题

**常见问题**：
```
问题 1: 启用 ASPM 后设备无响应
原因: L1 退出延迟超时，设备恢复失败
解决: 禁用 ASPM 或仅启用 L0s

问题 2: 性能显著下降
原因: 频繁进入/退出 L1，累积延迟
解决: 使用 performance 策略或仅 L0s

问题 3: dmesg 报 ASPM 错误
dmesg | grep -i aspm
  pci 0000:01:00.0: disabling ASPM on pre-1.1 PCIe device
原因: 旧设备 ASPM 实现有问题
解决: 内核自动禁用，无需干预
```

---

## 总结

### 关键要点

1. ✅ **ASPM 提供多级低功耗状态**：L0s, L1, L1.1, L1.2，功耗逐级降低
2. ✅ **延迟与功耗权衡**：L0s 快速但节能少，L1.2 慢但节能显著
3. ✅ **硬件自动控制**：状态转换由硬件根据空闲时间自动执行
4. ✅ **灵活配置**：OS 可根据应用场景选择策略
5. ✅ **L1 子状态需要 CLKREQ#**：平台必须支持动态时钟控制
6. ✅ **FEMU 模拟配置接口**，但不真正进入低功耗状态

**使用建议**：
- **台式机/服务器**：性能优先，禁用或仅 L0s
- **笔记本**：平衡模式，启用 L0s + L1
- **移动/嵌入式**：省电优先，启用 L1.2
- **高性能设备**：根据工作负载动态调整

---

## 下一步学习

- [设备电源状态](device-power-states.md) - D0-D3 状态详解
- [LTR](ltr.md) - Latency Tolerance Reporting（延迟容忍报告）
- [Clock Gating](clock-gating.md) - 时钟门控技术

---

## 参考资料

- **规范**：PCIe Base Spec Chapter 5.4 (Link Power Management)
- **L1 PM Substates**：PCIe Base Spec 5.5.4 (L1 PM Substates)
- **实现**：`hw/pci/pcie.c` (FEMU)
- **Linux**：`drivers/pci/pcie/aspm.c` (内核 ASPM 驱动)

---

**相关页面**：
- [← 电源管理概述](overview.md)
- [LTR →](ltr.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
