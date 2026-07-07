# NVMe 规范概念解析

这是一个面向中文读者的 NVMe Base 2.1 规范概念知识库。我们将官方规范中的技术概念改写为更易理解的中文文档，帮助开发者和工程师快速掌握 NVMe 的核心机制。

## 项目特色

- ✅ **中文友好**：摆脱生硬的机器翻译，使用自然流畅的中文表达
- 📚 **系统化组织**：按功能模块分类，便于查找和学习
- 🔍 **概念清晰**：关键术语中英对照，降低理解门槛
- 📖 **完整覆盖**：涵盖 NVMe Base 2.1 规范的所有核心概念
- 🔗 **追溯源头**：保留规范页码引用，方便查阅原文

## 快速开始

### 核心概念

如果你刚接触 NVMe，建议从这些基础概念开始：

1. [控制器（Controller）](concepts/controller.md) - 主机与存储系统之间的桥梁
2. [命名空间（Namespace）](concepts/namespace.md) - NVMe 的存储抽象单元
3. [队列对（Queue Pair）](concepts/queue-pair.md) - 命令提交与完成的机制
4. [命令集（Command Sets）](concepts/command-sets.md) - 命令的分类与路由
5. [NVM 子系统（NVM Subsystem）](concepts/nvm-subsystem.md) - 系统边界与组成

### 按主题浏览

- **系统架构**：了解 NVMe 的整体结构和组件
- **通信机制**：掌握命令和数据的传输方式
- **存储组织**：理解底层资源的管理
- **控制平面**：学习配置和监控方法
- **生命周期管理**：熟悉初始化、错误恢复和电源管理

详细的主题分类请参见 [概念索引](CONCEPT_INDEX.md)。

## 文档结构

```
.
├── CONCEPT_INDEX.md          # 概念索引（按功能分类）
├── CONTEXT.md                # 术语表与定义
├── concepts/                 # 概念文档目录
│   ├── controller.md         # 控制器
│   ├── namespace.md          # 命名空间
│   ├── queue-pair.md         # 队列对
│   └── ...                   # 更多概念
└── maps/                     # 架构图与文档映射
```

## 规范来源

本知识库基于 **NVM Express Base Specification Revision 2.1**，这是 NVMe 标准的最新版本。

官方规范下载：[NVMe 官网](https://nvmexpress.org/)

## 如何使用

### 查找特定概念

1. 浏览 [概念索引](CONCEPT_INDEX.md) 查找感兴趣的主题
2. 使用 GitHub 搜索功能搜索关键词
3. 通过术语表 [CONTEXT.md](CONTEXT.md) 查找专业术语

### 深入学习

每个概念文档都包含：
- **概念说明**：清晰的中文解释
- **工作原理**：图表和示例
- **关键机制**：详细的技术细节
- **规范依据**：原始规范的页码引用

## 贡献指南

欢迎提出改进建议！如果你发现：
- 翻译不够准确或自然
- 概念解释不够清晰
- 缺少重要的说明

请提交 Issue 或 Pull Request。

## 许可证

本项目基于 NVMe 规范创建，遵循相关知识共享原则。

## 相关资源

- [NVMe 官方网站](https://nvmexpress.org/)
- [NVMe 规范文档](https://nvmexpress.org/specifications/)
- [NVMe 开发者社区](https://nvmexpress.org/developers/)
