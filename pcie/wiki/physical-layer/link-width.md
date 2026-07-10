# 链路宽度 (Link Width)

## 概述

PCIe 链路宽度（Link Width）指的是在两个设备之间并行使用的 Lane（通道）数量。链路可以由 x1、x2、x4、x8、x16 或 x32 个 Lane 组成，每个 Lane 都是一个独立的差分信号对，支持全双工通信。链路宽度是决定 PCIe 链路总带宽的关键参数之一。

## 支持的链路宽度

### 标准链路宽度配置

PCIe 规范定义了以下标准链路宽度：

```
支持的链路宽度：
- x1:  1 个 Lane  (最小配置)
- x2:  2 个 Lane
- x4:  4 个 Lane  (常见于 NVMe SSD)
- x8:  8 个 Lane  (常见于高性能网卡)
- x16: 16 个 Lane (常见于显卡)
- x32: 32 个 Lane (PCIe 6.0+，服务器应用)
```

### 物理连接

每个 Lane 由 4 根信号线组成（2 对差分信号）：

```
单个 Lane 的信号：
+--------+
| TX+    | ← 发送正信号
| TX-    | ← 发送负信号
| RX+    | ← 接收正信号
| RX-    | ← 接收负信号
+--------+

x4 链路示例：
设备 A                           设备 B
+-------+                       +-------+
| Lane0 |<--------------------->| Lane0 |
| Lane1 |<--------------------->| Lane1 |
| Lane2 |<--------------------->| Lane2 |
| Lane3 |<--------------------->| Lane3 |
+-------+                       +-------+
共 16 根信号线 (4 Lane × 4 信号/Lane)
```

### 链路宽度与带宽关系

链路的总带宽等于单 Lane 带宽乘以 Lane 数量：

| 链路宽度 | Gen 3 带宽 | Gen 4 带宽 | Gen 5 带宽 | 信号线数 |
|---------|-----------|-----------|-----------|---------|
| x1      | ~985 MB/s | ~1.97 GB/s | ~3.94 GB/s | 4 线 |
| x2      | ~1.97 GB/s | ~3.94 GB/s | ~7.88 GB/s | 8 线 |
| x4      | ~3.94 GB/s | ~7.88 GB/s | ~15.75 GB/s | 16 线 |
| x8      | ~7.88 GB/s | ~15.75 GB/s | ~31.51 GB/s | 32 线 |
| x16     | ~15.75 GB/s | ~31.51 GB/s | ~63.01 GB/s | 64 线 |
| x32     | ~31.51 GB/s | ~63.01 GB/s | ~126.03 GB/s | 128 线 |

```c
// 带宽计算公式
float calculate_bandwidth_gbps(int gen, int width) {
    // 每代的单 Lane 速率 (GT/s)
    const float data_rate[] = {
        0,      // 索引 0 未使用
        2.5,    // Gen 1
        5.0,    // Gen 2
        8.0,    // Gen 3
        16.0,   // Gen 4
        32.0,   // Gen 5
        64.0    // Gen 6
    };
    
    // 编码效率
    float encoding_efficiency;
    if (gen <= 2) {
        encoding_efficiency = 0.8;  // 8b/10b = 80%
    } else {
        encoding_efficiency = 0.9846;  // 128b/130b ≈ 98.46%
    }
    
    // 总带宽 (Gbps) = 速率 × 宽度 × 效率
    return data_rate[gen] * width * encoding_efficiency;
}

// 示例
float bw_gen4_x4 = calculate_bandwidth_gbps(4, 4);
printf("Gen 4 x4 带宽: %.2f Gbps\n", bw_gen4_x4);  // 输出: 15.75 Gbps
```

## 链路宽度协商

### 协商机制

PCIe 设备在链路训练（Link Training）过程中自动协商链路宽度：

```
链路宽度协商流程：
+------------------+
| 1. 检测阶段      |
|    (Detect)      | ← 检测对端是否存在
+------------------+
         ↓
+------------------+
| 2. 轮询阶段      |
|    (Polling)     | ← 确定可用的 Lane 数量
+------------------+
         ↓
+------------------+
| 3. 配置阶段      |
|    (Config)      | ← 协商最终链路宽度
+------------------+
         ↓
+------------------+
| 4. L0 状态       |
|    (运行)        | ← 以协商的宽度运行
+------------------+
```

### 协商原则

协商遵循以下原则：

1. **最大能力匹配**：选择双方都支持的最大宽度
2. **降级支持**：如果部分 Lane 失败，自动降级到较窄的宽度
3. **对称性**：上行和下行使用相同的宽度

```c
// 链路宽度协商示例
typedef struct {
    uint8_t max_width;      // 最大支持宽度
    uint8_t trained_width;  // 实际训练宽度
    bool    upconfigure_capable;  // 是否支持宽度升级
} link_width_negotiation_t;

uint8_t negotiate_link_width(uint8_t local_max, uint8_t remote_max) {
    // 选择双方都支持的最大宽度
    uint8_t negotiated = (local_max < remote_max) ? local_max : remote_max;
    
    // 验证宽度是否有效（必须是 1, 2, 4, 8, 16, 32）
    const uint8_t valid_widths[] = {1, 2, 4, 8, 16, 32};
    bool valid = false;
    
    for (int i = 0; i < 6; i++) {
        if (negotiated == valid_widths[i]) {
            valid = true;
            break;
        }
    }
    
    if (!valid) {
        // 降级到下一个有效宽度
        for (int i = 5; i >= 0; i--) {
            if (valid_widths[i] < negotiated) {
                negotiated = valid_widths[i];
                break;
            }
        }
    }
    
    return negotiated;
}
```

### Lane 反转 (Lane Reversal)

PCIe 支持 Lane 反转，以适应某些物理连接：

```
正常连接：
设备 A                    设备 B
Lane 0 -----------------> Lane 0
Lane 1 -----------------> Lane 1
Lane 2 -----------------> Lane 2
Lane 3 -----------------> Lane 3

反转连接：
设备 A                    设备 B
Lane 0 -----------------> Lane 3
Lane 1 -----------------> Lane 2
Lane 2 -----------------> Lane 1
Lane 3 -----------------> Lane 0

硬件自动检测并适应反转情况
```

## 配置空间寄存器

### Link Capabilities Register

链路能力寄存器（偏移 0xC）报告设备支持的最大链路宽度：

```
Link Capabilities Register (Offset 0xC):
+--------------------------------------------------+
| 31:10 | 9:4 | 3:0                                 |
|-------|-----|-------------------------------------|
| ...   | MLW | Maximum Link Width                  |
+--------------------------------------------------+

MLW (Maximum Link Width) 编码：
0x01 = x1
0x02 = x2
0x04 = x4
0x08 = x8
0x0C = x12  (不常见)
0x10 = x16
0x20 = x32
```

```c
// 读取最大链路宽度
#define PCI_EXP_LNKCAP          0x0C
#define PCI_EXP_LNKCAP_MLW_MASK 0x3F0
#define PCI_EXP_LNKCAP_MLW_SHIFT 4

uint8_t read_max_link_width(uint16_t bus, uint8_t dev, uint8_t func) {
    uint32_t lnkcap = pci_read_config_dword(bus, dev, func, 
                                            PCI_EXP_LNKCAP);
    
    uint8_t mlw = (lnkcap & PCI_EXP_LNKCAP_MLW_MASK) >> 
                  PCI_EXP_LNKCAP_MLW_SHIFT;
    
    // 将编码转换为实际宽度
    switch (mlw) {
        case 0x01: return 1;
        case 0x02: return 2;
        case 0x04: return 4;
        case 0x08: return 8;
        case 0x0C: return 12;
        case 0x10: return 16;
        case 0x20: return 32;
        default:   return 0;  // 无效
    }
}
```

### Link Status Register

链路状态寄存器（偏移 0x12）报告当前协商的链路宽度：

```
Link Status Register (Offset 0x12):
+--------------------------------------------------+
| 15:10 | 9:4 | 3:0                                 |
|-------|-----|-------------------------------------|
| ...   | NLW | Negotiated Link Width               |
+--------------------------------------------------+

NLW (Negotiated Link Width) 编码：
与 MLW 相同的编码方式
```

```c
// 读取当前链路宽度
#define PCI_EXP_LNKSTA          0x12
#define PCI_EXP_LNKSTA_NLW_MASK 0x3F0
#define PCI_EXP_LNKSTA_NLW_SHIFT 4

uint8_t read_current_link_width(uint16_t bus, uint8_t dev, uint8_t func) {
    uint16_t lnksta = pci_read_config_word(bus, dev, func, 
                                           PCI_EXP_LNKSTA);
    
    uint8_t nlw = (lnksta & PCI_EXP_LNKSTA_NLW_MASK) >> 
                  PCI_EXP_LNKSTA_NLW_SHIFT;
    
    // 将编码转换为实际宽度
    switch (nlw) {
        case 0x01: return 1;
        case 0x02: return 2;
        case 0x04: return 4;
        case 0x08: return 8;
        case 0x0C: return 12;
        case 0x10: return 16;
        case 0x20: return 32;
        default:   return 0;
    }
}
```

### Link Control Register

链路控制寄存器可用于触发链路重训练：

```c
// 触发链路重训练
#define PCI_EXP_LNKCTL           0x10
#define PCI_EXP_LNKCTL_RL        0x20  // Retrain Link

void retrain_link(uint16_t bus, uint8_t dev, uint8_t func) {
    uint16_t lnkctl = pci_read_config_word(bus, dev, func, 
                                           PCI_EXP_LNKCTL);
    
    // 设置 Retrain Link 位
    lnkctl |= PCI_EXP_LNKCTL_RL;
    pci_write_config_word(bus, dev, func, PCI_EXP_LNKCTL, lnkctl);
    
    // 等待训练完成
    uint16_t lnksta;
    do {
        lnksta = pci_read_config_word(bus, dev, func, PCI_EXP_LNKSTA);
    } while (lnksta & 0x800);  // Link Training bit
    
    printf("链路重训练完成\n");
}
```

## 动态链路宽度管理

### 链路宽度降级

当部分 Lane 出现故障时，链路可以降级到较窄的宽度继续运行：

```
降级场景：
初始: x16 链路，所有 Lane 正常
     [0][1][2][3][4][5][6][7][8][9][10][11][12][13][14][15]
      ✓  ✓  ✓  ✓  ✓  ✓  ✓  ✓  ✓  ✓   ✓   ✓   ✓   ✓   ✓   ✓

故障: Lane 8-15 出现问题
     [0][1][2][3][4][5][6][7][8][9][10][11][12][13][14][15]
      ✓  ✓  ✓  ✓  ✓  ✓  ✓  ✓  ✗  ✗   ✗   ✗   ✗   ✗   ✗   ✗

降级: 自动降级到 x8
     [0][1][2][3][4][5][6][7]
      ✓  ✓  ✓  ✓  ✓  ✓  ✓  ✓
```

```c
// 检测链路宽度降级
typedef struct {
    uint8_t max_width;
    uint8_t current_width;
    bool    degraded;
} link_width_status_t;

link_width_status_t check_link_degradation(uint16_t bus, 
                                           uint8_t dev, 
                                           uint8_t func) {
    link_width_status_t status;
    
    status.max_width = read_max_link_width(bus, dev, func);
    status.current_width = read_current_link_width(bus, dev, func);
    status.degraded = (status.current_width < status.max_width);
    
    if (status.degraded) {
        printf("警告: 链路宽度降级检测到!\n");
        printf("  最大宽度: x%d\n", status.max_width);
        printf("  当前宽度: x%d\n", status.current_width);
        printf("  性能损失: %.1f%%\n", 
               (1.0 - (float)status.current_width / status.max_width) * 100);
    }
    
    return status;
}
```

### 链路宽度升级 (Upconfigure)

某些设备支持在运行时将链路宽度从较窄升级到较宽：

```c
// 检查是否支持 Upconfigure
#define PCI_EXP_LNKCAP2         0x2C
#define PCI_EXP_LNKCAP2_SLS     0x1E  // Supported Link Speeds Vector

bool supports_upconfigure(uint16_t bus, uint8_t dev, uint8_t func) {
    uint32_t lnkcap = pci_read_config_dword(bus, dev, func, 
                                            PCI_EXP_LNKCAP);
    
    // 检查 Upconfigure Capable 位 (bit 31)
    return (lnkcap & 0x80000000) != 0;
}

// 触发 Upconfigure
void trigger_upconfigure(uint16_t bus, uint8_t dev, uint8_t func) {
    if (!supports_upconfigure(bus, dev, func)) {
        printf("设备不支持 Upconfigure\n");
        return;
    }
    
    // 通过重训练链路尝试升级宽度
    retrain_link(bus, dev, func);
    
    uint8_t new_width = read_current_link_width(bus, dev, func);
    printf("Upconfigure 后的链路宽度: x%d\n", new_width);
}
```

## 实际应用场景

### 不同设备类型的典型链路宽度

```
设备类型          典型宽度    说明
-----------------------------------------------
NVMe SSD          x4         足够的带宽用于存储
网卡 (1/10G)      x4         适合大多数网络应用
网卡 (25/100G)    x8/x16     高性能网络
显卡 (集成)       x4/x8      入门级图形
显卡 (独立)       x16        高性能图形
RAID 卡           x8         多磁盘聚合
HBA              x8         主机总线适配器
Thunderbolt       x4         外部扩展
```

### 带宽需求分析

```python
# 计算应用所需的最小链路宽度
def calculate_required_width(bandwidth_gbps, gen):
    """
    bandwidth_gbps: 所需带宽 (GB/s)
    gen: PCIe 代数
    返回: 所需的最小链路宽度
    """
    # 单 Lane 带宽 (GB/s)
    lane_bandwidth = {
        3: 0.985,   # Gen 3
        4: 1.969,   # Gen 4
        5: 3.938    # Gen 5
    }
    
    if gen not in lane_bandwidth:
        raise ValueError("不支持的 PCIe 代数")
    
    # 计算所需 Lane 数
    required_lanes = bandwidth_gbps / lane_bandwidth[gen]
    
    # 向上取整到标准宽度
    standard_widths = [1, 2, 4, 8, 16, 32]
    for width in standard_widths:
        if width >= required_lanes:
            return width
    
    return 32  # 最大宽度

# 示例：NVMe SSD 需求分析
# 假设 SSD 顺序读取速度为 7 GB/s
required_width_gen3 = calculate_required_width(7.0, 3)
required_width_gen4 = calculate_required_width(7.0, 4)

print(f"7 GB/s SSD on Gen 3: 需要 x{required_width_gen3}")  # x8
print(f"7 GB/s SSD on Gen 4: 需要 x{required_width_gen4}")  # x4
```

### 主板插槽配置

典型的桌面主板 PCIe 插槽配置：

```
主板示例: Z690 芯片组
+----------------------------------+
| CPU PCIe Lanes (20 个 Gen 5)     |
+----------------------------------+
| Slot 1: x16 (Gen 5) ← 显卡       |
| Slot 2: x4  (Gen 4) ← NVMe/网卡  |
+----------------------------------+

+----------------------------------+
| 芯片组 PCIe Lanes (12 个 Gen 4)  |
+----------------------------------+
| Slot 3: x1  (Gen 3) ← 声卡       |
| Slot 4: x1  (Gen 3) ← Wi-Fi      |
| M.2_1:  x4  (Gen 4) ← NVMe       |
| M.2_2:  x4  (Gen 4) ← NVMe       |
+----------------------------------+

Lane 分配策略：
- 高性能设备连接到 CPU
- 低带宽设备连接到芯片组
- 某些插槽可能共享 Lane (互斥使用)
```

## 调试和诊断

### Linux 下查看链路宽度

```bash
# 使用 lspci 查看链路信息
lspci -vvv -s 01:00.0 | grep -E "LnkCap|LnkSta"

# 输出示例：
# LnkCap: Port #0, Speed 16GT/s, Width x16, ASPM L1, Exit Latency L1 <4us
# LnkSta: Speed 16GT/s (ok), Width x16 (ok)

# 检查所有 PCIe 设备的链路宽度
for dev in $(lspci -d ::0604 | cut -d' ' -f1); do
    echo "=== Device $dev ==="
    lspci -vvv -s $dev | grep -E "LnkCap|LnkSta"
done
```

### 使用 sysfs 接口

```bash
# 读取当前链路宽度
cat /sys/bus/pci/devices/0000:01:00.0/current_link_width

# 读取最大链路宽度
cat /sys/bus/pci/devices/0000:01:00.0/max_link_width

# 触发链路重训练 (需要 root 权限)
echo 1 > /sys/bus/pci/devices/0000:01:00.0/reset
```

### 链路宽度监控工具

```c
// 持续监控链路宽度
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void monitor_link_width(uint16_t bus, uint8_t dev, uint8_t func) {
    uint8_t last_width = 0;
    
    printf("开始监控链路宽度...\n");
    
    while (1) {
        uint8_t current_width = read_current_link_width(bus, dev, func);
        
        if (current_width != last_width) {
            time_t now = time(NULL);
            printf("[%s] 链路宽度变化: x%d -> x%d\n",
                   ctime(&now), last_width, current_width);
            
            if (current_width < last_width) {
                printf("  警告: 检测到链路降级!\n");
            } else if (current_width > last_width) {
                printf("  链路已升级\n");
            }
            
            last_width = current_width;
        }
        
        sleep(1);  // 每秒检查一次
    }
}
```

## 性能优化建议

### 最大化链路带宽

1. **使用适当的插槽**：
   - 高带宽设备使用 x16 或 x8 插槽
   - 确保插槽直连 CPU 而非芯片组

2. **避免 Lane 共享**：
   - 检查主板手册了解 Lane 分配
   - 某些 M.2 插槽可能与 SATA 或其他插槽共享

3. **定期检查链路状态**：
   - 监控链路宽度是否降级
   - 检查链路训练错误

4. **优化 PCB 设计**：
   - 确保所有 Lane 等长
   - 良好的信号完整性

### 功耗与性能权衡

```c
// 根据负载动态调整链路宽度
typedef enum {
    LINK_WIDTH_POLICY_MAX_PERF,     // 最大性能
    LINK_WIDTH_POLICY_BALANCED,     // 平衡
    LINK_WIDTH_POLICY_MAX_POWER_SAVE // 最大节能
} link_width_policy_t;

void apply_link_width_policy(uint16_t bus, uint8_t dev, uint8_t func,
                             link_width_policy_t policy) {
    uint8_t max_width = read_max_link_width(bus, dev, func);
    uint8_t target_width;
    
    switch (policy) {
        case LINK_WIDTH_POLICY_MAX_PERF:
            target_width = max_width;
            break;
            
        case LINK_WIDTH_POLICY_BALANCED:
            // 使用一半的 Lane
            target_width = max_width / 2;
            if (target_width < 1) target_width = 1;
            break;
            
        case LINK_WIDTH_POLICY_MAX_POWER_SAVE:
            // 使用最小宽度
            target_width = 1;
            break;
    }
    
    printf("应用链路宽度策略: 目标宽度 x%d\n", target_width);
    // 注意: 实际实现需要设备支持动态宽度调整
}
```

## 常见问题

### 链路宽度降级的原因

1. **硬件故障**：
   - Lane 信号完整性问题
   - 接触不良
   - PCB 走线损坏

2. **配置问题**：
   - 插槽物理限制（x16 插槽可能只有 x8 电气连接）
   - BIOS 设置限制

3. **热插拔**：
   - 热插拔过程中的暂时降级

### 如何修复链路宽度问题

```bash
# 1. 检查物理连接
# 确保设备完全插入插槽

# 2. 清洁金手指
# 使用橡皮擦轻擦设备金手指

# 3. 更新 BIOS/固件
# 访问主板厂商网站下载最新 BIOS

# 4. 检查 BIOS 设置
# 进入 BIOS，确保 PCIe 设置为 Auto 或最大宽度

# 5. 尝试不同的插槽
# 测试设备在其他插槽中是否正常

# 6. 触发链路重训练
setpci -s 01:00.0 CAP_EXP+10.w=0020:0020
```

## 总结

PCIe 链路宽度是影响系统性能的关键参数：

**关键要点**：
- 链路宽度从 x1 到 x32，总带宽与 Lane 数量成正比
- 链路训练时自动协商宽度，选择双方支持的最大值
- 支持动态降级以适应 Lane 故障
- 通过配置空间寄存器读取和监控链路宽度
- 不同设备类型有不同的典型宽度需求

**最佳实践**：
- 根据设备带宽需求选择合适的插槽
- 定期监控链路状态，及时发现降级
- 优化系统配置以最大化可用带宽
- 理解主板 Lane 分配避免资源冲突

理解链路宽度的工作原理对于 PCIe 系统设计、故障诊断和性能优化至关重要。
