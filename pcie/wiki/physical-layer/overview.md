# 物理层概述 (Physical Layer Overview)

## 概述

PCIe 物理层（Physical Layer）是 PCIe 协议栈的最底层，负责在物理介质上传输数据。物理层分为两个主要子块：逻辑子块（Logical Sub-block）和电气子块（Electrical Sub-block），它们协同工作以实现高速、可靠的点对点串行通信。

## 物理层架构

### 整体结构

```
+----------------------------------------------------------+
|                    事务层 (Transaction Layer)             |
+----------------------------------------------------------+
                           ↕
+----------------------------------------------------------+
|                  数据链路层 (Data Link Layer)             |
+----------------------------------------------------------+
                           ↕
+----------------------------------------------------------+
|                   物理层 (Physical Layer)                 |
|  +----------------------------------------------------+  |
|  |         逻辑子块 (Logical Sub-block)               |  |
|  |  - 8b/10b 或 128b/130b 编码/解码                  |  |
|  |  - 扰码/解扰                                      |  |
|  |  - Lane 聚合/分散                                 |  |
|  |  - 链路训练状态机 (LTSSM)                         |  |
|  +----------------------------------------------------+  |
|                           ↕                              |
|  +----------------------------------------------------+  |
|  |         电气子块 (Electrical Sub-block)            |  |
|  |  - 串行器/解串器 (SerDes)                         |  |
|  |  - 发送器 (TX Driver)                             |  |
|  |  - 接收器 (RX Equalizer)                          |  |
|  |  - 时钟恢复 (CDR)                                 |  |
|  +----------------------------------------------------+  |
+----------------------------------------------------------+
                           ↕
              物理介质 (PCB 走线或电缆)
```

### 主要功能

物理层负责：

1. **数据编码/解码**：将数据转换为适合传输的格式
2. **差分信号传输**：在物理 Lane 上传输串行数据
3. **时钟恢复**：从接收的数据流中提取时钟
4. **链路训练**：初始化和维护链路状态
5. **速度和宽度协商**：确定最优的工作参数
6. **均衡和补偿**：克服信号完整性问题

## 逻辑子块详解

### 编码机制

PCIe 使用不同的编码方案来保证时钟恢复和直流平衡：

#### 8b/10b 编码 (Gen 1/2)

**原理**：将 8 位数据编码为 10 位符号，增加 25% 开销

```
8b/10b 编码特性：
- 保证足够的跳变密度（时钟恢复需要）
- 维持直流平衡（Running Disparity）
- 提供特殊控制字符（K-code）
- 错误检测能力

编码效率：8/10 = 80%
```

**编码示例**：

```c
// 8b/10b 编码表（部分）
typedef struct {
    uint8_t  data_byte;     // 输入 8 位数据
    uint16_t encoded_10b;   // 输出 10 位符号
    bool     is_k_code;     // 是否为控制字符
} encoding_table_entry_t;

// 常用的 K-code 符号
#define K28_5  0xBC  // COM 符号（训练序列）
#define K27_7  0xFB  // STP 符号（帧起始）
#define K29_7  0xFD  // SDP 符号（数据包起始）
#define K30_7  0xFE  // END 符号（帧结束）
#define K23_7  0xF7  // PAD 符号（填充）

// 编码函数（简化）
uint16_t encode_8b10b(uint8_t data, bool is_k_code, bool running_disparity) {
    // 查表或使用编码逻辑
    // 根据当前 running disparity 选择正/负编码
    if (is_k_code) {
        return lookup_k_code_table(data, running_disparity);
    } else {
        return lookup_d_code_table(data, running_disparity);
    }
}
```

**Gen 1/2 数据率计算**：

```
Gen 1: 2.5 GT/s × 8/10 = 2.0 Gbps (每 Lane)
Gen 2: 5.0 GT/s × 8/10 = 4.0 Gbps (每 Lane)
```

#### 128b/130b 编码 (Gen 3+)

**原理**：将 128 位数据编码为 130 位，仅增加 1.5% 开销

```
128b/130b 编码特性：
- 更高的编码效率（98.46%）
- 2 位同步头（00b, 01b, 10b, 11b）
- 扰码器保证跳变密度
- 支持更高速率

编码效率：128/130 ≈ 98.46%
```

**编码格式**：

```
130 位块结构：
+--------+---------------------------+
| Sync   | Payload (128 bits)        |
| Header |                           |
| (2bit) |                           |
+--------+---------------------------+

Sync Header 类型：
- 00b: 全部为数据块
- 01b: 数据块 + Ordered Set 起始
- 10b: Ordered Set 结束 + 数据块
- 11b: 保留

扰码多项式：
G(x) = x^23 + x^21 + x^16 + x^8 + x^5 + x^2 + 1
```

**Gen 3+ 数据率计算**：

```
Gen 3:  8.0 GT/s × 128/130 ≈ 7.88 Gbps (每 Lane)
Gen 4: 16.0 GT/s × 128/130 ≈ 15.75 Gbps (每 Lane)
Gen 5: 32.0 GT/s × 128/130 ≈ 31.51 Gbps (每 Lane)
Gen 6: 64.0 GT/s × 128/130 ≈ 63.01 Gbps (每 Lane)
```

### 扰码器 (Scrambler)

扰码器用于打散数据模式，提高信号质量：

```c
// LFSR (Linear Feedback Shift Register) 扰码器
#define SCRAMBLER_POLYNOMIAL 0xA9  // 示例多项式

typedef struct {
    uint32_t lfsr_state;  // LFSR 状态
    uint32_t seed;        // 初始种子
} scrambler_t;

// 初始化扰码器
void scrambler_init(scrambler_t* scr, uint32_t seed) {
    scr->seed = seed;
    scr->lfsr_state = seed;
}

// 扰码一个字节
uint8_t scramble_byte(scrambler_t* scr, uint8_t data) {
    uint8_t scrambled = 0;
    
    for (int i = 0; i < 8; i++) {
        // 从 LFSR 生成伪随机位
        bool feedback = ((scr->lfsr_state >> 22) & 1) ^ 
                       ((scr->lfsr_state >> 21) & 1);
        
        scr->lfsr_state = (scr->lfsr_state << 1) | feedback;
        
        // XOR 数据位和 LFSR 输出
        bool data_bit = (data >> i) & 1;
        bool scrambled_bit = data_bit ^ (scr->lfsr_state & 1);
        scrambled |= (scrambled_bit << i);
    }
    
    return scrambled;
}

// 解扰（使用相同的 LFSR 序列）
uint8_t descramble_byte(scrambler_t* scr, uint8_t data) {
    // 解扰与扰码使用相同的操作（XOR 的性质）
    return scramble_byte(scr, data);
}
```

### Lane 聚合与分散

多 Lane 链路需要正确地分散和聚合数据：

```
发送端 Lane 分散：
+------------------+
| TLP / DLLP Data  |
+------------------+
        ↓
   [条带化]
        ↓
   +----+----+----+----+
   | L0 | L1 | L2 | L3 | (x4 链路示例)
   +----+----+----+----+
        ↓
    物理 Lane

接收端 Lane 聚合：
   +----+----+----+----+
   | L0 | L1 | L2 | L3 |
   +----+----+----+----+
        ↓
   [去条带化]
        ↓
   [Lane 对齐]
        ↓
+------------------+
| TLP / DLLP Data  |
+------------------+
```

**Lane 对齐**使用 SKP Ordered Set：

```c
// SKP 符号用于补偿时钟偏差和 Lane 对齐
#define SKP_INTERVAL_SYMBOLS  1536  // Gen 1/2
#define SKP_INTERVAL_BLOCKS   370   // Gen 3+ (128b/130b blocks)

// Lane 去偏斜缓冲区
typedef struct {
    uint8_t buffer[16][256];  // 最多 16 Lane，每个 256 字节缓冲
    uint16_t write_ptr[16];
    uint16_t read_ptr[16];
    bool aligned[16];
} lane_deskew_buffer_t;

// Lane 对齐过程
void align_lanes(lane_deskew_buffer_t* deskew, int num_lanes) {
    bool all_aligned = false;
    
    while (!all_aligned) {
        all_aligned = true;
        
        // 等待所有 Lane 接收到 SKP Ordered Set
        for (int i = 0; i < num_lanes; i++) {
            if (!detect_skp_ordered_set(i)) {
                all_aligned = false;
            }
        }
    }
    
    // 从 SKP 边界开始同步读取所有 Lane
    for (int i = 0; i < num_lanes; i++) {
        deskew->read_ptr[i] = find_skp_boundary(i);
        deskew->aligned[i] = true;
    }
}
```

## 电气子块详解

### 差分信号传输

PCIe 使用差分信号对（TX+/TX-, RX+/RX-）来传输数据：

```
差分信号特性：
- 每个 Lane 由 2 对差分信号组成（TX 和 RX）
- 全双工通信（同时发送和接收）
- 差分电压：400-1200 mV (取决于代数)
- 共模电压：0-3.6 V
- 阻抗：100Ω ± 10% (差分)
```

**物理连接**：

```
设备 A                        设备 B
+-------+                    +-------+
| TX+   |-------- Lane 0 ------->| RX+   |
| TX-   |------  (正向)   ----->| RX-   |
|       |                    |       |
| RX+   |<------- Lane 0 ------| TX+   |
| RX-   |<-----  (反向)   -----| TX-   |
+-------+                    +-------+

每个 Lane = 4 根信号线（2 对差分）
x4 链路 = 16 根信号线
x16 链路 = 64 根信号线
```

### Lane 结构

```c
// Lane 配置
typedef enum {
    LANE_WIDTH_x1  = 1,
    LANE_WIDTH_x2  = 2,
    LANE_WIDTH_x4  = 4,
    LANE_WIDTH_x8  = 8,
    LANE_WIDTH_x16 = 16,
    LANE_WIDTH_x32 = 32  // PCIe 6.0
} pcie_lane_width_t;

// Lane 电气参数
typedef struct {
    float tx_voltage_swing_mv;      // 发送电压摆幅
    float tx_de_emphasis_db;        // 发送去加重
    float rx_termination_ohm;       // 接收端阻抗
    float differential_impedance;   // 差分阻抗
    int   data_rate_gtps;          // 数据率 (GT/s)
} lane_electrical_params_t;

// 不同代数的电气参数
const lane_electrical_params_t gen_params[] = {
    // Gen 1
    { .tx_voltage_swing_mv = 800, .tx_de_emphasis_db = -3.5, 
      .rx_termination_ohm = 50, .differential_impedance = 100, 
      .data_rate_gtps = 2.5 },
    
    // Gen 2
    { .tx_voltage_swing_mv = 800, .tx_de_emphasis_db = -3.5,
      .rx_termination_ohm = 50, .differential_impedance = 100,
      .data_rate_gtps = 5.0 },
    
    // Gen 3
    { .tx_voltage_swing_mv = 800, .tx_de_emphasis_db = -3.5,
      .rx_termination_ohm = 50, .differential_impedance = 100,
      .data_rate_gtps = 8.0 },
    
    // Gen 4
    { .tx_voltage_swing_mv = 600, .tx_de_emphasis_db = 0,  // 使用均衡器
      .rx_termination_ohm = 50, .differential_impedance = 100,
      .data_rate_gtps = 16.0 },
    
    // Gen 5
    { .tx_voltage_swing_mv = 500, .tx_de_emphasis_db = 0,
      .rx_termination_ohm = 50, .differential_impedance = 100,
      .data_rate_gtps = 32.0 }
};
```

### SerDes (串行器/解串器)

SerDes 负责并行数据和串行数据之间的转换：

```
发送端 SerDes：
+----------------+       +------------+       +----------+
| Parallel Data  | ----> | Serializer | ----> | TX Driver|
| (多字节宽)      |       | (PISO)     |       | (差分)    |
+----------------+       +------------+       +----------+
    32/64/128 bit           1 bit              Analog

接收端 SerDes：
+----------+       +--------------+       +----------------+
| RX Front | ----> | Deserializer | ----> | Parallel Data  |
| End      |       | (SIPO)       |       | (多字节宽)      |
+----------+       +--------------+       +----------------+
  Analog              1 bit                 32/64/128 bit
```

**SerDes 时钟域**：

```
发送端：
- 参考时钟：100 MHz (通用)
- PLL 倍频：100 MHz × N = 串行时钟
  * Gen 1: 100 MHz × 25 = 2.5 GHz
  * Gen 3: 100 MHz × 80 = 8.0 GHz
  * Gen 5: 100 MHz × 320 = 32.0 GHz

接收端：
- CDR (Clock Data Recovery) 从数据中恢复时钟
- PLL 锁定到接收时钟
- 并行时钟 = 串行时钟 / 宽度
```

### 时钟数据恢复 (CDR)

CDR 电路从接收的串行数据流中提取时钟：

```c
// CDR 原理（简化）
typedef struct {
    float vco_frequency;      // VCO (压控振荡器) 频率
    float phase_error;        // 相位误差
    bool  locked;             // PLL 锁定状态
    int   lock_counter;       // 锁定计数器
} cdr_state_t;

// CDR 锁定过程
void cdr_process(cdr_state_t* cdr, bool data_edge) {
    if (data_edge) {
        // 检测数据跳变
        cdr->phase_error = measure_phase_error();
        
        // 调整 VCO 频率
        adjust_vco_frequency(cdr->vco_frequency, cdr->phase_error);
        
        // 检查锁定状态
        if (fabs(cdr->phase_error) < LOCK_THRESHOLD) {
            cdr->lock_counter++;
            if (cdr->lock_counter > LOCK_COUNT_THRESHOLD) {
                cdr->locked = true;
            }
        } else {
            cdr->lock_counter = 0;
            cdr->locked = false;
        }
    }
}
```

**CDR 要求的跳变密度**：

```
8b/10b 编码：保证最多连续 5 个相同的位
128b/130b + 扰码：保证足够的跳变密度

没有跳变 → CDR 失锁 → 接收错误
```

### 发送器 (TX Driver)

发送器将数字信号转换为模拟差分信号：

```
TX Driver 组成：
+----------------+
| Pre-emphasis   | ← 补偿高频衰减
+----------------+
        ↓
+----------------+
| De-emphasis    | ← 减少 ISI（符号间干扰）
+----------------+
        ↓
+----------------+
| Differential   |
| Driver         | ← 差分输出级
+----------------+
        ↓
      TX+/TX-
```

**去加重 (De-emphasis)**：

```c
// 去加重系数
typedef enum {
    DE_EMPHASIS_0_0_DB  = 0,   // Gen 4+ (使用均衡器代替)
    DE_EMPHASIS_3_5_DB  = 1,   // Gen 1/2/3 常用
    DE_EMPHASIS_6_0_DB  = 2    // 长距离链路
} de_emphasis_level_t;

// 发送均衡（Gen 3+）
typedef struct {
    int8_t pre_cursor;      // 前置光标系数 (-6 到 +6)
    int8_t cursor;          // 主光标系数
    int8_t post_cursor;     // 后置光标系数 (-6 到 +6)
} tx_equalization_t;

// 3-tap 均衡器输出
float tx_equalize(tx_equalization_t* eq, float prev, float curr, float next) {
    return eq->pre_cursor * prev + 
           eq->cursor * curr + 
           eq->post_cursor * next;
}
```

### 接收器均衡器 (RX Equalizer)

接收器使用均衡器补偿信道损耗：

```
RX Equalizer 类型：

1. CTLE (Continuous Time Linear Equalizer)
   - 模拟均衡器
   - 补偿高频衰减
   - 低功耗

2. DFE (Decision Feedback Equalizer)
   - 数字均衡器
   - 消除 ISI
   - 适应性强

3. FFE (Feed-Forward Equalizer)
   - 线性均衡器
   - 预测性补偿
```

```c
// CTLE 参数
typedef struct {
    float dc_gain_db;           // 直流增益
    float high_freq_gain_db;    // 高频增益
    float zero_freq_ghz;        // 零点频率
    float pole_freq_ghz;        // 极点频率
} ctle_params_t;

// DFE 参数
typedef struct {
    int8_t tap_coefficients[8];  // 8 个抽头系数
    int    num_taps;              // 使用的抽头数量
} dfe_params_t;

// 自适应均衡
void adaptive_equalization(ctle_params_t* ctle, dfe_params_t* dfe) {
    // 1. 测量眼图质量
    float eye_height = measure_eye_height();
    float eye_width = measure_eye_width();
    
    // 2. 调整 CTLE
    if (eye_height < TARGET_EYE_HEIGHT) {
        ctle->high_freq_gain_db += 0.5;
    }
    
    // 3. 调整 DFE 系数（LMS 算法）
    for (int i = 0; i < dfe->num_taps; i++) {
        dfe->tap_coefficients[i] = calculate_lms_coefficient(i);
    }
}
```

## 性能特性

### 各代 PCIe 对比

| 代数 | 编码 | 速率 (GT/s) | 带宽/Lane | 带宽 x16 | 最大链路长度 |
|-----|------|-----------|----------|---------|-------------|
| Gen 1 | 8b/10b | 2.5 | 250 MB/s | 4 GB/s | ~50 cm |
| Gen 2 | 8b/10b | 5.0 | 500 MB/s | 8 GB/s | ~50 cm |
| Gen 3 | 128b/130b | 8.0 | ~985 MB/s | ~16 GB/s | ~40 cm |
| Gen 4 | 128b/130b | 16.0 | ~1.97 GB/s | ~32 GB/s | ~30 cm |
| Gen 5 | 128b/130b | 32.0 | ~3.94 GB/s | ~63 GB/s | ~20 cm |
| Gen 6 | FLIT (256B) | 64.0 | ~7.88 GB/s | ~126 GB/s | ~15 cm |

### 信号完整性挑战

随着速率提高，面临的挑战：

```
挑战因素：
1. 插入损耗 (Insertion Loss)：信号能量衰减
2. 回波损耗 (Return Loss)：阻抗不匹配导致反射
3. 串扰 (Crosstalk)：相邻信号线之间的耦合
4. 抖动 (Jitter)：时钟和数据的时序偏差
5. ISI (Inter-Symbol Interference)：符号间干扰

应对措施：
- Gen 1/2: 简单去加重
- Gen 3: 发送均衡 + 接收 CTLE
- Gen 4/5: 全面均衡（TX FIR + RX CTLE + DFE）
- Gen 6: PAM4 调制 + 先进 DSP
```

## 功耗管理

物理层支持多种低功耗状态：

```c
// 链路电源状态
typedef enum {
    L0_STATE,      // 全速运行
    L0s_STATE,     // 待机（快速恢复）
    L1_STATE,      // 低功耗（中速恢复）
    L1_1_STATE,    // L1 子状态（更低功耗）
    L1_2_STATE,    // L1 子状态（最低功耗）
    L2_STATE,      // 关闭（需要软件唤醒）
    L3_STATE       // 电源关闭
} link_power_state_t;

// 状态转换延迟
const uint32_t exit_latency_us[] = {
    [L0_STATE]   = 0,
    [L0s_STATE]  = 1,      // < 1 μs
    [L1_STATE]   = 10,     // < 10 μs
    [L1_1_STATE] = 20,     // < 20 μs
    [L1_2_STATE] = 100,    // < 100 μs
    [L2_STATE]   = 1000,   // < 1 ms
    [L3_STATE]   = 10000   // > 10 ms
};
```

## 调试接口

### 物理层测试点

```
常见测试点：
1. TX 眼图测试
2. RX 灵敏度测试
3. 抖动容限测试
4. BER (Bit Error Rate) 测试
5. 均衡器性能测试
```

```bash
# Linux 下查看物理层状态
lspci -vvv -s 01:00.0 | grep -E "LnkCap|LnkSta"

# 输出示例：
# LnkCap: Port #0, Speed 16GT/s, Width x16
# LnkSta: Speed 16GT/s (ok), Width x16 (ok)

# 查看链路训练错误
cat /sys/bus/pci/devices/0000:01:00.0/link_training_errors
```

## 最佳实践

### PCB 设计建议

1. **阻抗控制**：保持 100Ω ± 10% 差分阻抗
2. **等长设计**：Lane 内和 Lane 间长度匹配
3. **参考平面**：完整的接地/电源平面
4. **过孔优化**：最小化过孔数量和残桩
5. **EMI 控制**：适当的滤波和屏蔽

### 信号完整性优化

```python
# 链路预算分析
def link_budget_analysis(gen, length_cm, connector_count):
    """
    gen: PCIe 代数 (1-5)
    length_cm: 走线长度（厘米）
    connector_count: 连接器数量
    """
    # 损耗参数（简化）
    loss_per_cm = {1: 0.5, 2: 1.0, 3: 2.0, 4: 3.5, 5: 6.0}  # dB/cm
    connector_loss = 0.5  # dB per connector
    
    total_loss = length_cm * loss_per_cm[gen] + connector_count * connector_loss
    
    # TX 输出功率（典型）
    tx_power_dbm = {1: 0, 2: 0, 3: -1, 4: -2, 5: -3}
    
    # RX 灵敏度（典型）
    rx_sensitivity_dbm = {1: -10, 2: -8, 3: -7, 4: -6, 5: -5}
    
    # 计算余量
    margin = tx_power_dbm[gen] - total_loss - rx_sensitivity_dbm[gen]
    
    return {
        'total_loss_db': total_loss,
        'margin_db': margin,
        'pass': margin > 3  # 建议至少 3 dB 余量
    }

# 示例
result = link_budget_analysis(gen=4, length_cm=30, connector_count=2)
print(f"总损耗: {result['total_loss_db']} dB")
print(f"余量: {result['margin_db']} dB")
print(f"通过: {result['pass']}")
```

## 总结

PCIe 物理层是实现高速串行通信的基础，分为逻辑子块和电气子块：

**逻辑子块**：
- 编码/解码（8b/10b 或 128b/130b）
- 扰码/解扰
- Lane 聚合/分散
- 链路训练状态机

**电气子块**：
- SerDes（串行器/解串器）
- 差分信号传输
- 时钟恢复
- 均衡和补偿

随着 PCIe 代数的演进，物理层技术不断进步以支持更高的数据率，同时也带来了更严格的信号完整性要求。理解物理层的工作原理对于 PCIe 系统的设计、调试和优化至关重要。
