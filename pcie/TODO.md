# PCIe Wiki 待完成内容清单

**当前完成度**：9/50+ 页面 (18%)

本文档列出了所有待完成的内容，按优先级和重要性排序。

---

## 🔴 优先级 1：核心协议内容（最重要）

这些是理解 PCIe 的关键内容，应该优先完成。

### 事务层 (Transaction Layer) - 6 页

| 页面 | 文件路径 | 重要性 | 预计字数 |
|------|---------|--------|---------|
| ✅ TLP 格式详解 | `transaction-layer/tlp-format.md` | ⭐⭐⭐⭐⭐ | 已完成 |
| ⬜ **事务类型详解** | `transaction-layer/transaction-types.md` | ⭐⭐⭐⭐⭐ | 4000 |
| ⬜ **流控制机制** | `transaction-layer/flow-control.md` | ⭐⭐⭐⭐⭐ | 3500 |
| ⬜ **排序规则** | `transaction-layer/ordering.md` | ⭐⭐⭐⭐ | 3000 |
| ⬜ 路由机制 | `transaction-layer/routing.md` | ⭐⭐⭐⭐ | 2500 |
| ⬜ 虚拟通道 | `transaction-layer/virtual-channels.md` | ⭐⭐⭐ | 2500 |
| ⬜ ECRC | `transaction-layer/ecrc.md` | ⭐⭐⭐ | 2000 |

**建议优先完成**：事务类型详解、流控制机制、排序规则

### 数据链路层 (Data Link Layer) - 5 页

| 页面 | 文件路径 | 重要性 | 预计字数 |
|------|---------|--------|---------|
| ⬜ **DLLP 格式** | `data-link-layer/dllp-format.md` | ⭐⭐⭐⭐⭐ | 3000 |
| ⬜ **ACK/NAK 协议** | `data-link-layer/ack-nak.md` | ⭐⭐⭐⭐⭐ | 3500 |
| ⬜ 重试缓冲区 | `data-link-layer/retry-buffer.md` | ⭐⭐⭐⭐ | 2500 |
| ⬜ LCRC | `data-link-layer/lcrc.md` | ⭐⭐⭐ | 2000 |
| ⬜ 流控制初始化 | `data-link-layer/fc-init.md` | ⭐⭐⭐ | 2500 |

**建议优先完成**：DLLP 格式、ACK/NAK 协议

### 物理层 (Physical Layer) - 8 页

| 页面 | 文件路径 | 重要性 | 预计字数 |
|------|---------|--------|---------|
| ⬜ **LTSSM 状态机** | `physical-layer/ltssm.md` | ⭐⭐⭐⭐⭐ | 5000 |
| ⬜ **编码方案** | `physical-layer/encoding.md` | ⭐⭐⭐⭐⭐ | 4000 |
| ⬜ 物理层概述 | `physical-layer/overview.md` | ⭐⭐⭐⭐ | 3000 |
| ⬜ 链路训练 | `physical-layer/link-training.md` | ⭐⭐⭐⭐ | 3500 |
| ⬜ 链路均衡 | `physical-layer/link-equalization.md` | ⭐⭐⭐ | 3000 |
| ⬜ 时钟架构 | `physical-layer/clocking.md` | ⭐⭐⭐ | 2500 |
| ⬜ Retimer | `physical-layer/retimer.md` | ⭐⭐ | 2500 |
| ⬜ 电气规范 | `physical-layer/electrical.md` | ⭐⭐⭐ | 3500 |

**建议优先完成**：LTSSM 状态机、编码方案、物理层概述

---

## 🟠 优先级 2：软件接口（驱动开发必需）

### 配置空间 (Configuration) - 6 页

| 页面 | 文件路径 | 重要性 | 预计字数 |
|------|---------|--------|---------|
| ⬜ **配置空间布局** | `configuration/config-space.md` | ⭐⭐⭐⭐⭐ | 4000 |
| ⬜ **BARs 详解** | `configuration/bars.md` | ⭐⭐⭐⭐⭐ | 3500 |
| ⬜ Type 0/1 Header | `configuration/header-types.md` | ⭐⭐⭐⭐ | 3000 |
| ⬜ 能力结构 | `configuration/capabilities.md` | ⭐⭐⭐⭐ | 3500 |
| ⬜ PCIe 能力详解 | `configuration/pcie-capability.md` | ⭐⭐⭐⭐ | 4000 |
| ⬜ 扩展能力 | `configuration/extended-capabilities.md` | ⭐⭐⭐ | 3500 |

**建议优先完成**：配置空间布局、BARs 详解

### 中断机制 (Interrupts) - 5 页

| 页面 | 文件路径 | 重要性 | 预计字数 |
|------|---------|--------|---------|
| ⬜ **中断概述** | `interrupts/overview.md` | ⭐⭐⭐⭐⭐ | 3000 |
| ⬜ **MSI-X 详解** | `interrupts/msix.md` | ⭐⭐⭐⭐⭐ | 4000 |
| ⬜ MSI 详解 | `interrupts/msi.md` | ⭐⭐⭐⭐ | 3500 |
| ⬜ Legacy INTx | `interrupts/intx.md` | ⭐⭐⭐ | 2500 |
| ⬜ 中断路由 | `interrupts/routing.md` | ⭐⭐⭐ | 2500 |

**建议优先完成**：中断概述、MSI-X 详解

---

## 🟡 优先级 3：错误处理和电源管理

### 错误处理 (Error Handling) - 5 页

| 页面 | 文件路径 | 重要性 | 预计字数 |
|------|---------|--------|---------|
| ⬜ **AER 详解** | `error-handling/aer.md` | ⭐⭐⭐⭐⭐ | 4500 |
| ⬜ 错误分类 | `error-handling/error-types.md` | ⭐⭐⭐⭐ | 3000 |
| ⬜ 错误检测 | `error-handling/detection.md` | ⭐⭐⭐⭐ | 3000 |
| ⬜ 错误恢复 | `error-handling/recovery.md` | ⭐⭐⭐ | 3000 |
| ⬜ 错误注入 | `error-handling/injection.md` | ⭐⭐ | 2000 |

**建议优先完成**：AER 详解、错误分类

### 电源管理 (Power Management) - 5 页

| 页面 | 文件路径 | 重要性 | 预计字数 |
|------|---------|--------|---------|
| ⬜ **ASPM** | `power-management/aspm.md` | ⭐⭐⭐⭐⭐ | 4000 |
| ⬜ 电源状态 | `power-management/power-states.md` | ⭐⭐⭐⭐ | 3500 |
| ⬜ L1 子状态 | `power-management/l1-substates.md` | ⭐⭐⭐ | 2500 |
| ⬜ 设备电源状态 | `power-management/device-states.md` | ⭐⭐⭐ | 2500 |
| ⬜ 时钟电源管理 | `power-management/clkreq.md` | ⭐⭐ | 2000 |

**建议优先完成**：ASPM、电源状态

---

## 🟢 优先级 4：高级特性

### 高级特性 (Advanced Features) - 9 页

| 页面 | 文件路径 | 重要性 | 预计字数 |
|------|---------|--------|---------|
| ⬜ **SR-IOV** | `advanced-features/sriov.md` | ⭐⭐⭐⭐⭐ | 5000 |
| ⬜ 热插拔 | `advanced-features/hot-plug.md` | ⭐⭐⭐⭐ | 3500 |
| ⬜ ACS | `advanced-features/acs.md` | ⭐⭐⭐⭐ | 3000 |
| ⬜ ARI | `advanced-features/ari.md` | ⭐⭐⭐ | 2500 |
| ⬜ AtomicOps | `advanced-features/atomic-ops.md` | ⭐⭐⭐ | 3000 |
| ⬜ TPH | `advanced-features/tph.md` | ⭐⭐ | 2500 |
| ⬜ PTM | `advanced-features/ptm.md` | ⭐⭐ | 2500 |
| ⬜ DOE | `advanced-features/doe.md` | ⭐⭐ | 3000 |
| ⬜ IDE | `advanced-features/ide.md` | ⭐⭐ | 3000 |

**建议优先完成**：SR-IOV、热插拔

---

## 🔵 优先级 5：实现和实践

### 实现参考 (Implementation) - 6 页

| 页面 | 文件路径 | 重要性 | 预计字数 |
|------|---------|--------|---------|
| ⬜ **驱动开发指南** | `implementation/driver-guide.md` | ⭐⭐⭐⭐⭐ | 5000 |
| ⬜ **NVMe over PCIe** | `implementation/nvme.md` | ⭐⭐⭐⭐⭐ | 4500 |
| ⬜ FEMU 代码导读 | `implementation/femu-overview.md` | ⭐⭐⭐⭐ | 4000 |
| ⬜ 配置空间实现 | `implementation/config-space.md` | ⭐⭐⭐ | 3000 |
| ⬜ 中断实现 | `implementation/interrupts.md` | ⭐⭐⭐ | 3000 |
| ⬜ AER 实现 | `implementation/aer.md` | ⭐⭐⭐ | 3000 |

**建议优先完成**：驱动开发指南、NVMe over PCIe

### 架构补充 (Architecture) - 3 页

| 页面 | 文件路径 | 重要性 | 预计字数 |
|------|---------|--------|---------|
| ⬜ 数据包类型 | `architecture/packet-types.md` | ⭐⭐⭐⭐ | 3000 |
| ⬜ 设备枚举 | `architecture/enumeration.md` | ⭐⭐⭐⭐ | 3500 |
| ⬜ 地址空间 | `architecture/address-space.md` | ⭐⭐⭐ | 3000 |

### 入门指南补充 (Introduction) - 2 页

| 页面 | 文件路径 | 重要性 | 预计字数 |
|------|---------|--------|---------|
| ⬜ 与 PCI 的区别 | `introduction/pci-vs-pcie.md` | ⭐⭐⭐ | 2500 |
| ⬜ 学习路径 | `introduction/learning-path.md` | ⭐⭐⭐ | 2000 |

### 参考资料补充 (Glossary) - 3 页

| 页面 | 文件路径 | 重要性 | 预计字数 |
|------|---------|--------|---------|
| ⬜ 寄存器速查 | `glossary/register-quick-ref.md` | ⭐⭐⭐⭐ | 4000 |
| ⬜ FAQ | `glossary/faq.md` | ⭐⭐⭐ | 3000 |
| ⬜ 规范章节索引 | `glossary/spec-index.md` | ⭐⭐ | 2000 |

---

## 📊 内容增强需求

除了新页面，现有内容也需要增强：

### 1. 图表增强（高优先级）

**需要添加的图表类型**：

#### Mermaid 流程图和状态图
- [ ] LTSSM 完整状态机图
- [ ] 数据链路层状态机
- [ ] TLP 处理流程图
- [ ] ACK/NAK 协议时序图
- [ ] 链路训练流程图
- [ ] 设备枚举流程图
- [ ] 中断处理流程图
- [ ] 错误处理流程图

#### 架构图（Draw.io 或手绘）
- [ ] 完整的 PCIe 拓扑示例图
- [ ] 三层协议栈详细图
- [ ] Switch 内部架构图
- [ ] Root Complex 架构图
- [ ] 虚拟通道示意图
- [ ] SR-IOV 架构图

#### 格式图（精美化）
- [ ] TLP 各种格式的详细图
- [ ] DLLP 格式图
- [ ] 配置空间布局图（256B + 4KB）
- [ ] BAR 结构图
- [ ] MSI/MSI-X 表格结构图
- [ ] AER 寄存器布局图

#### 时序图
- [ ] Memory Read 完整时序
- [ ] Memory Write 时序
- [ ] 流控制信用交换时序
- [ ] 链路训练完整时序
- [ ] 热插拔事件时序

**预计需要**：30-40 个高质量图表

### 2. 代码示例增强

需要添加更多实际代码示例：

#### Linux Kernel 示例
- [ ] PCIe 设备初始化代码
- [ ] BAR 映射代码
- [ ] MSI-X 配置代码
- [ ] DMA 映射代码
- [ ] 配置空间访问代码
- [ ] 错误处理代码

#### FEMU/QEMU 示例
- [ ] PCIe 设备模拟完整示例
- [ ] 能力结构添加示例
- [ ] 中断注入示例
- [ ] 配置空间读写示例
- [ ] AER 错误注入示例

#### 用户空间示例
- [ ] 使用 libpci 访问设备
- [ ] 使用 sysfs 访问配置空间
- [ ] UIO 驱动示例
- [ ] VFIO 使用示例

**预计需要**：30+ 个代码示例

### 3. 实践案例

需要添加真实的使用案例：

- [ ] **GPU 直通配置**（虚拟化场景）
- [ ] **NVMe SSD 性能调优**
- [ ] **网卡 SR-IOV 配置**
- [ ] **PCIe 链路问题诊断**
- [ ] **热插拔操作实践**
- [ ] **电源管理优化**
- [ ] **错误恢复实践**
- [ ] **驱动开发完整案例**

### 4. 性能和调试

新增内容：

- [ ] **性能优化指南**
  - Max Payload Size 调整
  - Max Read Request Size 优化
  - Relaxed Ordering 使用
  - ASPM 配置权衡
  - 拓扑优化建议

- [ ] **调试和故障排查**
  - lspci 高级用法
  - setpci 使用技巧
  - dmesg 日志分析
  - PCIe 错误日志解读
  - 链路训练问题诊断
  - 性能问题定位

- [ ] **测试和验证**
  - PCIe 一致性测试
  - 压力测试方法
  - 错误注入测试
  - 性能基准测试

---

## 🎨 互动工具开发（长期）

这些可以作为 Web 应用开发：

### 基础工具
- [ ] **TLP 构建器**
  - 可视化构建 TLP
  - 显示字段含义
  - 生成二进制或十六进制

- [ ] **BDF 地址解析器**
  - 输入 Bus:Device.Function
  - 显示地址组成
  - 计算配置空间偏移

- [ ] **流控制计算器**
  - 输入缓冲区大小
  - 计算信用数量
  - 模拟流控制过程

### 高级工具
- [ ] **LTSSM 可视化器**
  - 动画展示状态转换
  - 显示各状态含义
  - 模拟链路训练过程

- [ ] **配置空间编辑器**
  - 可视化配置空间
  - 高亮关键字段
  - 解释每个位的含义

- [ ] **排序规则检查器**
  - 输入 TLP 序列
  - 检查是否违反排序
  - 解释排序约束

---

## 📅 建议的完成顺序

### 第 1 周：核心协议（6-8 页）
1. 事务类型详解
2. 流控制机制
3. DLLP 格式
4. ACK/NAK 协议
5. 排序规则
6. 配置空间布局

### 第 2 周：物理层和软件接口（6-8 页）
1. LTSSM 状态机
2. 编码方案
3. BARs 详解
4. 中断概述
5. MSI-X 详解
6. 物理层概述

### 第 3 周：错误处理和高级特性（6-8 页）
1. AER 详解
2. ASPM
3. SR-IOV
4. 驱动开发指南
5. NVMe over PCIe
6. 错误分类

### 第 4 周：补充和完善（6-8 页）
1. 补充剩余页面
2. 添加 Mermaid 图表
3. 增加代码示例
4. 添加实践案例
5. 完善现有页面

---

## 📈 预计工作量

| 类别 | 页面数 | 预计字数 | 图表数 | 代码示例 |
|------|--------|---------|--------|---------|
| **已完成** | 9 | 25,000 | 25 | 15 |
| **优先级 1** | 19 | 60,000 | 15 | 20 |
| **优先级 2** | 11 | 35,000 | 10 | 15 |
| **优先级 3** | 10 | 30,000 | 8 | 10 |
| **优先级 4** | 9 | 27,000 | 5 | 8 |
| **优先级 5** | 14 | 42,000 | 12 | 25 |
| **总计** | **72** | **219,000** | **75** | **93** |

**预计总工作时间**：
- 编写内容：60-80 小时
- 制作图表：20-30 小时
- 代码示例：15-20 小时
- 测试验证：10-15 小时
- **总计**：105-145 小时（约 3-4 周全职工作）

---

## 💡 快速启动建议

如果时间有限，建议按以下"最小可用集"完成：

### 核心最小集（15 页，约 50,000 字）

**必须完成**：
1. 事务类型详解 ⭐⭐⭐⭐⭐
2. 流控制机制 ⭐⭐⭐⭐⭐
3. DLLP 格式 ⭐⭐⭐⭐⭐
4. ACK/NAK 协议 ⭐⭐⭐⭐⭐
5. LTSSM 状态机 ⭐⭐⭐⭐⭐
6. 编码方案 ⭐⭐⭐⭐⭐
7. 配置空间布局 ⭐⭐⭐⭐⭐
8. BARs 详解 ⭐⭐⭐⭐⭐
9. 中断概述 ⭐⭐⭐⭐⭐
10. MSI-X 详解 ⭐⭐⭐⭐⭐
11. AER 详解 ⭐⭐⭐⭐⭐
12. ASPM ⭐⭐⭐⭐⭐
13. SR-IOV ⭐⭐⭐⭐⭐
14. 驱动开发指南 ⭐⭐⭐⭐⭐
15. NVMe over PCIe ⭐⭐⭐⭐⭐

完成这 15 页后，Wiki 就具备了基本的可用性。

---

## 🎯 总结

**当前进度**：9/72 页 (12.5%)

**核心缺失内容**：
- 事务层详细机制（流控制、排序）
- 数据链路层可靠传输机制
- 物理层状态机和编码
- 配置空间和 BARs
- 中断机制详解
- 实际代码和案例

**建议行动**：
1. 先完成"核心最小集"15 页
2. 逐步添加图表和代码示例
3. 根据反馈调整优先级
4. 持续迭代和完善

---

*文档版本：v1.0*  
*最后更新：2026-07-06*
