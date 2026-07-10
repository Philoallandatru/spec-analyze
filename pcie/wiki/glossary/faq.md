# PCIe 常见问题解答 (FAQ)

## 入门问题

### Q1: PCIe 是什么？

**A**: PCIe (PCI Express) 是一种高速串行计算机扩展总线标准，用于连接主板和外围设备（如显卡、SSD、网卡等）。它是 PCI 和 PCI-X 的继任者。

**关键特点**：
- 点对点串行连接（不是共享总线）
- 高速率：Gen1 2.5GT/s 到 Gen6 64GT/s
- 可扩展：x1, x4, x8, x16 等链路宽度
- 热插拔支持
- 内置电源管理

### Q2: PCIe 各代速度是多少？

| 代数 | 速率 | 编码 | 有效带宽(x16) | 年份 |
|------|------|------|---------------|------|
| Gen 1 | 2.5 GT/s | 8b/10b | ~4 GB/s | 2003 |
| Gen 2 | 5.0 GT/s | 8b/10b | ~8 GB/s | 2007 |
| Gen 3 | 8.0 GT/s | 128b/130b | ~16 GB/s | 2010 |
| Gen 4 | 16.0 GT/s | 128b/130b | ~32 GB/s | 2017 |
| Gen 5 | 32.0 GT/s | 128b/130b | ~64 GB/s | 2019 |
| Gen 6 | 64.0 GT/s | 242b/256b | ~128 GB/s | 2022 |

### Q3: 如何查看系统中的 PCIe 设备？

```bash
# 列出所有 PCIe 设备
lspci

# 详细信息
lspci -vv

# 树状显示
lspci -tv
```

### Q4: MSI 和 MSI-X 有什么区别？

| 特性 | MSI | MSI-X |
|------|-----|-------|
| 最大向量数 | 32 | 2048 |
| 向量配置 | 2的幂次 | 任意数量 |
| 地址/数据 | 共享 | 独立 |

**推荐**：优先使用 MSI-X。

### Q5: 如何提高 PCIe 性能？

1. 增大 Max Payload Size
2. 增大 Max Read Request Size
3. 启用 Relaxed Ordering
4. 使用 MSI-X 多队列
5. DMA 优化

### Q6: ASPM 是什么？

**A**: ASPM (Active State Power Management) 是链路级自动省电机制。

**状态**：
- L0s：快速省电，< 4 μs 恢复
- L1：深度省电，< 10 μs 恢复

### Q7: SR-IOV 是什么？

**A**: SR-IOV 允许单个物理设备虚拟化为多个虚拟设备。

**概念**：
- PF (Physical Function)：物理功能
- VF (Virtual Function)：虚拟功能

### Q8: 有哪些常用的 PCIe 调试工具？

```bash
lspci      # 列出设备
setpci     # 读写配置空间
dmesg      # 内核日志
trace-cmd  # 内核追踪
perf       # 性能分析
```

## 总结

本 FAQ 涵盖了 PCIe 最常见的问题。详细说明请参考 Wiki 其他页面。

参见：
- [PCIe 概述](../introduction/overview.md)
- [驱动开发指南](../implementation/driver-guide.md)
