# PCIe 错误检测

## 概述

PCIe 在各个协议层实现了多层次的错误检测机制，确保数据传输的完整性和可靠性。本文详细介绍各层的错误检测方法和实现。

## 检测层次结构

```
应用数据
    |
    v
+------------------+
| Transaction Layer |  <-- ECRC (端到端完整性)
+------------------+
    |
    v
+------------------+
| Data Link Layer  |  <-- LCRC (链路层完整性)
+------------------+      Sequence Number (顺序检测)
    |                     Ack/Nak (确认机制)
    v
+------------------+
| Physical Layer   |  <-- 8b/10b Decode (符号检测)
+------------------+      Disparity Check (极性检测)
    |                     Elastic Buffer (时钟补偿)
    v
物理链路
```

## 物理层错误检测

### 8b/10b 编码检测 (Gen 1/2)

```
8b/10b 编码特性：
- 每 8 位数据编码为 10 位符号
- 控制字符和数据字符有特定模式
- Running Disparity 跟踪

检测机制：

1. 非法符号检测
   收到：0b1111111111 (10 个连续 1)
   --> 不是有效的 8b/10b 符号
   --> Receiver Error

2. Disparity 错误
   当前 RD: Positive
   收到符号: D10.2 (Positive disparity 版本)
   期望: D10.2 (Negative disparity 版本)
   --> Disparity Error
   --> Receiver Error

3. 控制字符验证
   收到：K28.7 在数据位置
   期望：数据字符
   --> Control Symbol Error
```

#### 硬件实现

```verilog
// 8b/10b 解码器错误检测
module decoder_8b10b (
    input wire clk,
    input wire [9:0] encoded_data,
    input wire rd_in,              // Running Disparity input
    
    output reg [7:0] decoded_data,
    output reg is_control,
    output reg rd_out,             // Running Disparity output
    output reg decode_error,       // 解码错误
    output reg disparity_error     // 极性错误
);

    reg [9:0] lookup_table [0:511];
    reg [10:0] current_disparity;
    
    always @(posedge clk) begin
        decode_error <= 1'b0;
        disparity_error <= 1'b0;
        
        // 查找解码表
        decoded_data <= lookup_table[encoded_data];
        
        // 检查是否为有效符号
        if (decoded_data == 8'hFF) begin
            decode_error <= 1'b1;
        end
        
        // 计算当前符号的极性
        current_disparity <= count_ones(encoded_data) - 
                             count_zeros(encoded_data);
        
        // 检查极性是否匹配
        if (rd_in == POSITIVE && current_disparity != NEGATIVE &&
            current_disparity != NEUTRAL) begin
            disparity_error <= 1'b1;
        end
        
        // 更新 Running Disparity
        rd_out <= (current_disparity == POSITIVE) ? NEGATIVE : POSITIVE;
    end

endmodule
```

### 128b/130b 编码检测 (Gen 3+)

```
128b/130b 特性：
- 2-bit Sync Header: 01b 或 10b
- 128-bit Data Block

检测机制：

1. Sync Header 验证
   收到：00b 或 11b
   --> 无效的 Sync Header
   --> Block Lock Loss

2. 块对齐检测
   搜索有效的 Sync Header 模式
   如果连续多个块没有有效 Sync Header
   --> 失去块锁定

3. Scrambler 同步
   使用 LFSR (Linear Feedback Shift Register)
   如果解扰后数据无意义
   --> Scrambler 失步
```

#### 块锁定状态机

```
       +--------+
       | Search |  <-- 初始状态
       +--------+
            |
            | 找到有效 Sync Header
            v
       +--------+
       | Lock   |  <-- 锁定状态
       +--------+
            |
            | 连续错误超过阈值
            v
       +--------+
       | Search |
       +--------+
```

### 接收器错误处理

```c
// 物理层错误处理
void phy_handle_receiver_error(struct pcie_port *port)
{
    struct phy_error_counters *counters = &port->phy_counters;
    
    // 增加错误计数
    counters->receiver_errors++;
    
    // 记录可纠正错误
    aer_report_correctable_error(port->pdev, PCI_ERR_COR_RCVR);
    
    // 如果错误率过高，触发链路重训练
    if (counters->receiver_errors > PHY_ERROR_THRESHOLD) {
        pr_warn("High receiver error rate, initiating link retrain\n");
        pcie_retrain_link(port);
        counters->receiver_errors = 0;
    }
}
```

## 数据链路层错误检测

### LCRC (Link CRC)

```
LCRC 计算：
- 32-bit CRC
- 覆盖：Sequence Number + TLP
- 不覆盖：LCRC 本身

生成多项式：
G(x) = x^32 + x^26 + x^23 + x^22 + x^16 + 
       x^12 + x^11 + x^10 + x^8 + x^7 + 
       x^5 + x^4 + x^2 + x + 1

TLP 传输格式：
+-------+------+----------+------+
| Start | TLP  | Seq#     | LCRC |
| Token | Data | (2 bytes)| (4 B)|
+-------+------+----------+------+

LCRC 覆盖范围：
        [  Seq#  |  TLP  ]
        └─────────────────┘
             LCRC 计算范围
```

#### LCRC 计算实现

```c
// CRC-32 计算（标准以太网 CRC）
uint32_t calculate_lcrc(const uint8_t *data, size_t len)
{
    uint32_t crc = 0xFFFFFFFF;
    
    for (size_t i = 0; i < len; i++) {
        crc ^= data[i];
        for (int j = 0; j < 8; j++) {
            if (crc & 1)
                crc = (crc >> 1) ^ 0xEDB88320;  // 反转多项式
            else
                crc = crc >> 1;
        }
    }
    
    return ~crc;
}

// 验证 LCRC
bool verify_lcrc(const uint8_t *tlp_data, size_t tlp_len,
                 uint16_t seq_num, uint32_t received_lcrc)
{
    uint8_t buffer[tlp_len + 2];
    
    // 构造 LCRC 计算缓冲区：Seq# + TLP
    buffer[0] = seq_num & 0xFF;
    buffer[1] = (seq_num >> 8) & 0xFF;
    memcpy(buffer + 2, tlp_data, tlp_len);
    
    // 计算 LCRC
    uint32_t calculated_lcrc = calculate_lcrc(buffer, tlp_len + 2);
    
    return (calculated_lcrc == received_lcrc);
}
```

#### LCRC 错误处理

```
LCRC 检测到错误：
1. 丢弃错误的 TLP
2. 不更新 Ack 序列号
3. 不发送 Ack DLLP
4. 等待发送端重传超时
5. 发送端重传该 TLP

如果 LCRC 错误频繁：
1. 记录到 Correctable Error Status
2. 可能触发链路重训练
3. 检查信号完整性

示例流程：
发送端: TLP (Seq=10) + LCRC_A
         ↓
物理链路 (噪声导致 bit 翻转)
         ↓
接收端: TLP (Seq=10, 有错误) + LCRC_A
         ↓
      计算 LCRC_B ≠ LCRC_A
         ↓
      LCRC Error! 丢弃 TLP
         ↓
      不发送 Ack(10)
         ↓
发送端: Replay Timer 超时
         ↓
      重传 TLP (Seq=10)
```

### 序列号检测

```
序列号机制：
- 12-bit 序列号 (0-4095)
- 循环使用
- 检测丢失、重复、乱序

检测场景：

1. 序列号跳跃（丢包）
   收到：Seq=100
   期望：Seq=99
   --> 发送 NAK(99)

2. 序列号重复
   收到：Seq=100 (第二次)
   期望：Seq=101
   --> 丢弃重复的 TLP

3. 序列号回退
   收到：Seq=98
   期望：Seq=100
   --> 可能是重传，检查是否已确认

实现：
struct dll_receiver {
    uint16_t next_expected_seq;
    uint16_t last_acked_seq;
    bool nak_sent[4096];  // 跟踪已发送的 NAK
};

bool check_sequence_number(struct dll_receiver *rx, uint16_t seq)
{
    if (seq == rx->next_expected_seq) {
        // 正确的序列号
        rx->next_expected_seq = (rx->next_expected_seq + 1) % 4096;
        return true;
    } else if (seq < rx->next_expected_seq) {
        // 可能是重传
        if (seq > rx->last_acked_seq) {
            // 重传的 TLP，接受
            return true;
        } else {
            // 重复的 TLP，丢弃
            return false;
        }
    } else {
        // 序列号跳跃
        if (!rx->nak_sent[rx->next_expected_seq]) {
            send_nak(rx->next_expected_seq);
            rx->nak_sent[rx->next_expected_seq] = true;
        }
        return false;
    }
}
```

### DLLP CRC

```
DLLP CRC-16:
- 16-bit CRC
- 覆盖：DLLP Type + Reserved + Payload
- 不覆盖：CRC 本身

生成多项式：
G(x) = x^16 + x^12 + x^5 + 1

DLLP 格式：
+------+----------+------+--------+
| Type | Reserved | Data | CRC-16 |
| 1B   | 1B       | 2B   | 2B     |
+------+----------+------+--------+
└──────────────────────┘
       CRC 计算范围

检测：
if (calculate_crc16(dllp, 4) != received_crc) {
    // Bad DLLP
    discard_dllp();
    report_correctable_error(PCI_ERR_COR_BAD_DLLP);
}
```

## 事务层错误检测

### ECRC (End-to-End CRC)

```
ECRC 特性：
- 可选特性
- 32-bit CRC
- 端到端完整性保护
- 覆盖整个 TLP（不包括 Seq# 和 LCRC）

覆盖范围：
+--------+----------+------+
| Header | Payload  | ECRC |
+--------+----------+------+
└──────────────────┘
     ECRC 计算范围

与 LCRC 的区别：
LCRC: 链路层保护（逐跳）
ECRC: 事务层保护（端到端）

       发送端              中间节点            接收端
         |                    |                  |
      生成 ECRC            不检查 ECRC        检查 ECRC
         |                    |                  |
         └────────── 端到端保护 ──────────────┘
              (Switch 不修改 TLP，ECRC 保持有效)
```

#### ECRC 生成和验证

```c
// ECRC 生成
uint32_t generate_ecrc(const struct tlp *tlp)
{
    uint8_t buffer[4096];
    size_t len = 0;
    
    // TLP Header
    memcpy(buffer, &tlp->header, tlp->header_len);
    len += tlp->header_len;
    
    // TLP Data Payload (如果有)
    if (tlp->data_len > 0) {
        memcpy(buffer + len, tlp->data, tlp->data_len);
        len += tlp->data_len;
    }
    
    // 计算 CRC-32
    return calculate_crc32(buffer, len);
}

// ECRC 验证
bool verify_ecrc(const struct tlp *tlp, uint32_t received_ecrc)
{
    uint32_t calculated_ecrc = generate_ecrc(tlp);
    
    if (calculated_ecrc != received_ecrc) {
        // ECRC Error
        pr_err("ECRC mismatch: calculated=0x%08x, received=0x%08x\n",
               calculated_ecrc, received_ecrc);
        
        // 记录不可纠正错误
        aer_report_uncorrectable_error(dev, PCI_ERR_UNC_ECRC);
        
        return false;
    }
    
    return true;
}
```

#### ECRC 配置

```c
// 启用/禁用 ECRC
static void configure_ecrc(struct pci_dev *pdev, bool enable)
{
    int pos = pci_find_ext_capability(pdev, PCI_EXT_CAP_ID_ERR);
    u32 cap;
    
    if (!pos)
        return;
    
    // 读取 Advanced Error Capabilities and Control
    pci_read_config_dword(pdev, pos + PCI_ERR_CAP, &cap);
    
    if (enable) {
        // 启用 ECRC 生成
        cap |= PCI_ERR_CAP_ECRC_GENC;
        // 启用 ECRC 检查
        cap |= PCI_ERR_CAP_ECRC_CHKE;
    } else {
        // 禁用 ECRC
        cap &= ~(PCI_ERR_CAP_ECRC_GENC | PCI_ERR_CAP_ECRC_CHKE);
    }
    
    pci_write_config_dword(pdev, pos + PCI_ERR_CAP, cap);
    
    dev_info(&pdev->dev, "ECRC %s\n", enable ? "enabled" : "disabled");
}
```

### TLP 格式验证

```
检查项目：

1. Header 完整性
   - Fmt 和 Type 组合是否有效
   - Length 字段是否合理 (0-1024 DW)
   - Reserved 位是否为 0

2. 地址对齐
   - Memory Request 地址必须与 Length 对齐
   - First/Last BE 一致性检查

3. 字段一致性
   - Length 与实际 Data Payload 匹配
   - Completion 的 Byte Count 合理性

4. 配置请求验证
   - Register Number < 256 (Type 0/1)
   - Register Number < 4096 (PCIe 扩展)
   - Bus/Device/Function 范围

检测代码：
bool validate_tlp_header(const struct tlp_header *hdr)
{
    // 检查 Fmt/Type 组合
    if (!is_valid_fmt_type(hdr->fmt, hdr->type)) {
        return false;
    }
    
    // 检查 Length
    if (hdr->length > 1024 || hdr->length == 0) {
        if (!is_length_zero_allowed(hdr->type)) {
            return false;
        }
    }
    
    // 检查 Reserved 位
    if (hdr->reserved != 0) {
        pr_warn("TLP has non-zero reserved bits\n");
    }
    
    // 检查地址对齐
    if (is_memory_request(hdr)) {
        if ((hdr->address & 0x3) && hdr->length > 1) {
            return false;  // 非对齐的多 DW 访问
        }
    }
    
    return true;
}
```

### Poisoned TLP 检测

```
EP (Error Poisoned) 位：
- 位于 TLP Header Byte 0, Bit 14
- 只在 Completion TLP 和 Memory Write TLP 中有效

检测流程：
收到 TLP --> 检查 EP 位
              |
              v
         EP = 1 ?
              |
              v (是)
      Poisoned TLP Detected
              |
              v
      记录 UCE Status[12]
              |
              v
      根据策略处理：
      - 丢弃 TLP
      - 传播错误到应用
      - 触发 Machine Check Exception

处理示例：
void handle_poisoned_tlp(struct pci_dev *pdev, struct tlp *tlp)
{
    // 记录错误
    aer_report_uncorrectable_error(pdev, PCI_ERR_UNC_POISON_TLP);
    
    // 记录 TLP 头到 Header Log
    save_tlp_header_log(pdev, tlp);
    
    // 根据 Severity 决定是否致命
    if (is_fatal_poison(pdev)) {
        // 触发致命错误处理
        handle_fatal_error(pdev);
    } else {
        // 非致命，通知驱动程序
        notify_driver_error(pdev, -EIO);
    }
}
```

## 综合错误检测示例

### 完整的接收路径

```c
// PCIe 接收路径错误检测
enum tlp_rx_result {
    TLP_RX_OK,
    TLP_RX_PHY_ERROR,
    TLP_RX_LCRC_ERROR,
    TLP_RX_SEQ_ERROR,
    TLP_RX_ECRC_ERROR,
    TLP_RX_MALFORMED,
    TLP_RX_POISONED
};

enum tlp_rx_result receive_tlp(struct pcie_port *port, 
                                 struct tlp *tlp)
{
    // 1. 物理层检测
    if (port->phy->decode_error) {
        port->stats.phy_errors++;
        return TLP_RX_PHY_ERROR;
    }
    
    // 2. LCRC 检测
    if (!verify_lcrc(tlp->data, tlp->len, tlp->seq, tlp->lcrc)) {
        port->stats.lcrc_errors++;
        send_nak(port, tlp->seq);
        return TLP_RX_LCRC_ERROR;
    }
    
    // 3. 序列号检测
    if (!check_sequence_number(&port->dll_rx, tlp->seq)) {
        port->stats.seq_errors++;
        return TLP_RX_SEQ_ERROR;
    }
    
    // 4. 发送 ACK
    send_ack(port, tlp->seq);
    
    // 5. TLP 格式验证
    if (!validate_tlp_header(&tlp->header)) {
        port->stats.malformed_tlps++;
        aer_report_uncorrectable_error(port->pdev, 
                                        PCI_ERR_UNC_MALF_TLP);
        return TLP_RX_MALFORMED;
    }
    
    // 6. ECRC 检测（如果启用）
    if (port->ecrc_enabled && tlp->header.td) {
        if (!verify_ecrc(tlp, tlp->ecrc)) {
            port->stats.ecrc_errors++;
            aer_report_uncorrectable_error(port->pdev,
                                            PCI_ERR_UNC_ECRC);
            return TLP_RX_ECRC_ERROR;
        }
    }
    
    // 7. Poisoned TLP 检测
    if (tlp->header.ep) {
        port->stats.poisoned_tlps++;
        handle_poisoned_tlp(port->pdev, tlp);
        return TLP_RX_POISONED;
    }
    
    // 8. 传递到上层处理
    process_tlp(port, tlp);
    
    return TLP_RX_OK;
}
```

## 错误检测性能影响

### 各层开销

| 检测机制 | 延迟增加 | 带宽影响 | 硬件复杂度 |
|---------|---------|---------|-----------|
| 8b/10b Decode | < 1 ns | 20% 编码开销 | 低 |
| LCRC | 2-5 ns | 4 字节/TLP | 中 |
| Sequence# | < 1 ns | 2 字节/TLP | 低 |
| ECRC | 5-10 ns | 4 字节/TLP | 中 |
| Format Check | 1-2 ns | 0% | 低 |

### 优化策略

```c
// 硬件流水线并行检测
void parallel_error_detection(struct tlp *tlp)
{
    // 各层检测同时进行
    parallel_section {
        phy_decode_check(tlp);      // 物理层
        lcrc_check(tlp);             // LCRC
        sequence_check(tlp);         // 序列号
        format_check(tlp);           // 格式
        ecrc_check(tlp);             // ECRC
    }
    
    // 汇总结果
    if (any_error_detected()) {
        handle_error(tlp);
    } else {
        forward_to_upper_layer(tlp);
    }
}
```

## 调试和监控

### 错误率监控

```c
// 错误率计算
struct error_rate_monitor {
    uint64_t window_start_time;
    uint64_t window_size_ns;
    
    uint32_t phy_errors;
    uint32_t lcrc_errors;
    uint32_t ecrc_errors;
    uint32_t total_tlps;
};

void update_error_rates(struct error_rate_monitor *mon)
{
    uint64_t now = ktime_get_ns();
    
    if (now - mon->window_start_time > mon->window_size_ns) {
        // 计算错误率
        double phy_rate = (double)mon->phy_errors / mon->total_tlps;
        double lcrc_rate = (double)mon->lcrc_errors / mon->total_tlps;
        double ecrc_rate = (double)mon->ecrc_errors / mon->total_tlps;
        
        pr_info("Error rates: PHY=%.6f%%, LCRC=%.6f%%, ECRC=%.6f%%\n",
                phy_rate * 100, lcrc_rate * 100, ecrc_rate * 100);
        
        // 复位计数器
        mon->window_start_time = now;
        mon->phy_errors = 0;
        mon->lcrc_errors = 0;
        mon->ecrc_errors = 0;
        mon->total_tlps = 0;
    }
}
```

### 内核追踪点

```bash
# 追踪 PCIe 错误检测
trace-cmd record -e pcie:pcie_error_detected \
                  -e aer:aer_event

# 查看追踪
trace-cmd report
```

## 总结

PCIe 错误检测的关键点：

1. **多层检测**：物理层、数据链路层、事务层
2. **CRC 保护**：LCRC（逐跳）、ECRC（端到端）
3. **序列检测**：防止丢包、重复、乱序
4. **格式验证**：确保协议一致性
5. **毒化机制**：传播数据错误到关心的层
6. **性能平衡**：检测开销 vs 可靠性

参见：
- [错误类型](error-types.md)
- [AER 详解](aer.md)
- [错误恢复](recovery.md)
- [LCRC 详解](../data-link-layer/lcrc.md)
