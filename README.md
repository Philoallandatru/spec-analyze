# 存储与互连规范解析

这是一个面向中文读者的技术规范知识库，包含 **NVMe** 和 **PCIe** 两大核心技术的完整文档。我们将官方规范改写为更易理解的中文文档，帮助开发者和工程师快速掌握存储和互连技术。

## 📚 包含的规范

### 🔵 NVMe Base 2.1 规范

NVMe（Non-Volatile Memory Express）是专为固态存储设计的高性能存储协议。

**特色：**
- ✅ **中文友好**：摆脱生硬的机器翻译，使用自然流畅的中文表达
- 📚 **系统化组织**：按功能模块分类，便于查找和学习
- 🔍 **概念清晰**：关键术语中英对照，降低理解门槛
- 📖 **完整覆盖**：涵盖 NVMe Base 2.1 规范的所有核心概念
- 🔗 **追溯源头**：保留规范页码引用，方便查阅原文

**快速入口：** [NVMe 文档目录](nvme/) | [概念索引](nvme/CONCEPT_INDEX.md)

**核心概念：**
1. [控制器（Controller）](nvme/concepts/controller.md) - 主机与存储系统之间的桥梁
2. [命名空间（Namespace）](nvme/concepts/namespace.md) - NVMe 的存储抽象单元
3. [队列对（Queue Pair）](nvme/concepts/queue-pair.md) - 命令提交与完成的机制
4. [命令集（Command Sets）](nvme/concepts/command-sets.md) - 命令的分类与路由
5. [NVM 子系统（NVM Subsystem）](nvme/concepts/nvm-subsystem.md) - 系统边界与组成

---

### 🟢 PCIe 5.0 规范

PCIe（PCI Express）是现代计算机系统中最重要的高速互连总线标准。

**特色：**
- ✅ **双重视角**：结合官方规范和实际代码实现
- 🎨 **图文并茂**：大量图表和代码示例
- 📈 **渐进式学习**：从基础到高级的清晰路径
- 💻 **实践导向**：包含真实代码和案例
- 🔧 **完整覆盖**：从物理层到事务层的完整技术栈

**快速入口：** [PCIe Wiki 主页](pcie/wiki/README.md) | [快速开始](pcie/QUICKSTART.md)

**热门主题：**
- [PCIe 简介](pcie/wiki/introduction/overview.md) - 什么是 PCIe？历史、特点、应用
- [拓扑结构](pcie/wiki/introduction/topology.md) - Root Complex、Switch、Endpoint
- [三层协议模型](pcie/wiki/architecture/layering.md) - TL、DLL、PHY 完整解析
- [配置空间](pcie/wiki/configuration/config-space.md) - 设备配置机制
- [中断机制](pcie/wiki/interrupts/overview.md) - INTx、MSI、MSI-X

---

## 🚀 快速开始

### 选择你的学习路径

**如果你是：**

- **存储开发者** → 从 [NVMe 控制器](nvme/concepts/controller.md) 开始
- **驱动开发者** → 从 [PCIe 配置空间](pcie/wiki/configuration/config-space.md) 和 [中断机制](pcie/wiki/interrupts/overview.md) 开始
- **硬件工程师** → 从 [PCIe 物理层](pcie/wiki/physical-layer/overview.md) 开始
- **系统架构师** → 同时学习 [NVMe 系统模型](nvme/CONCEPT_INDEX.md#系统模型) 和 [PCIe 拓扑结构](pcie/wiki/introduction/topology.md)

### 按主题浏览

#### NVMe 主题
- **系统架构**：了解 NVMe 的整体结构和组件
- **通信机制**：掌握命令和数据的传输方式
- **存储组织**：理解底层资源的管理
- **控制平面**：学习配置和监控方法
- **生命周期管理**：熟悉初始化、错误恢复和电源管理

详细分类请参见 [NVMe 概念索引](nvme/CONCEPT_INDEX.md)

#### PCIe 主题
- **基础概念**：协议分层、拓扑结构、设备类型
- **物理层**：链路训练、编码方式、电气规范
- **数据链路层**：流量控制、确认/重传、错误检测
- **事务层**：TLP 格式、路由、排序规则
- **高级特性**：SR-IOV、ACS、热插拔、电源管理

详细内容请参见 [PCIe Wiki 主页](pcie/wiki/README.md)

---

## 📂 文档结构

```
spec-repo/
├── README.md                    # 本文件
├── nvme/                        # NVMe Base 2.1 规范
│   ├── CONCEPT_INDEX.md         # 概念索引
│   ├── CONTEXT.md               # 术语表
│   ├── concepts/                # 73 个概念文档
│   │   ├── controller.md
│   │   ├── namespace.md
│   │   ├── queue-pair.md
│   │   └── ...
│   ├── maps/                    # 架构图与文档映射
│   ├── _meta/                   # 元数据
│   ├── _source/                 # 源规范引用
│   └── _review/                 # 审阅材料
└── pcie/                        # PCIe 5.0 规范
    ├── README.md                # PCIe 项目介绍
    ├── QUICKSTART.md            # 快速开始
    ├── SPEC_INDEX.md            # 规范索引
    └── wiki/                    # 84 个 Wiki 文档
        ├── introduction/        # 入门介绍
        ├── architecture/        # 架构设计
        ├── physical-layer/      # 物理层
        ├── data-link-layer/     # 数据链路层
        ├── transaction-layer/   # 事务层
        ├── configuration/       # 配置空间
        ├── interrupts/          # 中断机制
        ├── power-management/    # 电源管理
        ├── error-handling/      # 错误处理
        ├── advanced-features/   # 高级特性
        ├── implementation/      # 实现指南
        └── glossary/            # 术语表
```

---

## 🎯 规范来源

- **NVMe**：基于 NVM Express Base Specification Revision 2.1
- **PCIe**：基于 PCI Express Base Specification Revision 5.0

官方规范下载：
- [NVMe 官网](https://nvmexpress.org/)
- [PCI-SIG 官网](https://pcisig.com/)

---

## 💡 如何使用

### 查找特定内容

1. **NVMe 内容**：浏览 [NVMe 概念索引](nvme/CONCEPT_INDEX.md) 或搜索 `nvme/concepts/` 目录
2. **PCIe 内容**：浏览 [PCIe Wiki 主页](pcie/wiki/README.md) 或查看 [规范索引](pcie/SPEC_INDEX.md)
3. **术语查询**：
   - NVMe 术语：[CONTEXT.md](nvme/CONTEXT.md)
   - PCIe 术语：[术语表](pcie/wiki/glossary/index.md)

### 深入学习

每个文档都包含：
- **清晰的中文解释**：自然流畅的语言
- **工作原理图表**：直观的示意图
- **关键机制详解**：深入的技术细节
- **规范依据**：原始规范的页码引用

---

## 🤝 贡献指南

欢迎提出改进建议！如果你发现：
- 翻译不够准确或自然
- 概念解释不够清晰
- 缺少重要的说明
- 有更好的图表或示例

请提交 Issue 或 Pull Request。

---

## 📜 许可证

本项目基于官方技术规范创建，遵循相关知识共享原则。

---

## 🔗 相关资源

### NVMe
- [NVMe 官方网站](https://nvmexpress.org/)
- [NVMe 规范文档](https://nvmexpress.org/specifications/)
- [NVMe 开发者社区](https://nvmexpress.org/developers/)

### PCIe
- [PCI-SIG 官方网站](https://pcisig.com/)
- [PCIe 规范下载](https://pcisig.com/specifications)
- [PCIe 开发者论坛](https://pcisig.com/developers)

---

**⭐ 如果这个项目对你有帮助，请给个 Star！**
