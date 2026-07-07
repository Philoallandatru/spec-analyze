# 命令集（Command Sets）

NVMe 将命令划分为三大类：管理命令集（Admin Command Set）、一个或多个 I/O 命令集，以及 Fabrics 命令集。命令应该提交到哪条队列、在控制器的何种状态下被处理，都由它所属的命令集决定。

[规范 PDF 第 41 页](../_source/pages/page-041.md)

## 理解命令集的组织结构

```text
NVMe 命令集分类
│
├── 管理命令集（Admin Command Set）
│   └── 提交到 → 管理提交队列（Admin Submission Queue）
│
├── I/O 命令集（I/O Command Sets）
│   ├── NVM 命令集
│   ├── 分区命名空间命令集（Zoned Namespace）
│   └── 键值命令集（Key Value，示例）
│   └── 提交到 → I/O 提交队列
│
└── Fabrics 命令集（Fabrics Command Set）
    └── 提交到 → 管理提交队列；部分命令也可用 I/O 队列
```

这是对规范图 5 的文字化重建。[规范 PDF 第 41 页](../_source/pages/page-041.md)

**Fabrics 命令的特殊性：**
与管理命令和 I/O 命令不同，Fabrics 命令无论控制器是否启用（enabled），控制器都会处理它们。这使得 Fabrics 命令可以用于建立连接等基础操作。

[规范 PDF 第 41 页](../_source/pages/page-041.md)

## 命名空间与 I/O 命令集的绑定

命名空间在创建时会绑定到一个特定的 I/O 命令集，这个绑定关系在命名空间的整个生命周期内都不会改变：

```text
创建命名空间
    |
    └─→ 选择唯一一个 I/O 命令集
            |
            └─→ 绑定关系在命名空间生命周期内固定不变

控制器必须支持并启用该命令集
    |
    └─→ 命名空间才能挂载；命令按该命令集的规则解释
```

**I/O 命令集决定数据模型：**
- **NVM 命令集**：将数据表示为逻辑块（logical blocks）
- **键值命令集**：将数据表示为键值对（key-value pairs）

命名空间只能挂载到既支持又启用了其 I/O 命令集的控制器上。一个控制器可以同时支持多个 I/O 命令集。

[规范 PDF 第 52 页](../_source/pages/page-052.md)

## I/O 命令集配置（Profile）

当控制器配置寄存器 `CC.CSS=110b` 选择"所有支持的 I/O 命令集"模式时，可以通过 I/O 命令集配置特性（Feature ID `19h`）来选择一个已声明的命令集组合。

**配置规则：**
- 索引值为零的组合会被拒绝
- 如果组合中排除了某个已挂载命名空间正在使用的命令集，也会被拒绝
- 设置成功后，立即切换到该组合

**其他 CSS 模式下：**
对于其他 `CC.CSS` 值，该特性设置会成功完成，但不会改变实际的命令集选择。

[规范 PDF 第 410-411 页](../_source/pages/page-410.md)

## 规范依据

- [命令集的类型与路由，PDF 第 41 页](../_source/pages/page-041.md)
- [命名空间与 I/O 命令集的绑定，PDF 第 52 页](../_source/pages/page-052.md)
- [I/O 命令集配置特性，PDF 第 410-411 页](../_source/pages/page-410.md)
