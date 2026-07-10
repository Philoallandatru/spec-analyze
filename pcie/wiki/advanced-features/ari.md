# PCIe 备选路由 ID（ARI - Alternative Routing-ID Interpretation）

## 概述

备选路由 ID 解释（ARI）是 PCIe 2.0 规范引入的一项扩展特性，旨在突破传统 PCIe 设备功能数量的限制。在传统 PCIe 中，每个设备最多支持 8 个功能（Function 0-7），而 ARI 将这一限制扩展到 256 个功能，极大地增强了设备的功能密度。

ARI 对于支持大量虚拟功能的 SR-IOV（Single Root I/O Virtualization）设备尤为重要，使单个物理设备能够虚拟化为数百个独立的功能，每个功能可以分配给不同的虚拟机或容器。

## 传统 PCIe 设备寻址限制

### BDF 寻址模型

传统 PCIe 使用 BDF（Bus:Device:Function）模型来唯一标识每个功能：

```
Requester ID / BDF：16 位
┌─────────────┬──────────────┬────────────────┐
│  Bus Number │ Device Number│ Function Number│
│   (8 bits)  │   (5 bits)   │    (3 bits)    │
└─────────────┴──────────────┴────────────────┘
     0-255         0-31           0-7
```

**限制：**
- 每条总线最多 32 个设备（Device 0-31）
- 每个设备最多 8 个功能（Function 0-7）
- 每条总线最多 256 个功能（32 × 8）

### 多功能设备的限制

对于单个物理设备（占用一个 Device Number），最多只能实现 8 个功能：

```
传统多功能设备：
Bus 1, Device 0
  ├─ Function 0: 网络控制器
  ├─ Function 1: 存储控制器
  ├─ Function 2: 管理接口
  ├─ Function 3: 调试接口
  ├─ Function 4-7: (可用)
  └─ 最多 8 个功能
```

**对 SR-IOV 的影响：**

SR-IOV 允许一个物理功能（PF）创建多个虚拟功能（VF）。在没有 ARI 的情况下：

```
SR-IOV 设备（无 ARI）：
01:00.0 - Physical Function (PF)
01:00.1 - Virtual Function (VF) 0
01:00.2 - Virtual Function (VF) 1
01:00.3 - Virtual Function (VF) 2
01:00.4 - Virtual Function (VF) 3
01:00.5 - Virtual Function (VF) 4
01:00.6 - Virtual Function (VF) 5
01:00.7 - Virtual Function (VF) 6
最多 7 个 VF（因为 Function 0 是 PF）
```

这严重限制了 SR-IOV 的可扩展性，特别是在需要大量虚拟机或容器的云环境中。

## ARI 的解决方案

### 重新解释 Device 和 Function 字段

ARI 将原本的 Device Number（5 位）和 Function Number（3 位）合并，形成一个 8 位的 Function Number：

```
ARI 寻址模型：
Requester ID / BDF：16 位
┌─────────────┬──────────────────────────────┐
│  Bus Number │    ARI Function Number       │
│   (8 bits)  │         (8 bits)             │
└─────────────┴──────────────────────────────┘
     0-255              0-255
```

**关键变化：**
- Device Number 不再使用（实际上与 Function Number 合并）
- Function Number 扩展为 8 位
- 每个设备最多 256 个功能（Function 0-255）

### ARI 设备表示

```
ARI 启用的 SR-IOV 设备：
Bus 1, Device 0（Device Number 实际被忽略）
  ├─ Function 0:   Physical Function (PF)
  ├─ Function 1:   Virtual Function (VF) 0
  ├─ Function 2:   Virtual Function (VF) 1
  ├─ Function 3:   Virtual Function (VF) 2
  │     ...
  ├─ Function 100: Virtual Function (VF) 99
  │     ...
  └─ Function 255: Virtual Function (VF) 254
  最多 255 个 VF
```

**BDF 表示方式：**

虽然 ARI 内部将 Device 和 Function 合并为 8 位，但在工具输出中仍然使用传统的 BDF 格式：

```bash
# lspci 输出示例（ARI 设备）
01:00.0 Ethernet controller [Physical Function]
01:00.1 Ethernet controller [Virtual Function]
01:00.2 Ethernet controller [Virtual Function]
...
01:1f.7 Ethernet controller [Virtual Function] # Function 255 (0x1F × 8 + 7 = 255)
```

注意：当 Function Number 超过 7 时，会"溢出"到 Device Number 字段，但这只是表示方式，ARI 硬件内部将其视为单一的 8 位 Function Number。

## ARI Capability 结构

ARI 通过两个 PCIe 扩展 Capability 结构实现：

### 1. ARI Extended Capability（设备端）

位于支持 ARI 的端点设备的 PCIe Extended Configuration Space 中。

**Capability ID**：0x000E（ARI）

**结构：**

```
偏移 0x00: PCI Express Extended Capability Header
  Bits 0-15 : Capability ID (0x000E)
  Bits 16-19: Capability Version
  Bits 20-31: Next Capability Offset

偏移 0x04: ARI Capability Register
  Bit 0     : MFVC Function Groups Capability (M)
  Bit 1     : ACS Function Groups Capability (A)
  Bits 2-7  : Reserved
  Bits 8-15 : Next Function Number

偏移 0x06: ARI Control Register
  Bit 0     : MFVC Function Groups Enable
  Bit 1     : ACS Function Groups Enable
  Bits 2-3  : Reserved
  Bit 4     : Function Group (read-only)
  Bits 5-15 : Reserved
```

**关键字段：**

**Next Function Number（8 位）**
- 指示下一个功能的编号
- 用于软件枚举所有功能
- 值为 0 表示这是最后一个功能

**示例枚举过程：**

```
设备有 4 个功能：Function 0, 1, 2, 3

Function 0 的 Next Function Number = 1
Function 1 的 Next Function Number = 2
Function 2 的 Next Function Number = 3
Function 3 的 Next Function Number = 0 (最后一个)

软件枚举：
1. 读取 Function 0 的 ARI Capability
2. Next Function Number = 1，继续
3. 读取 Function 1 的 ARI Capability
4. Next Function Number = 2，继续
5. 读取 Function 2 的 ARI Capability
6. Next Function Number = 3，继续
7. 读取 Function 3 的 ARI Capability
8. Next Function Number = 0，结束
```

### 2. ARI Forwarding（Root Port / Switch 端）

ARI 需要上游组件（Root Port 或 Switch Downstream Port）的支持。

**位置**：PCIe Capability Structure 的 Device Capabilities 2 和 Device Control 2 寄存器

**Device Capabilities 2 Register（偏移 0x24）**

```
Bit 5: ARI Forwarding Supported
```

该位指示此 Root Port 或 Downstream Port 是否支持 ARI 转发。

**Device Control 2 Register（偏移 0x28）**

```
Bit 5: ARI Forwarding Enable
```

软件设置此位以启用 ARI 转发。

## ARI Forwarding Enable 机制

### 为什么需要 Forwarding？

ARI 改变了 Requester ID 的解释方式，但 TLP（Transaction Layer Packet）的格式没有改变。上游组件需要知道如何正确解释来自下游的 Requester ID。

**传统模式（ARI Forwarding Disabled）：**

```
Root Port 收到 TLP
  ↓
提取 Requester ID: Bus=1, Dev=5, Func=3
  ↓
解释为：总线 1 上的设备 5 的功能 3
```

**ARI 模式（ARI Forwarding Enabled）：**

```
Root Port 收到 TLP
  ↓
提取 Requester ID: Bus=1, Dev=5, Func=3
  ↓
重新解释：总线 1 上的功能 (5 << 3 | 3) = 功能 43
```

### 启用 ARI 的步骤

**步骤 1：检查设备是否支持 ARI**

```c
// 伪代码
int pcie_ari_supported(struct pci_dev *dev) {
    int pos;
    
    // 查找 ARI Extended Capability
    pos = pci_find_ext_capability(dev, PCI_EXT_CAP_ID_ARI);
    if (!pos)
        return 0; // 不支持
    
    return 1;
}
```

**步骤 2：检查上游端口是否支持 ARI Forwarding**

```c
// 伪代码
int pcie_ari_forwarding_supported(struct pci_dev *bridge) {
    u32 cap2;
    
    // 读取 Device Capabilities 2
    pci_read_config_dword(bridge, 
                          bridge->pcie_cap + PCI_EXP_DEVCAP2, 
                          &cap2);
    
    return !!(cap2 & PCI_EXP_DEVCAP2_ARI);
}
```

**步骤 3：启用 ARI Forwarding**

```c
// 伪代码
void pcie_ari_forwarding_enable(struct pci_dev *bridge) {
    u16 ctrl2;
    
    // 读取 Device Control 2
    pci_read_config_word(bridge, 
                         bridge->pcie_cap + PCI_EXP_DEVCTL2, 
                         &ctrl2);
    
    // 设置 ARI Forwarding Enable 位
    ctrl2 |= PCI_EXP_DEVCTL2_ARI;
    
    // 写回
    pci_write_config_word(bridge, 
                          bridge->pcie_cap + PCI_EXP_DEVCTL2, 
                          ctrl2);
    
    dev_info(&bridge->dev, "ARI Forwarding enabled\n");
}
```

**步骤 4：重新枚举设备**

启用 ARI Forwarding 后，需要重新扫描总线以发现所有功能：

```c
// 伪代码
void pcie_rescan_bus_with_ari(struct pci_bus *bus) {
    int func;
    
    // ARI 模式：扫描功能 0-255
    for (func = 0; func < 256; func++) {
        struct pci_dev *dev;
        
        // 尝试读取功能
        dev = pci_scan_single_device(bus, PCI_DEVFN(0, func));
        if (!dev) {
            // 如果设备有 ARI，使用 Next Function Number
            // 否则，在第一个不存在的功能处停止
            if (last_dev && pcie_ari_enabled(last_dev)) {
                int next = pcie_ari_get_next_func(last_dev);
                if (next == 0)
                    break;
                func = next - 1; // -1 因为循环会 ++
                continue;
            } else {
                break;
            }
        }
        
        last_dev = dev;
        pci_device_add(dev, bus);
    }
}
```

## ARI 与 SR-IOV 的集成

### SR-IOV Capability 中的 ARI 支持

SR-IOV Capability 结构包含与 ARI 相关的字段：

**SR-IOV Capability Register（偏移 0x04）**

```
Bit 0: ARI Capable Hierarchy
```

该位由硬件设置，指示系统层次结构支持 ARI。

**SR-IOV Control Register（偏移 0x08）**

```
Bit 4: ARI Capable Hierarchy (Read-Write)
```

软件设置此位以在 SR-IOV 上下文中启用 ARI。

### VF 的编号方式

**无 ARI：**

```
First VF Offset = 1
VF Stride = 1
NumVFs = 7

VF 编号：
Function 1, 2, 3, 4, 5, 6, 7
```

**有 ARI：**

```
First VF Offset = 1
VF Stride = 1
NumVFs = 200

VF 编号：
Function 1, 2, 3, ..., 200
```

**VF 的 BDF 计算：**

```
VF[i] 的 BDF = PF_BDF + First VF Offset + (VF Stride × i)

示例（ARI 启用）：
PF = 01:00.0
First VF Offset = 1
VF Stride = 1

VF[0] = 01:00.0 + 1 + (1 × 0) = 01:00.1
VF[1] = 01:00.0 + 1 + (1 × 1) = 01:00.2
...
VF[99] = 01:00.0 + 1 + (1 × 99) = 01:00.100 (显示为 01:0c.4)
```

### 完整的 SR-IOV + ARI 初始化流程

```
1. 系统启动，PCI 枚举开始
   ↓
2. 发现 Root Port，检查 ARI Forwarding 支持
   ↓
3. 扫描总线，发现 SR-IOV PF
   ↓
4. 检查 PF 的 ARI Extended Capability
   ↓
5. 如果都支持，启用 Root Port 的 ARI Forwarding
   ↓
6. 读取 SR-IOV Capability：
   - Total VFs：设备支持的最大 VF 数量
   - 根据 ARI 支持情况决定实际可用的 VF 数量
   ↓
7. 操作系统或用户设置 NumVFs
   ↓
8. 启用 SR-IOV（设置 SR-IOV Control 的 VF Enable 位）
   ↓
9. 硬件创建 VF，每个 VF 显示为独立的 PCIe 功能
   ↓
10. 重新扫描总线，枚举所有 VF
   ↓
11. 为每个 VF 分配资源（BAR、中断等）
   ↓
12. VF 可以独立分配给虚拟机
```

## 软件支持和配置

### Linux 内核

**检查 ARI 支持：**

```bash
# 查看设备是否支持 ARI
lspci -vvv -s 01:00.0 | grep -i "alternative routing"

# 输出示例
Capabilities: [160] Alternative Routing-ID Interpretation (ARI)
    ARICap: MFVC- ACS-, Next Function: 1
    ARICtl: MFVC- ACS-, Function Group: 0

# 查看 Root Port 的 ARI Forwarding 状态
lspci -vvv -s 00:01.0 | grep -i "ari"

# 输出示例
DevCap2: ... ARIFwd+
DevCtl2: ... ARIFwd+
```

**启用 SR-IOV with ARI：**

```bash
# 查看设备支持的 VF 数量
cat /sys/bus/pci/devices/0000:01:00.0/sriov_totalvfs
# 输出：200（有 ARI）或 7（无 ARI）

# 启用 VF
echo 100 > /sys/bus/pci/devices/0000:01:00.0/sriov_numvfs

# 验证 VF 创建
lspci | grep "Virtual Function"
# 应显示 100 个 VF
```

**内核配置检查：**

```bash
# 确保内核支持 SR-IOV 和 ARI
zcat /proc/config.gz | grep -E "IOMMU|SRIOV|ARI"

# 应包含：
CONFIG_PCI_IOV=y
CONFIG_INTEL_IOMMU=y 或 CONFIG_AMD_IOMMU=y
```

### BIOS/UEFI 设置

许多系统需要在 BIOS/UEFI 中启用相关选项：

```
Advanced → PCI Configuration
  ├─ SR-IOV Support: [Enabled]
  ├─ ARI Support: [Enabled]
  └─ IOMMU/VT-d/AMD-Vi: [Enabled]
```

### 虚拟化平台配置

**QEMU/KVM：**

```bash
# 将 VF 分配给虚拟机
qemu-system-x86_64 \
  -device vfio-pci,host=01:00.1 \  # VF 0
  -device vfio-pci,host=01:00.2 \  # VF 1
  ...
```

**libvirt XML：**

```xml
<hostdev mode='subsystem' type='pci' managed='yes'>
  <source>
    <address domain='0x0000' bus='0x01' slot='0x00' function='0x1'/>
  </source>
</hostdev>
```

**Docker/Kubernetes：**

使用 SR-IOV Device Plugin：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sriov-pod
spec:
  containers:
  - name: app
    image: myapp
    resources:
      limits:
        intel.com/sriov_net: 1  # 请求一个 SR-IOV VF
```

## ARI 的限制和注意事项

### 1. 硬件依赖

**完整的 ARI 支持链：**

```
CPU/Root Complex
  ↓ (必须支持 ARI Forwarding)
Root Port
  ↓ (必须支持 ARI Forwarding)
Switch (如果存在)
  ↓ (必须支持 ARI Forwarding)
Downstream Port
  ↓
End Device (必须支持 ARI)
```

任何一个环节不支持 ARI，整个链路就无法使用 ARI。

### 2. 拓扑限制

ARI 要求设备位于 Device 0 位置：

```
正确的 ARI 设备拓扑：
Bus 1
  └─ Device 0
      ├─ Function 0
      ├─ Function 1
      │     ...
      └─ Function 255

错误的拓扑（无法使用 ARI）：
Bus 1
  ├─ Device 0
  │   └─ Function 0
  └─ Device 1  ← ARI 设备不能在 Device 1
      └─ Function 0
```

### 3. 软件兼容性

某些旧版操作系统或驱动程序可能不支持 ARI：

- Windows Server 2008 及更早版本
- 旧版 Linux 内核（< 2.6.35）
- 某些嵌入式 RTOS

### 4. BIOS/UEFI 支持

BIOS/UEFI 必须在启动时正确配置 ARI：

- 检测 ARI capable 设备
- 启用 Root Port 的 ARI Forwarding
- 正确分配资源（考虑最多 256 个功能）

如果 BIOS 不支持 ARI，操作系统可能无法看到所有功能。

## 性能和可扩展性

### VF 数量对比

| 场景 | 无 ARI | 有 ARI | 提升 |
|------|--------|--------|------|
| 单 PF SR-IOV | 最多 7 VF | 最多 255 VF | 36× |
| 高端网卡 | 受限 | 可支持数百虚拟机 | 显著 |
| GPU 虚拟化 | 有限分割 | 细粒度分割 | 灵活性大增 |

### 实际应用案例

**1. 数据中心网络：**

```
Intel XL710 40GbE 网卡（支持 ARI）
  ├─ Physical Function
  └─ 128 个 Virtual Functions
      → 每个 VF 分配给一个容器/虚拟机
      → 每个获得专用的网络队列和带宽
```

**2. 存储加速：**

```
NVMe SSD 控制器（支持 ARI）
  ├─ Physical Function
  └─ 64 个 Virtual Functions
      → 多租户存储隔离
      → 每个租户独立的 I/O 路径
```

**3. GPU 虚拟化：**

```
NVIDIA A100 GPU（支持 ARI 和 MIG）
  ├─ Physical Function
  └─ 多个 Virtual Functions
      → 每个 VF 对应一个 GPU 切片
      → 实现 GPU 的多租户共享
```

## 故障排查

### 问题 1：VF 数量受限于 7 个

**症状：**
```bash
cat /sys/bus/pci/devices/0000:01:00.0/sriov_totalvfs
7  # 期望 255
```

**排查步骤：**

1. 检查设备是否支持 ARI：
```bash
lspci -vvv -s 01:00.0 | grep ARI
# 应该有 "Alternative Routing-ID Interpretation (ARI)" 条目
```

2. 检查 Root Port 是否支持 ARI Forwarding：
```bash
lspci -vvv -s 00:01.0 | grep ARIFwd
# DevCap2 和 DevCtl2 应该都是 ARIFwd+
```

3. 检查 BIOS 设置：
- 进入 BIOS/UEFI
- 确认 SR-IOV 和 ARI 选项都已启用

4. 检查内核日志：
```bash
dmesg | grep -i ari
# 查找 ARI 相关的错误或警告
```

**解决方案：**
- 更新 BIOS/UEFI 到最新版本
- 确保主板和 CPU 支持 ARI
- 更新设备固件

### 问题 2：启用 VF 后系统不稳定

**可能原因：**

1. **资源不足**：256 个 VF 需要大量的 MMIO 空间和中断资源
2. **IOMMU 配置问题**：ARI 与 IOMMU 的交互问题
3. **驱动 bug**：驱动程序可能在高 VF 数量下有问题

**解决方案：**

```bash
# 逐步增加 VF 数量，找到稳定点
echo 10 > /sys/bus/pci/devices/0000:01:00.0/sriov_numvfs
# 测试稳定性
echo 20 > /sys/bus/pci/devices/0000:01:00.0/sriov_numvfs
# 继续测试...

# 检查资源分配
lspci -vvv -s 01:00.0 | grep -A 10 "Memory at"

# 检查 IOMMU 状态
dmesg | grep -i dmar
dmesg | grep -i iommu
```

### 问题 3：某些 VF 无法分配给虚拟机

**症状：**
```bash
# VFIO 报错
vfio-pci 0000:01:00.50: Failed to setup group
```

**排查：**

```bash
# 检查 IOMMU 分组
ls /sys/kernel/iommu_groups/*/devices/ | grep 01:00

# 检查 VF 是否被其他驱动绑定
lspci -k -s 01:00.50

# 检查 ACS 设置（影响 IOMMU 分组）
lspci -vvv | grep -i "access control"
```

## 最佳实践

### 1. 硬件选型

在需要大量 VF 的场景中，确保：
- CPU/芯片组支持 ARI（Intel Xeon、AMD EPYC 最新代）
- 主板 BIOS 支持 ARI 配置
- 设备（网卡、存储控制器等）明确支持 ARI
- 充足的 PCIe 带宽和 MMIO 地址空间

### 2. 系统配置

```bash
# 在 GRUB 中启用必要的内核参数
GRUB_CMDLINE_LINUX="intel_iommu=on iommu=pt"

# 或者对于 AMD
GRUB_CMDLINE_LINUX="amd_iommu=on iommu=pt"

# 更新 GRUB
update-grub
```

### 3. 监控和管理

```bash
# 创建监控脚本
#!/bin/bash
# ari-monitor.sh

PF_BDF="0000:01:00.0"

echo "=== ARI Status ==="
lspci -vvv -s $PF_BDF | grep -A 2 "Alternative Routing"

echo "=== VF Status ==="
cat /sys/bus/pci/devices/$PF_BDF/sriov_totalvfs
cat /sys/bus/pci/devices/$PF_BDF/sriov_numvfs

echo "=== Active VFs ==="
lspci | grep -c "Virtual Function"
```

### 4. 性能优化

- 为高性能场景预留专用的 PCIe 插槽
- 使用 NUMA 亲和性绑定 VF 到正确的 CPU
- 合理分配 VF 数量，避免资源过度分割
- 监控 PCIe 带宽利用率

## 总结

ARI 是 PCIe 生态系统中的关键扩展特性，通过将功能数量从 8 个扩展到 256 个，为 SR-IOV 和现代虚拟化应用提供了必要的可扩展性。理解 ARI 的寻址机制、ARI Forwarding Enable 的作用以及与 SR-IOV 的集成，对于设计和部署高密度虚拟化环境至关重要。

在实际应用中，ARI 使得单个物理设备能够服务数百个虚拟机或容器，显著提高了硬件利用率和投资回报。随着云计算和边缘计算的发展，ARI 将继续在数据中心和高性能计算领域发挥重要作用，成为实现高效资源共享和隔离的基础技术。
