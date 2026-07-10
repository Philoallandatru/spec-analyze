# PCIe 速度和编码

## 概述

PCIe 通过提高数据传输速率和改进编码方案来增加带宽。本文详细介绍各代 PCIe 的速度、编码方式和带宽计算。

## PCIe 代数演进

### 速度对比表

| 代数 | 速率 | 编码 | 编码效率 | 有效带宽(x1) | 发布年份 |
|------|------|------|---------|-------------|---------|
| Gen 1 | 2.5 GT/s | 8b/10b | 80% | 250 MB/s | 2003 |
| Gen 2 | 5.0 GT/s | 8b/10b | 80% | 500 MB/s | 2007 |
| Gen 3 | 8.0 GT/s | 128b/130b | 98.5% | 985 MB/s | 2010 |
| Gen 4 | 16.0 GT/s | 128b/130b | 98.5% | 1.97 GB/s | 2017 |
| Gen 5 | 32.0 GT/s | 128b/130b | 98.5% | 3.94 GB/s | 2019 |
| Gen 6 | 64.0 GT/s | 242b/256b | 94.5% | 6.06 GB/s | 2022 |

**GT/s**: Giga Transfers per second（每秒十亿次传输）

## 编码方案

### 8b/10b 编码 (Gen 1/2)

**原理**：
```
每 8 位数据编码为 10 位符号
编码效率 = 8/10 = 80%

目的：
1. 保证直流平衡（Running Disparity）
2. 足够的状态转换（时钟恢复）
3. 控制字符编码
4. 错误检测能力

示例：
数据: 0xA5 (10100101)
编码: K28.5 或 D21.5 (10位符号)
```

**带宽计算**：
```
Gen1 x1: 2.5 GT/s × 8/10 = 2.0 Gb/s = 250 MB/s
Gen2 x1: 5.0 GT/s × 8/10 = 4.0 Gb/s = 500 MB/s

Gen2 x16: 5.0 GT/s × 16 × 8/10 = 64 Gb/s = 8 GB/s
```

**优缺点**：
- ✅ 成熟可靠，易于实现
- ✅ 良好的时钟恢复特性
- ❌ 20% 编码开销
- ❌ 限制了高速性能

### 128b/130b 编码 (Gen 3/4/5)

**原理**：
```
每 128 位数据添加 2 位同步头
编码效率 = 128/130 = 98.5%

同步头模式：
01b = 数据块
10b = 控制块

不需要 Running Disparity
使用加扰器 (Scrambler) 保证转换密度
```

**带宽计算**：
```
Gen3 x1: 8.0 GT/s × 128/130 = 7.88 Gb/s ≈ 985 MB/s
Gen4 x1: 16.0 GT/s × 128/130 = 15.75 Gb/s ≈ 1.97 GB/s
Gen5 x1: 32.0 GT/s × 128/130 = 31.5 Gb/s ≈ 3.94 GB/s

Gen4 x16: 16.0 GT/s × 16 × 128/130 = 252 Gb/s ≈ 31.5 GB/s
```

**优势**：
- ✅ 编码效率高（98.5%）
- ✅ 低延迟
- ✅ 适合高速传输
- ❌ 需要加扰器和块锁定

### 242b/256b 编码 (Gen 6)

**原理**：
```
FLIT (Flow Control Unit) 模式
每 256 位包含：
- 242 位有效数据
- 8 位 CRC
- 6 位控制信息

编码效率 = 242/256 = 94.5%

增强的错误检测和前向纠错能力
```

**带宽计算**：
```
Gen6 x1: 64.0 GT/s × 242/256 = 60.5 Gb/s ≈ 6.06 GB/s
Gen6 x16: 64.0 GT/s × 16 × 242/256 = 968 Gb/s ≈ 121 GB/s

理论 vs 实际：
理论带宽考虑协议开销后约为有效带宽的 85-95%
```

## 速度协商

### 链路训练序列

```
速度协商过程：

1. 初始速度: Gen1 (2.5 GT/s)
   所有 PCIe 设备必须支持 Gen1

2. 交换能力: 通过 TS1/TS2 报告最高支持速度
   
3. 速度切换:
   Gen1 训练完成 → 进入 Recovery
   → 切换到 Gen2 → 重新训练
   → Gen2 完成 → 切换到 Gen3 ...

4. 最终速度: 双方都支持的最高速度
```

### Link Capabilities 寄存器

```
PCIe Capability + 0x0C: Link Capabilities

Bits [3:0]: Max Link Speed
  0x1 = 2.5 GT/s (Gen1)
  0x2 = 5.0 GT/s (Gen2)
  0x3 = 8.0 GT/s (Gen3)
  0x4 = 16.0 GT/s (Gen4)
  0x5 = 32.0 GT/s (Gen5)
  0x6 = 64.0 GT/s (Gen6)
```

### Link Control 2 寄存器

```
PCIe Capability + 0x30: Link Control 2

Bits [3:0]: Target Link Speed
  软件可以写入目标速度
  触发速度变更

Bit 4: Enter Compliance
  进入合规模式（测试用）

Bit 6: Hardware Autonomous Speed Disable
  禁用硬件自动降速
```

## 带宽计算公式

### 通用公式

```
理论带宽 (GB/s) = 速率 (GT/s) × Lane数 × 编码效率 / 8

编码效率：
Gen1/2:    8/10 = 0.8
Gen3/4/5:  128/130 = 0.9846
Gen6:      242/256 = 0.9453

示例：
Gen3 x4 = 8 × 4 × 128/130 / 8 = 3.94 GB/s
Gen4 x8 = 16 × 8 × 128/130 / 8 = 15.75 GB/s
```

### 有效带宽

```
实际应用中需要考虑协议开销：

有效带宽 = 理论带宽 × 效率因子

效率因子（典型值）：
- TLP Header 开销: ~5-10%
- ACK/NAK DLLP: ~2-3%
- 流控制开销: ~1-2%
- 其他开销: ~2-5%

总效率因子: 85-92%

示例：
Gen3 x4 理论: 3.94 GB/s
Gen3 x4 实际: 3.94 × 0.9 ≈ 3.5 GB/s
```

## 速度限制因素

### 物理限制

1. **信号完整性**
   - 高频损耗
   - 反射和串扰
   - PCB 走线质量

2. **时钟恢复**
   - 需要足够的边沿转换
   - 抖动和相位噪声

3. **功耗和散热**
   - 高速度意味着高功耗
   - SerDes 功耗增加

### 软件限制

```bash
# BIOS 可能限制速度
# 操作系统可能限制速度

# 查看实际速度
lspci -vv | grep "LnkSta:"

# 强制设置速度
setpci -s 01:00.0 CAP_EXP+30.W=0043  # Gen3
setpci -s 01:00.0 CAP_EXP+30.W=0044  # Gen4
```

## 速度测试

### 带宽测试

```bash
# NVMe SSD 测试
fio --name=test --filename=/dev/nvme0n1 --rw=read \
    --bs=128k --iodepth=32 --direct=1 --runtime=60

# 网卡测试
iperf3 -c server_ip -t 60

# 理论带宽对比
Gen3 x4: ~3.5 GB/s 实际读写
Gen4 x4: ~7.0 GB/s 实际读写
```

## 向后兼容

### 兼容性规则

```
PCIe 完全向后兼容：

Gen6 设备 可以工作在 Gen1 插槽
Gen1 设备 可以工作在 Gen6 插槽

协商结果：
双方支持的最高共同速度

示例：
Gen4 设备 + Gen3 插槽 = Gen3 速度
Gen2 设备 + Gen5 插槽 = Gen2 速度
```

## 代码示例

### 读取链路速度

```c
static void check_link_speed(struct pci_dev *pdev)
{
    int pos;
    u32 link_cap;
    u16 link_sta;
    
    pos = pci_pcie_cap(pdev);
    
    // 读取最大支持速度
    pci_read_config_dword(pdev, pos + PCI_EXP_LNKCAP, &link_cap);
    int max_speed = link_cap & 0xF;
    
    // 读取当前速度
    pci_read_config_word(pdev, pos + PCI_EXP_LNKSTA, &link_sta);
    int cur_speed = link_sta & 0xF;
    
    const char *speed_str[] = {
        "Unknown", "2.5GT/s", "5GT/s", "8GT/s",
        "16GT/s", "32GT/s", "64GT/s"
    };
    
    dev_info(&pdev->dev, "Max: %s, Current: %s\n",
             speed_str[max_speed], speed_str[cur_speed]);
}
```

## 总结

PCIe 速度和编码的关键点：

1. **速度演进**：2.5 GT/s → 64 GT/s
2. **编码效率**：8b/10b (80%) → 128b/130b (98.5%)
3. **带宽计算**：速率 × Lane × 效率
4. **向后兼容**：自动协商最高共同速度
5. **实际性能**：考虑协议开销（85-92%）

参见：
- [PCIe 概述](../introduction/overview.md)
- [链路宽度](link-width.md)
- [编码方案](../physical-layer/encoding.md)
- [链路训练](../physical-layer/link-training.md)
