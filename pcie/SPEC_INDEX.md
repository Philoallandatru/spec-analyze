# PCI Express 规范索引

> PCIe 5.0 规范完整索引 - 快速查找和导航

[![基于规范](https://img.shields.io/badge/PCIe-5.0-green)]()
[![页面数](https://img.shields.io/badge/页面-72-blue)]()

---

## 📋 目录

- [概述与入门](#概述与入门)
- [架构基础](#架构基础)
- [三层协议模型](#三层协议模型)
  - [事务层 (TL)](#事务层-transaction-layer)
  - [数据链路层 (DLL)](#数据链路层-data-link-layer)
  - [物理层 (PHY)](#物理层-physical-layer)
- [配置与初始化](#配置与初始化)
- [中断机制](#中断机制)
- [错误处理](#错误处理)
- [电源管理](#电源管理)
- [高级特性](#高级特性)
- [实现参考](#实现参考)
- [术语与参考](#术语与参考)

---

## 概述与入门

### PCIe 基础知识
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **PCIe 概述** | [overview.md](wiki/introduction/overview.md) | Ch 1 | PCIe 历史、特点、应用场景 |
| **拓扑结构** | [topology.md](wiki/introduction/topology.md) | Ch 1.3 | Root Complex、Switch、Endpoint |
| **PCI vs PCIe** | [pci-vs-pcie.md](wiki/introduction/pci-vs-pcie.md) | Ch 1 | 并行总线到串行总线的演进 |
| **术语速查** | [terminology.md](wiki/introduction/terminology.md) | - | 常用缩写和术语解释 |
| **学习路径** | [learning-path.md](wiki/introduction/learning-path.md) | - | 针对不同角色的学习路线 |

**相关规范章节**：
- Chapter 1: Introduction
- Chapter 1.3: Topology

---

## 架构基础

### 系统架构
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **三层协议模型** | [layering.md](wiki/architecture/layering.md) | Ch 1.2 | TL、DLL、PHY 分层架构 |
| **数据包类型** | [packet-types.md](wiki/architecture/packet-types.md) | Ch 2 | TLP、DLLP、Ordered Sets |
| **地址空间** | [address-space.md](wiki/architecture/address-space.md) | Ch 2.2 | Memory、I/O、Configuration 空间 |
| **枚举过程** | [enumeration.md](wiki/architecture/enumeration.md) | Ch 6.6 | 设备发现和配置流程 |
| **时钟架构** | [clock-architecture.md](wiki/architecture/clock-architecture.md) | Ch 4 | 时钟分布和同步机制 |

**相关规范章节**：
- Chapter 1.2: Layered Architecture
- Chapter 2: Transaction, Data Link, and Physical Layers
- Chapter 6.6: Configuration

---

## 三层协议模型

### 事务层 (Transaction Layer)

#### TLP 基础
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **TLP 格式** | [tlp-format.md](wiki/transaction-layer/tlp-format.md) | Ch 2.2 | Header、Data、ECRC 格式 |
| **事务类型** | [transaction-types.md](wiki/transaction-layer/transaction-types.md) | Ch 2.2 | Memory、I/O、Config、Message |
| **路由机制** | [routing.md](wiki/transaction-layer/routing.md) | Ch 2.2.8 | 地址路由、ID 路由、隐式路由 |

#### 流控与顺序
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **流量控制** | [flow-control.md](wiki/transaction-layer/flow-control.md) | Ch 2.4 | Credit-based 流控机制 |
| **排序规则** | [ordering.md](wiki/transaction-layer/ordering.md) | Ch 2.4 | Producer-Consumer 排序模型 |
| **虚拟通道** | [virtual-channels.md](wiki/transaction-layer/virtual-channels.md) | Ch 2.4.1 | VC0-VC7 和流量分类 |

#### 数据完整性
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **ECRC** | [ecrc.md](wiki/transaction-layer/ecrc.md) | Ch 2.7 | End-to-End CRC 端到端校验 |

**相关规范章节**：
- Chapter 2.2: Transaction Layer
- Chapter 2.4: Flow Control
- Chapter 2.7: End-to-End Data Integrity

---

### 数据链路层 (Data Link Layer)

#### DLLP 与流控
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **DLLP 格式** | [dllp-format.md](wiki/data-link-layer/dllp-format.md) | Ch 2.5 | Flow Control、Ack/Nak、PM |
| **流控初始化** | [fc-init.md](wiki/data-link-layer/fc-init.md) | Ch 2.5.2 | Credit 初始化过程 |

#### 可靠传输
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **Ack/Nak 协议** | [ack-nak.md](wiki/data-link-layer/ack-nak.md) | Ch 2.6 | 确认和否定应答机制 |
| **重传缓冲区** | [retry-buffer.md](wiki/data-link-layer/retry-buffer.md) | Ch 2.6.1 | Replay Buffer 实现 |
| **LCRC** | [lcrc.md](wiki/data-link-layer/lcrc.md) | Ch 2.6.2 | Link CRC 链路级校验 |

**相关规范章节**：
- Chapter 2.5: Data Link Layer
- Chapter 2.6: Link Reliability

---

### 物理层 (Physical Layer)

#### 物理层基础
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **物理层概述** | [overview.md](wiki/physical-layer/overview.md) | Ch 4 | Lane、Link Width、速度等级 |
| **编码机制** | [encoding.md](wiki/physical-layer/encoding.md) | Ch 4.2 | 8b/10b、128b/130b 编码 |
| **电气规范** | [electrical-spec.md](wiki/physical-layer/electrical-spec.md) | Ch 4.3 | 电压、眼图、抖动规范 |

#### 链路训练与管理
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **LTSSM** | [ltssm.md](wiki/physical-layer/ltssm.md) | Ch 4.2.6 | Link Training 状态机 |
| **链路训练** | [link-training.md](wiki/physical-layer/link-training.md) | Ch 4.2.6 | 速度协商、Lane 反转、极性 |
| **Retimer** | [retimer.md](wiki/physical-layer/retimer.md) | Ch 4.5 | 信号中继和恢复 |

**相关规范章节**：
- Chapter 4: Physical Layer
- Chapter 4.2: Link Training and Status State Machine (LTSSM)
- Chapter 4.3: Electrical Specifications

---

## 配置与初始化

### 配置空间
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **配置空间结构** | [config-space.md](wiki/configuration/config-space.md) | Ch 7 | Type 0、Type 1 Header |
| **Header 类型** | [header-types.md](wiki/configuration/header-types.md) | Ch 7.5 | Endpoint、Bridge、CardBus |
| **BARs** | [bars.md](wiki/configuration/bars.md) | Ch 7.5.1 | Base Address Registers |
| **Capabilities** | [capabilities.md](wiki/configuration/capabilities.md) | Ch 7.7 | PCIe Capability Structure |
| **Extended Capabilities** | [extended-capabilities.md](wiki/configuration/extended-capabilities.md) | Ch 7.9 | 扩展配置空间 (256B-4KB) |

**相关规范章节**：
- Chapter 7: Configuration Space
- Chapter 7.5: Type 0/1 Configuration Space Header
- Chapter 7.7: Capabilities
- Chapter 7.9: Extended Capabilities

---

## 中断机制

### 中断类型
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **中断概述** | [overview.md](wiki/interrupts/overview.md) | Ch 6.1 | INTx、MSI、MSI-X 对比 |
| **传统中断** | [intx.md](wiki/interrupts/intx.md) | Ch 6.1.1 | INTx (INTA-INTD) |
| **MSI** | [msi.md](wiki/interrupts/msi.md) | Ch 6.1.2 | Message Signaled Interrupts |
| **MSI-X** | [msix.md](wiki/interrupts/msix.md) | Ch 6.1.3 | MSI-X 扩展版本 |
| **中断路由** | [routing.md](wiki/interrupts/routing.md) | Ch 6.1.4 | 中断消息传递 |

**相关规范章节**：
- Chapter 6.1: Interrupt and PME Support
- Chapter 6.1.2: MSI Capability
- Chapter 6.1.3: MSI-X Capability

---

## 错误处理

### 错误检测与恢复
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **错误类型** | [error-types.md](wiki/error-handling/error-types.md) | Ch 6.2 | Correctable、Uncorrectable |
| **错误检测** | [detection.md](wiki/error-handling/detection.md) | Ch 6.2.1 | CRC、Timeout、Poisoned TLP |
| **错误恢复** | [recovery.md](wiki/error-handling/recovery.md) | Ch 6.2.2 | Link Recovery、Hot Reset |
| **AER** | [aer.md](wiki/error-handling/aer.md) | Ch 6.2.3 | Advanced Error Reporting |
| **错误注入** | [error-injection.md](wiki/error-handling/error-injection.md) | Ch 6.2.4 | 测试和验证 |

**相关规范章节**：
- Chapter 6.2: Error Handling
- Chapter 6.2.3: Advanced Error Reporting (AER)

---

## 电源管理

### 电源状态与优化
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **电源状态** | [power-states.md](wiki/power-management/power-states.md) | Ch 5.3 | D0-D3hot、D3cold |
| **ASPM** | [aspm.md](wiki/power-management/aspm.md) | Ch 5.4 | Active State Power Management |
| **CLKREQ#** | [clkreq.md](wiki/power-management/clkreq.md) | Ch 5.5 | 时钟请求信号 |

**相关规范章节**：
- Chapter 5: Power Management
- Chapter 5.3: Device Power Management States
- Chapter 5.4: Link Power Management (ASPM)

---

## 高级特性

### 虚拟化与隔离
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **SR-IOV** | [sriov.md](wiki/advanced-features/sriov.md) | Ch 9.3 | Single Root I/O Virtualization |
| **ACS** | [acs.md](wiki/advanced-features/acs.md) | Ch 6.12 | Access Control Services |
| **ARI** | [ari.md](wiki/advanced-features/ari.md) | Ch 6.13 | Alternative Routing-ID |

### 性能与安全
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **Atomic Operations** | [atomic-ops.md](wiki/advanced-features/atomic-ops.md) | Ch 6.15 | 原子操作 |
| **TPH** | [tph.md](wiki/advanced-features/tph.md) | Ch 6.17 | TLP Processing Hints |
| **PTM** | [ptm.md](wiki/advanced-features/ptm.md) | Ch 6.18 | Precision Time Measurement |
| **IDE** | [ide.md](wiki/advanced-features/ide.md) | Ch 6.33 | Integrity and Data Encryption |
| **DOE** | [doe.md](wiki/advanced-features/doe.md) | Ch 6.30 | Data Object Exchange |

### 热插拔
| 主题 | 文档 | 规范章节 | 说明 |
|------|------|----------|------|
| **Hot-Plug** | [hot-plug.md](wiki/advanced-features/hot-plug.md) | Ch 6.7 | 热插拔支持 |

**相关规范章节**：
- Chapter 6.7: Hot-Plug
- Chapter 6.12-6.18: Advanced Features
- Chapter 6.30-6.33: Security Features
- Chapter 9.3: SR-IOV

---

## 实现参考

### 代码实现
| 主题 | 文档 | 代码库 | 说明 |
|------|------|--------|------|
| **FEMU 概述** | [femu-overview.md](wiki/implementation/femu-overview.md) | QEMU/FEMU | PCIe 设备模拟 |
| **配置空间实现** | [config-space.md](wiki/implementation/config-space.md) | hw/pci | 配置空间读写 |
| **中断实现** | [interrupts.md](wiki/implementation/interrupts.md) | hw/pci | MSI/MSI-X 实现 |
| **AER 实现** | [aer.md](wiki/implementation/aer.md) | hw/pci | 错误报告 |
| **NVMe over PCIe** | [nvme.md](wiki/implementation/nvme.md) | hw/block | NVMe 设备示例 |
| **驱动开发指南** | [driver-guide.md](wiki/implementation/driver-guide.md) | Linux Kernel | PCIe 驱动开发 |

**代码分析文档**：
- [FEMU 代码深度分析](pcie-deep-analysis.md) - 17KB 详细分析
- [PCIe 5.0 规范分析](pcie-spec-analysis.md) - 规范结构解析

---

## 术语与参考

### 参考资料
| 主题 | 文档 | 说明 |
|------|------|------|
| **术语表** | [glossary/index.md](wiki/glossary/index.md) | 200+ 术语速查 |
| **Wiki 主页** | [wiki/README.md](wiki/README.md) | 完整导航 |
| **快速开始** | [QUICKSTART.md](QUICKSTART.md) | 入门指南 |
| **项目总结** | [PROJECT_SUMMARY.md](PROJECT_SUMMARY.md) | 项目进度 |

---

## 按规范章节索引

| 章节 | 标题 | 主要内容 | 相关文档 |
|------|------|----------|----------|
| **Ch 1** | Introduction | 概述、拓扑、分层 | [overview.md](wiki/introduction/overview.md), [topology.md](wiki/introduction/topology.md), [layering.md](wiki/architecture/layering.md) |
| **Ch 2** | Transaction/Data Link/Physical | TLP、DLLP、流控 | [tlp-format.md](wiki/transaction-layer/tlp-format.md), [dllp-format.md](wiki/data-link-layer/dllp-format.md) |
| **Ch 4** | Physical Layer | 物理层规范 | [overview.md](wiki/physical-layer/overview.md), [ltssm.md](wiki/physical-layer/ltssm.md) |
| **Ch 5** | Power Management | 电源管理 | [power-states.md](wiki/power-management/power-states.md), [aspm.md](wiki/power-management/aspm.md) |
| **Ch 6** | System Architecture | 中断、错误、高级特性 | [overview.md](wiki/interrupts/overview.md), [error-types.md](wiki/error-handling/error-types.md) |
| **Ch 7** | Configuration Space | 配置空间 | [config-space.md](wiki/configuration/config-space.md), [bars.md](wiki/configuration/bars.md) |
| **Ch 9** | SR-IOV | 虚拟化 | [sriov.md](wiki/advanced-features/sriov.md) |

---

## 按角色快速索引

### 软件开发者
**核心主题**：配置空间 → BARs → 中断 → DMA → 错误处理

- [配置空间](wiki/configuration/config-space.md)
- [BARs](wiki/configuration/bars.md)
- [MSI/MSI-X](wiki/interrupts/msi.md)
- [驱动开发指南](wiki/implementation/driver-guide.md)
- [AER](wiki/error-handling/aer.md)

### 硬件工程师
**核心主题**：物理层 → LTSSM → 链路训练 → 电气规范

- [物理层概述](wiki/physical-layer/overview.md)
- [LTSSM](wiki/physical-layer/ltssm.md)
- [链路训练](wiki/physical-layer/link-training.md)
- [电气规范](wiki/physical-layer/electrical-spec.md)
- [编码机制](wiki/physical-layer/encoding.md)

### 系统架构师
**核心主题**：拓扑 → 路由 → 虚拟化 → 性能优化

- [拓扑结构](wiki/introduction/topology.md)
- [路由机制](wiki/transaction-layer/routing.md)
- [SR-IOV](wiki/advanced-features/sriov.md)
- [ASPM](wiki/power-management/aspm.md)
- [虚拟通道](wiki/transaction-layer/virtual-channels.md)

### 验证工程师
**核心主题**：错误处理 → 流控 → 排序 → 协议验证

- [错误类型](wiki/error-handling/error-types.md)
- [错误注入](wiki/error-handling/error-injection.md)
- [流量控制](wiki/transaction-layer/flow-control.md)
- [排序规则](wiki/transaction-layer/ordering.md)
- [Ack/Nak](wiki/data-link-layer/ack-nak.md)

---

## 快速查找表

### 关键数据结构
| 结构 | 位置 | 文档 |
|------|------|------|
| TLP Header | Transaction Layer | [tlp-format.md](wiki/transaction-layer/tlp-format.md) |
| DLLP | Data Link Layer | [dllp-format.md](wiki/data-link-layer/dllp-format.md) |
| Config Space Header | Configuration | [config-space.md](wiki/configuration/config-space.md) |
| Capability Structure | Configuration | [capabilities.md](wiki/configuration/capabilities.md) |
| MSI-X Table | Interrupts | [msix.md](wiki/interrupts/msix.md) |

### 关键寄存器
| 寄存器 | 偏移 | 文档 |
|--------|------|------|
| Command/Status | 0x04 | [config-space.md](wiki/configuration/config-space.md) |
| BAR0-5 | 0x10-0x24 | [bars.md](wiki/configuration/bars.md) |
| Capability Pointer | 0x34 | [capabilities.md](wiki/configuration/capabilities.md) |
| Device Control | Cap+0x08 | [capabilities.md](wiki/configuration/capabilities.md) |
| Link Control | Cap+0x10 | [capabilities.md](wiki/configuration/capabilities.md) |

### 关键概念速查
| 概念 | 简述 | 文档 |
|------|------|------|
| Credit | 流控单位 | [flow-control.md](wiki/transaction-layer/flow-control.md) |
| Posted/Non-Posted | 事务类型 | [transaction-types.md](wiki/transaction-layer/transaction-types.md) |
| Completion | 响应机制 | [transaction-types.md](wiki/transaction-layer/transaction-types.md) |
| Replay Buffer | 重传缓冲区 | [retry-buffer.md](wiki/data-link-layer/retry-buffer.md) |
| TS1/TS2 | 训练序列 | [link-training.md](wiki/physical-layer/link-training.md) |

---

## 图表索引

### 架构图
- 三层协议模型：[layering.md](wiki/architecture/layering.md)
- 拓扑结构：[topology.md](wiki/introduction/topology.md)
- 数据包格式：[packet-types.md](wiki/architecture/packet-types.md)

### 流程图
- 枚举过程：[enumeration.md](wiki/architecture/enumeration.md)
- 链路训练：[link-training.md](wiki/physical-layer/link-training.md)
- 流控初始化：[fc-init.md](wiki/data-link-layer/fc-init.md)
- 错误恢复：[recovery.md](wiki/error-handling/recovery.md)

### 状态机
- LTSSM：[ltssm.md](wiki/physical-layer/ltssm.md)
- Link Recovery：[recovery.md](wiki/error-handling/recovery.md)

---

## 代码示例索引

### C 语言示例
- 配置空间读写：[config-space.md](wiki/implementation/config-space.md)
- MSI/MSI-X 设置：[interrupts.md](wiki/implementation/interrupts.md)
- DMA 映射：[driver-guide.md](wiki/implementation/driver-guide.md)
- 错误处理：[aer.md](wiki/implementation/aer.md)

### Verilog HDL 示例
- TLP 生成：[tlp-format.md](wiki/transaction-layer/tlp-format.md)
- CRC 计算：[lcrc.md](wiki/data-link-layer/lcrc.md)
- 8b/10b 编码：[encoding.md](wiki/physical-layer/encoding.md)

---

## 统计信息

**内容规模**：
- 总页面数：72 页
- 总字数：200,000+ 字
- 代码示例：600+ 个
- ASCII 图表：300+ 个
- 术语条目：200+ 个

**完成度**：100% ✅

**最后更新**：2026-07-07

---

## 使用说明

### 如何使用本索引

1. **按主题浏览**：使用上方的分类目录快速定位感兴趣的主题
2. **按规范章节**：如果你在阅读规范，可以使用"按规范章节索引"部分
3. **按角色**：根据你的工作角色（开发者、硬件工程师等）查看推荐路径
4. **快速查找**：使用"快速查找表"定位具体的数据结构或寄存器

### 相关资源

- **Wiki 主页**：[wiki/README.md](wiki/README.md)
- **快速开始**：[QUICKSTART.md](QUICKSTART.md)
- **项目首页**：[README.md](README.md)
- **官方规范**：PCI Express Base Specification.pdf

---

**返回**：[项目首页](README.md) | [Wiki 主页](wiki/README.md)

---

*本索引基于 PCIe 5.0 Base Specification 创建*
