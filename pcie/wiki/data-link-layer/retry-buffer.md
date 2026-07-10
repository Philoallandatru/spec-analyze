# 重试缓冲区 (Retry Buffer)

## 概述

重试缓冲区（Retry Buffer）是 PCIe 数据链路层（Data Link Layer, DLL）中实现可靠数据传输的核心机制。它确保了在物理层或数据链路层发生错误时，能够自动重传丢失或损坏的数据包，而无需上层软件干预。这种端到端的硬件级重传机制是 PCIe 协议可靠性的基石。

## 重试缓冲区的作用

### 核心功能

重试缓冲区在发送端维护一个已发送但尚未确认的 TLP（Transaction Layer Packet）副本。其主要作用包括：

1. **保存已发送的 TLP**：在收到接收端的 ACK 确认之前，发送端必须保留所有已发送 TLP 的完整副本
2. **支持重传操作**：当检测到错误或收到 NAK 信号时，能够快速重新发送缓冲区中的数据
3. **维护传输顺序**：确保重传的数据包保持原有的发送顺序
4. **实现流控制**：与流控制机制配合，防止接收端缓冲区溢出

### 缓冲区大小要求

PCIe 规范对重试缓冲区的大小有明确要求：

```
最小重试缓冲区大小 = 4 × Max_Payload_Size + 额外开销
```

| 设备类型 | 最小缓冲区要求 | 典型实现 |
|---------|---------------|---------|
| Root Complex | ≥ 4 KB | 8-16 KB |
| Switch | ≥ 4 KB | 8-32 KB |
| Endpoint | ≥ 4 KB | 4-8 KB |

实际实现中，许多设备提供远超最小要求的缓冲区大小，以提高传输性能和容错能力。

## 重传机制详解

### ACK/NAK 协议

PCIe 使用 ACK（Acknowledgment）和 NAK（Negative Acknowledgment）DLLP 来实现确认机制：

```
发送端                              接收端
  |                                   |
  |---- TLP (Seq=100) --------------->|
  |                                   | (检查 LCRC，正确)
  |<--- ACK DLLP (Ack#=101) ----------|
  |                                   |
  | (释放 Seq=100 的 TLP)              |
  |                                   |
  |---- TLP (Seq=101) --------------->|
  |                                   | (检查 LCRC，错误)
  |<--- NAK DLLP (Nak#=101) ----------|
  |                                   |
  | (重传 Seq=101 及后续 TLP)          |
  |---- TLP (Seq=101) --------------->|
  |                                   |
```

### 序列号机制

每个 TLP 在数据链路层被分配一个 12 位序列号（Sequence Number），范围从 0 到 4095，循环使用：

```c
// 序列号管理伪代码
typedef struct {
    uint16_t next_transmit_seq;    // 下一个要发送的序列号 (0-4095)
    uint16_t ack_seq;              // 已确认的序列号
    uint16_t retry_start_seq;      // 重试起始序列号
} seq_manager_t;

// 分配序列号
uint16_t assign_sequence_number(seq_manager_t* mgr) {
    uint16_t seq = mgr->next_transmit_seq;
    mgr->next_transmit_seq = (mgr->next_transmit_seq + 1) % 4096;
    return seq;
}

// 处理 ACK
void process_ack(seq_manager_t* mgr, uint16_t ack_num) {
    // 释放序列号小于 ack_num 的所有 TLP
    while (mgr->ack_seq != ack_num) {
        release_tlp_from_retry_buffer(mgr->ack_seq);
        mgr->ack_seq = (mgr->ack_seq + 1) % 4096;
    }
}

// 处理 NAK
void process_nak(seq_manager_t* mgr, uint16_t nak_num) {
    // 从 nak_num 开始重传所有 TLP
    mgr->retry_start_seq = nak_num;
    mgr->next_transmit_seq = nak_num;
    trigger_replay();
}
```

### 重传触发条件

重传可以由以下几种情况触发：

#### 1. NAK 触发的重传

接收端在以下情况会发送 NAK：
- **LCRC 错误**：检测到 TLP 的链路层 CRC 校验失败
- **序列号错误**：收到的序列号与期望的序列号不匹配
- **接收缓冲区溢出**：由于流控制错误导致的溢出

#### 2. Replay Timer 超时

Replay Timer 是发送端维护的定时器，用于检测 ACK 丢失的情况：

```
Replay Timer 超时时间：
- Gen 1/2: 300 ns × N (N 为链路上的符号数)
- Gen 3+: 基于链路延迟动态计算
```

**Replay Timer 工作流程**：

```
1. 发送 TLP 时启动 Replay Timer
2. 收到 ACK DLLP 时重置 Timer
3. Timer 超时前未收到 ACK：
   ├─ 第一次超时：触发重传
   ├─ 连续超时次数 < 最大重试次数：继续重传
   └─ 达到最大重试次数：上报链路失败
```

#### 3. Completion Timeout

这是事务层的超时机制，但会影响数据链路层的重试：

```c
// Completion Timeout 范围
typedef enum {
    COMPLETION_TIMEOUT_50US_50MS = 0,    // 50 μs to 50 ms
    COMPLETION_TIMEOUT_50US_100US = 1,   // 50 μs to 100 μs
    COMPLETION_TIMEOUT_1MS_10MS = 2,     // 1 ms to 10 ms
    COMPLETION_TIMEOUT_16MS_55MS = 5,    // 16 ms to 55 ms
    COMPLETION_TIMEOUT_65MS_210MS = 6,   // 65 ms to 210 ms
    COMPLETION_TIMEOUT_1S_3_5S = 9,      // 1 s to 3.5 s
    COMPLETION_TIMEOUT_4S_13S = 10,      // 4 s to 13 s
    COMPLETION_TIMEOUT_17S_64S = 14      // 17 s to 64 s
} completion_timeout_range_t;
```

## 超时处理机制

### Replay Timer 详解

Replay Timer 是数据链路层最重要的超时机制：

**计算公式**（简化版）：

```
Replay_Timer_Limit = 2 × Round_Trip_Latency + Ack_Latency + Margin
```

**典型时序**：

```
时间轴 (单位: ns)
  0 ─┬─ 发送 TLP (Seq=100)，启动 Replay Timer
     │
  50 ─┤  TLP 传播到接收端
     │
  60 ─┤  接收端处理 TLP
     │
  70 ─┤  接收端发送 ACK DLLP
     │
 120 ─┤  ACK 传播到发送端
     │
 130 ─┴─ 发送端收到 ACK，停止 Timer
```

如果在 Replay Timer Limit（例如 300 ns）内未收到 ACK，则触发重传。

### 重传计数器

PCIe 设备维护重传计数器，用于跟踪重传尝试次数：

```c
typedef struct {
    uint32_t replay_count;           // 当前重传次数
    uint32_t max_replay_attempts;    // 最大重传次数（通常为 3-5 次）
    uint32_t total_replays;          // 统计：总重传次数
    uint32_t nak_received;           // 统计：收到的 NAK 数量
    uint32_t timeout_replays;        // 统计：超时触发的重传
} retry_statistics_t;

// 重传处理
int handle_replay(retry_statistics_t* stats) {
    stats->replay_count++;
    stats->total_replays++;
    
    if (stats->replay_count >= stats->max_replay_attempts) {
        // 达到最大重试次数，上报链路失败
        report_link_failure();
        return -1;
    }
    
    // 执行重传
    retransmit_all_unacked_tlps();
    return 0;
}
```

### 错误恢复流程

当重传失败达到阈值时，PCIe 链路会进入恢复流程：

```
错误检测
    ↓
NAK 或 Replay Timer 超时
    ↓
触发重传（从 NAK 序列号或最早未确认 TLP 开始）
    ↓
重传成功？
    ├─ 是：继续正常传输
    └─ 否：增加重传计数
        ↓
    达到最大重试次数？
        ├─ 否：继续重传
        └─ 是：进入链路恢复状态
            ↓
        发送 TS1（Training Sequence 1）
            ↓
        链路重训练（Link Retraining）
            ↓
        恢复成功？
            ├─ 是：返回 L0 状态
            └─ 否：链路失败，可能进入 Disabled 状态
```

## 重试缓冲区管理

### 缓冲区数据结构

典型的重试缓冲区实现：

```c
#define RETRY_BUFFER_SIZE 8192  // 8 KB
#define MAX_TLP_SIZE 4096       // 最大 TLP 大小

typedef struct {
    uint16_t sequence_number;   // 序列号
    uint16_t tlp_length;        // TLP 长度（字节）
    uint8_t  tlp_data[MAX_TLP_SIZE];  // TLP 数据
    uint32_t timestamp;         // 发送时间戳
    bool     acked;             // 是否已确认
} retry_buffer_entry_t;

typedef struct {
    retry_buffer_entry_t entries[128];  // 支持 128 个待确认 TLP
    uint16_t write_ptr;         // 写指针
    uint16_t read_ptr;          // 读指针
    uint16_t count;             // 当前条目数
    uint32_t bytes_used;        // 已用字节数
} retry_buffer_t;

// 添加 TLP 到重试缓冲区
int add_to_retry_buffer(retry_buffer_t* buf, uint16_t seq, 
                        const uint8_t* tlp, uint16_t len) {
    if (buf->bytes_used + len > RETRY_BUFFER_SIZE) {
        return -1;  // 缓冲区满
    }
    
    retry_buffer_entry_t* entry = &buf->entries[buf->write_ptr];
    entry->sequence_number = seq;
    entry->tlp_length = len;
    memcpy(entry->tlp_data, tlp, len);
    entry->timestamp = get_current_time_ns();
    entry->acked = false;
    
    buf->write_ptr = (buf->write_ptr + 1) % 128;
    buf->count++;
    buf->bytes_used += len;
    
    return 0;
}

// 从重试缓冲区移除已确认的 TLP
void remove_acked_tlps(retry_buffer_t* buf, uint16_t ack_seq) {
    while (buf->count > 0) {
        retry_buffer_entry_t* entry = &buf->entries[buf->read_ptr];
        
        // 检查序列号是否在已确认范围内
        if (seq_less_than(entry->sequence_number, ack_seq)) {
            buf->bytes_used -= entry->tlp_length;
            buf->read_ptr = (buf->read_ptr + 1) % 128;
            buf->count--;
        } else {
            break;
        }
    }
}
```

### 缓冲区优化策略

高性能实现通常采用以下优化：

1. **循环缓冲区**：避免内存拷贝和碎片
2. **零拷贝设计**：直接从 DMA 缓冲区引用数据
3. **多级缓存**：使用 SRAM/Cache 加速重传
4. **预测性释放**：在预期 ACK 到达前预先准备释放

## 性能影响分析

### 缓冲区大小对性能的影响

| 缓冲区大小 | 优点 | 缺点 | 适用场景 |
|-----------|------|------|---------|
| 4 KB（最小） | 硬件成本低 | 可能限制吞吐量 | 低速端点 |
| 8-16 KB | 平衡性能与成本 | 中等硬件开销 | 通用设备 |
| 32+ KB | 高吞吐量，容错性强 | 硬件成本高 | 高性能 NIC/SSD |

### 延迟分析

重试机制引入的额外延迟：

```
正常传输延迟：
  传播延迟 + 处理延迟 = 约 100-500 ns

发生重传时的延迟：
  检测延迟 + Replay Timer 超时 + 重传时间
  = 300 ns + (300-1000 ns) + 传播延迟
  ≈ 700-2000 ns

最坏情况（多次重传失败）：
  多次重传 + 链路重训练
  = 数毫秒到数十毫秒
```

### 带宽利用率

在不同错误率下的带宽利用率：

```python
# 带宽利用率计算
def calculate_throughput_efficiency(ber, avg_packet_size, link_speed_gbps):
    """
    ber: 误码率（Bit Error Rate）
    avg_packet_size: 平均包大小（字节）
    link_speed_gbps: 链路速度（Gbps）
    """
    bits_per_packet = avg_packet_size * 8
    packet_error_rate = 1 - (1 - ber) ** bits_per_packet
    
    # 假设重传开销为 500 ns
    retransmit_overhead_ns = 500
    packet_time_ns = (bits_per_packet / link_speed_gbps)
    
    # 考虑重传的有效吞吐量
    efficiency = 1 / (1 + packet_error_rate * retransmit_overhead_ns / packet_time_ns)
    return efficiency * 100

# 示例：Gen 3 (8 Gbps)，256 字节包，不同 BER
print(f"BER 1e-12: {calculate_throughput_efficiency(1e-12, 256, 8):.2f}%")
print(f"BER 1e-10: {calculate_throughput_efficiency(1e-10, 256, 8):.2f}%")
print(f"BER 1e-9:  {calculate_throughput_efficiency(1e-9, 256, 8):.2f}%")
```

## 调试与监控

### 重试缓冲区状态监控

关键监控指标：

```c
typedef struct {
    uint64_t tlps_transmitted;      // 发送的 TLP 总数
    uint64_t tlps_retransmitted;    // 重传的 TLP 数量
    uint64_t acks_received;         // 收到的 ACK 数量
    uint64_t naks_received;         // 收到的 NAK 数量
    uint64_t replay_timeouts;       // Replay Timer 超时次数
    uint64_t buffer_full_events;    // 缓冲区满事件
    uint32_t current_buffer_usage;  // 当前缓冲区使用率
    uint32_t peak_buffer_usage;     // 峰值缓冲区使用率
    uint32_t link_recovery_count;   // 链路恢复次数
} dll_statistics_t;

// 性能指标计算
float calculate_retry_rate(dll_statistics_t* stats) {
    if (stats->tlps_transmitted == 0) return 0.0;
    return (float)stats->tlps_retransmitted / stats->tlps_transmitted * 100;
}
```

### Linux 下的调试接口

在 Linux 系统中，可以通过以下方式查看重试统计：

```bash
# 查看 PCIe 错误计数
lspci -vvv -s 01:00.0 | grep -A 10 "Advanced Error Reporting"

# 使用 pcieport 驱动的调试接口
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_correctable
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_fatal

# 使用 perf 监控 PCIe 重传事件（如果硬件支持）
perf stat -e pcie/replay_event/ -a sleep 10
```

### 常见问题诊断

| 问题现象 | 可能原因 | 诊断方法 |
|---------|---------|---------|
| 频繁重传 | 信号完整性问题、EMI | 示波器检查眼图、减少链路速率测试 |
| Replay Timer 超时 | 链路延迟过大、时钟偏差 | 调整 Timer 参数、检查 SKP 符号 |
| 缓冲区溢出 | 流控制失败、接收端阻塞 | 检查 FC Credit 状态、监控接收端 |
| 链路频繁恢复 | 硬件故障、驱动问题 | 检查物理连接、更新固件/驱动 |

## 最佳实践

### 设计建议

1. **缓冲区大小设计**
   - 至少满足 4× Max_Payload_Size
   - 考虑链路延迟和往返时间
   - 为突发流量预留余量

2. **超时参数配置**
   - Replay Timer 应适配实际链路延迟
   - 考虑多级交换机的累积延迟
   - 为温度变化预留裕量

3. **错误处理策略**
   - 设置合理的最大重试次数（3-5 次）
   - 快速上报不可恢复错误
   - 记录详细的错误日志用于分析

### 性能调优

```c
// 自适应 Replay Timer 调整
typedef struct {
    uint32_t min_rtt_ns;           // 最小往返时间
    uint32_t avg_rtt_ns;           // 平均往返时间
    uint32_t max_rtt_ns;           // 最大往返时间
    uint32_t replay_timer_limit;   // 动态 Timer 限制
} adaptive_timer_t;

void update_replay_timer(adaptive_timer_t* timer, uint32_t measured_rtt) {
    // 更新 RTT 统计
    timer->avg_rtt_ns = (timer->avg_rtt_ns * 7 + measured_rtt) / 8;
    
    if (measured_rtt < timer->min_rtt_ns) {
        timer->min_rtt_ns = measured_rtt;
    }
    if (measured_rtt > timer->max_rtt_ns) {
        timer->max_rtt_ns = measured_rtt;
    }
    
    // 设置 Timer 为平均值 + 2× 标准差
    timer->replay_timer_limit = timer->avg_rtt_ns * 2 + 100;  // 加 100ns 余量
}
```

## 总结

重试缓冲区是 PCIe 数据链路层实现可靠传输的核心机制。通过维护已发送但未确认的 TLP 副本，结合 ACK/NAK 协议和 Replay Timer 超时机制，PCIe 能够在硬件层面自动处理传输错误，无需软件干预。

关键要点：
- **可靠性**：确保所有 TLP 最终被正确接收
- **性能**：在正常情况下几乎无额外开销
- **容错性**：能够应对物理层的瞬时错误
- **透明性**：对上层软件完全透明

理解重试缓冲区的工作原理对于 PCIe 系统设计、性能优化和故障诊断都至关重要。
