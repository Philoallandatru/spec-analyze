# PCIe 访问控制服务（ACS - Access Control Services）

## 概述

访问控制服务（ACS）是 PCIe 规范中引入的一项安全特性，旨在控制和限制 Peer-to-Peer（P2P）流量在 PCIe 拓扑中的传播。ACS 对于实现设备隔离、增强系统安全性以及支持虚拟化环境中的 I/O 虚拟化（如 SR-IOV）至关重要。

ACS 通过一组控制位和验证机制，允许系统软件精确控制哪些设备可以相互通信，以及哪些流量必须经过特定的控制点（如 Root Complex）进行验证和路由。

## ACS 的重要性

### 安全隔离

在没有 ACS 的情况下，PCIe 设备可以直接进行 P2P 通信，绕过 Root Complex 和操作系统的监控。这带来几个安全问题：

1. **恶意设备攻击**：一个被攻陷的设备可以直接访问同一 Switch 下其他设备的内存空间
2. **数据泄露**：敏感数据可能在设备间直接传输而不经过系统监控
3. **权限提升**：低权限设备可能利用 P2P 通道访问高权限资源

ACS 通过强制特定流量必须经过 Root Complex，确保所有关键通信都在操作系统的控制之下。

### 虚拟化支持

在虚拟化环境中，不同的虚拟机可能被分配不同的物理设备或 SR-IOV 虚拟功能。如果没有 ACS：

1. **虚拟机逃逸**：一个虚拟机控制的设备可能访问另一个虚拟机的设备
2. **IOMMU 绕过**：P2P 流量可能绕过 IOMMU 的地址转换和保护
3. **隔离失效**：虚拟机之间的隔离保证失效

ACS 与 IOMMU 配合，为虚拟化环境提供了必要的硬件隔离保证。

### 拓扑发现和管理

ACS 还帮助系统软件理解 PCIe 拓扑的实际特性，识别哪些设备可以进行 P2P 通信，从而做出正确的资源分配和隔离决策。

## ACS Capability 结构

ACS 通过一个扩展 Capability 结构实现，通常位于 Root Port、Switch Upstream Port 和 Switch Downstream Port 的配置空间中。

### ACS Capability Register（偏移 0x04）

该寄存器指示端口支持的 ACS 功能：

```
Bit 0  : Source Validation (V)
Bit 1  : Translation Blocking (B)
Bit 2  : P2P Request Redirect (R)
Bit 3  : P2P Completion Redirect (C)
Bit 4  : Upstream Forwarding (U)
Bit 5  : P2P Egress Control (E)
Bit 6  : Direct Translated P2P (T)
Bits 7 : Enhanced Capability (EC)
Bits 8-15: Reserved
```

### ACS Control Register（偏移 0x06）

该寄存器允许软件启用或禁用各项 ACS 功能：

```
Bit 0  : Source Validation Enable
Bit 1  : Translation Blocking Enable
Bit 2  : P2P Request Redirect Enable
Bit 3  : P2P Completion Redirect Enable
Bit 4  : Upstream Forwarding Enable
Bit 5  : P2P Egress Control Enable
Bit 6  : Direct Translated P2P Enable
Bits 7-15: Reserved
```

### ACS Egress Control Vector（可选）

如果支持 P2P Egress Control，此寄存器包含一个位向量，指示允许向哪些端口转发 P2P 流量。

## ACS 核心机制

### 1. Source Validation（源验证）

Source Validation 确保所有从下游接收的 TLP（Transaction Layer Packet）都具有正确的 Requester ID。

**工作原理：**

当 Switch 或 Root Port 接收到来自下游的 TLP 时，会验证：
- TLP 的 Requester ID 中的 Bus Number 是否属于该端口的次级总线范围
- Function Number 和 Device Number 是否在有效范围内

**验证流程：**

```
接收 TLP from Downstream
  ↓
提取 Requester ID (Bus:Device:Function)
  ↓
检查 Bus Number 是否在 [Secondary Bus, Subordinate Bus] 范围
  ↓
NO → 丢弃 TLP，记录错误
  ↓
YES → 继续处理
```

**应用场景：**

- 防止设备伪造其他设备的 Requester ID
- 检测配置错误或硬件故障
- 增强系统安全性，防止 ID 欺骗攻击

**示例：**

假设一个 Switch Downstream Port 配置为：
- Primary Bus: 0
- Secondary Bus: 1  
- Subordinate Bus: 5

如果接收到一个 Requester ID 为 06:00.0 的 TLP，Source Validation 将拒绝该 TLP，因为 Bus 6 不在 [1, 5] 范围内。

### 2. Translation Blocking（转换阻止）

Translation Blocking 防止已经过地址转换的 TLP（带有 AT 字段 = "Translation Request" 或 "Translated"）在 P2P 路径上传输。

**背景：**

在支持 ATS（Address Translation Services）的系统中，设备可以请求 IOMMU 将虚拟地址转换为物理地址，转换后的 TLP 带有 "Translated" 标记。Translation Blocking 确保这些已转换的地址不能用于 P2P 通信。

**工作原理：**

检查 TLP Header 的 Address Type（AT）字段：
- `00b`：默认，未转换
- `01b`：Translation Request
- `10b`：Translated
- `11b`：保留

如果 AT 字段指示 TLP 已转换，且 TLP 的目标是 P2P（不是上游），则：
1. 阻止该 TLP 继续传输
2. 返回 Unsupported Request 错误

**重要性：**

- 已转换的地址是物理地址，只对 IOMMU 和内存控制器有意义
- 其他 PCIe 设备可能将这些地址解释为 MMIO 地址，导致错误行为
- 防止设备使用转换后的地址绕过 IOMMU 保护

**虚拟化场景：**

```
VM1 的设备请求地址转换
  ↓
IOMMU 转换: GPA → HPA
  ↓
TLP 标记为 "Translated"
  ↓
Translation Blocking 检查
  ↓
如果目标是 P2P → 阻止
如果目标是 Root Complex → 允许
```

### 3. P2P Request Redirect（P2P 请求重定向）

P2P Request Redirect 强制所有 P2P Memory Request（读写请求）重定向到上游，而不是直接转发到其他下游端口。

**正常 P2P 路由（无 ACS）：**

```
Device A (Port 1)
    ↓
   Switch
    ↓
Device B (Port 2)

Device A 向 Device B 的 BAR 写入 → 直接从 Port 1 路由到 Port 2
```

**启用 P2P Request Redirect：**

```
Device A (Port 1)
    ↓
   Switch → 上游到 Root Complex
    ↓
Root Complex 重新路由
    ↓
   Switch
    ↓
Device B (Port 2)

Device A 的请求 → 上游 → Root Complex → 下游 → Device B
```

**受影响的 TLP 类型：**

- Memory Read Request（MRd）
- Memory Write Request（MWr）
- Atomic Operations（Fetch-Add、Swap、CAS）

**不受影响：**

- Completions（完成包，由 P2P Completion Redirect 控制）
- Messages（消息 TLP）
- Configuration Requests（配置请求）

**优势：**

1. **可见性**：所有 P2P 请求通过 Root Complex，操作系统可以监控和控制
2. **IOMMU 保护**：请求可以经过 IOMMU 验证地址合法性
3. **审计和日志**：可以记录所有设备间通信

**性能影响：**

- 增加了路径长度，延迟增加
- 可能增加 Root Complex 的负载
- 对于高性能 P2P 应用（如 GPU 直连）可能不适合

### 4. P2P Completion Redirect（P2P 完成重定向）

P2P Completion Redirect 强制完成包（Completion TLP）重定向到上游，即使对应的请求是从下游来的。

**背景：**

在 PCIe 中，Memory Read Request 会产生 Completion with Data 响应。正常情况下，Switch 会将 Completion 直接路由到发起请求的设备。

**启用 Completion Redirect：**

```
设备流程：
Device A → MRd Request → Device B
Device B → Completion → (正常：直接到 Device A)
                      → (ACS：上游到 Root Complex，然后下游到 Device A)
```

**与 Request Redirect 的协同：**

通常 P2P Request Redirect 和 P2P Completion Redirect 一起启用：

```
完整流程（两者都启用）：
Device A → Request → Switch → Root Complex → Switch → Device B
Device B → Completion → Switch → Root Complex → Switch → Device A

所有 P2P 流量都经过 Root Complex
```

**优势：**

- 完整的 P2P 流量可见性
- 对称的路径（请求和响应都经过 Root Complex）
- 简化审计和监控

### 5. Upstream Forwarding（上游转发）

Upstream Forwarding 控制从一个 Downstream Port 接收的 TLP 是否可以转发到 Upstream Port。

**应用场景：**

主要用于 Multi-Root（多根）和复杂的 Switch 拓扑：

```
Root Complex 1     Root Complex 2
      ↓                   ↓
  Upstream Port      Upstream Port
      ↓                   ↓
    Multi-Root Switch
      ↓         ↓
   Port 1     Port 2
```

如果禁用 Upstream Forwarding，可以防止从 Port 1 的流量到达 Root Complex 2。

**标准拓扑：**

在大多数标准的单根拓扑中，此功能通常启用，以允许设备与 Root Complex 通信。

### 6. P2P Egress Control（P2P 出口控制）

P2P Egress Control 提供细粒度的 P2P 流量控制，允许指定哪些 Downstream Port 可以接收 P2P 流量。

**Egress Control Vector：**

这是一个位向量，每一位对应一个 Downstream Port：

```
假设 Switch 有 8 个 Downstream Port：
Egress Control Vector = 0b00001010

Bit 0 (Port 0): 0 - 禁止 P2P 流量
Bit 1 (Port 1): 1 - 允许 P2P 流量
Bit 2 (Port 2): 0 - 禁止 P2P 流量
Bit 3 (Port 3): 1 - 允许 P2P 流量
Bit 4-7: 禁止
```

**工作流程：**

```
Switch 接收 P2P TLP
  ↓
确定目标 Downstream Port
  ↓
检查 Egress Control Vector 中对应位
  ↓
位 = 0 → 重定向到上游
位 = 1 → 转发到目标端口
```

**应用场景：**

- **混合隔离策略**：允许某些设备对进行 P2P，而其他设备必须经过 Root Complex
- **性能优化**：为高性能设备对（如 GPU）启用直接 P2P，同时隔离其他设备
- **分组管理**：将设备分为不同的安全区域

**示例配置：**

```
场景：4 端口 Switch，端口分配：
- Port 0: GPU 0
- Port 1: GPU 1  
- Port 2: Network Card
- Port 3: Storage Controller

策略：允许 GPU 间 P2P，隔离网络和存储

Egress Vector for Port 0: 0b0010 (只能 P2P 到 Port 1)
Egress Vector for Port 1: 0b0001 (只能 P2P 到 Port 0)
Egress Vector for Port 2: 0b0000 (无 P2P)
Egress Vector for Port 3: 0b0000 (无 P2P)
```

### 7. Direct Translated P2P（直接转换的 P2P）

Direct Translated P2P 允许已转换的地址在特定条件下进行 P2P 通信。

**与 Translation Blocking 的关系：**

Translation Blocking 阻止所有已转换的 P2P 流量。Direct Translated P2P 提供了例外机制。

**使用条件：**

1. 系统必须支持一致的地址空间
2. 所有相关设备必须在同一 IOMMU 域
3. 地址转换在所有设备上是一致的

**应用：**

主要用于高性能计算场景，其中多个设备共享统一的虚拟地址空间，允许它们使用转换后的地址直接通信，而无需往返 Root Complex。

## ACS 在虚拟化中的应用

### IOMMU 隔离组（Isolation Group）

ACS 与 IOMMU 配合定义隔离组：

**无 ACS：**
```
同一 Switch 下的所有设备在同一隔离组
→ 无法将它们分配给不同的虚拟机
```

**有 ACS：**
```
每个 Downstream Port 是独立的隔离组
→ 可以安全地分配给不同的虚拟机
```

### VFIO 和设备分配

Linux 的 VFIO（Virtual Function I/O）框架依赖 ACS：

```bash
# 检查 ACS 支持
lspci -vv | grep -i "access control"

# 查看 IOMMU 隔离组
ls /sys/kernel/iommu_groups/

# 隔离组示例（有 ACS）
iommu_group 1: 01:00.0 Network controller
iommu_group 2: 02:00.0 Storage controller

# 隔离组示例（无 ACS）
iommu_group 1: 01:00.0 Network controller
               02:00.0 Storage controller
               (两个设备在同一组，必须同时分配)
```

### ACS Override（谨慎使用）

某些系统的硬件不支持 ACS，但软件可以强制启用（存在安全风险）：

```bash
# Linux 内核参数（不推荐用于生产环境）
pcie_acs_override=downstream,multifunction
```

**风险：**
- 破坏了硬件隔离保证
- 一个虚拟机可能访问另一个虚拟机的设备
- 仅应在完全信任所有虚拟机的环境中使用

## 软件配置示例

### Linux 内核配置

```c
// 伪代码：在 Switch Port 上启用 ACS
void pcie_enable_acs(struct pci_dev *dev) {
    int pos;
    u16 cap, ctrl;
    
    // 查找 ACS Capability
    pos = pci_find_ext_capability(dev, PCI_EXT_CAP_ID_ACS);
    if (!pos)
        return;
    
    // 读取 ACS Capability
    pci_read_config_word(dev, pos + PCI_ACS_CAP, &cap);
    
    // 构建控制字
    ctrl = 0;
    if (cap & PCI_ACS_SV)
        ctrl |= PCI_ACS_SV;  // Source Validation
    if (cap & PCI_ACS_TB)
        ctrl |= PCI_ACS_TB;  // Translation Blocking
    if (cap & PCI_ACS_RR)
        ctrl |= PCI_ACS_RR;  // P2P Request Redirect
    if (cap & PCI_ACS_CR)
        ctrl |= PCI_ACS_CR;  // P2P Completion Redirect
    if (cap & PCI_ACS_UF)
        ctrl |= PCI_ACS_UF;  // Upstream Forwarding
    
    // 写入 ACS Control
    pci_write_config_word(dev, pos + PCI_ACS_CTRL, ctrl);
    
    dev_info(&dev->dev, "ACS enabled: 0x%04x\n", ctrl);
}
```

### 检查 ACS 状态

```bash
# 使用 lspci 查看 ACS 状态
lspci -vvv -s 00:01.0 | grep -A 10 "Access Control Services"

# 输出示例
Access Control Services:
    ACSCap: SrcValid+ TransBlk+ ReqRedir+ CmpltRedir+ UpstreamFwd+ EgressCtrl- DirectTranslatedP2P-
    ACSCtl: SrcValid+ TransBlk+ ReqRedir+ CmpltRedir+ UpstreamFwd+ EgressCtrl- DirectTranslatedP2P-
```

符号说明：
- `+`：支持且已启用
- `-`：不支持或未启用

## 性能与安全的权衡

### 禁用 ACS（最高性能）

**优势：**
- P2P 流量直接路由，延迟最低
- 不增加 Root Complex 负载
- 最大化吞吐量

**劣势：**
- 无设备隔离
- 虚拟化不安全
- 难以监控和审计

**适用场景：**
- 单用户工作站
- 高性能计算（所有应用可信）
- GPU 渲染农场

### 启用完整 ACS（最高安全）

**优势：**
- 完整的设备隔离
- 支持安全的虚拟化
- 所有流量可监控

**劣势：**
- P2P 延迟增加
- Root Complex 成为瓶颈
- 吞吐量可能降低

**适用场景：**
- 多租户云环境
- 虚拟化主机
- 安全敏感应用

### 混合策略（Egress Control）

使用 Egress Control 实现平衡：

```
配置示例：
- 高安全区域（Web 服务器、数据库）：禁用 P2P
- 性能区域（GPU 集群）：启用选择性 P2P
- Root Complex 监控关键流量
```

## 故障排查

### 问题 1：虚拟机无法使用 PCIe 直通

**症状：**
- VFIO 报告设备在同一隔离组
- 无法单独分配设备

**排查步骤：**

1. 检查硬件是否支持 ACS：
```bash
lspci -vvv | grep -i "Access Control Services"
```

2. 检查 BIOS/UEFI 设置：
- 确认 IOMMU 已启用（VT-d 或 AMD-Vi）
- 检查是否有 ACS 相关选项

3. 检查内核参数：
```bash
cat /proc/cmdline | grep iommu
# 应包含 intel_iommu=on 或 amd_iommu=on
```

4. 查看隔离组：
```bash
for d in /sys/kernel/iommu_groups/*/devices/*; do
    n=${d#*/iommu_groups/*}; n=${n%%/*}
    printf 'IOMMU Group %s ' "$n"
    lspci -nns "${d##*/}"
done
```

**解决方案：**
- 升级到支持 ACS 的硬件
- 如果可接受风险，使用 ACS override（不推荐）
- 重新设计 PCIe 拓扑

### 问题 2：P2P 性能下降

**症状：**
- GPU 间传输速度显著降低
- 延迟增加

**原因分析：**
- ACS 强制 P2P 流量经过 Root Complex
- Root Complex 带宽不足

**解决方案：**

1. 评估是否真的需要 ACS：
```bash
# 禁用 ACS（仅测试，评估性能差异）
setpci -s 00:01.0 0x100.W=0x0000
```

2. 使用 Egress Control 选择性启用 P2P：
```bash
# 为特定端口对启用 P2P
# (需要硬件和驱动支持)
```

3. 升级到更高带宽的 Root Complex

4. 考虑 NVLink 等专用互连（对于 GPU）

### 问题 3：ACS 设置不生效

**症状：**
- 配置寄存器写入后，读取值不变
- 系统行为未改变

**可能原因：**

1. **硬件不支持**：检查 ACS Capability 寄存器
2. **固件锁定**：BIOS 可能锁定了 ACS 设置
3. **驱动冲突**：其他驱动可能覆盖了设置

**解决：**
```bash
# 检查是否有驱动管理 ACS
dmesg | grep -i acs

# 在内核参数中强制设置
pci=pcie_acs_override=id:8086:1234
```

## 未来发展

### PCIe 6.0 和 CXL

PCIe 6.0 和 CXL（Compute Express Link）引入了新的互连模型，ACS 的作用可能演变：

1. **更细粒度的控制**：支持更复杂的访问策略
2. **动态配置**：运行时调整 ACS 策略
3. **与 CXL.mem 集成**：控制共享内存访问

### 机密计算（Confidential Computing）

在机密计算环境中，ACS 与 TEE（可信执行环境）配合：

- 确保机密虚拟机的设备隔离
- 防止侧信道攻击
- 支持设备级加密（IDE）

## 总结

ACS 是 PCIe 生态系统中的关键安全特性，为现代虚拟化和云计算提供了必要的硬件隔离保证。理解 ACS 的各项机制——Source Validation、Translation Blocking、P2P Redirect 等——对于设计安全的系统架构至关重要。

在实际部署中，需要在性能和安全之间找到平衡点。完全启用 ACS 提供最强的隔离但可能牺牲 P2P 性能；选择性使用 Egress Control 可以实现灵活的策略。硬件选型时，确认关键组件（尤其是 Root Port 和 Switch）支持 ACS 是构建可靠虚拟化平台的前提。

随着 PCIe 和 CXL 技术的演进，ACS 将继续发展以应对新的安全挑战，成为异构计算和机密计算的重要基础设施。
