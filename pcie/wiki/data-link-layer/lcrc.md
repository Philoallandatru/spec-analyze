# LCRC - 链路层 CRC

## 概述

LCRC（Link CRC）是 PCIe 数据链路层用于保证数据完整性的关键机制。它使用 CRC-32 算法对每个 TLP（Transaction Layer Packet）进行校验，能够检测传输过程中发生的数据损坏。LCRC 与重试缓冲区机制配合，构成了 PCIe 端到端可靠传输的基础。

## CRC-32 算法详解

### 算法基础

LCRC 使用标准的 CRC-32 算法，与以太网等协议使用的算法相同：

**多项式**：
```
G(x) = x^32 + x^26 + x^23 + x^22 + x^16 + x^12 + x^11 + x^10 + x^8 + x^7 + x^5 + x^4 + x^2 + x + 1
```

**十六进制表示**：
```
0x104C11DB7 (33 位，包含隐含的最高位)
0x04C11DB7  (32 位，常用表示)
```

### CRC 计算流程

```
输入数据流 → 按位处理 → 多项式除法 → 余数即为 CRC 值
```

**伪代码实现**：

```c
#define CRC32_POLYNOMIAL 0x04C11DB7
#define CRC32_INIT_VALUE 0xFFFFFFFF

// 位级 CRC-32 计算（理论实现）
uint32_t crc32_bit_level(const uint8_t* data, size_t length) {
    uint32_t crc = CRC32_INIT_VALUE;
    
    for (size_t i = 0; i < length; i++) {
        uint8_t byte = data[i];
        for (int bit = 0; bit < 8; bit++) {
            bool msb = (crc & 0x80000000) != 0;
            crc <<= 1;
            if ((byte & (0x80 >> bit)) != 0) {
                crc |= 1;
            }
            if (msb) {
                crc ^= CRC32_POLYNOMIAL;
            }
        }
    }
    
    return crc;
}
```

### 查表法优化

硬件和高性能软件实现通常使用查表法加速计算：

```c
// 预计算的 CRC 查找表
static uint32_t crc32_table[256];

// 初始化 CRC 查找表
void init_crc32_table(void) {
    for (uint32_t i = 0; i < 256; i++) {
        uint32_t crc = i << 24;
        for (int j = 0; j < 8; j++) {
            if (crc & 0x80000000) {
                crc = (crc << 1) ^ CRC32_POLYNOMIAL;
            } else {
                crc <<= 1;
            }
        }
        crc32_table[i] = crc;
    }
}

// 字节级 CRC-32 计算（查表实现）
uint32_t crc32_table_lookup(const uint8_t* data, size_t length) {
    uint32_t crc = CRC32_INIT_VALUE;
    
    for (size_t i = 0; i < length; i++) {
        uint8_t index = (crc >> 24) ^ data[i];
        crc = (crc << 8) ^ crc32_table[index];
    }
    
    return crc;
}

// 32 位并行 CRC 计算（高性能实现）
uint32_t crc32_fast(const uint8_t* data, size_t length) {
    uint32_t crc = CRC32_INIT_VALUE;
    
    // 按 4 字节对齐处理
    while (length >= 4 && ((uintptr_t)data & 3) == 0) {
        uint32_t word = *(uint32_t*)data;
        crc ^= word;
        crc = crc32_table[(crc >> 24) & 0xFF] ^
              crc32_table[(crc >> 16) & 0xFF] ^
              crc32_table[(crc >> 8) & 0xFF] ^
              crc32_table[crc & 0xFF];
        data += 4;
        length -= 4;
    }
    
    // 处理剩余字节
    while (length--) {
        uint8_t index = (crc >> 24) ^ *data++;
        crc = (crc << 8) ^ crc32_table[index];
    }
    
    return crc;
}
```

## LCRC 计算范围

### TLP 结构与 LCRC 位置

LCRC 覆盖整个 TLP，包括 Header 和 Data Payload（如果有）：

```
+------------------+
| Sequence Number  |  ← 数据链路层添加（12 位）
+------------------+
| TLP Header       |  ← 3-4 DW (12-16 字节)
| (3 或 4 DW)      |  ↓
+------------------+  |
| Data Payload     |  | LCRC 计算范围
| (0-1024 DW)      |  |
+------------------+  ↑
| LCRC             |  ← 数据链路层添加（32 位）
+------------------+
```

### 详细计算范围

**包含在 LCRC 计算中**：
1. **TLP Header**：所有 Header 字段（包括保留位）
2. **Data Payload**：如果存在的话
3. **TLP Digest (ECRC)**：如果存在的话

**不包含在 LCRC 计算中**：
1. **Sequence Number**：在 LCRC 之前添加
2. **LCRC 字段本身**：LCRC 是计算结果
3. **物理层帧头**：Start/End tokens

### 不同 TLP 类型的 LCRC 计算

| TLP 类型 | Header 大小 | Data Payload | LCRC 覆盖范围 |
|---------|------------|-------------|--------------|
| Memory Read Request | 3 DW | 无 | 3 DW Header |
| Memory Read Request (64-bit) | 4 DW | 无 | 4 DW Header |
| Memory Write Request | 3/4 DW | 1-1024 DW | Header + Data |
| Completion without Data | 3 DW | 无 | 3 DW Header |
| Completion with Data | 3 DW | 1-1024 DW | Header + Data |
| Message | 4 DW | 0-1 DW | Header + Data (if any) |
| Configuration Request | 3 DW | 1 DW | 4 DW 总计 |

### 计算示例

**Memory Write TLP (64-bit 地址，32 字节数据)**：

```c
typedef struct {
    // Header (4 DW)
    uint32_t dw0;  // Fmt, Type, TC, Attr, Length, etc.
    uint32_t dw1;  // Requester ID, Tag, Last/First BE
    uint32_t dw2;  // Address [63:32]
    uint32_t dw3;  // Address [31:2], PH
    
    // Data Payload (8 DW = 32 字节)
    uint32_t data[8];
    
    // LCRC (1 DW)
    uint32_t lcrc;
} memory_write_tlp_t;

uint32_t calculate_tlp_lcrc(const memory_write_tlp_t* tlp) {
    // 计算范围：Header (16 bytes) + Data (32 bytes) = 48 bytes
    const uint8_t* data_start = (const uint8_t*)&tlp->dw0;
    size_t length = 16 + 32;  // Header + Data
    
    return crc32_fast(data_start, length);
}
```

## 错误检测能力

### 理论检测能力

CRC-32 能够检测以下类型的错误：

1. **所有单比特错误**：100% 检测率
2. **所有双比特错误**：100% 检测率
3. **所有奇数个比特错误**：100% 检测率
4. **所有长度 ≤32 的突发错误**：100% 检测率
5. **长度 >32 的突发错误**：99.9999998% 检测率
6. **随机多比特错误**：99.9999998% 检测率

### 汉明距离

CRC-32 提供的汉明距离特性：

| 数据长度 | 最小汉明距离 | 可检测错误模式 |
|---------|-------------|---------------|
| ≤ 2974 bits | HD = 6 | 所有 ≤5 比特错误 |
| ≤ 91607 bits | HD = 4 | 所有 ≤3 比特错误 |
| 任意长度 | HD = 4 | 所有突发长度 ≤32 的错误 |

对于典型的 PCIe TLP：
- 最小 TLP：12 字节 Header = 96 bits
- 最大 TLP：4KB Header + Data = ~33,000 bits
- HD = 4 对所有 PCIe TLP 都适用

### 未检测错误概率

理论上，CRC-32 的未检测错误概率为：

```
P(undetected error) ≈ 2^(-32) = 2.328 × 10^(-10) ≈ 0.00000002%
```

对于实际应用：

```python
def calculate_undetected_error_rate(ber, tlp_size_bytes, throughput_gbps):
    """
    计算未被 LCRC 检测到的错误率
    
    ber: 物理层误码率
    tlp_size_bytes: 平均 TLP 大小（字节）
    throughput_gbps: 吞吐量（Gbps）
    """
    bits_per_tlp = tlp_size_bytes * 8
    tlp_error_rate = 1 - (1 - ber) ** bits_per_tlp
    
    tlps_per_second = (throughput_gbps * 1e9) / bits_per_tlp
    errors_per_second = tlps_per_second * tlp_error_rate
    
    # CRC-32 未检测概率
    undetected_rate = errors_per_second * (2 ** -32)
    
    return undetected_rate

# 示例计算
ber = 1e-12           # 典型 BER
tlp_size = 256        # 256 字节 TLP
throughput = 8        # 8 Gbps (Gen 3 x1)

rate = calculate_undetected_error_rate(ber, tlp_size, throughput)
print(f"未检测错误率: {rate:.2e} errors/sec")
print(f"平均无故障时间: {1/rate/3600/24/365:.1f} 年")
```

## LCRC 与 ECRC 的区别

### 架构层次差异

```
应用层
   ↓
+-------------------+
| 事务层 (TL)        |
+-------------------+
   ↓ (添加 ECRC - 可选)
   ↓
+-------------------+
| 数据链路层 (DLL)   |  ← 添加 Seq# 和 LCRC
+-------------------+
   ↓
+-------------------+
| 物理层 (PL)        |
+-------------------+
```

### 功能对比

| 特性 | LCRC | ECRC |
|-----|------|------|
| **全称** | Link CRC | End-to-End CRC |
| **层次** | 数据链路层 | 事务层 |
| **必需性** | 强制 | 可选 |
| **保护范围** | TLP Header + Data + ECRC | TLP Header + Data |
| **覆盖链路** | 单个 PCIe 链路 (点对点) | 端到端 (穿越 Switch) |
| **错误处理** | 触发链路层重传 | 上报给软件处理 |
| **算法** | CRC-32 | CRC-32 (相同算法) |
| **位置** | 数据链路层帧尾 | TLP 末尾（在 LCRC 之前） |

### 保护域示例

```
端点 A → Switch → 端点 B

[端点 A]
  TLP: Header + Data
  添加 ECRC (可选): Header + Data + ECRC
  ↓
[端点 A 数据链路层]
  添加 Seq# 和 LCRC: Seq# + Header + Data + ECRC + LCRC
  ↓
[物理链路 A]
  ↓
[Switch 端口 1]
  验证 LCRC ✓ → 移除 Seq# 和 LCRC
  不检查 ECRC (透传)
  ↓
[Switch 内部路由]
  ↓
[Switch 端口 2]
  重新添加 Seq# 和 LCRC (新值)
  ↓
[物理链路 B]
  ↓
[端点 B 数据链路层]
  验证 LCRC ✓ → 移除 Seq# 和 LCRC
  ↓
[端点 B 事务层]
  验证 ECRC ✓ (端到端检查)
  
LCRC 保护：各段链路独立
ECRC 保护：端到端完整路径
```

### 使用场景

**仅使用 LCRC（典型配置）**：
- 适用于大多数 PCIe 设备
- 依赖 Switch 内部逻辑的正确性
- 性能开销最小

**LCRC + ECRC（高可靠性）**：
- 关键任务系统（服务器、存储）
- 检测 Switch 内部数据损坏
- 检测内存中的 TLP 篡改
- 轻微性能开销（~1-2%）

### ECRC 配置

```c
// PCIe 配置空间 - Advanced Error Reporting Capability
#define PCI_EXP_AER_CAP_ECRC_GEN_CAPABLE  (1 << 5)
#define PCI_EXP_AER_CAP_ECRC_GEN_ENABLE   (1 << 6)
#define PCI_EXP_AER_CAP_ECRC_CHK_CAPABLE  (1 << 7)
#define PCI_EXP_AER_CAP_ECRC_CHK_ENABLE   (1 << 8)

// 启用 ECRC 生成
void enable_ecrc_generation(int pci_fd, int bus, int dev, int func) {
    uint16_t aer_cap_offset = find_ext_capability(pci_fd, PCI_EXT_CAP_ID_ERR);
    if (aer_cap_offset == 0) {
        printf("设备不支持 AER\n");
        return;
    }
    
    uint32_t aer_cap_ctrl;
    pci_read_config_dword(pci_fd, aer_cap_offset + 0x18, &aer_cap_ctrl);
    
    if (aer_cap_ctrl & PCI_EXP_AER_CAP_ECRC_GEN_CAPABLE) {
        aer_cap_ctrl |= PCI_EXP_AER_CAP_ECRC_GEN_ENABLE;
        pci_write_config_dword(pci_fd, aer_cap_offset + 0x18, aer_cap_ctrl);
        printf("ECRC 生成已启用\n");
    }
}
```

## LCRC 错误处理

### 接收端处理流程

```c
typedef enum {
    LCRC_CHECK_PASS,
    LCRC_CHECK_FAIL,
    LCRC_CHECK_ERROR
} lcrc_check_result_t;

lcrc_check_result_t process_received_tlp(const uint8_t* tlp_data, size_t length) {
    // 1. 提取接收到的 LCRC
    uint32_t received_lcrc = *(uint32_t*)(tlp_data + length - 4);
    
    // 2. 重新计算 LCRC (不包括最后 4 字节的 LCRC 字段)
    uint32_t calculated_lcrc = crc32_fast(tlp_data, length - 4);
    
    // 3. 比较
    if (calculated_lcrc == received_lcrc) {
        // LCRC 正确
        increment_counter(STAT_LCRC_PASS);
        return LCRC_CHECK_PASS;
    } else {
        // LCRC 错误
        increment_counter(STAT_LCRC_ERROR);
        log_error("LCRC mismatch: expected 0x%08X, got 0x%08X", 
                  calculated_lcrc, received_lcrc);
        
        // 发送 NAK DLLP
        send_nak_dllp(extract_sequence_number(tlp_data));
        
        return LCRC_CHECK_FAIL;
    }
}
```

### 错误处理状态机

```
接收 TLP
    ↓
提取 Sequence Number
    ↓
计算 LCRC
    ↓
比较 LCRC
    ↓
  正确？
    ├─ 是 → 发送 ACK DLLP
    │       ↓
    │    传递给事务层
    │
    └─ 否 → 丢弃 TLP
            ↓
         发送 NAK DLLP
            ↓
         等待重传
```

### NAK DLLP 格式

```c
typedef struct {
    uint8_t  type;       // DLLP Type = NAK (0x10)
    uint8_t  reserved1;
    uint16_t ack_nak_seq_num;  // 期望的下一个序列号
    uint16_t crc16;      // DLLP 自己的 CRC-16
    uint8_t  reserved2[2];
} nak_dllp_t;

void send_nak_dllp(uint16_t expected_seq_num) {
    nak_dllp_t nak = {
        .type = 0x10,  // NAK
        .reserved1 = 0,
        .ack_nak_seq_num = expected_seq_num,
        .reserved2 = {0, 0}
    };
    
    // 计算 DLLP 的 CRC-16
    nak.crc16 = calculate_dllp_crc16((uint8_t*)&nak, 6);
    
    // 发送 NAK
    transmit_dllp((uint8_t*)&nak, sizeof(nak));
    
    // 统计
    increment_counter(STAT_NAK_SENT);
}
```

## 硬件实现考虑

### 流水线 CRC 计算

现代 PCIe 控制器使用流水线架构实现实时 CRC 计算：

```
发送路径：
TLP 生成 → CRC 计算引擎 → 添加 LCRC → 物理层编码
   |           ↓              ↓              ↓
  DW0        DW0            DW0            DW0
  DW1    →   DW1      →     DW1      →     DW1
  DW2        DW2 (CRC)      DW2            DW2
  DW3        DW3            DW3            DW3
                            LCRC           LCRC

接收路径：
物理层解码 → 并行 CRC 计算 → 比较 → 传递给 TL
                ↓              ↓        或丢弃
             累积 CRC      最终 CRC
```

### 并行 CRC 引擎

高速实现（Gen 3+）需要多字节并行处理：

```verilog
// 简化的 4 字节并行 CRC 模块 (Verilog)
module crc32_4byte_parallel (
    input wire clk,
    input wire rst_n,
    input wire [31:0] data_in,
    input wire data_valid,
    input wire crc_clear,
    output reg [31:0] crc_out
);

    wire [31:0] crc_next;
    
    // 并行 CRC 计算逻辑（使用预计算的矩阵）
    assign crc_next = crc_matrix_multiply(crc_out, data_in);
    
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            crc_out <= 32'hFFFFFFFF;
        end else if (crc_clear) begin
            crc_out <= 32'hFFFFFFFF;
        end else if (data_valid) begin
            crc_out <= crc_next;
        end
    end

endmodule
```

### 性能优化

| 实现方式 | 吞吐量 | 延迟 | 硬件资源 |
|---------|--------|------|---------|
| 位串行 | 低 (125 Mbps) | 高 (256 周期) | 最少 |
| 字节并行 | 中 (1 Gbps) | 中 (32 周期) | 适中 |
| 4 字节并行 | 高 (4+ Gbps) | 低 (8 周期) | 较多 |
| 全 TLP 并行 | 最高 (16+ Gbps) | 最低 (1-2 周期) | 最多 |

Gen 4 (16 GT/s) 和 Gen 5 (32 GT/s) 需要至少 4 字节并行 CRC 引擎。

## 调试与测试

### LCRC 错误注入

用于测试错误处理路径：

```c
// LCRC 错误注入（仅用于测试）
void inject_lcrc_error(uint8_t* tlp_data, size_t length) {
    uint32_t* lcrc_field = (uint32_t*)(tlp_data + length - 4);
    
    // 翻转一个比特
    *lcrc_field ^= (1 << 15);
    
    printf("已注入 LCRC 错误，原始: 0x%08X, 修改后: 0x%08X\n",
           *lcrc_field ^ (1 << 15), *lcrc_field);
}
```

### 监控 LCRC 错误

```bash
# Linux 下监控 PCIe 错误
#!/bin/bash

DEVICE="0000:01:00.0"
AER_PATH="/sys/bus/pci/devices/${DEVICE}/aer_dev_correctable"

echo "监控 PCIe LCRC 错误..."
watch -n 1 'cat '"${AER_PATH}"' | grep -E "BadTLP|BadDLLP"'

# 或使用 lspci
watch -n 1 'lspci -vvv -s '"${DEVICE}"' | grep -A 5 "Corrected errors"'
```

### PCIe 分析仪捕获

使用硬件分析仪（如 Lecroy, Teledyne）捕获 LCRC 错误：

```
触发条件设置：
- Event Type: NAK DLLP
- Filter: LCRC Error
- Capture: Before/After 100 packets

分析内容：
1. 错误 TLP 的完整内容
2. 接收到的 LCRC 值
3. 期望的 LCRC 值
4. NAK DLLP 的发送时间
5. 重传的 TLP 内容
```

## 性能影响

### 计算延迟

| 实现 | 延迟 (周期) | 延迟 (ns @ 250 MHz) |
|-----|------------|-------------------|
| 软件查表 | ~100-500 | 400-2000 |
| 硬件串行 | ~256 | 1024 |
| 硬件并行 (4B) | ~8 | 32 |
| 硬件并行 (全TLP) | 1-2 | 4-8 |

### 带宽开销

LCRC 增加的带宽开销：

```
开销 = 4 字节 LCRC / (TLP Header + Data + LCRC)

最坏情况（最小 TLP）：
开销 = 4 / (12 + 4) = 25%

典型情况（256 字节 TLP）：
开销 = 4 / (16 + 256 + 4) = 1.45%

最佳情况（4 KB TLP）：
开销 = 4 / (16 + 4096 + 4) = 0.097%
```

更大的 TLP 能够分摊 LCRC 开销，提高效率。

## 最佳实践

### 系统设计建议

1. **总是启用 LCRC**：这是 PCIe 规范的强制要求
2. **考虑启用 ECRC**：对于关键数据路径（存储、金融）
3. **监控 LCRC 错误率**：作为链路健康指标
4. **优化 TLP 大小**：使用较大的 TLP 减少开销

### 错误处理策略

```c
// 建议的错误计数器阈值
#define LCRC_ERROR_WARNING_THRESHOLD  10     // 每小时
#define LCRC_ERROR_CRITICAL_THRESHOLD 100    // 每小时
#define LCRC_ERROR_FATAL_THRESHOLD    1000   // 每小时

void monitor_lcrc_health(dll_statistics_t* stats, uint32_t time_window_sec) {
    float error_rate = (float)stats->lcrc_errors / time_window_sec * 3600;
    
    if (error_rate > LCRC_ERROR_FATAL_THRESHOLD) {
        // 严重错误，建议下线链路
        log_critical("LCRC error rate critical: %.1f/hour", error_rate);
        trigger_link_disable();
    } else if (error_rate > LCRC_ERROR_CRITICAL_THRESHOLD) {
        // 高错误率，降低链路速度
        log_error("LCRC error rate high: %.1f/hour", error_rate);
        trigger_link_speed_downgrade();
    } else if (error_rate > LCRC_ERROR_WARNING_THRESHOLD) {
        // 警告级别，记录日志
        log_warning("LCRC error rate elevated: %.1f/hour", error_rate);
    }
}
```

## 总结

LCRC 是 PCIe 数据链路层的核心保护机制，提供以下关键功能：

- **强制性保护**：所有 TLP 必须包含 LCRC
- **高检测率**：CRC-32 能够检测几乎所有传输错误
- **自动恢复**：与重试机制配合实现透明纠错
- **逐跳验证**：每个链路独立验证数据完整性

与 ECRC 的协同使用可以提供端到端的数据完整性保证，满足高可靠性应用的需求。理解 LCRC 的工作原理对于 PCIe 系统的设计、调试和优化都至关重要。
