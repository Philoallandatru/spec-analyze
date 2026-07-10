# PCIe 链路均衡

## 概述

链路均衡 (Link Equalization) 是 PCIe Gen3+ 引入的信号完整性优化技术，通过调整发送端预加重和接收端均衡器参数来补偿信号传输损耗，确保高速链路的可靠性。

## 为什么需要均衡

### 信号传输问题

```
PCIe Gen3 (8 GT/s) 及以上面临的挑战：

1. 频率相关损耗
   - 高频信号衰减更快
   - PCB 走线、连接器损耗
   - 趋肤效应

2. 码间干扰 (ISI)
   - 符号之间相互影响
   - 导致眼图闭合

3. 反射和串扰
   - 阻抗不匹配
   - 相邻信号线干扰

解决方案：链路均衡
- 发送端预加重 (Pre-emphasis/De-emphasis)
- 接收端连续时间线性均衡器 (CTLE)
- 决策反馈均衡器 (DFE)
```

### 均衡效果

```
无均衡:
眼图高度: 100 mV → 30 mV (严重衰减)
眼图宽度: 80 ps → 40 ps (闭合)
误码率: 10^-6 (不可接受)

有均衡:
眼图高度: 100 mV → 80 mV (良好)
眼图宽度: 80 ps → 70 ps (开阔)
误码率: 10^-12 (可靠)
```

## 均衡技术

### 1. 发送端预加重

```
预加重原理：
提前放大高频分量，补偿传输损耗

波形调整：
Original:    ___     ___
            |   |   |   |
            |   |___|   |___

Pre-emphasis: ^___     ^___
             /|   \   /|   \
            | |   |\_/|   |___
            强调边沿，减弱平坦部分

参数：
- Cursor (C0): 主抽头
- Pre-cursor (C-1): 前抽头
- Post-cursor (C+1): 后抽头

典型配置：
C-1 = -2 dB
C0  = 0 dB (参考)
C+1 = -3.5 dB
```

### 2. 接收端均衡

**CTLE (Continuous Time Linear Equalizer)**:
```
连续时间线性均衡器

特性：
- 频域均衡
- 放大高频，衰减低频
- 补偿频率相关损耗

传输函数：
H(f) = A × (1 + f/fp)
      ─────────────
       (1 + f/fz)

fp: 零点频率
fz: 极点频率
A: 增益

配置参数：
- DC Gain
- Peaking Frequency
- Peaking Magnitude
```

**DFE (Decision Feedback Equalizer)**:
```
决策反馈均衡器

原理：
1. 检测已判决的符号
2. 估计其对当前符号的影响
3. 从当前符号中减去估计值

优势：
- 消除后游标 ISI
- 不放大噪声
- 改善眼图

Tap 配置：
DFE-Tap1: 最重要，补偿最近符号
DFE-Tap2-4: 补偿更早的符号
```

## 均衡过程

### Phase 0: 8 GT/s 训练

```
Gen3 训练开始：
1. 在 2.5 GT/s (Gen1) 完成基本训练
2. 速度切换到 8 GT/s
3. 进入 Recovery.RcvrLock
4. 使用默认均衡系数

目标：
建立基本链路，准备均衡
```

### Phase 1: 均衡评估

```
评估阶段：
1. 发送端使用多组预设系数
2. 接收端评估每组系数的效果
3. 记录最佳系数

Preset 预设：
P0-P10: 预定义的系数组合
例如：
P0: C-1=0dB, C0=0dB, C+1=0dB (无预加重)
P4: C-1=-2dB, C0=0dB, C+1=-3.5dB (典型)
P7: C-1=-3dB, C0=0dB, C+1=-6dB (强预加重)

接收端反馈：
- 眼图高度
- 眼图宽度
- 误码率
```

### Phase 2: 系数优化

```
优化阶段：
1. 从 Phase 1 最佳 Preset 开始
2. 接收端请求微调系数
3. 发送端逐步调整
4. 接收端持续评估
5. 达到最优后停止

调整命令：
- Increment/Decrement C-1
- Increment/Decrement C0
- Increment/Decrement C+1

收敛：
接收端发送 "No More Changes" 表示满意
```

### Phase 3: 旁路阶段

```
可选阶段：
允许一端跳过均衡，使用固定系数

用途：
- 短距离链路可能不需要复杂均衡
- 快速训练
```

## 软件配置

### 均衡控制

```c
// 配置链路均衡
static int configure_link_equalization(struct pci_dev *pdev)
{
    int pos;
    u32 link_cap2, link_ctl3;
    u16 link_ctl2, link_sta2;
    
    pos = pci_pcie_cap(pdev);
    if (!pos)
        return -EINVAL;
    
    // 读取 Link Capabilities 2
    pci_read_config_dword(pdev, pos + PCI_EXP_LNKCAP2, &link_cap2);
    
    // 检查是否支持均衡
    if (!(link_cap2 & PCI_EXP_LNKCAP2_SLS_8_0GB)) {
        dev_info(&pdev->dev, "Gen3+ not supported, no equalization\n");
        return 0;
    }
    
    // 配置 Link Control 2
    pci_read_config_word(pdev, pos + PCI_EXP_LNKCTL2, &link_ctl2);
    
    // 设置目标链路速度为 Gen3
    link_ctl2 &= ~PCI_EXP_LNKCTL2_TLS;
    link_ctl2 |= PCI_EXP_LNKCTL2_TLS_8_0GT;
    
    // 启用硬件自主速度禁用
    link_ctl2 |= PCI_EXP_LNKCTL2_HASD;
    
    pci_write_config_word(pdev, pos + PCI_EXP_LNKCTL2, link_ctl2);
    
    // 触发链路均衡
    link_ctl2 |= PCI_EXP_LNKCTL2_EQU_REQ;
    pci_write_config_word(pdev, pos + PCI_EXP_LNKCTL2, link_ctl2);
    
    // 等待均衡完成
    msleep(100);
    
    // 检查均衡状态
    pci_read_config_word(pdev, pos + PCI_EXP_LNKSTA2, &link_sta2);
    
    if (link_sta2 & PCI_EXP_LNKSTA2_EQU_COMP) {
        dev_info(&pdev->dev, "Link equalization completed\n");
    } else {
        dev_warn(&pdev->dev, "Link equalization failed\n");
        return -EIO;
    }
    
    // 读取均衡状态
    if (link_sta2 & PCI_EXP_LNKSTA2_EQU_P1) {
        dev_info(&pdev->dev, "Phase 1 equalization complete\n");
    }
    if (link_sta2 & PCI_EXP_LNKSTA2_EQU_P2) {
        dev_info(&pdev->dev, "Phase 2 equalization complete\n");
    }
    if (link_sta2 & PCI_EXP_LNKSTA2_EQU_P3) {
        dev_info(&pdev->dev, "Phase 3 equalization complete\n");
    }
    
    return 0;
}
```

### Lane Margining

```c
// Lane Margining 测试
static int test_lane_margining(struct pci_dev *pdev, int lane)
{
    int pos;
    u32 margin_ctrl, margin_status;
    
    pos = pci_find_ext_capability(pdev, PCI_EXT_CAP_ID_LMR);
    if (!pos) {
        dev_err(&pdev->dev, "Lane Margining not supported\n");
        return -ENOTSUPP;
    }
    
    // 配置 Margining 控制
    margin_ctrl = (lane << 8) |           // Lane Number
                  (0x1 << 4) |             // Margin Type: Timing
                  (0x1);                   // Start Margining
    
    pci_write_config_dword(pdev, pos + PCI_LMR_CTRL, margin_ctrl);
    
    // 等待测试完成
    msleep(1000);
    
    // 读取结果
    pci_read_config_dword(pdev, pos + PCI_LMR_STATUS, &margin_status);
    
    u8 left_margin = (margin_status >> 0) & 0xFF;
    u8 right_margin = (margin_status >> 8) & 0xFF;
    
    dev_info(&pdev->dev,
             "Lane %d timing margin: left=%d, right=%d\n",
             lane, left_margin, right_margin);
    
    // 测试电压裕度
    margin_ctrl = (lane << 8) |
                  (0x2 << 4) |             // Margin Type: Voltage
                  (0x1);
    
    pci_write_config_dword(pdev, pos + PCI_LMR_CTRL, margin_ctrl);
    msleep(1000);
    
    pci_read_config_dword(pdev, pos + PCI_LMR_STATUS, &margin_status);
    
    u8 up_margin = (margin_status >> 0) & 0xFF;
    u8 down_margin = (margin_status >> 8) & 0xFF;
    
    dev_info(&pdev->dev,
             "Lane %d voltage margin: up=%d, down=%d\n",
             lane, up_margin, down_margin);
    
    return 0;
}
```

## 硬件实现

### 均衡控制器

```verilog
// 链路均衡控制器
module link_equalizer (
    input wire clk,
    input wire rst_n,
    
    // 均衡请求
    input wire eq_start,
    input wire [3:0] preset,
    
    // 系数控制
    output reg signed [5:0] c_minus_1,
    output reg signed [5:0] c_0,
    output reg signed [5:0] c_plus_1,
    
    // 接收端反馈
    input wire [7:0] eye_height,
    input wire [7:0] eye_width,
    input wire [15:0] ber,
    
    // 状态输出
    output reg eq_complete,
    output reg eq_phase
);

    // Preset 系数表
    reg signed [5:0] preset_table [0:10][0:2];
    
    initial begin
        // P0: No pre-emphasis
        preset_table[0][0] = 0;   // C-1
        preset_table[0][1] = 0;   // C0
        preset_table[0][2] = 0;   // C+1
        
        // P4: Moderate pre-emphasis
        preset_table[4][0] = -2;  // C-1
        preset_table[4][1] = 0;   // C0
        preset_table[4][2] = -4;  // C+1
        
        // P7: Strong pre-emphasis
        preset_table[7][0] = -3;
        preset_table[7][1] = 0;
        preset_table[7][2] = -6;
    end
    
    // 均衡状态机
    reg [2:0] state;
    parameter IDLE = 0, PHASE1 = 1, PHASE2 = 2, PHASE3 = 3, DONE = 4;
    
    reg [3:0] best_preset;
    reg [15:0] best_quality;
    
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            state <= IDLE;
            eq_complete <= 1'b0;
            eq_phase <= 2'b00;
        end else begin
            case (state)
                IDLE: begin
                    if (eq_start) begin
                        state <= PHASE1;
                        eq_phase <= 2'b01;
                    end
                end
                
                PHASE1: begin
                    // 评估 Preset
                    c_minus_1 <= preset_table[preset][0];
                    c_0 <= preset_table[preset][1];
                    c_plus_1 <= preset_table[preset][2];
                    
                    // 计算质量指标
                    best_quality <= eye_height * eye_width;
                    
                    // 所有 Preset 评估完成后进入 Phase 2
                    if (preset == 10) begin
                        state <= PHASE2;
                        eq_phase <= 2'b10;
                    end
                end
                
                PHASE2: begin
                    // 微调系数优化
                    // 根据接收端反馈调整
                    // ...
                    
                    // 收敛后完成
                    if (converged) begin
                        state <= DONE;
                        eq_complete <= 1'b1;
                    end
                end
                
                DONE: begin
                    eq_complete <= 1'b1;
                end
            endcase
        end
    end

endmodule
```

## 调试和优化

### 查看均衡状态

```bash
# lspci 显示链路状态
lspci -vvv -s 01:00.0 | grep -i "equalization\|preset"

# 使用 setpci 读取寄存器
setpci -s 01:00.0 CAP_EXP+30.W  # Link Control 2
setpci -s 01:00.0 CAP_EXP+32.W  # Link Status 2
```

### 眼图测试

```bash
# 使用硬件分析仪
# - Tektronix DSA
# - Keysight Infiniium
# - LeCroy LabMaster

# 测量参数：
# - Eye Height (垂直开度)
# - Eye Width (水平开度)
# - Jitter (抖动)
# - BER (误码率)
```

## 最佳实践

1. **使用合适的 Preset**：根据链路长度选择
2. **验证均衡效果**：测量眼图和误码率
3. **考虑温度影响**：温度变化影响均衡
4. **预留裕度**：不要工作在极限状态
5. **定期重训练**：长时间运行后可能需要重新均衡

## 总结

链路均衡的关键点：

1. **必要性**：Gen3+ 高速链路必需
2. **多阶段**：评估、优化、旁路
3. **发送端**：预加重补偿损耗
4. **接收端**：CTLE 和 DFE 均衡
5. **软件控制**：可触发和监控均衡过程

参见：
- [物理层概述](overview.md)
- [链路训练](link-training.md)
- [LTSSM](ltssm.md)
- [编码方案](encoding.md)
