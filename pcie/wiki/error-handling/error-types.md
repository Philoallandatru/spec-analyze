# PCIe 错误类型

## 概述

PCIe 定义了多种错误类型，用于检测和报告链路和设备的异常情况。理解这些错误类型对于系统调试和可靠性设计至关重要。

## 错误分类

### 按严重性分类

```
┌─────────────────────────────────────┐
│     PCIe 错误严重性层次              │
├─────────────────────────────────────┤
│                                     │
│  ┌───────────────────────────────┐ │
│  │   致命错误 (Fatal Errors)     │ │
│  │   - 系统无法恢复              │ │
│  │   - 需要系统重启              │ │
│  │   - 例如：链路完全失效        │ │
│  └───────────────────────────────┘ │
│              ↑                      │
│  ┌───────────────────────────────┐ │
│  │ 非致命错误 (Non-Fatal Errors) │ │
│  │   - 可能导致数据丢失          │ │
│  │   - 设备可以恢复              │ │
│  │   - 例如：TLP 格式错误        │ │
│  └───────────────────────────────┘ │
│              ↑                      │
│  ┌───────────────────────────────┐ │
│  │ 可纠正错误 (Correctable Errors)│ │
│  │   - 硬件自动纠正              │ │
│  │   - 不影响数据完整性          │ │
│  │   - 例如：接收器错误          │ │
│  └───────────────────────────────┘ │
│                                     │
└─────────────────────────────────────┘
```

### 按层次分类

| 层次 | 错误类型 | 示例 |
|------|---------|------|
| **Transaction Layer** | TLP 相关错误 | Malformed TLP, ECRC Error, Poisoned TLP |
| **Data Link Layer** | DLLP/链路错误 | LCRC Error, NAK Timeout, Replay Timeout |
| **Physical Layer** | 物理信号错误 | Receiver Error, 8b/10b Decode Error |

## 可纠正错误 (Correctable Errors)

### 错误列表

| 位 | 错误名称 | 描述 | 恢复机制 |
|----|---------|------|---------|
| 0 | Receiver Error | 物理层接收错误 | 自动重训练 |
| 6 | Bad TLP | TLP 格式错误但可忽略 | 丢弃 TLP |
| 7 | Bad DLLP | DLLP 格式错误 | 丢弃 DLLP |
| 8 | REPLAY_NUM Rollover | 重传计数溢出 | 链路重训练 |
| 12 | Replay Timer Timeout | 重传超时 | 请求重传 |
| 13 | Advisory Non-Fatal | 建议性非致命错误 | 记录日志 |
| 14 | Corrected Internal Error | 内部可纠正错误 | 内部纠正 |
| 15 | Header Log Overflow | 头日志溢出 | 覆盖旧条目 |

### Receiver Error

```
原因：
- 8b/10b 解码错误
- 物理层符号错误
- 信号完整性问题
- 噪声干扰

检测：
Physical Layer --> Receiver Error
                    |
                    v
            自动记录到 CE Status

恢复：
1. 物理层尝试重新同步
2. 如果持续错误，触发链路重训练
3. 链路恢复后继续传输

影响：
- 透明恢复，上层无感知
- 可能增加延迟
- 频繁出现需要检查信号质量
```

### Replay Timer Timeout

```
原因：
- TLP 发送后长时间未收到 ACK
- 接收端处理缓慢
- 链路拥塞

检测：
发送 TLP --> 启动重传定时器（timeout 值可配置）
              |
              v
        超时未收到 ACK --> Replay Timer Timeout

恢复流程：
1. 记录 Correctable Error Status
2. 发送 NAK DLLP 请求重传
3. 发送端从重试缓冲区重传
4. 如果反复超时，触发链路重训练

配置：
- Replay Timer: 通常 1-10 ms
- Replay Counter: 最大重传次数
```

### Bad TLP/DLLP

```
Bad TLP:
- TLP 头格式错误
- Length 字段不一致
- Reserved 位非零
处理：丢弃 TLP，不影响其他传输

Bad DLLP:
- DLLP CRC 错误
- DLLP 格式错误
处理：丢弃 DLLP，不影响 TLP 传输
```

## 不可纠正非致命错误 (Uncorrectable Non-Fatal)

### 错误列表

| 位 | 错误名称 | 描述 | 严重性 |
|----|---------|------|--------|
| 4 | Data Link Protocol Error | 数据链路协议违规 | Non-Fatal |
| 12 | Poisoned TLP Received | 收到中毒 TLP | Non-Fatal |
| 13 | Flow Control Protocol Error | 流控制协议错误 | Non-Fatal |
| 14 | Completion Timeout | Completion 超时 | Non-Fatal |
| 15 | Completer Abort | Completer 中止 | Non-Fatal |
| 16 | Unexpected Completion | 意外的 Completion | Non-Fatal |
| 17 | Receiver Overflow | 接收器溢出 | Non-Fatal |
| 18 | Malformed TLP | 格式错误的 TLP | Non-Fatal |
| 19 | ECRC Error | ECRC 校验失败 | Non-Fatal |
| 20 | Unsupported Request | 不支持的请求 | Non-Fatal |
| 21 | ACS Violation | 访问控制违规 | Non-Fatal |
| 22 | Uncorrectable Internal Error | 不可纠正内部错误 | Non-Fatal |
| 23 | MC Blocked TLP | 多播阻塞 TLP | Non-Fatal |
| 24 | AtomicOp Egress Blocked | 原子操作出口阻塞 | Non-Fatal |
| 25 | TLP Prefix Blocked Error | TLP 前缀阻塞错误 | Non-Fatal |

### Poisoned TLP

```
中毒机制：
1. 数据在传输或处理中检测到错误
2. 设置 TLP 的 EP (Error Poisoned) 位
3. TLP 继续传输，但标记为"中毒"
4. 接收端检测到 EP 位，记录错误

场景示例：
CPU --> Memory Read Request
        |
        v
     设备读取内存
        |
        v (ECC 错误检测到)
     设备设置 EP=1
        |
        v
     Completion (EP=1)
        |
        v
     CPU 收到，触发 Machine Check Exception

优点：
- 错误不会被静默忽略
- 保证数据完整性
- 允许错误传播到关心的软件层

Completion TLP with EP=1:
DW0: Fmt=010, Type=01010, EP=1, ...
```

### Flow Control Protocol Error

```
原因：
- 信用违规（发送超过信用的数据）
- 信用初始化错误
- UpdateFC DLLP 错误

检测：
接收端维护信用计数
if (received_data > available_credits) {
    报告 Flow Control Protocol Error
}

示例：
初始信用：8 个 Header Credits
已发送：8 个 TLP
收到 UpdateFC：+2 Credits (现在有 2)
又发送：3 个 TLP --> 违规！

影响：
- 可能导致接收缓冲区溢出
- 数据可能丢失
- 需要复位链路或设备

恢复：
1. 记录错误
2. 通知驱动程序
3. 驱动执行设备复位
4. 重新初始化流控制
```

### Completion Timeout

```
原因：
- 请求的设备无响应
- 请求被丢弃
- 路由错误

超时时间：
- 可配置：50 μs - 50 seconds
- 典型值：1-10 ms

流程：
CPU --> Non-Posted Request (Memory Read)
        |
        v
     启动 Completion Timer
        |
        v (超时)
     Completion Timeout Error

处理：
1. 记录 UCE Status[14]
2. 可能触发 AER 中断
3. 驱动程序收到错误通知
4. 根据 error_detected 回调决定恢复策略

配置（Device Control 2 寄存器）：
Completion Timeout Value:
0000: 50 μs - 50 ms
0001: 50 μs - 100 μs
0010: 1 ms - 10 ms
...
1101: 16 s - 55 s
1110: 65 s - 210 s
```

### Unsupported Request

```
原因：
- 访问不支持的功能
- 非法的配置访问
- 访问不存在的 BAR
- Type 1 配置访问到 Endpoint

示例 1：访问未实现的 BAR
CPU --> Memory Write to BAR2
        |
        v
     设备只实现了 BAR0
        |
        v
     设备返回 Unsupported Request (UR)

示例 2：64-bit 访问 32-bit 设备
CPU --> 64-bit Memory Write
        |
        v
     设备只支持 32-bit
        |
        v
     返回 UR Completion

处理：
- Completion Status = Unsupported Request (001)
- 记录 UCE Status[20]
- CPU 收到 Completion 后可能产生异常
```

## 不可纠正致命错误 (Uncorrectable Fatal)

### 默认致命的错误

这些错误通常被配置为致命（UE Severity 寄存器）：

| 位 | 错误名称 | 为什么致命 |
|----|---------|-----------|
| 4 | Data Link Protocol Error | 链路协议崩溃 |
| 5 | Surprise Down Error | 链路意外断开 |
| 13 | Flow Control Protocol Error | 信用系统崩溃 |
| 18 | Malformed TLP | 协议违规 |
| 19 | ECRC Error | 端到端完整性失败 |
| 22 | Uncorrectable Internal Error | 内部硬件故障 |

### Data Link Protocol Error

```
原因：
- LTSSM 异常状态
- Ack/Nak 协议失败
- 序列号错误

示例：
发送端：Seq# = 10
接收端：期望 Seq# = 10
收到：   Seq# = 12  --> 跳号！

检测到 Data Link Protocol Error

影响：
- 链路状态机进入 Recovery 或 Detect
- 所有待处理的事务失败
- 可能需要设备复位

恢复：
1. 触发 Fatal Error 中断
2. 系统尝试热复位
3. 如果失败，标记设备不可用
```

### Malformed TLP

```
严重的格式错误：
- Length = 0 但有 Data Payload
- Memory Write 的 Length 不匹配实际数据
- Reserved 编码的 Type 字段
- Configuration Request 的地址越界

检测：
TLP Checker --> 格式验证
                 |
                 v (违规)
            Malformed TLP Error

处理：
1. 记录到 UCE Status[18]
2. 丢弃 TLP
3. 如果配置为 Fatal，触发致命错误流程
4. 可能导致系统挂起或 panic

与 Bad TLP 的区别：
- Bad TLP：轻微格式问题，可纠正
- Malformed TLP：严重违规，不可纠正
```

## 错误严重性配置

### Uncorrectable Error Severity 寄存器

```
AER Capability + 0x0C: UE Severity Register

Bit 位对应 UE Status 的错误类型：
0: Non-Fatal
1: Fatal

默认配置（PCIe 规范建议）：
Bit 4  (Data Link Protocol): 1 (Fatal)
Bit 12 (Poisoned TLP):       0 (Non-Fatal)
Bit 13 (Flow Control):       1 (Fatal)
Bit 14 (Completion Timeout): 0 (Non-Fatal)
Bit 15 (Completer Abort):    0 (Non-Fatal)
Bit 18 (Malformed TLP):      1 (Fatal)
Bit 19 (ECRC):               1 (Fatal)
Bit 20 (Unsupported Request):0 (Non-Fatal)

软件可以修改这些配置
```

### 配置示例

```c
// 配置错误严重性
static void configure_error_severity(struct pci_dev *pdev)
{
    int pos = pdev->aer_cap;
    u32 severity;
    
    // 读取当前严重性配置
    pci_read_config_dword(pdev, pos + PCI_ERR_UNCOR_SEVER, &severity);
    
    // 将 Completion Timeout 配置为 Fatal（生产环境可能不这样做）
    severity |= PCI_ERR_UNC_COMP_TIME;
    
    // 将 Poisoned TLP 配置为 Non-Fatal（允许软件处理）
    severity &= ~PCI_ERR_UNC_POISON_TLP;
    
    pci_write_config_dword(pdev, pos + PCI_ERR_UNCOR_SEVER, severity);
    
    dev_info(&pdev->dev, "Error severity configured: 0x%08x\n", severity);
}
```

## 错误掩码

### Uncorrectable Error Mask

```
AER Capability + 0x08: UE Mask Register

设置某位为 1 = 屏蔽该错误
设置某位为 0 = 启用该错误报告

示例：屏蔽 Completion Timeout
u32 mask;
pci_read_config_dword(pdev, pos + PCI_ERR_UNCOR_MASK, &mask);
mask |= PCI_ERR_UNC_COMP_TIME;  // 屏蔽
pci_write_config_dword(pdev, pos + PCI_ERR_UNCOR_MASK, mask);

用途：
- 调试时临时屏蔽已知问题
- 某些错误在特定系统中不适用
- 防止错误风暴

注意：屏蔽不代表错误不发生，只是不报告
```

### Correctable Error Mask

```
AER Capability + 0x14: CE Mask Register

通常不屏蔽可纠正错误，除非：
- 频繁的 Receiver Error 影响性能
- 已知的硬件缺陷导致误报
- 测试环境中的预期错误
```

## 错误统计和监控

### 错误计数

```c
struct pcie_error_stats {
    atomic64_t ce_count;         // 可纠正错误
    atomic64_t nfe_count;        // 非致命错误
    atomic64_t fe_count;         // 致命错误
    
    // 细分统计
    atomic64_t receiver_error;
    atomic64_t bad_tlp;
    atomic64_t replay_timeout;
    atomic64_t completion_timeout;
    atomic64_t poisoned_tlp;
    atomic64_t malformed_tlp;
};

void update_error_stats(struct pci_dev *pdev, u32 ce_status, u32 ue_status)
{
    struct pcie_error_stats *stats = &pdev->aer_stats;
    
    if (ce_status) {
        atomic64_inc(&stats->ce_count);
        if (ce_status & PCI_ERR_COR_RCVR)
            atomic64_inc(&stats->receiver_error);
        if (ce_status & PCI_ERR_COR_BAD_TLP)
            atomic64_inc(&stats->bad_tlp);
    }
    
    if (ue_status) {
        if (is_fatal_error(pdev, ue_status))
            atomic64_inc(&stats->fe_count);
        else
            atomic64_inc(&stats->nfe_count);
    }
}
```

### sysfs 接口

```bash
# 查看错误统计
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_correctable
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_nonfatal
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_fatal

# 查看详细错误类型
cat /sys/kernel/debug/pci/0000:01:00.0/aer_stats
```

## 错误影响分析

### 影响矩阵

| 错误类型 | 数据完整性 | 系统可用性 | 性能影响 | 恢复时间 |
|---------|-----------|-----------|---------|---------|
| Receiver Error | ✅ 无影响 | ✅ 无影响 | ⚠️ 轻微 | < 1 μs |
| Replay Timeout | ✅ 无影响 | ✅ 无影响 | ⚠️ 中等 | < 10 ms |
| Poisoned TLP | ❌ 数据错误 | ⚠️ 任务失败 | ⚠️ 中等 | 取决于软件 |
| Completion Timeout | ⚠️ 请求失败 | ⚠️ 任务失败 | ⚠️ 中等 | 50 μs - 50 s |
| Malformed TLP | ❌ 协议违规 | ❌ 设备不可用 | ❌ 严重 | 需要复位 |
| Data Link Protocol | ❌ 链路失效 | ❌ 设备不可用 | ❌ 严重 | 需要热复位 |

## 调试技巧

### 查看错误状态

```bash
# 使用 lspci
lspci -vvv -s 01:00.0 | grep -A 5 "Advanced Error Reporting"

# 使用 setpci 读取 AER 寄存器
setpci -s 01:00.0 ECAP_AER+0x04.L  # UE Status
setpci -s 01:00.0 ECAP_AER+0x10.L  # CE Status
```

### 内核日志分析

```bash
# 查看 AER 错误日志
dmesg | grep -i "aer\|pcie"

# 典型日志：
# pcieport 0000:00:1c.0: AER: Corrected error received
# pcieport 0000:00:1c.0: PCIe Bus Error: severity=Corrected, type=Physical Layer
# pcieport 0000:00:1c.0:   device [8086:a110] error status/mask=00000001/00002000
# pcieport 0000:00:1c.0:    [ 0] RxErr
```

## 总结

PCIe 错误类型的关键点：

1. **三级严重性**：可纠正、非致命、致命
2. **层次化检测**：物理层、数据链路层、事务层
3. **灵活配置**：可以配置严重性和掩码
4. **透明恢复**：可纠正错误对软件透明
5. **错误传播**：Poisoned TLP 机制保证数据完整性
6. **统计监控**：帮助诊断系统健康状况

参见：
- [AER 详解](aer.md)
- [错误检测](detection.md)
- [错误恢复](recovery.md)
- [错误注入](injection.md)
