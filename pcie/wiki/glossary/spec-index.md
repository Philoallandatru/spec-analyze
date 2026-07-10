# PCIe 规范索引

## 官方规范

### PCI Express Base Specification

| 版本 | 发布时间 | 主要特性 |
|------|---------|---------|
| 1.0 | 2002 | 初始版本，2.5 GT/s |
| 1.1 | 2005 | 规范澄清 |
| 2.0 | 2007 | 5.0 GT/s |
| 2.1 | 2009 | 规范更新 |
| 3.0 | 2010 | 8.0 GT/s，128b/130b编码 |
| 3.1 | 2014 | 规范完善 |
| 4.0 | 2017 | 16.0 GT/s |
| 5.0 | 2019 | 32.0 GT/s |
| 6.0 | 2022 | 64.0 GT/s，242b/256b编码 |

### 重要章节索引

**第1章 - 介绍**
- 1.1 PCIe 架构概述
- 1.2 拓扑结构
- 1.3 与 PCI 的兼容性

**第2章 - Transaction Layer**
- 2.1 TLP 格式
- 2.2 事务类型
- 2.3 流控制
- 2.4 排序规则

**第3章 - Data Link Layer**
- 3.1 DLLP 格式
- 3.2 ACK/NAK 协议
- 3.3 LCRC
- 3.4 重试机制

**第4章 - Physical Layer**
- 4.1 LTSSM 状态机
- 4.2 编码方案
- 4.3 链路训练
- 4.4 电气规范

**第6章 - 配置空间**
- 6.1 配置机制
- 6.2 标准寄存器
- 6.3 Capability 结构

**第7章 - 错误处理**
- 7.1 错误分类
- 7.2 AER 机制
- 7.3 错误恢复

## 相关规范

### PCI-SIG 规范
- PCIe Card Electromechanical Specification
- PCIe M.2 Specification
- PCIe OCuLink Specification
- PCIe Mini Card Specification

### 设备类规范
- NVMe Specification (NVM Express)
- AHCI Specification (SATA)
- xHCI Specification (USB)
- GPU 互连规范

## 在线资源

### 官方网站
- PCI-SIG: https://pcisig.com/
- PCI-SIG Members: https://members.pcisig.com/

### 规范下载
- 需要 PCI-SIG 会员资格
- 部分旧版本公开可用

### 开源实现
- QEMU: https://www.qemu.org/
- Linux Kernel: https://www.kernel.org/
- FEMU: https://github.com/vtess/FEMU

## 工具和软件

### 调试工具
- lspci (pciutils)
- setpci
- PCIe 分析仪

### 开发工具
- QEMU/KVM
- Xilinx Vivado
- Intel Quartus

## 学习资源

### 书籍推荐
- "PCI Express System Architecture" by Ravi Budruk
- "PCI Express Technology" by Mike Jackson

### 在线教程
- 本 Wiki
- PCI-SIG 培训材料
- 各厂商应用笔记

## 总结

本索引提供 PCIe 规范的导航和相关资源链接。

参见：
- [PCIe 概述](../introduction/overview.md)
- [学习路径](../introduction/learning-path.md)
- [术语表](index.md)
