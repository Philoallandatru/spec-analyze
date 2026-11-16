# PCIe Wiki 快速开始指南

欢迎使用 **PCI Express 技术 Wiki**！本仓库系统覆盖 PCIe 架构、三层协议栈（事务层 / 数据链路层 / 物理层）、配置空间、中断、错误处理、电源管理、安全/虚拟化特性及 QEMU/FEMU/Linux 驱动实现。

---

## 1. 从哪里开始？

- **零基础入门**：[PCIe 概述](wiki/introduction/overview.md) → [拓扑结构](wiki/introduction/topology.md) → [三层协议模型](wiki/architecture/layering.md)
- **按角色选路线**：[完整学习路径](wiki/introduction/learning-path.md)
- **查全站目录**：[Wiki 主页](wiki/README.md) · [规范与文档总索引](SPEC_INDEX.md)
- **随手查术语/寄存器**：[术语表](wiki/glossary/index.md) · [寄存器速查表](wiki/glossary/register-quick-ref.md) · [常见问题 (FAQ)](wiki/glossary/faq.md)

---

## 2. 按工程角色推荐路径

### 硬件工程师（物理层与链路）
1. [物理层概述](wiki/physical-layer/overview.md) — Lane、链路宽度与速率等级
2. [编码机制](wiki/physical-layer/encoding.md) — 8b/10b、128b/130b 与 PAM4
3. [LTSSM 状态机](wiki/physical-layer/ltssm.md) — 链路训练与恢复状态机
4. [链路均衡](wiki/physical-layer/link-equalization.md) · [电气规范](wiki/physical-layer/electrical-spec.md) · [Retimer](wiki/physical-layer/retimer.md)

### 驱动与系统软件开发者（配置、DMA 与中断）
1. [配置空间结构](wiki/configuration/config-space.md) — Type 0/1 头部与 ECAM
2. [BAR 寄存器](wiki/configuration/bars.md) — MMIO / I/O 地址映射
3. [中断机制概述](wiki/interrupts/overview.md) · [MSI-X](wiki/interrupts/msix.md) — 现代多队列中断
4. [驱动开发指南](wiki/implementation/driver-guide.md) · [NVMe over PCIe](wiki/implementation/nvme.md)

### 系统架构师与验证工程师（协议、QoS 与安全）
1. [TLP 格式](wiki/transaction-layer/tlp-format.md) · [事务类型](wiki/transaction-layer/transaction-types.md)
2. [流量控制](wiki/transaction-layer/flow-control.md) · [排序规则](wiki/transaction-layer/ordering.md) · [虚拟通道](wiki/transaction-layer/virtual-channels.md)
3. [错误类型](wiki/error-handling/error-types.md) · [AER 高级错误报告](wiki/error-handling/aer.md)
4. [SR-IOV 虚拟化](wiki/advanced-features/sriov.md) · [ACS 隔离](wiki/advanced-features/acs.md) · [IDE 链路加密](wiki/advanced-features/ide.md)

---

## 3. 仓库结构概览

```text
pcie/
├── README.md                              # 仓库总览
├── QUICKSTART.md                          # 本快速开始指南
├── SPEC_INDEX.md                          # 按规范章节与角色的完整索引
├── pcie-deep-analysis.md                  # QEMU/FEMU PCIe 实现深度分析
├── pcie-spec-analysis.md                  # PCIe 5.0 规范结构分析
├── PCI Express Base Specification.pdf     # 官方规范原文
└── wiki/                                  # 70+ 篇专题文档
    ├── README.md                          # Wiki 导航主页
    ├── introduction/                      # 入门（概述、拓扑、PCI vs PCIe、学习路径）
    ├── architecture/                      # 架构（分层、包类型、地址空间、枚举、时钟）
    ├── transaction-layer/                 # 事务层（TLP、路由、流控、排序、VC、ECRC）
    ├── data-link-layer/                   # 数据链路层（DLLP、流控初始化、Ack/Nak、LCRC）
    ├── physical-layer/                    # 物理层（编码、LTSSM、均衡、电气、Retimer）
    ├── configuration/                     # 配置空间（Type 0/1、BAR、Capabilities）
    ├── interrupts/                        # 中断（INTx、MSI、MSI-X、路由）
    ├── error-handling/                    # 错误处理（检测、恢复、AER、错误注入）
    ├── power-management/                  # 电源管理（D/L 状态、ASPM、L1 子状态、CLKREQ#）
    ├── advanced-features/                 # 高级特性（SR-IOV、ACS、ARI、TPH、PTM、DOE、IDE）
    ├── implementation/                    # 实现参考（FEMU、Linux 驱动、NVMe）
    └── glossary/                          # 参考资料（术语表、寄存器速查、FAQ、规范索引）
```

---

## 4. 离线阅读与检索技巧

- **推荐编辑器**：使用 **VS Code**（搭配 Markdown Preview Enhanced）或 **Obsidian**（直接打开 `wiki/` 目录即可获得双向链接与图谱视图）。
- **命令行快速检索术语**：

```bash
# 在 wiki/ 中搜索指定关键词（如 MSI-X 或 Replay Buffer）
grep -rn "MSI-X" wiki/
```

---

## 关联与引申文档

- **全局导航**：[Wiki 主页](wiki/README.md) · [完整规范索引](SPEC_INDEX.md) · [仓库说明](README.md)
- **速查手册**：[PCIe 术语表](wiki/glossary/index.md) · [寄存器速查表](wiki/glossary/register-quick-ref.md) · [常见问题 (FAQ)](wiki/glossary/faq.md)
