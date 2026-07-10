# CLKREQ# 信号与时钟管理

CLKREQ# (Clock Request) 是 PCIe 的边带信号，用于动态控制参考时钟，是实现深度电源管理（特别是 L1.2 状态）的关键机制。

---

## 概述

**CLKREQ# 的作用**：
- **动态时钟控制**：设备可请求或释放参考时钟
- **深度节能**：允许平台在不需要时关闭参考时钟
- **L1.2 必需**：L1.2 状态依赖 CLKREQ# 机制
- **平台协作**：设备和平台共同管理时钟资源

**CLKREQ# 信号特性**：

| 属性 | 说明 |
|------|------|
| 信号类型 | 边带信号（Sideband），不在PCIe差分对中 |
| 电平逻辑 | 低电平有效（Active Low） |
| 方向 | 设备 → 平台（单向） |
| 用途 | 请求/释放 100 MHz 参考时钟 |
| 必需性 | L1.2 必需，其他状态可选 |

**系统架构**：
```
┌─────────────────────────────────────────────────────┐
│                    Platform                         │
│  ┌──────────────────────────────────────────────┐   │
│  │  Clock Generator (时钟发生器)                │   │
│  │  - 生成 100 MHz PCIe 参考时钟                │   │
│  │  - 接收 CLKREQ# 控制                         │   │
│  │  - 动态启用/禁用时钟输出                     │   │
│  └──────────────┬──────────────┬────────────────┘   │
│                 │              │                     │
│          REFCLK │              │ CLKREQ#             │
│       (100 MHz) │              │ (控制信号)          │
└─────────────────┼──────────────┼─────────────────────┘
                  │              │
                  ↓              ↑
         ┌────────────────────────────────┐
         │         PCIe Device            │
         │  ┌──────────────────────────┐  │
         │  │  CLKREQ# 逻辑            │  │
         │  │  - Low: 请求时钟         │  │
         │  │  - High: 不需要时钟      │  │
         │  └──────────────────────────┘  │
         │                                │
         │  使用参考时钟:                 │
         │  - 生成内部时钟               │
         │  - 链路训练                   │
         │  - 数据传输                   │
         └────────────────────────────────┘
```

---

## CLKREQ# 信号协议

### 基本操作

**请求时钟（Assert CLKREQ#）**：
```
设备拉低 CLKREQ# (Low)
    ↓
平台检测到请求
    ↓
平台启动时钟发生器
    ↓
100 MHz REFCLK 输出稳定（需要时间）
    ↓
设备接收到稳定时钟
    ↓
设备可以使用时钟进行操作
```

**释放时钟（Deassert CLKREQ#）**：
```
设备拉高 CLKREQ# (High)
    ↓
平台检测到释放
    ↓
平台可以关闭时钟发生器
    ↓
REFCLK 停止输出（节省功耗）
```

### 时序要求

**时钟请求时序**：
```
CLKREQ#    ‾‾‾‾‾‾‾‾‾‾\_____________/‾‾‾‾‾‾‾‾‾‾
            (High)    (Low - 请求)   (High)
                           
REFCLK     无时钟     等待 T_clkon   稳定时钟
                      ↓         ↓
                      |---------|
                       T_clkon
                     (典型 10-100 μs)

T_clkon: 时钟启动时间
  - 取决于平台时钟发生器特性
  - 典型值: 10-100 μs
  - PCIe 规范: 最大允许 < T_POWER_ON
```

**时钟关闭时序**：
```
CLKREQ#    ‾‾‾‾\_____________/‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾
           请求     释放
                   
REFCLK     稳定时钟  可以关闭
                    ↓
                    立即或延迟关闭
                    (平台策略决定)

注意: 平台可能延迟关闭时钟，
      以应对短暂的释放-请求周期
```

---

## CLKREQ# 在电源管理中的应用

### L1.2 状态中的 CLKREQ#

**L1.2 进入流程**：
```
L0 (Active)
  │
  ├─> 链路空闲检测
  │
  ├─> 协商进入 L1
  │     - 发送 PM_Enter_L1 DLLP
  │     - 双方确认
  │
  ├─> 进入 L1
  │     - 发送 EIOS
  │     - 关闭 PLL
  │
  ├─> 进入 L1.2 (如果启用)
  │     - 设备 Deassert CLKREQ# (拉高)
  │     - 告知平台不需要参考时钟
  │
  └─> L1.2 状态
        - REFCLK 停止
        - 最低功耗
        - 设备保持最小电路运行
```

**L1.2 退出流程**：
```
L1.2 状态
  │
  ├─> 唤醒事件触发
  │     - 设备内部事件
  │     - 或 WAKE# 信号
  │
  ├─> Assert CLKREQ# (拉低)
  │     - 请求参考时钟
  │
  ├─> 等待 REFCLK 稳定
  │     - 延迟: T_POWER_ON (配置中定义)
  │     - 典型: 10-100 μs
  │
  ├─> 启动 PLL
  │     - 使用恢复的 REFCLK
  │     - 生成内部时钟
  │
  ├─> 链路训练
  │     - 发送 TS1/TS2
  │     - 恢复链路同步
  │
  └─> 返回 L0
        - 链路恢复活动状态
```

**关键时间参数**：
```
L1 PM Substates Control 2 Register:
┌───────────────────────────────────────┐
│ T_POWER_ON Value (5 bits)             │
│ T_POWER_ON Scale (2 bits)             │
│                                       │
│ T_POWER_ON = Value × Scale            │
│                                       │
│ Scale:                                │
│   00b = 2 μs                          │
│   01b = 10 μs                         │
│   10b = 100 μs                        │
│   11b = Reserved                      │
│                                       │
│ 例如: Value=5, Scale=01b              │
│       T_POWER_ON = 5 × 10 μs = 50 μs  │
└───────────────────────────────────────┘

设备必须在 Assert CLKREQ# 后等待至少 T_POWER_ON
才能假设 REFCLK 已稳定
```

### CLKREQ# 在其他状态中的使用

**L0/L0s 状态**：
```
CLKREQ# 保持 Low (Asserted)
  - 设备需要持续的参考时钟
  - 支持快速数据传输
  - 快速状态恢复
```

**L1 状态（无 L1.2）**：
```
CLKREQ# 可以保持 Low
  - 平台可继续提供 REFCLK
  - 设备关闭 PLL 但保留时钟路径
  - 或者设备 Deassert CLKREQ# 以节省更多功耗
    (取决于实现)
```

**L2/L3 状态（更深度休眠）**：
```
CLKREQ# 通常 High (Deasserted)
  - 设备主电源关闭或极低功耗
  - 不需要参考时钟
  - 恢复需要完整的初始化
```

---

## 硬件实现

### 平台端实现

**时钟发生器控制**：
```c
// 伪代码: 平台时钟管理逻辑

typedef struct {
    bool clkreq_asserted;  // CLKREQ# 引脚状态
    bool clock_enabled;     // 时钟输出是否启用
    uint32_t t_clkon_us;   // 时钟启动延迟
} clk_controller_t;

void clkreq_handler(clk_controller_t *ctrl)
{
    // 监听 CLKREQ# 信号变化
    bool clkreq = read_clkreq_pin();
    
    if (clkreq == LOW && !ctrl->clock_enabled) {
        // 设备请求时钟
        start_clock_generator();
        udelay(ctrl->t_clkon_us);  // 等待时钟稳定
        ctrl->clock_enabled = true;
        
    } else if (clkreq == HIGH && ctrl->clock_enabled) {
        // 设备释放时钟
        // 可以立即关闭，或延迟以避免频繁开关
        if (idle_time > CLKOFF_DELAY_MS) {
            stop_clock_generator();
            ctrl->clock_enabled = false;
        }
    }
}
```

**平台策略**：
```
策略 1: 立即响应
  - CLKREQ# High → 立即关闭时钟
  - CLKREQ# Low → 立即启动时钟
  - 优点: 最大功耗节省
  - 缺点: 频繁开关可能影响时钟发生器寿命

策略 2: 延迟关闭
  - CLKREQ# High → 等待一段时间才关闭
  - CLKREQ# Low → 立即启动时钟
  - 优点: 减少开关频率
  - 缺点: 功耗节省略少

策略 3: 合并请求
  - 多个设备共享时钟源
  - 只有所有设备都释放才关闭
  - 优点: 简化管理
  - 缺点: 单个设备无法独立控制
```

### 设备端实现

**CLKREQ# 控制逻辑**：
```verilog
// Verilog 示例: 设备端 CLKREQ# 控制

module clkreq_controller (
    input wire clk,
    input wire rst_n,
    input wire enter_l1_2,      // 进入 L1.2 请求
    input wire exit_l1_2,       // 退出 L1.2 请求
    input wire refclk_stable,   // 参考时钟稳定指示
    output reg clkreq_n         // CLKREQ# 输出 (低有效)
);

    localparam ST_L0 = 2'b00;
    localparam ST_L1_2 = 2'b01;
    localparam ST_WAIT_CLK = 2'b10;
    
    reg [1:0] state;
    reg [15:0] t_power_on_cnt;
    
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            state <= ST_L0;
            clkreq_n <= 1'b0;  // 默认请求时钟
            t_power_on_cnt <= 0;
        end else begin
            case (state)
                ST_L0: begin
                    clkreq_n <= 1'b0;  // L0: 需要时钟
                    if (enter_l1_2) begin
                        clkreq_n <= 1'b1;  // 释放时钟
                        state <= ST_L1_2;
                    end
                end
                
                ST_L1_2: begin
                    clkreq_n <= 1'b1;  // L1.2: 不需要时钟
                    if (exit_l1_2) begin
                        clkreq_n <= 1'b0;  // 请求时钟
                        t_power_on_cnt <= T_POWER_ON;
                        state <= ST_WAIT_CLK;
                    end
                end
                
                ST_WAIT_CLK: begin
                    clkreq_n <= 1'b0;  // 保持请求
                    if (t_power_on_cnt > 0) begin
                        t_power_on_cnt <= t_power_on_cnt - 1;
                    end else if (refclk_stable) begin
                        state <= ST_L0;  // 时钟稳定，返回 L0
                    end
                end
            endcase
        end
    end
endmodule
```

### PCIe Capability 配置

**L1 PM Substates Capability**：
```c
// FEMU 示例: 配置 CLKREQ# 相关能力

void pcie_cap_l1pm_init(PCIDevice *dev, uint16_t offset)
{
    uint8_t *cfg = dev->config + offset;
    uint32_t cap = 0;
    uint32_t ctrl2 = 0;
    
    // L1 PM Substates Capabilities
    cap |= PCI_L1PM_CAP_ASPM_L1_2;   // 支持 L1.2
    cap |= PCI_L1PM_CAP_PCIPM_L1_2;
    cap |= PCI_L1PM_CAP_CM_RESTORE_TIME(10);  // 10 μs
    cap |= PCI_L1PM_CAP_PORT_COMMON_MODE;     // 支持公共模式
    
    pci_set_long(cfg + PCI_L1PM_CAP, cap);
    
    // L1 PM Substates Control 2: T_POWER_ON
    // T_POWER_ON = 50 μs (Value=5, Scale=10μs)
    ctrl2 |= (5 << 0);   // Value
    ctrl2 |= (1 << 5);   // Scale: 01b = 10 μs
    
    pci_set_long(cfg + PCI_L1PM_CTRL2, ctrl2);
    
    // wmask: 允许软件配置
    pci_set_long(dev->wmask + offset + PCI_L1PM_CTRL2,
                 PCI_L1PM_CTRL2_TPOWERON_MASK);
}
```

---

## 配置与调试

### 检查 CLKREQ# 支持

**使用 lspci**：
```bash
# 查看 L1 PM Substates 能力
lspci -vvv -s 01:00.0 | grep -A 10 "L1 PM Substates"

L1SubCap: PCI-PM_L1.2+ PCI-PM_L1.1+ ASPM_L1.2+ ASPM_L1.1+ L1_PM_Substates+
          PortCommonModeRestoreTime=10us PortTPowerOnTime=50us
L1SubCtl1: PCI-PM_L1.2+ PCI-PM_L1.1+ ASPM_L1.2+ ASPM_L1.1+
           T_CommonMode=10us LTR1.2_Threshold=0ns
L1SubCtl2: T_PwrOn=50us

# 解读:
# - PortTPowerOnTime=50us: 平台时钟启动需要 50 μs
# - T_PwrOn=50us: 设备配置的等待时间
```

**使用 setpci**：
```bash
# 假设 L1 PM Substates Cap 在 Extended Config Space 0x200

# 读取 Capabilities
setpci -s 01:00.0 200.l
  # 检查 L1.2 支持位

# 读取 Control 2 (T_POWER_ON)
setpci -s 01:00.0 20c.l
  # Bit 4-0: Value
  # Bit 6-5: Scale
```

### CLKREQ# 信号测试

**硬件调试**：
```
使用示波器或逻辑分析仪:

1. 连接 CLKREQ# 引脚
   - 找到 PCIe 插槽的 CLKREQ# 引脚
   - Mini PCIe: Pin 22
   - M.2: 根据 key type (通常 Pin 20 或 32)

2. 连接 REFCLK 信号
   - 100 MHz 差分时钟对

3. 触发 L1.2 进入/退出
   - 在系统中启用 L1.2
   - 观察设备空闲时的行为

4. 预期波形:
   CLKREQ#   ‾‾‾‾\_____________/‾‾‾‾‾‾‾‾‾
             L0   L1.2 进入      L1.2 退出
             
   REFCLK    活动  停止(~T_clkon) 恢复
                   ←----------→
                    T_POWER_ON
```

**软件测试**：
```bash
# 启用 L1.2
echo powersupersave > /sys/module/pcie_aspm/parameters/policy

# 监控电源状态
watch -n 0.1 'cat /sys/bus/pci/devices/0000:01:00.0/power/runtime_status'
# 预期: 空闲后变为 suspended

# 触发 I/O 观察唤醒
dd if=/dev/nvme0n1 of=/dev/null bs=4k count=1
# CLKREQ# 应该从 High 变为 Low，然后 REFCLK 恢复

# dmesg 查看电源状态转换
dmesg | grep -i "L1"
```

### 常见问题排查

**问题 1: L1.2 无法进入**
```
症状: lspci 显示支持 L1.2，但实际不进入

检查:
1. BIOS 是否启用 CLKREQ#
   - 某些平台 BIOS 可禁用 CLKREQ# 功能
   
2. 硬件连接
   - CLKREQ# 引脚是否正确连接
   - 使用万用表测试引脚连续性

3. 软件配置
   cat /sys/module/pcie_aspm/parameters/policy
   # 应该是 powersupersave

4. 内核日志
   dmesg | grep -i clkreq
   # 查看是否有 CLKREQ 相关错误
```

**问题 2: REFCLK 启动超时**
```
症状: 从 L1.2 恢复失败，设备不响应

原因:
- T_POWER_ON 配置过短
- 平台时钟发生器启动慢于预期

解决:
1. 增加 T_POWER_ON 值
   setpci -s 01:00.0 20c.l=0x0000000A
   # 设置更大的 Value

2. 检查平台规格
   - 查阅主板手册，确认时钟启动时间
   - 可能需要 BIOS 更新

3. 禁用 L1.2 作为临时方案
   setpci -s 01:00.0 208.l=0x00000000
```

**问题 3: 频繁进出 L1.2 导致性能下降**
```
症状: 随机读性能显著降低

原因:
- 每次 I/O 都触发 L1.2 → L0 转换
- T_POWER_ON 延迟累积

解决:
1. 调整 LTR (Latency Tolerance Reporting)
   - 增加 LTR 阈值，减少进入 L1.2 频率
   
2. 使用 L1.1 代替 L1.2
   - L1.1 不关闭 REFCLK，恢复更快
   setpci -s 01:00.0 208.l=0x00000005
   # 启用 L1.1，禁用 L1.2

3. 针对工作负载调整策略
   # 高性能工作负载
   echo performance > /sys/module/pcie_aspm/parameters/policy
   
   # 低频访问工作负载
   echo powersupersave > /sys/module/pcie_aspm/parameters/policy
```

---

## 系统级考虑

### 多设备 CLKREQ# 管理

**独立时钟源**：
```
每个 PCIe 端口有独立的 CLKREQ#
  设备 1                设备 2
    │                      │
    │ CLKREQ#1            │ CLKREQ#2
    ↓                      ↓
  ┌────────┐          ┌────────┐
  │ CLK 1  │          │ CLK 2  │
  └────────┘          └────────┘
    │                      │
    └──────────┬───────────┘
               │
          ┌─────────┐
          │ Platform│
          └─────────┘

优点: 设备独立控制时钟
缺点: 成本高，需要多个时钟源
```

**共享时钟源**：
```
多个设备共享一个时钟源
  设备 1        设备 2        设备 3
    │              │              │
    │ CLKREQ#1    │ CLKREQ#2    │ CLKREQ#3
    └──────┬───────┴──────┬───────┘
           │              │
           ↓              ↓
       ┌──────────────────────┐
       │  CLKREQ# OR Gate     │
       │  (任一 Low = Low)     │
       └───────────┬──────────┘
                   │
              ┌─────────┐
              │Shared CLK│
              └─────────┘
                   │
         ┌─────────┼─────────┐
         │         │         │
      设备 1     设备 2     设备 3

逻辑: 任一设备请求时钟，所有设备都有时钟
优点: 成本低
缺点: 无法完全独立省电
```

### 功耗预算

**CLKREQ# 带来的功耗节省**：
```
示例系统: 笔记本，4 个 PCIe 设备

场景 1: 无 CLKREQ# (所有时钟始终运行)
  4 × 时钟分发功耗 = 4 × 50 mW = 200 mW
  4 × 设备 L1 功耗 = 4 × 100 mW = 400 mW
  总计 = 600 mW

场景 2: 有 CLKREQ# (空闲设备关闭时钟)
  3 个设备空闲，1 个活动
  1 × 时钟分发 = 50 mW
  3 × L1.2 功耗 = 3 × 10 mW = 30 mW
  1 × 活动 = 500 mW
  总计 = 580 mW

场景 3: 所有设备 L1.2
  0 × 时钟分发 = 0 mW
  4 × L1.2 = 4 × 10 mW = 40 mW
  总计 = 40 mW

节省: 场景 1 → 场景 3: 93% 功耗降低
```

---

## 总结

### 关键要点

1. **CLKREQ# 是 L1.2 的关键**：允许平台动态关闭参考时钟，实现最低功耗
2. **双向协作**：设备通过 CLKREQ# 通知平台需求，平台控制时钟供应
3. **时间权衡**：关闭时钟节省功耗，但恢复需要 T_POWER_ON 延迟
4. **配置灵活性**：T_POWER_ON 可根据平台特性配置
5. **系统设计**：多设备系统需考虑时钟源架构（独立 vs 共享）
6. **调试重要**：CLKREQ# 问题需要硬件测试和软件配置结合

**最佳实践**：
- **移动设备**：充分利用 CLKREQ# 和 L1.2，最大化续航
- **服务器**：根据工作负载决定，低延迟需求可禁用 L1.2
- **台式机**：平衡模式，使用 L1.1（保留 REFCLK）
- **调试**：从 L0s/L1 开始，逐步启用 L1.1/L1.2

---

## 下一步学习

- [ASPM](aspm.md) - Active State Power Management 详解
- [LTR](ltr.md) - Latency Tolerance Reporting（与 L1.2 配合）
- [设备电源状态](power-states.md) - D0-D3 状态管理

---

## 参考资料

- **规范**：PCIe Base Spec 5.5.2 (CLKREQ#)
- **L1 PM Substates**：PCIe Base Spec 5.5.4
- **实现**：`hw/pci/pcie.c` (FEMU)
- **Linux 驱动**：`drivers/pci/pcie/aspm.c`

---

**相关页面**：
- [← ASPM](aspm.md)
- [LTR →](ltr.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
