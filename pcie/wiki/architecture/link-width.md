# PCIe 链路宽度

## 概述

PCIe 链路宽度 (Link Width) 指并行使用的数据通道（Lane）数量。通过增加链路宽度可以成比例地提高总带宽。

## 支持的链路宽度

### 标准宽度

| 宽度 | Lane数 | 符号 | 应用场景 |
|------|--------|------|---------|
| x1 | 1 | PCIe x1 | 低速设备、扩展卡 |
| x2 | 2 | PCIe x2 | M.2 接口 |
| x4 | 4 | PCIe x4 | NVMe SSD、网卡 |
| x8 | 8 | PCIe x8 | 高性能网卡、RAID卡 |
| x12 | 12 | PCIe x12 | 少见 |
| x16 | 16 | PCIe x16 | 显卡、GPU |
| x32 | 32 | PCIe x32 | 服务器专用（已废弃） |

### 带宽计算

```
Gen3 带宽计算：
x1:  8 GT/s × 1 × 128/130 ≈ 1 GB/s
x4:  8 GT/s × 4 × 128/130 ≈ 4 GB/s
x8:  8 GT/s × 8 × 128/130 ≈ 8 GB/s
x16: 8 GT/s × 16 × 128/130 ≈ 16 GB/s

Gen4 带宽计算：
x1:  16 GT/s × 1 × 128/130 ≈ 2 GB/s
x4:  16 GT/s × 4 × 128/130 ≈ 8 GB/s
x16: 16 GT/s × 16 × 128/130 ≈ 32 GB/s
```

## 链路宽度协商

### 训练过程

```
1. 初始状态：发现可用的 Lane
2. 交换能力：双方报告支持的宽度
3. 协商宽度：选择双方都支持的最大宽度
4. 配置链路：使用协商后的宽度

协商规则：
- 选择 min(发送端能力, 接收端能力)
- 必须是标准宽度 (x1, x2, x4, x8, x16)
- 物理连接决定最大可能宽度
```

### Link Capabilities 寄存器

```
PCIe Capability + 0x0C: Link Capabilities

Bits [9:4]: Max Link Width
  0x01 = x1
  0x02 = x2
  0x04 = x4
  0x08 = x8
  0x0C = x12
  0x10 = x16
  0x20 = x32

查看：
lspci -vv | grep "LnkCap:"
输出：LnkCap: Port #0, Speed 16GT/s, Width x16
```

### Link Status 寄存器

```
PCIe Capability + 0x12: Link Status

Bits [9:4]: Negotiated Link Width
  当前协商的实际宽度

查看：
lspci -vv | grep "LnkSta:"
输出：LnkSta: Speed 8GT/s (ok), Width x4 (ok)
```

## 宽度降级

### 原因

1. **物理连接问题**
   - Lane 损坏或接触不良
   - 连接器插入不完全
   - PCB 走线问题

2. **训练失败**
   - 某些 Lane 无法锁定
   - 误码率过高
   - 信号完整性问题

3. **配置限制**
   - BIOS 设置
   - 主板槽位限制
   - 设备不支持全宽

### 检测降级

```bash
# 查看协商宽度
lspci -vv -s 01:00.0 | grep LnkSta

# 对比能力和状态
lspci -vv -s 01:00.0 | grep -E "LnkCap|LnkSta"

# 示例输出：
# LnkCap: Width x16  (设备支持 x16)
# LnkSta: Width x4   (实际运行在 x4) --> 降级！
```

### 带宽影响

```
x16 降级到 x8：带宽减半
x16 降级到 x4：带宽降为 1/4
x8 降级到 x1：带宽降为 1/8

影响：
- I/O 密集型应用性能下降明显
- GPU 渲染性能降低
- NVMe SSD 吞吐量受限
```

## 动态宽度调整

### Hardware Autonomous Width

某些设备支持自动宽度调整以节省功耗：

```
运行时根据负载动态改变宽度：
高负载: x16
中负载: x8
低负载: x4
空闲:   x1

Link Control 寄存器:
Bit 6: Hardware Autonomous Width Disable
  0 = 允许自动调整
  1 = 禁用自动调整
```

## 槽位兼容性

### 物理兼容性

```
卡槽兼容规则：
- x1 卡 可插入 x1, x4, x8, x16 槽
- x4 卡 可插入 x4, x8, x16 槽
- x8 卡 可插入 x8, x16 槽
- x16 卡 只能插入 x16 槽

电气兼容：
- 小卡插大槽：正常工作，使用卡的宽度
- 大卡插小槽：物理不兼容（除非开放式槽位）
```

### 开放式槽位

```
Open-Ended Slot:
- 槽位末端开放
- 允许更长的卡插入
- 电气连接受限于槽位宽度

示例：
x4 开放式槽位可以插入 x16 卡
但只能以 x4 宽度运行
```

## 查看和配置

### 软件查看

```bash
# 查看链路宽度
lspci -vv | grep Width

# sysfs 接口
cat /sys/bus/pci/devices/0000:01:00.0/current_link_width
cat /sys/bus/pci/devices/0000:01:00.0/max_link_width
```

### 代码读取

```c
// 读取链路宽度
static void read_link_width(struct pci_dev *pdev)
{
    int pos;
    u32 link_cap;
    u16 link_sta;
    
    pos = pci_pcie_cap(pdev);
    
    // 读取 Link Capabilities
    pci_read_config_dword(pdev, pos + PCI_EXP_LNKCAP, &link_cap);
    int max_width = (link_cap >> 4) & 0x3F;
    
    // 读取 Link Status
    pci_read_config_word(pdev, pos + PCI_EXP_LNKSTA, &link_sta);
    int cur_width = (link_sta >> 4) & 0x3F;
    
    dev_info(&pdev->dev, "Link: max x%d, current x%d\n",
             max_width, cur_width);
    
    if (cur_width < max_width) {
        dev_warn(&pdev->dev, "Link width degraded!\n");
    }
}
```

## 故障排查

### 宽度降级调试

1. **检查物理连接**
   - 重新插拔设备
   - 清洁连接器
   - 尝试其他槽位

2. **检查 BIOS 设置**
   - PCIe 配置选项
   - Lane 分配设置

3. **查看内核日志**
```bash
dmesg | grep -i pcie
# 查找训练失败或降级消息
```

4. **测试不同速度**
```bash
# 强制 Gen3 速度测试
setpci -s 01:00.0 CAP_EXP+30.W=0083
echo 1 > /sys/bus/pci/devices/0000:01:00.0/reset
```

## 总结

PCIe 链路宽度的关键点：

1. **可扩展性**：x1 到 x16 成比例带宽
2. **自动协商**：双方协商最大公共宽度
3. **降级检测**：对比能力和状态寄存器
4. **兼容性**：小卡可插大槽
5. **动态调整**：某些设备支持节能

参见：
- [PCIe 概述](../introduction/overview.md)
- [速度和编码](speed-encoding.md)
- [链路训练](../physical-layer/link-training.md)
