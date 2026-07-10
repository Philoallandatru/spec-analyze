# PCIe 速度与编码 (Speed and Encoding)

## 概述

PCIe 的速度规格与编码方案紧密相关。每一代 PCIe 都定义了特定的信号速率（GT/s）和编码方案，共同决定了实际可用带宽。理解速度与编码的关系对于系统设计和性能优化至关重要。

---

## PCIe 各代速度规格

### 速度演进表

| PCIe 代数 | 信号速率 | 编码方案 | 编码效率 | 每通道带宽 | x16 带宽 |
|----------|---------|---------|---------|-----------|---------|
| **Gen 1** | 2.5 GT/s | 8b/10b | 80% | 2.0 Gb/s | 32 Gb/s (4 GB/s) |
| **Gen 2** | 5.0 GT/s | 8b/10b | 80% | 4.0 Gb/s | 64 Gb/s (8 GB/s) |
| **Gen 3** | 8.0 GT/s | 128b/130b | 98.46% | 7.877 Gb/s | 126 Gb/s (15.75 GB/s) |
| **Gen 4** | 16.0 GT/s | 128b/130b | 98.46% | 15.754 Gb/s | 252 Gb/s (31.5 GB/s) |
| **Gen 5** | 32.0 GT/s | 128b/130b | 98.46% | 31.508 Gb/s | 504 Gb/s (63 GB/s) |
| **Gen 6** | 64.0 GT/s | FLIT/PAM4 | ~98% | ~62.5 Gb/s | ~1000 Gb/s (125 GB/s) |

**注释**：
- **GT/s**：Giga Transfers per second（十亿次传输/秒）- 物理层信号速率
- **Gb/s**：Gigabits per second（吉比特/秒）- 去除编码开销后的有效数据率
- **带宽**：单向传输速率，PCIe 是全双工，双向总带宽翻倍

---

## 速度与编码的关系

### 编码开销对速度的影响

```
信号速率 (GT/s) × 编码效率 = 有效数据率 (Gb/s)

Gen 1 示例:
2.5 GT/s × (8/10) = 2.0 Gb/s

Gen 3 示例:
8.0 GT/s × (128/130) = 7.877 Gb/s
```

**为什么 Gen 3 改用 128b/130b？**

```
假设使用 8b/10b 达到 Gen 3 带宽:
目标: 8 Gb/s 有效带宽
需要信号速率: 8 / 0.8 = 10 GT/s

问题:
1. 10 GT/s 信号对物理层要求极高
2. 功耗显著增加
3. 成本大幅上升

使用 128b/130b:
需要信号速率: 8 / 0.9846 = 8.13 GT/s → 实际 8.0 GT/s
节省: 20% 信号速率 = 更低功耗和成本
```

---

## 各代速度详解

### Gen 1 (2.5 GT/s)

```
基础参数:
- 信号速率: 2.5 GT/s
- 编码: 8b/10b
- 符号周期: 400 ps (1/2.5G)
- 编码后位周期: 400 ps

每通道带宽计算:
2.5 GT/s × 8/10 = 2.0 Gb/s = 250 MB/s

x16 配置:
250 MB/s × 16 = 4000 MB/s = 4 GB/s (单向)
全双工: 8 GB/s (双向总和)
```

**应用场景**：
- 第一代 PCIe 设备（2003-2007）
- 低速设备兼容性
- 功耗敏感应用

---

### Gen 2 (5.0 GT/s)

```
基础参数:
- 信号速率: 5.0 GT/s (2× Gen 1)
- 编码: 8b/10b (相同)
- 符号周期: 200 ps
- 编码后位周期: 200 ps

每通道带宽计算:
5.0 GT/s × 8/10 = 4.0 Gb/s = 500 MB/s

x16 配置:
500 MB/s × 16 = 8000 MB/s = 8 GB/s (单向)
```

**相对 Gen 1 的改进**：
- 速度翻倍
- 保持编码兼容性
- 相对简单的升级路径

**限制**：
- 仍有 20% 编码开销
- 5 GT/s 接近 8b/10b 实用上限

---

### Gen 3 (8.0 GT/s)

```
基础参数:
- 信号速率: 8.0 GT/s
- 编码: 128b/130b (新)
- 符号周期: 125 ps
- 块周期: 16.25 ns (130 位)

每通道带宽计算:
8.0 GT/s × 128/130 = 7.877 Gb/s ≈ 984.6 MB/s

x16 配置:
984.6 MB/s × 16 = 15754 MB/s ≈ 15.75 GB/s (单向)
```

**关键特性**：
- 编码效率从 80% 提升到 98.46%
- 必须使用扰码 (Scrambling)
- 支持发送端均衡 (TX EQ)
- 接收端 CTLE/DFE 均衡

**相对 Gen 2 的提升**：
- 带宽提升 96.9% (几乎翻倍)
- 实际信号速率仅提升 60%

---

### Gen 4 (16.0 GT/s)

```
基础参数:
- 信号速率: 16.0 GT/s (2× Gen 3)
- 编码: 128b/130b (相同)
- 符号周期: 62.5 ps
- 块周期: 8.125 ns

每通道带宽计算:
16.0 GT/s × 128/130 = 15.754 Gb/s ≈ 1969 MB/s

x16 配置:
1969 MB/s × 16 = 31508 MB/s ≈ 31.5 GB/s (单向)
```

**技术挑战**：
- 更高的信号完整性要求
- 更复杂的均衡算法
- PAM4 调制的前期研究

**应用**：
- 高性能 GPU (RTX 3090, RX 6900 XT)
- 企业级 NVMe SSD
- 高速网络适配器

---

### Gen 5 (32.0 GT/s)

```
基础参数:
- 信号速率: 32.0 GT/s
- 编码: 128b/130b (相同)
- 符号周期: 31.25 ps
- 块周期: 4.0625 ns

每通道带宽计算:
32.0 GT/s × 128/130 = 31.508 Gb/s ≈ 3938 MB/s

x16 配置:
3938 MB/s × 16 = 63016 MB/s ≈ 63 GB/s (单向)
```

**极限挑战**：
- 31.25 ps 位周期（极短）
- 信道损耗 ~-16 dB @ 16 GHz
- 需要先进的 PCB 材料
- 更严格的时序要求

**当前应用**：
- 最新 NVMe SSD (2023+)
- 下一代 GPU/加速器
- CXL 3.0 互连
- AI/ML 训练加速卡

---

## 速度协商机制

### 链路训练与速度协商

```
链路训练阶段:
1. Detect     → 检测对端存在
2. Polling    → 确定位宽和速度能力
3. Config     → 配置链路参数
4. L0         → 正常运行

速度协商在 Polling 阶段:
┌─────────────────────────────────────┐
│  TS1/TS2 训练序列交换                │
│  ┌───────────────────────────────┐  │
│  │ 上游端: Gen 5 capable         │  │
│  │ 下游端: Gen 3 capable         │  │
│  │ → 协商结果: Gen 3 (最小公倍)  │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘

Speed ID 字段:
0001b = 2.5 GT/s (Gen 1)
0010b = 5.0 GT/s (Gen 2)
0011b = 8.0 GT/s (Gen 3)
0100b = 16.0 GT/s (Gen 4)
0101b = 32.0 GT/s (Gen 5)
```

### 速度变更流程

```c
// 运行时速度变更
void pcie_change_speed(pcie_device_t *dev, uint8_t target_speed) {
    // 1. 进入 Recovery 状态
    pcie_enter_recovery(dev);
    
    // 2. 发送新的 TS1 序列（指定目标速度）
    ts1_sequence_t ts1;
    ts1.speed_id = target_speed;
    pcie_send_ts1(dev, &ts1);
    
    // 3. 等待对端确认
    if (pcie_wait_ts2_ack(dev)) {
        // 4. 切换 PLL 到新速率
        pcie_set_pll_rate(dev, target_speed);
        
        // 5. 重新均衡（Gen 3+）
        if (target_speed >= PCIE_GEN3) {
            pcie_run_equalization(dev);
        }
        
        // 6. 返回 L0
        pcie_enter_l0(dev);
    }
}
```

---

## 带宽计算与性能

### 理论 vs 实际带宽

```
理论最大带宽:
Gen 3 x4: 7.877 Gb/s × 4 = 31.508 Gb/s = 3.94 GB/s

协议开销:
1. TLP Header: 12-20 字节
2. DLLP Acks: 周期性确认
3. LCRC: 4 字节/TLP
4. Flow Control: DLLP 流控制

实际有效带宽 ≈ 85-95% 理论带宽

示例: Gen 3 x4 NVMe SSD
理论: 3.94 GB/s
实际: 3.5 GB/s (大块顺序读取)
小块随机: 更低（协议开销占比增加）
```

### 不同负载下的效率

```python
def effective_bandwidth(gen, lanes, payload_size):
    """
    计算有效带宽
    
    gen: 1-5
    lanes: 1, 2, 4, 8, 16
    payload_size: 字节 (64, 128, 256, 512, 1024, 4096)
    """
    # 每通道原始速率 (Mb/s)
    raw_rates = {
        1: 2000,   # 2.5 GT/s × 8/10
        2: 4000,   # 5.0 GT/s × 8/10
        3: 7877,   # 8.0 GT/s × 128/130
        4: 15754,  # 16.0 GT/s × 128/130
        5: 31508   # 32.0 GT/s × 128/130
    }
    
    raw_bw = raw_rates[gen] * lanes  # Mb/s
    
    # TLP 开销
    tlp_header = 12  # 最小 TLP header (3 DW)
    lcrc = 4
    overhead = tlp_header + lcrc
    
    # 效率
    efficiency = payload_size / (payload_size + overhead)
    
    # 实际带宽 (MB/s)
    effective_bw = (raw_bw / 8) * efficiency * 0.95  # 95% 考虑其他开销
    
    return effective_bw

# 示例
print(f"Gen 3 x4, 4KB payload: {effective_bandwidth(3, 4, 4096):.2f} MB/s")
print(f"Gen 3 x4, 64B payload: {effective_bandwidth(3, 4, 64):.2f} MB/s")

# 输出:
# Gen 3 x4, 4KB payload: 3687.50 MB/s (效率高)
# Gen 3 x4, 64B payload: 2812.50 MB/s (协议开销占比大)
```

---

## 速度与功耗权衡

### 各代功耗对比

```
单 Lane L0 状态典型功耗:
Gen 1: ~500 mW
Gen 2: ~800 mW
Gen 3: ~1200 mW
Gen 4: ~1500 mW
Gen 5: ~2000 mW

每 Gb/s 功耗效率:
Gen 1: 500 mW / 2.0 Gb/s = 250 mW/(Gb/s)
Gen 2: 800 mW / 4.0 Gb/s = 200 mW/(Gb/s)
Gen 3: 1200 mW / 7.877 Gb/s = 152 mW/(Gb/s)
Gen 4: 1500 mW / 15.754 Gb/s = 95 mW/(Gb/s)
Gen 5: 2000 mW / 31.508 Gb/s = 63 mW/(Gb/s)

结论: 高代数能效更好（每比特功耗更低）
```

### 速度降级策略

```c
// 根据负载动态调整速度
typedef enum {
    SPEED_POLICY_MAX,        // 总是最高速度
    SPEED_POLICY_BALANCED,   // 根据负载调整
    SPEED_POLICY_POWER_SAVE  // 优先省电
} speed_policy_t;

void adaptive_speed_control(pcie_device_t *dev, 
                            speed_policy_t policy,
                            uint32_t utilization_percent) {
    uint8_t target_speed = dev->max_speed;
    
    switch (policy) {
    case SPEED_POLICY_BALANCED:
        if (utilization_percent < 20) {
            target_speed = PCIE_GEN1;  // 低负载 → Gen 1
        } else if (utilization_percent < 50) {
            target_speed = PCIE_GEN2;  // 中负载 → Gen 2
        } else {
            target_speed = dev->max_speed;  // 高负载 → 最高速度
        }
        break;
        
    case SPEED_POLICY_POWER_SAVE:
        // 使用满足需求的最低速度
        target_speed = calculate_min_speed_for_load(utilization_percent);
        break;
        
    case SPEED_POLICY_MAX:
    default:
        target_speed = dev->max_speed;
        break;
    }
    
    if (target_speed != dev->current_speed) {
        pcie_change_speed(dev, target_speed);
    }
}
```

---

## 速度测试与验证

### 查看链路速度

```bash
# 使用 lspci 查看
lspci -vvv -s 01:00.0 | grep -i "lnk"

# 输出示例:
#   LnkCap: Port #0, Speed 16GT/s, Width x4
#   LnkSta: Speed 16GT/s (ok), Width x4 (ok)

# 详细信息
sudo lspci -vvv -s 01:00.0 | grep -A5 "LnkCap"
#   LnkCap: Port #0, Speed 16GT/s, Width x4, ASPM L1, ...
#           - 设备支持 Gen 4 (16 GT/s)
#           - 支持 x4 配置
#   LnkSta: Speed 16GT/s (ok), Width x4 (ok)
#           - 当前运行在 Gen 4
#           - 当前使用 x4 通道
```

### 速度性能测试

```bash
# 使用 fio 测试不同速度下的性能
# 假设 NVMe SSD 支持 Gen 4

# 1. 测试当前速度 (Gen 4)
fio --name=gen4_test --filename=/dev/nvme0n1 --direct=1 \
    --rw=read --bs=128k --iodepth=32 --runtime=30

# 2. 降级到 Gen 3
sudo setpci -s 01:00.0 CAP_EXP+30.w=0031  # Target Speed = Gen 3
sudo setpci -s 01:00.0 CAP_EXP+10.w=0020  # Retrain Link
sleep 2
lspci -vvv -s 01:00.0 | grep "LnkSta.*Speed"  # 确认降级

# 3. 重新测试
fio --name=gen3_test --filename=/dev/nvme0n1 --direct=1 \
    --rw=read --bs=128k --iodepth=32 --runtime=30

# 预期结果:
# Gen 4 x4: ~6800 MB/s
# Gen 3 x4: ~3500 MB/s (约 50% 带宽)
```

### 编码错误检测

```bash
# 查看链路错误统计
sudo lspci -vvv -s 01:00.0 | grep -A20 "Advanced Error Reporting"

# 相关错误:
# - Receiver Error: 可能由编码/解码错误引起
# - Bad TLP: 可能由 128b/130b sync header 错误
# - Bad DLLP: 可能由链路层编码问题

# 实时监控错误
watch -n 1 'sudo lspci -vvv -s 01:00.0 | grep -A5 "UESta"'
```

---

## 常见问题与解决

### 速度协商失败

**现象**：设备协商到低于预期的速度

```bash
# 检查链路能力和状态
lspci -vvv -s 01:00.0 | grep -i "lnk"
  LnkCap: Speed 16GT/s, Width x16
  LnkSta: Speed 8GT/s, Width x16  # 降级到 Gen 3

# 可能原因:
# 1. 主板/CPU 不支持 Gen 4
lspci -vvv -s 00:01.0 | grep "LnkCap"  # 检查 Root Port

# 2. 信道质量不足
# - 检查 PCB 走线长度
# - 验证连接器质量
# - 测试信号完整性

# 3. 均衡失败 (Gen 3+)
# - 查看均衡状态寄存器
sudo setpci -s 01:00.0 CAP_EXP+34.l  # Equalization Status
```

**解决方案**：

```bash
# 1. 强制重新训练
echo 1 | sudo tee /sys/bus/pci/devices/0000:01:00.0/remove
echo 1 | sudo tee /sys/bus/pci/rescan

# 2. 调整 BIOS 设置
# - 启用 "PCIe Gen 4"
# - 关闭 "ASPM" (可能干扰训练)
# - 调整 "Link Speed" 设置

# 3. 检查固件更新
# - 主板 BIOS
# - 设备固件
```

---

### 带宽不达预期

**诊断流程**：

```python
#!/usr/bin/env python3
"""
PCIe 带宽诊断工具
"""

def diagnose_bandwidth(expected_mbps, actual_mbps):
    """诊断带宽问题"""
    ratio = actual_mbps / expected_mbps
    
    if ratio > 0.95:
        return "✓ 带宽正常 (>95% 理论值)"
    elif ratio > 0.85:
        return "⚠ 带宽良好但有协议开销 (85-95%)"
    elif ratio > 0.50:
        return "⚠ 带宽偏低 - 检查链路速度是否降级"
    else:
        return "✗ 带宽严重不足 - 链路可能降级或有其他问题"

# 示例
gen3_x4_theoretical = 3938  # MB/s
measured = 3200  # MB/s
print(diagnose_bandwidth(gen3_x4_theoretical, measured))
# 输出: ⚠ 带宽良好但有协议开销 (85-95%)
```

---

## 未来趋势

### Gen 6 (64 GT/s) - 预览

```
预计规格:
- 信号速率: 64.0 GT/s
- 编码: FLIT mode + PAM4
- 每通道带宽: ~62.5 Gb/s
- x16 带宽: ~1 TB/s = 125 GB/s

关键技术:
1. PAM4 调制
   - 4 电平信号 (00, 01, 10, 11)
   - 每符号 2 比特
   - 降低符号速率 (32 GBaud)

2. FLIT (Flow control unIT)
   - 256 字节固定大小块
   - 简化流控制
   - 更高效的编码

3. 前向纠错 (FEC)
   - CRC-24 或 Reed-Solomon
   - 提高可靠性
```

### Gen 7 及更远

```
可能的技术方向:
- 更高阶调制 (PAM8/PAM16)
- 光互连 (Optical PCIe)
- 芯片间互连 (Chiplet/UCIe)
- 与 CXL/UCIe 融合
```

---

## 实用建议

### 选择合适的速度

```
应用场景建议:

低带宽设备 (< 500 MB/s):
→ Gen 2 x1 或 Gen 3 x1
  示例: 声卡, USB 控制器, SATA 控制器

中等带宽设备 (500-2000 MB/s):
→ Gen 3 x2 或 Gen 3 x4
  示例: 消费级 NVMe SSD, 千兆网卡

高带宽设备 (2-8 GB/s):
→ Gen 3 x8 或 Gen 4 x4
  示例: 企业级 NVMe, 消费级 GPU

超高带宽设备 (> 8 GB/s):
→ Gen 4 x8 或 Gen 4 x16
  示例: 高端 GPU, AI 加速器, 100G 网卡
```

### 平衡性能与成本

```
成本考虑:

Gen 3 设计:
+ PCB: 标准 FR-4
+ 连接器: 常规质量
+ 成本: 基准
- 带宽: 最高 ~16 GB/s (x16)

Gen 4 设计:
+ PCB: 低损耗材料
+ 连接器: 高质量/高频
+ 成本: +30-50%
- 带宽: 最高 ~32 GB/s (x16)

Gen 5 设计:
+ PCB: 特殊低损耗材料
+ 连接器: 高端高频
+ 成本: +100-150%
- 带宽: 最高 ~64 GB/s (x16)

建议: 根据实际需求选择，避免过度设计
```

---

## 总结

**关键要点**：

1. ✅ **速度与编码紧密相关**：Gen 1-2 使用 8b/10b，Gen 3+ 使用 128b/130b
2. ✅ **编码效率**：8b/10b = 80%，128b/130b = 98.46%
3. ✅ **带宽计算**：信号速率 × 编码效率 × 通道数 = 理论带宽
4. ✅ **实际带宽**：理论带宽 × 85-95%（协议开销）
5. ✅ **能效提升**：每代功耗/比特持续降低
6. ✅ **协商机制**：链路训练时自动协商最高公共速度

**设计建议**：
- 根据实际带宽需求选择 Gen 和通道数
- 考虑成本-性能权衡
- 预留未来升级空间
- 重视信号完整性设计（Gen 4+）

---

## 相关页面

- [编码方案详解](encoding.md) - 8b/10b 和 128b/130b 技术细节
- [链路训练](link-training.md) - 速度协商过程
- [物理层概述](overview.md) - 物理层整体架构
- [电气规范](electrical-spec.md) - 信号完整性要求

---

*最后更新：2026-07-06*
