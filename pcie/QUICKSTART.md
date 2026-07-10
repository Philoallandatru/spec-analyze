# PCIe Wiki 快速开始指南

欢迎使用 PCI Express 技术 Wiki！本指南将帮助您快速上手。

---

## 📚 如何阅读本 Wiki

### 从这里开始

1. **初学者**：从 [PCIe 简介](wiki/introduction/overview.md) 开始
2. **有基础**：直接跳到 [三层协议模型](wiki/architecture/layering.md)
3. **查找术语**：使用 [术语表](wiki/glossary/index.md)
4. **全局导航**：查看 [主页](wiki/README.md)

### 推荐学习路径

#### 路径 1：硬件工程师
```
1. PCIe 简介 → 2. 拓扑结构 → 3. 物理层 → 4. LTSSM → 5. 电气规范
```

#### 路径 2：驱动开发者
```
1. PCIe 简介 → 2. 配置空间 → 3. BARs → 4. 中断机制 → 5. 驱动开发
```

#### 路径 3：系统架构师
```
1. 拓扑结构 → 2. 三层协议 → 3. 流控制 → 4. 虚拟化 (SR-IOV)
```

---

## 🗂️ 项目结构

```
pcie-wiki/
│
├── PCI Express Base Specification.pdf   # 官方规范文档（11MB）
├── pcie-deep-analysis.md                # FEMU 代码分析报告（17KB）
├── pcie-spec-analysis.md                # PCIe 5.0 规范分析报告
├── pcie_spec_text.txt                   # PDF 提取文本（中间文件）
├── PROJECT_SUMMARY.md                   # 项目进度和计划
├── QUICKSTART.md                        # 本文件
│
└── wiki/                                # Wiki 主目录
    ├── README.md                        # Wiki 主页（从这里开始！）
    │
    ├── introduction/                    # 入门指南
    │   ├── overview.md                  # ✅ PCIe 简介
    │   └── topology.md                  # ✅ 拓扑结构
    │
    ├── architecture/                    # 架构
    │   └── layering.md                  # ✅ 三层协议模型
    │
    ├── transaction-layer/               # 事务层（待建设）
    ├── data-link-layer/                 # 数据链路层（待建设）
    ├── physical-layer/                  # 物理层（待建设）
    ├── configuration/                   # 配置空间（待建设）
    ├── interrupts/                      # 中断机制（待建设）
    ├── error-handling/                  # 错误处理（待建设）
    ├── power-management/                # 电源管理（待建设）
    ├── advanced-features/               # 高级特性（待建设）
    ├── implementation/                  # 实现参考（待建设）
    │
    ├── glossary/                        # 参考资料
    │   └── index.md                     # ✅ 术语表
    │
    └── assets/                          # 图片和资源（待添加）
```

---

## 🚀 浏览方式

### 方法 1：直接阅读 Markdown（推荐用于离线）

**使用 Markdown 编辑器**：
- VS Code + Markdown Preview Enhanced
- Typora
- Obsidian（支持双向链接）
- Mark Text

**从主页开始**：
```bash
cd pcie-wiki/wiki
# 用您喜欢的编辑器打开 README.md
code README.md
```

### 方法 2：部署为静态网站（推荐用于团队共享）

#### 使用 MkDocs（推荐）

**安装 MkDocs**：
```bash
pip install mkdocs mkdocs-material
```

**创建配置文件** `mkdocs.yml`：
```yaml
site_name: PCIe 技术 Wiki
site_description: 全面的 PCI Express 技术文档
theme:
  name: material
  language: zh
  features:
    - navigation.tabs
    - navigation.sections
    - navigation.expand
    - search.suggest
    - search.highlight
  palette:
    - scheme: default
      primary: indigo
      accent: indigo

nav:
  - 首页: README.md
  - 入门指南:
    - PCIe 简介: introduction/overview.md
    - 拓扑结构: introduction/topology.md
  - 架构:
    - 三层协议模型: architecture/layering.md
  - 参考资料:
    - 术语表: glossary/index.md

markdown_extensions:
  - pymdownx.highlight
  - pymdownx.superfences
  - pymdownx.tabbed
  - tables
  - toc:
      permalink: true
```

**启动开发服务器**：
```bash
mkdocs serve
# 访问 http://127.0.0.1:8000
```

**构建静态站点**：
```bash
mkdocs build
# 输出到 site/ 目录
```

#### 使用 Docusaurus

```bash
npx create-docusaurus@latest pcie-wiki-site classic
cd pcie-wiki-site
# 将 wiki/ 内容复制到 docs/
npm start
```

### 方法 3：在 GitHub 上浏览

如果推送到 GitHub，可以直接在线浏览 Markdown 文件，GitHub 会自动渲染。

---

## 🔍 搜索功能

### 临时方案：使用 grep

**搜索特定术语**：
```bash
cd pcie-wiki/wiki
grep -r "MSI-X" .
```

**搜索并显示上下文**：
```bash
grep -r -C 3 "flow control" .
```

### 永久方案：部署 MkDocs

MkDocs Material 主题内置强大的搜索功能，支持中文。

---

## 📖 已完成的内容

### ✅ 可以阅读的页面

1. **[Wiki 主页](wiki/README.md)**
   - 完整的导航结构
   - 学习路径建议
   - 50+ 个计划中的主题

2. **[PCIe 简介](wiki/introduction/overview.md)**
   - PCIe 是什么
   - 历史演进（Gen 1-6）
   - 与其他总线对比
   - 应用场景

3. **[拓扑结构](wiki/introduction/topology.md)**
   - Root Complex、Switch、Endpoint
   - 设备枚举过程
   - 拓扑示例
   - FEMU 实现参考

4. **[三层协议模型](wiki/architecture/layering.md)**
   - 事务层详解（TLP、流控制、排序）
   - 数据链路层详解（ACK/NAK、LCRC）
   - 物理层详解（编码、LTSSM、电气）
   - 层间交互示例

5. **[术语表](wiki/glossary/index.md)**
   - 100+ 个术语
   - 按字母排序
   - 包含缩写对照表

### 📊 内容统计

- **总页面数**：5 个
- **总字数**：约 15,000 字
- **表格数量**：20+
- **代码示例**：10+
- **ASCII 图表**：15+

---

## 📝 分析报告

除了 Wiki 内容，项目还包含两份深度分析报告：

### 1. FEMU 代码分析 (`pcie-deep-analysis.md`)

**内容**：
- 基于 FEMU/QEMU 源代码的实现分析
- 核心 PCIe 组件和文件结构
- 代码到规范的映射关系
- NVMe over PCIe 集成案例

**适合**：
- 想了解 PCIe 在软件中如何实现
- 驱动开发者
- 虚拟化工程师

### 2. PCIe 5.0 规范分析 (`pcie-spec-analysis.md`)

**内容**：
- 官方规范的完整章节结构
- 400+ 图表和 250+ 表格清单
- 关键技术细节提取
- 与 FEMU 分析的对比

**适合**：
- 想深入理解官方规范
- 硬件工程师
- 协议专家

---

## 🛠️ 开发和贡献

### 添加新页面

1. **创建 Markdown 文件**：
```bash
# 例如：添加 TLP 格式页面
touch wiki/transaction-layer/tlp-format.md
```

2. **使用模板**：
```markdown
# 页面标题

## 概述
简短介绍本页面内容...

---

## 主要内容

### 小节 1
内容...

### 小节 2
内容...

---

## 总结

### 关键要点
- 要点 1
- 要点 2

---

## 下一步学习
- [相关页面 1](link1.md)
- [相关页面 2](link2.md)

---

## 参考资料
- 规范章节
- 代码文件

---

**相关页面**：
- [← 上一页](prev.md)
- [下一页 →](next.md)
- [返回首页](../README.md)

---

*最后更新：YYYY-MM-DD*
```

3. **更新主页导航**：
在 `wiki/README.md` 中添加新页面链接。

### 添加图表

#### Mermaid 图表（推荐）
```markdown
​```mermaid
graph TD
    A[Root Complex] --> B[Switch]
    B --> C[GPU]
    B --> D[NVMe]
​```
```

#### ASCII 图表
```
┌─────────┐
│  CPU    │
└────┬────┘
     │
┌────▼────┐
│  Device │
└─────────┘
```

### 版本控制

**初始化 Git 仓库**：
```bash
cd pcie-wiki
git init
git add .
git commit -m "Initial commit: PCIe Wiki v1.0"
```

**推送到 GitHub**：
```bash
git remote add origin https://github.com/your-username/pcie-wiki.git
git push -u origin main
```

---

## 🎯 下一步建议

### 如果您想继续完善 Wiki

1. **优先级最高**：
   - 创建 TLP 格式页面
   - 创建 LTSSM 状态机页面
   - 添加更多 Mermaid 图表

2. **短期目标**：
   - 完成事务层所有页面
   - 完成数据链路层所有页面
   - 完成物理层核心页面

3. **部署**：
   - 安装 MkDocs
   - 部署到本地或云端

### 如果您想分享给他人

1. **最简单**：直接分享 `wiki/` 文件夹
2. **更专业**：部署 MkDocs 站点
3. **最方便**：推送到 GitHub

---

## 💡 使用技巧

### Markdown 编辑器推荐

- **VS Code**：
  - 安装 "Markdown All in One" 插件
  - 安装 "Markdown Preview Enhanced" 插件
  - 支持 Mermaid 预览

- **Obsidian**：
  - 支持双向链接
  - 图谱视图（可视化页面关系）
  - 本地知识库

- **Typora**：
  - 所见即所得
  - 美观的渲染
  - 支持导出 PDF

### 快速查找

**在 VS Code 中**：
- `Ctrl+P` 快速打开文件
- `Ctrl+Shift+F` 全局搜索
- `Ctrl+Click` 跟随链接

**在终端中**：
```bash
# 查找包含 "MSI-X" 的文件
find wiki -name "*.md" -exec grep -l "MSI-X" {} \;

# 搜索术语并显示位置
grep -rn "flow control" wiki/
```

---

## 📞 获取帮助

### 常见问题

**Q: Markdown 链接无法跳转？**  
A: 确保使用相对路径，例如 `[链接](../folder/page.md)`

**Q: 图表显示不正确？**  
A: 使用支持 Mermaid 的编辑器或部署 MkDocs

**Q: 如何导出 PDF？**  
A: 使用 Typora 或 Pandoc：
```bash
pandoc wiki/README.md -o output.pdf
```

### 资源

- **官方规范**：`PCI Express Base Specification.pdf`
- **深度分析**：`pcie-deep-analysis.md` 和 `pcie-spec-analysis.md`
- **项目计划**：`PROJECT_SUMMARY.md`

---

## 🎉 开始探索

**现在就开始阅读**：

```bash
# 从主页开始
cat wiki/README.md

# 或用编辑器打开
code wiki/README.md
```

**祝您学习愉快！** 🚀

---

*最后更新：2026-07-06*
