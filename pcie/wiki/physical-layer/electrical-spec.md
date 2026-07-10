# PCIe 电气规范 (Electrical Specification)

## 概述

PCIe 电气规范定义了物理层信号的电气特性，包括电压电平、阻抗、时序、功耗等关键参数。这些规范确保不同厂商的 PCIe 设备能够互操作，并在各种工作条件下保持可靠的通信。

## 差分信号基础

### 差分信号原理

PCIe 使用低压差分信号 (LVDS-like) 传输数据，每个 Lane 包含两对差分线：

```
差分对组成：
- TX+ / TX-：发送差分对
- RX+ / RX-：接收差分对

差分信号优势：
1. 抗共模噪声能力强
2. 电磁辐射低
3. 功耗相对较低
4. 支持高速传输
```

### 差分信号定义

```
差分电压 (VTX-DIFF)：
VTX-DIFF = V(TX+) - V(TX-)

共模电压 (VTX-CM)：
VTX-CM = [V(TX+) + V(TX-)] / 2

单端电压关系：
V(TX+) = VTX-CM + VTX-DIFF/2
V(TX-) = VTX-CM - VTX-DIFF/2
```

## 发送端电气规范

### Gen 1/2/3 发送参数

| 参数 | 最小值 | 典型值 | 最大值 | 单位 | 说明 |
|-----|-------|-------|-------|-----|-----|
| 差分输出电压 (VTX-DIFF-PP) | 800 | 1000 | 1200 | mV | 峰峰值 |
| 共模电压 (VTX-CM) | 0 | 1.8 | 3.6 | V | 直流电平 |
| 差分阻抗 (ZTX-DIFF) | 90 | 100 | 110 | Ω | 特征阻抗 |
| 单端阻抗 (ZTX-SE) | 45 | 50 | 55 | Ω | 对地阻抗 |
| 上升/下降时间 (tr/tf) | - | - | 100 | ps | 20%-80% |
| 去加重 (De-emphasis) | -3.5 | -3.5 | -3.5 | dB | @2.5/5 GHz |

### Gen 4/5 发送参数

| 参数 | 最小值 | 典型值 | 最大值 | 单位 | 说明 |
|-----|-------|-------|-------|-----|-----|
| 差分输出电压 (VTX-DIFF-PP) | 400 | 600 | 800 | mV | 峰峰值（降低） |
| 共模电压 (VTX-CM) | 0 | 1.8 | 3.6 | V | 直流电平 |
| 差分阻抗 (ZTX-DIFF) | 90 | 100 | 110 | Ω | 特征阻抗 |
| TX 均衡系数范围 | -6 | 0 | +6 | - | 3-tap FFE |

### 发送均衡 (TX Equalization)

Gen 3 及更高版本支持发送端均衡以补偿信道损耗：

```c
// 3-tap FFE (Feed-Forward Equalizer)
typedef struct {
    int8_t c_minus_1;  // 前置光标 (pre-cursor)
    int8_t c_0;        // 主光标 (main cursor)
    int8_t c_plus_1;   // 后置光标 (post-cursor)
} tx_ffe_coefficients_t;

// 系数约束
// |c_minus_1| + |c_0| + |c_plus_1| = 63 (归一化)
// c_0 必须是最大值
// c_minus_1, c_plus_1 范围: -6 到 +6

// 示例配置
tx_ffe_coefficients_t presets[] = {
    // P0: 无均衡
    {.c_minus_1 = 0, .c_0 = 63, .c_plus_1 = 0},
    
    // P1: 轻度均衡
    {.c_minus_1 = 0, .c_0 = 59, .c_plus_1 = -4},
    
    // P2: 中度均衡
    {.c_minus_1 = 0, .c_0 = 55, .c_plus_1 = -8},
    
    // P3: 重度均衡
    {.c_minus_1 = 0, .c_0 = 51, .c_plus_1 = -12},
};
```

### 发送抖动规范

```
总抖动 (Total Jitter, TJ)：
Gen 1 (2.5 GT/s):  < 0.30 UI
Gen 2 (5.0 GT/s):  < 0.30 UI
Gen 3 (8.0 GT/s):  < 0.30 UI
Gen 4 (16.0 GT/s): < 0.25 UI
Gen 5 (32.0 GT/s): < 0.20 UI

UI (Unit Interval) = 1 / 数据率
Gen 1: 1 UI = 400 ps
Gen 2: 1 UI = 200 ps
Gen 3: 1 UI = 125 ps
Gen 4: 1 UI = 62.5 ps
Gen 5: 1 UI = 31.25 ps

抖动组成：
TJ = RJ + DJ
- RJ (Random Jitter): 随机抖动
- DJ (Deterministic Jitter): 确定性抖动
  - DCD (Duty Cycle Distortion)
  - ISI (Inter-Symbol Interference)
  - PJ (Periodic Jitter)
```

## 接收端电气规范

### 接收端参数

| 参数 | 最小值 | 典型值 | 最大值 | 单位 | 说明 |
|-----|-------|-------|-------|-----|-----|
| 差分输入电压范围 | 150 | - | 1200 | mV | 峰峰值 |
| 共模电压范围 (VRX-CM) | 0 | - | 3.6 | V | 直流容忍 |
| 差分输入阻抗 | 90 | 100 | 110 | Ω | 端接阻抗 |
| 单端输入阻抗 | 45 | 50 | 55 | Ω | 对地阻抗 |

### 接收端均衡

```
CTLE (Continuous Time Linear Equalizer)：
- 类型：模拟均衡器
- 功能：补偿高频衰减
- 增益范围：0-20 dB @ Nyquist 频率
- 可调参数：零点、极点频率

DFE (Decision Feedback Equalizer)：
- 类型：数字均衡器
- 功能：消除后置符号干扰
- 抽头数量：4-8 tap
- 系数范围：根据具体实现
```

```c
// CTLE 传输函数（简化）
typedef struct {
    float dc_gain;          // 直流增益 (0-20 dB)
    float ac_gain;          // 交流增益 (0-20 dB)
    float zero_freq_ghz;    // 零点频率
    float pole1_freq_ghz;   // 极点1频率
    float pole2_freq_ghz;   // 极点2频率
} ctle_config_t;

// DFE 配置
typedef struct {
    int num_taps;           // 抽头数 (4-8)
    float tap_weights[8];   // 抽头权重
    float adaptation_step;  // 自适应步长
} dfe_config_t;

// 典型 CTLE 配置
ctle_config_t ctle_gen5 = {
    .dc_gain = 0,           // 0 dB
    .ac_gain = 15,          // 15 dB @ 16 GHz
    .zero_freq_ghz = 2.0,
    .pole1_freq_ghz = 8.0,
    .pole2_freq_ghz = 16.0
};
```

### 接收端抖动容限

```
接收端必须容忍的抖动：
Gen 1/2/3: > 0.40 UI
Gen 4:     > 0.35 UI
Gen 5:     > 0.30 UI

测试条件：
- BER (Bit Error Rate) < 10^-12
- 使用 PRBS (Pseudo-Random Bit Sequence) 测试
- PRBS-31 (Gen 1/2/3)
- PRBS-23 (Gen 4/5)
```

## 阻抗规范

### 差分阻抗

```
目标阻抗：100Ω ± 10%
有效范围：90Ω - 110Ω

差分阻抗计算：
Z_diff = 2 × Z_0 × (1 - k)
其中：
- Z_0：单端阻抗
- k：耦合系数

典型 PCB 参数：
Z_0 = 50Ω
k = 0.0 (理想隔离)
Z_diff = 100Ω
```

### 阻抗匹配要求

```
发送端：
- 源阻抗：50Ω (单端)
- 差分阻抗：100Ω
- 匹配容差：± 10%

接收端：
- 端接阻抗：50Ω (单端)
- 差分阻抗：100Ω
- 匹配容差：± 10%

PCB 走线：
- 特征阻抗：50Ω (单端)
- 差分阻抗：100Ω
- 长度容差：± 5 mil (Lane 内)
```

## 功耗规范

### 各代功耗对比

| 代数 | L0 状态 (mW/Lane) | L0s 状态 (mW/Lane) | L1 状态 (mW) | 说明 |
|-----|------------------|-------------------|-------------|-----|
| Gen 1 | 500-800 | 100-200 | < 50 | 完整链路 |
| Gen 2 | 800-1200 | 100-200 | < 50 | 完整链路 |
| Gen 3 | 1000-1500 | 100-200 | < 50 | 完整链路 |
| Gen 4 | 1200-2000 | 100-200 | < 50 | 完整链路 |
| Gen 5 | 1500-2500 | 100-200 | < 50 | 完整链路 |

### 功耗管理状态

```c
// 链路功耗状态定义
typedef enum {
    LINK_L0,    // 全速运行：正常数据传输
    LINK_L0s,   // 待机：TX/RX 关闭，快速恢复 (< 1 μs)
    LINK_L1,    // 睡眠：更深睡眠，中速恢复 (< 10 μs)
    LINK_L1_1,  // L1.1：CLKREQ# 关闭参考时钟
    LINK_L1_2,  // L1.2：共模电压关闭
    LINK_L2,    // 关闭：软件控制恢复
    LINK_L3     // 电源关闭：需要硬件复位
} link_power_state_t;

// 状态功耗（典型 x4 链路）
const float power_consumption_mw[] = {
    [LINK_L0]   = 4000,  // 4W
    [LINK_L0s]  = 800,   // 0.8W
    [LINK_L1]   = 200,   // 0.2W
    [LINK_L1_1] = 50,    // 50mW
    [LINK_L1_2] = 10,    // 10mW
    [LINK_L2]   = 5,     // 5mW
    [LINK_L3]   = 0      // 0W
};
```

### 插槽功率规范

```
PCIe 插槽供电：
- +12V 主供电
- +3.3V 辅助供电
- +3.3Vaux 待机供电

标准插槽功率：
x1  插槽：10W (+12V: 0.55A, +3.3V: 3A)
x4  插槽：25W (+12V: 2.1A, +3.3V: 3A)
x8  插槽：25W (+12V: 2.1A, +3.3V: 3A)
x16 插槽：75W (+12V: 5.5A, +3.3V: 3A)

高功率配置（需额外供电）：
x16 插槽 + 6-pin PCIe：150W
x16 插槽 + 8-pin PCIe：225W
x16 插槽 + 2×8-pin PCIe：375W
```

## 时序规范

### 参考时钟

```
参考时钟规范：
频率：100 MHz ± 300 ppm
电平：HCSL (High-Speed Current Steering Logic)
摆幅：250 mV - 700 mV (单端)
占空比：45% - 55%
抖动：< 2 ps RMS

时钟架构：
1. 公共时钟架构 (Common Clock)
   - RC 和端点共享参考时钟
   - 更好的抖动性能
   
2. 独立时钟架构 (Separate Clock)
   - RC 和端点使用独立时钟
   - 更灵活的布局
   - 需要 SSC (Spread Spectrum Clocking)
```

### SSC (Spread Spectrum Clocking)

```
SSC 参数：
调制类型：向下展频 (Down-spread)
调制深度：-0.5% (Gen 1/2/3)
           -0.5% (Gen 4/5, 推荐)
调制频率：30 - 33 kHz
调制波形：三角波

目的：
- 降低 EMI 辐射
- 将能量分散到更宽频带
- 符合 FCC 和其他 EMI 标准
```

### 链路训练时序

```
训练序列间隔：
TS1/TS2 间隔：1024 符号 (Gen 1/2)
              1024 符号 (Gen 3+)

超时时间：
- Polling.Active: 24 ms
- Configuration: 48 ms
- Recovery: 48 ms

链路初始化总时间：
Gen 1/2: < 100 ms (典型)
Gen 3:   < 120 ms (典型)
Gen 4/5: < 150 ms (典型，包含均衡）
```

## 信号质量要求

### 眼图规范

```
眼高 (Eye Height)：
Gen 1/2: > 175 mV (差分)
Gen 3:   > 150 mV (差分)
Gen 4:   > 100 mV (差分)
Gen 5:   > 80 mV (差分)

眼宽 (Eye Width)：
所有代数: > 0.6 UI @ BER = 10^-12

测量条件：
- 在接收端测量点
- 包含均衡效果
- 使用 PRBS 测试图案
```

### 信道损耗预算

```
最大允许信道损耗 (@ Nyquist 频率)：
Gen 1 (1.25 GHz):  -6 dB
Gen 2 (2.5 GHz):   -8 dB
Gen 3 (4.0 GHz):   -10 dB
Gen 4 (8.0 GHz):   -12 dB
Gen 5 (16.0 GHz):  -16 dB

损耗来源：
1. PCB 走线损耗：0.5-6 dB/cm (频率相关)
2. 连接器损耗：0.5-1.5 dB (每个)
3. 过孔损耗：0.1-0.3 dB (每个)
4. 阻抗不匹配：0.5-2 dB
```

### BER (误码率) 要求

```
链路 BER 要求：
操作 BER: < 10^-12
目标 BER: < 10^-15 (推荐)

测试时长：
10^-12 BER 验证：
- 需要传输 > 10^13 位
- Gen 5 (32 GT/s)：> 5 分钟
- Gen 3 (8 GT/s)：> 20 分钟
```

## 测试与合规

### 电气测试项目

```
发送端测试：
1. 差分输出电压
2. 共模电压
3. 上升/下降时间
4. 眼图（高度、宽度）
5. 抖动（TJ, RJ, DJ）
6. 均衡预设（Gen 3+）
7. 时钟频率和 SSC

接收端测试：
1. 输入阻抗
2. 抖动容限
3. 均衡器性能
4. BER 测试
5. 灵敏度测试
```

### 合规测试流程

```c
// 简化的合规测试流程
typedef enum {
    TEST_TX_VOLTAGE,        // 发送电压测试
    TEST_TX_JITTER,         // 发送抖动测试
    TEST_TX_EYE_DIAGRAM,    // 发送眼图测试
    TEST_RX_IMPEDANCE,      // 接收阻抗测试
    TEST_RX_JITTER_TOL,     // 接收抖动容限
    TEST_RX_SENSITIVITY,    // 接收灵敏度
    TEST_LINK_TRAINING,     // 链路训练测试
    TEST_BER,               // 误码率测试
    TEST_INTEROP            // 互操作性测试
} compliance_test_t;

// 测试设备
typedef struct {
    char oscilloscope[64];      // "Tektronix MSO/DPO70000"
    char bert[64];              // "Anritsu MP1800A"
    char signal_gen[64];        // "Keysight M8000A"
    char spectrum_analyzer[64]; // "R&S FSW"
} test_equipment_t;

// 执行合规测试
bool run_compliance_test(int pcie_gen) {
    bool all_passed = true;
    
    printf("开始 PCIe Gen%d 电气合规测试\\n", pcie_gen);
    
    // 1. TX 测试
    all_passed &= test_tx_voltage(pcie_gen);
    all_passed &= test_tx_jitter(pcie_gen);
    all_passed &= test_tx_eye_diagram(pcie_gen);
    
    // 2. RX 测试
    all_passed &= test_rx_impedance(pcie_gen);
    all_passed &= test_rx_jitter_tolerance(pcie_gen);
    all_passed &= test_rx_sensitivity(pcie_gen);
    
    // 3. 系统测试
    all_passed &= test_link_training(pcie_gen);
    all_passed &= test_ber(pcie_gen);
    all_passed &= test_interoperability(pcie_gen);
    
    return all_passed;
}
```

### PCI-SIG 认证

```
认证类型：
1. Integrators List
   - 完整系统认证
   - 需要通过所有电气和协议测试
   
2. Compliance Workshop
   - 参加 PCI-SIG 官方测试活动
   - 与其他厂商设备互操作测试
   - 获得认证证书

认证级别：
- Component Level: 单个芯片
- Card Level: 扩展卡
- System Level: 完整系统
```

## PCB 设计指南

### 走线要求

```
差分走线：
1. 差分阻抗：100Ω ± 10%
2. 单端阻抗：50Ω ± 10%
3. 走线间距：保持恒定
4. 走线宽度：保持恒定
5. 参考层：连续完整

长度匹配：
- Lane 内匹配：± 5 mil (0.127 mm)
- Lane 间匹配：± 20 mil (0.508 mm)
- 换算：1 mil ≈ 6 ps 延迟 (FR4)

过孔设计：
- 最小化过孔数量
- 使用背钻去除残桩
- Via stub < 10 mil
- GND via 靠近信号 via
```

### 布局建议

```
推荐布局：
1. 信号层配置
   L1: 信号层 (Top)
   L2: GND 平面
   L3: 电源平面 (3.3V/1.8V)
   L4: GND 平面
   L5: 信号层
   L6: GND 平面
   L7: 电源平面 (12V)
   L8: 信号层 (Bottom)

2. 走线规则
   - TX/RX 对分开布线
   - 避免 90° 转角（使用圆角）
   - 远离时钟和电源噪声源
   - 包地处理（GND stitching）

3. 元件放置
   - AC 耦合电容靠近接收端
   - 端接电阻靠近接收端
   - 去耦电容靠近电源引脚
```

### 材料选择

```
PCB 基材：
- FR-4：Gen 1/2 适用
- 高速 FR-4：Gen 3 适用
- Low-Loss 材料：Gen 4/5 推荐
  * Rogers RO4350B
  * Panasonic Megtron 6
  * Isola I-Speed

材料参数：
- 介电常数 (Dk)：3.5-4.5 @ 10 GHz
- 损耗角正切 (Df)：< 0.01 @ 10 GHz
- 铜箔粗糙度：< 3 μm (低粗糙度)
```

## 调试技巧

### 常见电气问题

```python
# 电气问题诊断流程
def diagnose_electrical_issue(symptom):
    """
    诊断 PCIe 电气问题
    """
    issues = {
        'no_link': [
            '检查电源电压是否正常',
            '检查参考时钟是否存在',
            '检查 PERST# 复位信号',
            '验证差分对极性是否正确'
        ],
        'link_unstable': [
            '测量信号眼图质量',
            '检查抖动是否超标',
            '验证阻抗匹配',
            '检查 EMI 干扰'
        ],
        'ber_high': [
            '调整 TX 均衡参数',
            '调整 RX 均衡参数',
            '检查走线长度匹配',
            '验证信道插入损耗'
        ],
        'link_downgrade': [
            '检查信道损耗',
            '验证均衡器配置',
            '检查参考时钟抖动',
            '验证链路训练过程'
        ]
    }
    
    return issues.get(symptom, ['未知症状'])

# 示例使用
print("链路不稳定诊断步骤：")
for step in diagnose_electrical_issue('link_unstable'):
    print(f"- {step}")
```

### 测量技巧

```
眼图测量：
1. 使用高带宽示波器（> 5× 数据率）
2. 使用差分探头或 SMA 接口
3. 测量点：接收端输入（去除均衡效果）
4. 触发：使用时钟恢复触发
5. 采样：收集足够样本（> 10^6）

抖动测量：
1. 使用抖动分析功能
2. 分离 RJ 和 DJ 成分
3. 使用 PRBS 测试图案
4. 长时间测量（> 1 分钟）

阻抗测量：
1. 使用 TDR (Time Domain Reflectometry)
2. 测量频段：DC - 20 GHz
3. 校准：短路、开路、负载
4. 分析：反射系数、阻抗曲线
```

## 总结

PCIe 电气规范涵盖了从信号电压、阻抗、时序到功耗的各个方面：

**关键电气参数**：
- 差分信号：100Ω 阻抗，低压差分传输
- 发送电压：800-1200 mV (Gen 1-3)，400-800 mV (Gen 4-5)
- 抖动要求：< 0.30 UI (TX)，> 0.40 UI 容限 (RX)
- BER 目标：< 10^-12

**均衡技术**：
- 发送端：3-tap FFE (Gen 3+)
- 接收端：CTLE + DFE (Gen 3+)
- 自适应均衡：链路训练阶段完成

**功耗管理**：
- L0：全速运行 (1-2.5W/Lane)
- L0s/L1：快速省电 (100-200 mW)
- L1.1/L1.2：深度省电 (< 50 mW)

**设计考虑**：
- PCB 阻抗控制和长度匹配
- 信号完整性和 EMI 控制
- 适当的均衡和补偿
- 充分的测试和验证

理解这些电气规范对于设计高质量、可靠的 PCIe 系统至关重要。随着速率的提升，电气设计的挑战也在增加，需要更先进的信号处理技术和更精细的 PCB 设计。
