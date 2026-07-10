# 物理层编码方案

PCIe 的物理层编码是确保数据可靠传输的关键技术，从 Gen 1 的 8b/10b 编码演进到 Gen 3+ 的 128b/130b 编码，在保证时钟恢复和 DC 平衡的同时大幅提升了传输效率。

---

## 概述

PCIe 物理层编码的主要目标：
- **时钟恢复**：接收方从数据流中提取时钟信号
- **DC 平衡**：保持直流分量接近零，避免基线漂移
- **错误检测**：通过特殊编码检测传输错误
- **带内信令**：使用特殊字符传递控制信息
- **带宽效率**：减少编码开销，提高有效传输率

**不同代的编码方案**：

| PCIe Gen | 编码方案 | 信号速率 | 有效带宽 | 编码效率 |
|----------|---------|---------|---------|---------|
| Gen 1    | 8b/10b  | 2.5 GT/s | 2.0 Gb/s | 80% |
| Gen 2    | 8b/10b  | 5.0 GT/s | 4.0 Gb/s | 80% |
| Gen 3    | 128b/130b | 8.0 GT/s | ~7.88 Gb/s | 98.46% |
| Gen 4    | 128b/130b | 16.0 GT/s | ~15.75 Gb/s | 98.46% |
| Gen 5    | 128b/130b | 32.0 GT/s | ~31.51 Gb/s | 98.46% |

**编码在协议栈中的位置**：
```
┌─────────────────────────────────┐
│  Transaction Layer              │
│  生成 TLP                       │
└─────────────┬───────────────────┘
              │ TLP
┌─────────────▼───────────────────┐
│  Data Link Layer                │
│  添加 Seq# + LCRC               │
└─────────────┬───────────────────┘
              │ DLLP / TLP
┌─────────────▼───────────────────┐
│  Physical Layer                 │
│  ┌──────────────────────────┐   │
│  │ 1. Scrambling (扰码)     │   │
│  │ 2. 8b/10b or 128b/130b   │◄──── 编码在这里
│  │ 3. 串行化 Serialization  │   │
│  └──────────────────────────┘   │
└─────────────┬───────────────────┘
              │ 差分信号
              ▼
        PCIe Lane (物理线路)
```

---

## 8b/10b 编码详解（Gen 1-2）

### 基本原理

8b/10b 编码将 **8 位数据** 编码为 **10 位符号**，IBM 在 1983 年发明，广泛用于高速串行通信。

**编码步骤**：
```
原始 8 位数据
    │
    ├─> 分为两部分
    │
┌───┴────────────────┐
│                    │
▼                    ▼
5 位 (EDCBA)        3 位 (HGF)
│                    │
├─> 5b/6b 编码       ├─> 3b/4b 编码
│                    │
▼                    ▼
6 位 (abcdei)       4 位 (fghj)
│                    │
└────────┬───────────┘
         │
         ▼
    10 位符号 (abcdeifghj)
```

**示例编码**：
```
输入: 0xBC (10111100)
  5 位: 11100 (0x1C)
  3 位: 101 (0x5)

查表:
  5b/6b: 11100 → 001111 (负向运行不等式)
  3b/4b: 101 → 1010

输出: 0011111010 (10 位)
```

### 特殊字符（K-codes）

8b/10b 定义了 **12 个特殊控制字符**（K-codes），用于帧边界和链路管理。

**常用 K-codes**：

| 符号 | 名称 | 10b 编码 | PCIe 用途 |
|------|------|---------|----------|
| K28.0 | COM | 001111 0100 | TS1/TS2 训练序列开始 |
| K28.1 | FTS | 001111 1001 | Fast Training Sequence |
| K28.2 | STP | 001111 0101 | TLP 起始标记 |
| K28.3 | SDP | 001111 0011 | DLLP 起始标记 |
| K28.5 | IDL | 001111 1010 | 空闲字符 |
| K28.7 | END | 001111 1000 | 包结束标记 |
| K23.7 | EDB | 111010 1000 | 错误结束（Bad End） |
| K27.7 | SKP | 110110 1000 | Skip 字符（时钟补偿） |
| K30.7 | PAD | 011110 1000 | 链路训练填充 |

**TLP 帧结构示例**：
```
┌─────┬──────┬──────────────────┬─────┬─────┐
│ STP │ TLP  │   TLP Payload    │ END │ IDL │
│K28.2│Header│   (数据字节)      │K28.7│K28.5│
└─────┴──────┴──────────────────┴─────┴─────┘
  1个   12-16      0-4096 字节      1个   填充
 符号   字节                        符号
```

**DLLP 帧结构示例**：
```
┌─────┬──────┬──────┬─────┐
│ SDP │ DLLP │ CRC  │ END │
│K28.3│ 2DW  │ 2B   │K28.7│
└─────┴──────┴──────┴─────┘
```

### Running Disparity（运行不等式）

**定义**：当前符号中 1 和 0 的累积差值，用于维持 DC 平衡。

**机制**：
- 每个 8b/10b 符号有两种编码（正向和负向）
- 根据当前 RD 选择编码，使 RD 保持在 0 附近
- RD 状态：**RD-**（负）或 **RD+**（正）

**示例**：
```
数据: D10.2 (0x4A)

RD- 编码: 010101 0101 (5 个 1, 5 个 0) → 切换到 RD+
RD+ 编码: 101010 1010 (5 个 1, 5 个 0) → 切换到 RD-

编码器根据当前 RD 选择编码，接收器验证 RD 转换
如果 RD 错误 → 检测到编码错误
```

### 时钟恢复

8b/10b 保证 **最多 5 个连续相同位**，确保频繁跳变：

```
最坏情况（5 个连续位）：
  000001111100000...
  ↑    ↑    ↑
  跳变 跳变  跳变

接收器使用 PLL 锁定这些跳变，提取时钟
```

### 错误检测

8b/10b 可检测以下错误：
1. **非法符号**：接收到不在编码表中的 10 位模式
2. **RD 违规**：Running Disparity 不连续
3. **K-code 误用**：在错误位置出现控制字符

---

## 128b/130b 编码详解（Gen 3-5）

### 为什么需要新编码？

**8b/10b 的问题**：
- **20% 带宽损失**：每 10 位只传输 8 位数据
- Gen 3 目标 8 GT/s，使用 8b/10b 需要 10 GT/s 信号速率
- 高速信号的物理实现困难且成本高

**128b/130b 优势**：
- **仅 1.54% 开销**：98.46% 效率
- 简单实现，无需复杂查表
- 适合高速率（Gen 3 及以上）

### 编码原理

128b/130b 将 **128 位数据**添加 **2 位同步头**，形成 **130 位符号块**。

**格式**：
```
┌────┬─────────────────────────────────────┐
│Sync│       128-bit Data Block           │
│2bit│            (16 字节)                │
└────┴─────────────────────────────────────┘
 130 位符号块

Sync Header:
  10b → Data Block（所有字节是数据）
  01b → Ordered Set Block（包含控制信息）
```

**Data Block（Sync=10b）**：
```
┌──┬────┬────┬────┬───┬────┐
│10│ D0 │ D1 │ D2 │...│D15 │
└──┴────┴────┴────┴───┴────┘
   16 个数据字节（纯数据）
```

**Ordered Set Block（Sync=01b）**：
```
┌──┬──────┬─────────────────┐
│01│ Type │   OS Payload    │
│  │ 8bit │   120 bits      │
└──┴──────┴─────────────────┘

Type 字段定义 Ordered Set 类型:
  0x00: EIOS (Electrical Idle Ordered Set)
  0x1E: SDS (Start Data Stream)
  0x2D: SKP (Skip Ordered Set)
  0x4A: EIEOS (Electrical Idle Exit Ordered Set)
  0x55: STP (Start TLP)
  0x5C: SDP (Start DLLP)
  0x66: COM (训练序列)
  0xF7: END (包结束)
  0xFE: EDB (错误结束)
  ...
```

### TLP/DLLP 帧结构

**TLP 示例**（跨越多个 Block）：
```
Block N (Sync=01, STP):
┌──┬────┬──────────────────────────────┐
│01│STP │ TLP Header (12 字节) + ...   │
└──┴────┴──────────────────────────────┘

Block N+1 (Sync=10, Data):
┌──┬──────────────────────────────────┐
│10│   TLP Payload (16 字节)          │
└──┴──────────────────────────────────┘

Block N+2 (Sync=10, Data):
┌──┬──────────────────────────────────┐
│10│   TLP Payload (16 字节)          │
└──┴──────────────────────────────────┘

Block N+3 (Sync=01, END):
┌──┬────┬──────────────────────────────┐
│01│END │ 剩余 Payload + LCRC          │
└──┴────┴──────────────────────────────┘
```

### Scrambling（扰码）

128b/130b **必须配合 Scrambling** 使用，解决以下问题：
- 长串 0 或 1 导致时钟恢复困难
- EMI（电磁干扰）问题
- 数据相关抖动

**LFSR Scrambler（线性反馈移位寄存器）**：
```
多项式: G(X) = X^23 + X^21 + X^16 + X^8 + X^5 + X^2 + 1

┌────────────────────────────────────────┐
│  23-bit LFSR                           │
│  ┌──┬──┬──┬───┬──┬──┬───┬──┬──┬──┐   │
│  │22│21│20│...│16│..│ 8│..│ 2│ 1│    │
│  └┬─┴┬─┴──┴───┴┬─┴──┴┬─┴──┴┬─┴──┘    │
│   │  │         │     │     │          │
│   └──┴─────────┴─────┴─────┴─> XOR   │
└────────────────────────────────────────┘
                                  ↓
                            Scrambled Data
                                  ↓
                         Data XOR LFSR_output

初始值: 0xFFFF (全1)
每 130 位块重新初始化（保持同步）
```

**效果**：
```
原始数据: 00000000 00000000 00000000 ...
LFSR 输出: 10110101 01110010 11001101 ...
扰码结果: 10110101 01110010 11001101 ... (伪随机)
```

### 时钟补偿（SKP Ordered Set）

由于发送和接收时钟略有偏差，需要定期插入/删除 SKP 字符：

```
发送器每 ~1180 个符号插入 SKP Ordered Set:
┌────┬────┬────┬────┬────┬────┬────┬────┐
│Data│Data│SKP │Data│Data│SKP │Data│Data│
└────┴────┴────┴────┴────┴────┴────┴────┘

接收器根据 FIFO 状态添加/删除 SKP:
  FIFO 快满 → 删除 SKP（接收时钟慢）
  FIFO 快空 → 复制 SKP（接收时钟快）
```

---

## 编码效率对比

### 带宽计算

**Gen 2 (8b/10b, 5.0 GT/s)**：
```
信号速率: 5.0 GT/s (Giga Transfers/秒)
有效速率: 5.0 × (8/10) = 4.0 Gb/s
x16 通道: 4.0 × 16 = 64 Gb/s = 8 GB/s (每方向)
```

**Gen 3 (128b/130b, 8.0 GT/s)**：
```
信号速率: 8.0 GT/s
有效速率: 8.0 × (128/130) = 7.877 Gb/s
x16 通道: 7.877 × 16 = 126 Gb/s ≈ 15.75 GB/s (每方向)
```

**对比表**：

| 项目 | 8b/10b (Gen 2) | 128b/130b (Gen 3) | 提升 |
|------|----------------|-------------------|------|
| 编码开销 | 20% | 1.54% | 92% 减少 |
| x16 带宽 | 8 GB/s | 15.75 GB/s | 96.9% |
| 编码复杂度 | 高（查表） | 低（简单封装） | - |
| 时钟恢复 | 编码保证 | Scrambling 保证 | - |
| 错误检测 | RD + 非法符号 | CRC + Sync 检测 | - |

### 实际性能影响

**延迟**：
- 8b/10b：编码/解码增加 ~2 个符号周期
- 128b/130b：块对齐增加 ~130 位周期

**功耗**：
- Gen 3 的 Scrambler 增加轻微功耗
- 但更高效率减少总传输时间，实际节能

---

## 实现细节

### FEMU 中的编码模拟

FEMU 作为功能模拟器，**不模拟物理层编码细节**，但体现概念：

```c
// hw/pci/pcie.c
// PCIe 链路速度定义
#define PCIE_LINK_SPEED_2_5 0x1  // Gen 1
#define PCIE_LINK_SPEED_5   0x2  // Gen 2
#define PCIE_LINK_SPEED_8   0x3  // Gen 3
#define PCIE_LINK_SPEED_16  0x4  // Gen 4

// 链路训练（抽象）
static void pcie_cap_slot_plug_cb(HotplugHandler *hotplug_dev,
                                   DeviceState *dev, Error **errp)
{
    // 设置链路速度和宽度
    PCIDevice *pci_dev = PCI_DEVICE(dev);
    pcie_cap_slot_plug_common(hotplug_dev, dev, errp);
    
    // Gen 3 设备会协商到 8.0 GT/s
    // 实际系统中会进行 8b/10b 或 128b/130b 编码
}
```

### 链路速度协商

```c
// hw/pci/pcie.c
void pcie_cap_slot_plug_common(HotplugHandler *hotplug_dev,
                                DeviceState *dev, Error **errp)
{
    PCIDevice *hotplug_pdev = PCI_DEVICE(hotplug_dev);
    uint8_t *exp_cap = hotplug_pdev->config + hotplug_pdev->exp.exp_cap;
    uint16_t sltsta = pci_get_word(exp_cap + PCI_EXP_SLTSTA);
    
    // 链路训练开始（实际硬件会执行 8b/10b 同步）
    sltsta |= PCI_EXP_SLTSTA_PDS;  // Presence Detect State
    pci_set_word(exp_cap + PCI_EXP_SLTSTA, sltsta);
    
    // 链路速度在 Link Status 寄存器中报告
    // Bit 3:0 = Current Link Speed
    //   0001b = 2.5 GT/s (Gen 1, 8b/10b)
    //   0010b = 5.0 GT/s (Gen 2, 8b/10b)
    //   0011b = 8.0 GT/s (Gen 3, 128b/130b)
}
```

### 带宽计算辅助函数

```c
// 计算不同 Gen 的实际带宽
static uint64_t pcie_lane_bandwidth(uint8_t gen, uint8_t lanes)
{
    uint64_t rate_mbps;  // 每通道速率 (Mb/s)
    
    switch (gen) {
    case 1:  // Gen 1: 2.5 GT/s × 8/10 = 2.0 Gb/s
        rate_mbps = 2000;
        break;
    case 2:  // Gen 2: 5.0 GT/s × 8/10 = 4.0 Gb/s
        rate_mbps = 4000;
        break;
    case 3:  // Gen 3: 8.0 GT/s × 128/130 = 7.877 Gb/s
        rate_mbps = 7877;
        break;
    case 4:  // Gen 4: 16.0 GT/s × 128/130 = 15.754 Gb/s
        rate_mbps = 15754;
        break;
    case 5:  // Gen 5: 32.0 GT/s × 128/130 = 31.508 Gb/s
        rate_mbps = 31508;
        break;
    default:
        return 0;
    }
    
    // 总带宽 = 单通道速率 × 通道数 × 2 (全双工)
    return rate_mbps * lanes;
}

// 使用示例
uint64_t bw = pcie_lane_bandwidth(3, 16);  // Gen 3 x16
printf("Bandwidth: %lu Mb/s (%.2f GB/s per direction)\n", 
       bw, bw / 8000.0);
// 输出: Bandwidth: 126032 Mb/s (15.75 GB/s per direction)
```

---

## 实用技巧

### 调试链路问题

**查看当前链路速度和编码**：
```bash
# 查看 PCIe 设备链路状态
lspci -vvv -s 01:00.0 | grep -i "lnkcap\|lnksta"
  LnkCap: Port #0, Speed 8GT/s, Width x16
  LnkSta: Speed 8GT/s, Width x16

# Speed 8GT/s → Gen 3 → 使用 128b/130b 编码
# Speed 5GT/s → Gen 2 → 使用 8b/10b 编码
```

**检查链路训练错误**：
```bash
# 查看 AER 错误（可能由编码错误导致）
lspci -vvv -s 01:00.0 | grep -A10 "Advanced Error Reporting"
  UESta:  DLP- SDES- TLP- FCP- CmpltTO- ...
  
# DLP (Data Link Protocol Error) 可能指示 8b/10b 错误
# TLP 错误可能由 Scrambling 或编码问题引起
```

### 性能优化

**强制特定 Gen**（用于测试）：
```bash
# 限制为 Gen 2 (5.0 GT/s, 8b/10b)
setpci -s 01:00.0 CAP_EXP+30.w=0021  # Target Link Speed = Gen 2

# 触发重新训练
setpci -s 01:00.0 CAP_EXP+10.w=0020  # Retrain Link

# 验证
lspci -vvv -s 01:00.0 | grep "LnkSta.*Speed"
  LnkSta: Speed 5GT/s (downgraded), Width x16
```

**对比不同编码的延迟**：
```bash
# Gen 2 vs Gen 3 延迟测试
# Gen 2: 8b/10b 编码延迟 + 较低信号速率
# Gen 3: 128b/130b 编码延迟 + Scrambling + 更高信号速率
# 通常 Gen 3 总延迟更低（带宽大幅提升）

# 使用 fio 测试延迟
fio --name=latency --rw=randread --bs=4k --numjobs=1 \
    --iodepth=1 --filename=/dev/nvme0n1 --direct=1
```

---

## 总结

### 关键要点

1. ✅ **8b/10b 编码**用于 Gen 1-2，提供 80% 效率，通过 RD 保证 DC 平衡
2. ✅ **128b/130b 编码**用于 Gen 3+，提供 98.46% 效率，必须配合 Scrambling
3. ✅ **K-codes** (8b/10b) 和 **Ordered Sets** (128b/130b) 提供帧边界和控制信令
4. ✅ **Scrambling** 使用 LFSR 解决长串 0/1 和 EMI 问题
5. ✅ **SKP 字符**用于时钟补偿，应对发送/接收时钟偏差
6. ✅ **编码效率**直接影响带宽：Gen 3 x16 达到 15.75 GB/s，Gen 2 仅 8 GB/s

**选择建议**：
- 新设计：使用 Gen 3+ (128b/130b)，带宽和效率更高
- 兼容性：Gen 1-2 设备可协商到 8b/10b
- 调试：Gen 2 的 8b/10b 更易分析（K-codes 明确）

---

## 下一步学习

- [链路训练](link-training.md) - 如何协商编码和速度
- [物理层概述](overview.md) - 物理层整体架构
- [信号完整性](signal-integrity.md) - 高速信号挑战

---

## 参考资料

- **规范**：PCIe Base Spec Chapter 4 (Physical Layer)
- **8b/10b**：IEEE 802.3 / ANSI X3.230-1994 (Fibre Channel)
- **图表**：Figure 4-14 (128b/130b Block Format)
- **实现**：`hw/pci/pcie.c` (FEMU 链路速度设置)

---

**相关页面**：
- [← 物理层概述](overview.md)
- [链路训练 →](link-training.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
