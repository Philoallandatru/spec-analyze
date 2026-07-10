# PCI Express 技术 Wiki

欢迎来到 PCI Express (PCIe) 技术 Wiki！本 Wiki 旨在提供全面、易懂、图文并茂的 PCIe 技术文档。

---

## 📚 关于本 Wiki

本 Wiki 基于以下权威资源构建：
- **PCI Express Base Specification Revision 5.0** (官方规范)
- **FEMU/QEMU 源代码** (实际实现参考)

**特点：**
- ✅ 图文并茂，易于理解
- ✅ 从入门到精通的渐进式学习路径
- ✅ 结合规范和实际代码实现
- ✅ 包含术语表和交叉引用
- ✅ 适合硬件工程师、驱动开发者、系统架构师

---

## 🗺️ 内容导航

### 1️⃣ 入门指南
- [PCIe 简介](introduction/overview.md) - 什么是 PCIe？历史和演进
- [拓扑结构](introduction/topology.md) - Root Complex、Switch、Endpoint
- [与 PCI 的区别](introduction/pci-vs-pcie.md) - 为什么要从 PCI 升级到 PCIe？
- [学习路径](introduction/learning-path.md) - 如何系统学习 PCIe

### 2️⃣ 架构概览
- [三层协议模型](architecture/layering.md) - 事务层、数据链路层、物理层
- [数据包类型](architecture/packet-types.md) - TLP、DLLP、Ordered Set
- [设备枚举](architecture/enumeration.md) - 系统如何发现 PCIe 设备
- [地址空间](architecture/address-space.md) - 内存、IO、配置空间

### 3️⃣ 事务层 (Transaction Layer)
- [TLP 格式](transaction-layer/tlp-format.md) - 事务层包结构详解
- [事务类型](transaction-layer/transaction-types.md) - Memory、IO、Config、Message、Completion
- [路由机制](transaction-layer/routing.md) - 地址路由、ID 路由、隐式路由
- [排序规则](transaction-layer/ordering.md) - 保证数据一致性的排序约束
- [虚拟通道](transaction-layer/virtual-channels.md) - 流量类别和服务质量
- [流控制](transaction-layer/flow-control.md) - 基于信用的流控机制
- [ECRC](transaction-layer/ecrc.md) - 端到端 CRC 校验

### 4️⃣ 数据链路层 (Data Link Layer)
- [DLLP 格式](data-link-layer/dllp-format.md) - 数据链路层包
- [ACK/NAK 协议](data-link-layer/ack-nak.md) - 确认和重传机制
- [重试缓冲区](data-link-layer/retry-buffer.md) - 保证可靠传输
- [LCRC](data-link-layer/lcrc.md) - 链路层 CRC 校验
- [流控制初始化](data-link-layer/fc-init.md) - InitFC1/InitFC2 协议

### 5️⃣ 物理层 (Physical Layer)
- [物理层概述](physical-layer/overview.md) - 逻辑子块和电气子块
- [编码方案](physical-layer/encoding.md) - 8b/10b 和 128b/130b
- [LTSSM 状态机](physical-layer/ltssm.md) - 链路训练和状态管理
- [链路训练](physical-layer/link-training.md) - 速度和宽度协商
- [链路均衡](physical-layer/link-equalization.md) - 信号质量优化
- [时钟架构](physical-layer/clocking.md) - 参考时钟和扩频
- [Retimer](physical-layer/retimer.md) - PCIe 4.0/5.0 信号中继器
- [电气规范](physical-layer/electrical.md) - 电压、阻抗、抖动、眼图

### 6️⃣ 配置空间与能力
- [配置空间布局](configuration/config-space.md) - 256 字节标准 + 4KB 扩展
- [Type 0/1 Header](configuration/header-types.md) - Endpoint 和 Bridge 配置头
- [BARs](configuration/bars.md) - 基地址寄存器和地址映射
- [能力结构](configuration/capabilities.md) - 可扩展的特性声明机制
- [PCIe 能力](configuration/pcie-capability.md) - PCIe 专有能力详解
- [扩展能力](configuration/extended-capabilities.md) - 4KB 配置空间中的高级特性

### 7️⃣ 中断机制
- [中断概述](interrupts/overview.md) - 三种中断模式对比
- [Legacy INTx](interrupts/intx.md) - 传统共享中断线
- [MSI](interrupts/msi.md) - 消息信号中断
- [MSI-X](interrupts/msix.md) - 扩展消息中断
- [中断路由](interrupts/routing.md) - 中断如何到达 CPU

### 8️⃣ 错误处理
- [错误分类](error-handling/error-types.md) - 可纠正 vs 不可纠正错误
- [AER](error-handling/aer.md) - 高级错误报告机制
- [错误检测](error-handling/detection.md) - ECRC、LCRC、超时等
- [错误恢复](error-handling/recovery.md) - 错误后的恢复流程
- [错误注入](error-handling/injection.md) - 测试和调试

### 9️⃣ 电源管理
- [电源状态](power-management/power-states.md) - L0、L0s、L1、L2/L3
- [ASPM](power-management/aspm.md) - 主动状态电源管理
- [L1 子状态](power-management/l1-substates.md) - L1.1 和 L1.2
- [设备电源状态](power-management/device-states.md) - D0-D3
- [时钟电源管理](power-management/clkreq.md) - CLKREQ# 信号

### 🔟 高级特性
- [SR-IOV](advanced-features/sriov.md) - 单根 IO 虚拟化
- [ACS](advanced-features/acs.md) - 访问控制服务
- [ARI](advanced-features/ari.md) - 备选路由 ID 解释
- [热插拔](advanced-features/hot-plug.md) - Native Hot-Plug 支持
- [AtomicOps](advanced-features/atomic-ops.md) - 原子操作
- [TPH](advanced-features/tph.md) - TLP 处理提示
- [PTM](advanced-features/ptm.md) - 精确时间测量
- [DOE](advanced-features/doe.md) - 数据对象交换
- [IDE](advanced-features/ide.md) - 完整性和数据加密

### 💻 实现参考
- [FEMU 代码导读](implementation/femu-overview.md) - PCIe 在 QEMU 中的实现
- [配置空间实现](implementation/config-space.md) - 代码示例
- [中断实现](implementation/interrupts.md) - MSI/MSI-X 代码
- [AER 实现](implementation/aer.md) - 错误报告代码
- [NVMe over PCIe](implementation/nvme.md) - 完整的设备实现案例
- [驱动开发指南](implementation/driver-guide.md) - 如何编写 PCIe 驱动

### 📖 参考资料
- [术语表](glossary/index.md) - 所有 PCIe 术语和缩写
- [寄存器速查](glossary/register-quick-ref.md) - 常用寄存器快速参考
- [规范章节索引](glossary/spec-index.md) - 规范文档导航
- [FAQ](glossary/faq.md) - 常见问题解答
- [延伸阅读](glossary/further-reading.md) - 推荐资源

---

## 🎯 学习路径建议

### 入门路径（硬件背景）
1. [PCIe 简介](introduction/overview.md)
2. [拓扑结构](introduction/topology.md)
3. [三层协议模型](architecture/layering.md)
4. [物理层概述](physical-layer/overview.md)
5. [LTSSM 状态机](physical-layer/ltssm.md)

### 入门路径（软件背景）
1. [PCIe 简介](introduction/overview.md)
2. [配置空间布局](configuration/config-space.md)
3. [BARs](configuration/bars.md)
4. [中断概述](interrupts/overview.md)
5. [MSI-X](interrupts/msix.md)
6. [驱动开发指南](implementation/driver-guide.md)

### 进阶路径（协议深入）
1. [TLP 格式](transaction-layer/tlp-format.md)
2. [排序规则](transaction-layer/ordering.md)
3. [流控制](transaction-layer/flow-control.md)
4. [ACK/NAK 协议](data-link-layer/ack-nak.md)
5. [错误处理](error-handling/aer.md)

### 高级路径（虚拟化和安全）
1. [SR-IOV](advanced-features/sriov.md)
2. [ACS](advanced-features/acs.md)
3. [ARI](advanced-features/ari.md)
4. [IDE](advanced-features/ide.md)

---

## 🔧 工具和资源

### 调试工具
- **lspci** - Linux 下查看 PCIe 设备
- **setpci** - 读写配置空间
- **PCIe Protocol Analyzer** - 硬件协议分析仪
- **QEMU/FEMU** - 软件模拟环境

### 规范文档
- PCI Express Base Specification (官方规范)
- PCI Local Bus Specification (PCI 规范)
- NVMe Specification (NVMe 存储协议)

### 代码参考
- Linux Kernel PCIe 子系统
- QEMU/FEMU PCIe 模拟
- FreeBSD PCIe 实现

---

## 🤝 贡献指南

本 Wiki 欢迎贡献！您可以：
- 修正错误
- 补充内容
- 添加图表和示例
- 改进解释

---

## 📄 许可证

本 Wiki 内容基于以下资源：
- PCI Express Base Specification © PCI-SIG
- FEMU/QEMU 源代码 (GPLv2)

---

## 📧 联系方式

如有问题或建议，欢迎联系或提交 Issue。

---

**版本**: 1.0  
**最后更新**: 2026-07-06  
**基于规范**: PCIe 5.0 Revision 1.0  
**状态**: 🚧 持续建设中
