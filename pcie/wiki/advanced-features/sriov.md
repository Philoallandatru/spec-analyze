# SR-IOV 虚拟化详解

Single Root I/O Virtualization (SR-IOV) 是 PCIe 的虚拟化扩展，允许单个物理设备虚拟为多个独立的虚拟设备，直接分配给虚拟机使用。

---

## 概述

SR-IOV 是 **PCIe 规范的关键虚拟化特性**，解决了传统 I/O 虚拟化的性能瓶颈。

### 为什么需要 SR-IOV？

**传统虚拟化问题**：

```
传统 I/O 虚拟化（软件模拟）：

VM1        VM2        VM3
 │          │          │
 └──────────┴──────────┘
            │
      Hypervisor (软件模拟)
            │
       物理设备

问题：
✗ 所有 I/O 必须经过 Hypervisor
✗ 上下文切换开销大
✗ CPU 占用高
✗ 延迟高、吞吐量低
```

**SR-IOV 解决方案**：

```
SR-IOV (硬件虚拟化)：

VM1        VM2        VM3
 │          │          │
VF1        VF2        VF3 (Virtual Functions)
 └──────────┴──────────┘
            │
           PF (Physical Function)
            │
       物理设备

优势：
✓ VM 直接访问硬件 (Direct Assignment)
✓ 接近原生性能 (Near-native Performance)
✓ 低 CPU 开销
✓ 低延迟、高吞吐量
```

### SR-IOV 核心概念

**Physical Function (PF)**：
- 完整的 PCIe 功能
- 包含 SR-IOV 扩展能力
- 管理和配置 VF
- 通常由 Hypervisor 控制

**Virtual Function (VF)**：
- 轻量级的 PCIe 功能
- 专注于数据路径
- 共享 PF 的资源
- 直接分配给 VM

```
PF vs VF 对比：

特性           PF              VF
────────────────────────────────────
完整配置空间    ✓               ✗ (简化)
SR-IOV 能力     ✓               ✗
MSI-X 中断      ✓               ✓
DMA 能力        ✓               ✓
独立 BARs       ✓               ✓
数量            1               多个 (最多 256)
用途            管理            数据传输
```

---

## SR-IOV 架构

### 拓扑结构

```
系统架构：

┌─────────────────────────────────────────────────┐
│  Host System                                    │
│                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │   VM1    │  │   VM2    │  │   VM3    │     │
│  │  Driver  │  │  Driver  │  │  Driver  │     │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘     │
│       │             │             │            │
│  ┌────▼─────┐  ┌───▼──────┐  ┌───▼──────┐     │
│  │   VF1    │  │   VF2    │  │   VF3    │     │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘     │
│       │             │             │            │
│  ┌────┴──────────────┴─────────────┴─────┐     │
│  │         Physical Function (PF)        │     │
│  │  - SR-IOV 配置                        │     │
│  │  - VF 管理                            │     │
│  │  - 资源分配                           │     │
│  └───────────────────┬───────────────────┘     │
└────────────────────────┼─────────────────────────┘
                         │
                    PCIe Link
                         │
              ┌──────────▼──────────┐
              │   Physical Device   │
              │  (NIC, GPU, NVMe)   │
              └─────────────────────┘
```

### BDF 地址分配

SR-IOV 设备使用特殊的 BDF (Bus:Device.Function) 地址方案：

```
PF 地址：Bus 1, Device 0, Function 0 → 01:00.0

VF 地址（连续分配）：
VF0: Bus 1, Device 0, Function 1 → 01:00.1
VF1: Bus 1, Device 0, Function 2 → 01:00.2
VF2: Bus 1, Device 0, Function 3 → 01:00.3
...
VF7: Bus 1, Device 1, Function 0 → 01:01.0 (跨设备号)

规则：
- VF 的 Bus 号 = PF Bus + Offset
- VF 的 Device.Function 连续递增
- 每个设备最多 8 个 Function，然后进位到下一个 Device
```

---

## SR-IOV 配置空间

### PF 配置空间结构

```
PF 配置空间布局：

0x00-0x3F: 标准 PCI Header
0x40+:     Capabilities
           ├── PCIe Capability
           ├── MSI-X Capability
           └── ...
0x100+:    Extended Capabilities
           ├── SR-IOV Extended Capability (关键!)
           ├── ARI (Alternative Routing-ID)
           └── ...

SR-IOV Extended Capability 位于扩展配置空间 (0x100+)
```

### SR-IOV Extended Capability 结构

```
SR-IOV Capability (偏移通常在 0x160 左右):

Offset  Size  Field                  Description
──────────────────────────────────────────────────────
0x00    2     Capability ID          0x0010 (SR-IOV)
0x02    2     Next Capability        下一个能力的偏移
0x04    4     SR-IOV Capabilities    能力位
0x08    2     SR-IOV Control         控制寄存器
0x0A    2     SR-IOV Status          状态寄存器
0x0C    2     Initial VFs            支持的 VF 总数
0x0E    2     Total VFs              当前 VF 总数
0x10    2     Num VFs                已启用的 VF 数量
0x12    1     Function Dependency    功能依赖链接
0x13    1     Reserved
0x14    2     First VF Offset        第一个 VF 的偏移
0x16    2     VF Stride              VF 之间的步长
0x18    2     Reserved
0x1A    2     VF Device ID           VF 的设备 ID
0x1C    4     Supported Page Sizes   支持的页大小
0x20    4     System Page Size       系统页大小
0x24    24    VF BAR0-5              VF 的 BAR 配置 (6个)
0x3C    4     VF Migration State     迁移状态数组偏移
```

### 关键寄存器详解

**SR-IOV Control Register (偏移 0x08)**：

```
Bits    Field                       Description
────────────────────────────────────────────────────────────
0       VF Enable                   使能 VF (1=启用)
1       VF Migration Enable         使能 VF 迁移
2       VF Migration Interrupt En   VF 迁移中断使能
3       VF MSE (Memory Space En)    VF 内存空间使能
4       ARI Capable Hierarchy       ARI 层次结构
5-15    Reserved
```

**Initial VFs / Total VFs / Num VFs**：

```
Initial VFs (只读):
  - 设备支持的最大 VF 数量
  - 硬件固定值 (如 256)

Total VFs (只读):
  - 当前可用的 VF 数量
  - 可能因系统资源限制而小于 Initial VFs

Num VFs (读写):
  - 当前启用的 VF 数量
  - 软件写入此字段来创建 VF (0-Total VFs)
  - 示例：写入 8 创建 8 个 VF
```

**First VF Offset / VF Stride**：

```
First VF Offset:
  - 第一个 VF 相对于 PF 的 Routing ID 偏移
  - 示例：PF = 01:00.0, Offset = 1 → VF0 = 01:00.1

VF Stride:
  - 相邻 VF 之间的 Routing ID 步长
  - 通常为 1 (连续分配)
  - 某些情况下可能为 2 或更大 (支持 Function Level Reset)

计算 VF 的 BDF：
VF[i] BDF = PF BDF + First VF Offset + (i * VF Stride)
```

### VF BARs 配置

VF BARs 与普通 BARs 类似，但配置在 PF 的 SR-IOV 能力中：

```
VF BAR 配置 (每个 BAR 4 字节):

VF BAR0 (偏移 0x24): VF 的第一个 BAR
VF BAR1 (偏移 0x28): VF 的第二个 BAR
...
VF BAR5 (偏移 0x38): VF 的第六个 BAR

示例：VF BAR0 配置为 64-bit Memory，4KB 大小
偏移 0x24: 0xFFFFF00C (低 32 位，64-bit + Prefetchable)
偏移 0x28: 0xFFFFFFFF (高 32 位，全 1 表示 4GB 以上地址)

每个 VF 都会获得独立的 BAR 地址：
VF0 BAR0: 0xF0000000 (4KB)
VF1 BAR0: 0xF0001000 (4KB)
VF2 BAR0: 0xF0002000 (4KB)
...
```

---

## SR-IOV 启用流程

### 1. 检测 SR-IOV 支持

```bash
# 使用 lspci 检测
lspci -vvv -s 01:00.0 | grep -i "SR-IOV"

输出示例：
Capabilities: [160] Single Root I/O Virtualization (SR-IOV)
    IOVCap: Migration-, Interrupt Message Number: 000
    IOVCtl: Enable+ Migration- Interrupt- MSE+ ARIHierarchy+
    IOVSta: Migration-
    Initial VFs: 256, Total VFs: 256, Number of VFs: 8, ...
    VF offset: 1, stride: 1, Device ID: 154c
```

### 2. 驱动初始化 PF

```c
// 驱动代码示例 (简化)
int sriov_init(struct pci_dev *pdev)
{
    int pos, num_vfs;
    u16 total_vfs;
    
    // 1. 查找 SR-IOV Capability
    pos = pci_find_ext_capability(pdev, PCI_EXT_CAP_ID_SRIOV);
    if (!pos) {
        pr_err("SR-IOV not supported\n");
        return -ENODEV;
    }
    
    // 2. 读取 Total VFs
    pci_read_config_word(pdev, pos + PCI_SRIOV_TOTAL_VF, &total_vfs);
    pr_info("Device supports %d VFs\n", total_vfs);
    
    // 3. 配置 VF BARs (由 PCI 子系统处理)
    
    // 4. 设置系统页大小
    pci_write_config_dword(pdev, pos + PCI_SRIOV_SYS_PGSIZE, 0x00001000); // 4KB
    
    return 0;
}
```

### 3. 启用 VF

```c
// 启用 VF
int enable_vfs(struct pci_dev *pdev, int num_vfs)
{
    int pos = pci_find_ext_capability(pdev, PCI_EXT_CAP_ID_SRIOV);
    u16 ctrl;
    
    // 1. 写入 Num VFs
    pci_write_config_word(pdev, pos + PCI_SRIOV_NUM_VF, num_vfs);
    
    // 2. 设置 VF Enable 位
    pci_read_config_word(pdev, pos + PCI_SRIOV_CTRL, &ctrl);
    ctrl |= PCI_SRIOV_CTRL_VFE | PCI_SRIOV_CTRL_MSE;
    pci_write_config_word(pdev, pos + PCI_SRIOV_CTRL, ctrl);
    
    // 3. 等待 VF 就绪
    msleep(100);
    
    pr_info("Enabled %d VFs\n", num_vfs);
    return 0;
}
```

### 4. Linux sysfs 接口

```bash
# 通过 sysfs 启用 VF (推荐方法)

# 查看支持的 VF 数量
cat /sys/bus/pci/devices/0000:01:00.0/sriov_totalvfs
256

# 启用 8 个 VF
echo 8 > /sys/bus/pci/devices/0000:01:00.0/sriov_numvfs

# 验证 VF 已创建
lspci | grep "Virtual Function"
01:00.1 Ethernet controller: Intel Virtual Function
01:00.2 Ethernet controller: Intel Virtual Function
...

# 禁用 VF
echo 0 > /sys/bus/pci/devices/0000:01:00.0/sriov_numvfs
```

---

## VF 分配给虚拟机

### 1. VFIO (推荐方法)

VFIO (Virtual Function I/O) 是现代 Linux 的设备直通框架。

```bash
# 1. 绑定 VF 到 vfio-pci 驱动
echo 0000:01:00.1 > /sys/bus/pci/devices/0000:01:00.1/driver/unbind
echo "8086 154c" > /sys/bus/pci/drivers/vfio-pci/new_id

# 2. 验证绑定
lspci -k -s 01:00.1
01:00.1 Ethernet controller: Intel Virtual Function
    Kernel driver in use: vfio-pci

# 3. 配置 QEMU/KVM 虚拟机
qemu-system-x86_64 \
  -device vfio-pci,host=01:00.1 \
  ...
```

### 2. libvirt 配置

```xml
<!-- VM 配置文件 -->
<domain type='kvm'>
  <name>vm-with-sriov</name>
  ...
  <devices>
    <!-- SR-IOV VF 直通 -->
    <hostdev mode='subsystem' type='pci' managed='yes'>
      <source>
        <address domain='0x0000' bus='0x01' slot='0x00' function='0x1'/>
      </source>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/>
    </hostdev>
  </devices>
</domain>
```

### 3. VM 内部视图

```bash
# 在 VM 内部
lspci
00:05.0 Ethernet controller: Intel Virtual Function

# VF 对 VM 来说就像真实设备
ip link show
eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...

# 可以直接使用标准驱动
modprobe ixgbevf  # Intel VF 驱动
```

---

## FEMU 实现

### SR-IOV 核心代码

```c
// hw/pci/pcie_sriov.c - SR-IOV 实现

// SR-IOV 设备结构
typedef struct PCIESriovPF {
    PCIDevice *pf;                  // Physical Function
    uint16_t num_vfs;               // 当前 VF 数量
    uint16_t total_vfs;             // 最大 VF 数量
    uint16_t vf_offset;             // VF 偏移
    uint16_t vf_stride;             // VF 步长
    PCIDevice **vfs;                // VF 设备数组
    MemoryRegion *vf_bars[PCI_NUM_REGIONS]; // VF BARs
} PCIESriovPF;

// 初始化 SR-IOV
void pcie_sriov_pf_init(PCIDevice *dev, uint16_t offset,
                        const char *vf_name, uint16_t vf_dev_id,
                        uint16_t init_vfs, uint16_t total_vfs,
                        uint16_t vf_offset, uint16_t vf_stride)
{
    uint8_t *cfg = dev->config + offset;
    
    // 写入 SR-IOV Capability Header
    pci_set_word(cfg + PCI_SRIOV_CAP, PCI_EXT_CAP_ID_SRIOV);
    
    // 设置 VF 数量
    pci_set_word(cfg + PCI_SRIOV_INITIAL_VF, init_vfs);
    pci_set_word(cfg + PCI_SRIOV_TOTAL_VF, total_vfs);
    pci_set_word(cfg + PCI_SRIOV_NUM_VF, 0); // 初始为 0
    
    // 设置 VF 偏移和步长
    pci_set_word(cfg + PCI_SRIOV_VF_OFFSET, vf_offset);
    pci_set_word(cfg + PCI_SRIOV_VF_STRIDE, vf_stride);
    
    // 设置 VF Device ID
    pci_set_word(cfg + PCI_SRIOV_VF_DID, vf_dev_id);
    
    // 分配 SR-IOV 结构
    PCIESriovPF *sriov = g_malloc0(sizeof(*sriov));
    sriov->pf = dev;
    sriov->total_vfs = total_vfs;
    sriov->vf_offset = vf_offset;
    sriov->vf_stride = vf_stride;
    dev->exp.sriov_pf = sriov;
}

// 注册 VF
void pcie_sriov_vf_register(PCIDevice *vf, PCIDevice *pf,
                            uint16_t vf_num)
{
    PCIESriovPF *sriov = pf->exp.sriov_pf;
    
    // 存储 VF 引用
    if (!sriov->vfs) {
        sriov->vfs = g_malloc0(sizeof(PCIDevice *) * sriov->total_vfs);
    }
    sriov->vfs[vf_num] = vf;
    
    // 设置 VF 的配置空间
    vf->exp.sriov_vf_pf = pf;
    vf->exp.sriov_vf_num = vf_num;
}
```

### 配置空间写入处理

```c
// 处理 Num VFs 寄存器写入
static void sriov_num_vfs_write(PCIDevice *dev, uint32_t addr,
                                uint32_t val, int len)
{
    PCIESriovPF *sriov = dev->exp.sriov_pf;
    uint16_t num_vfs = val & 0xFFFF;
    
    if (num_vfs > sriov->total_vfs) {
        return; // 超出范围
    }
    
    // 如果 VF Enable 位已设置
    uint16_t ctrl = pci_get_word(dev->config + sriov->offset + PCI_SRIOV_CTRL);
    if (ctrl & PCI_SRIOV_CTRL_VFE) {
        // 创建或销毁 VF
        if (num_vfs > sriov->num_vfs) {
            // 创建新 VF
            for (int i = sriov->num_vfs; i < num_vfs; i++) {
                create_vf(dev, i);
            }
        } else if (num_vfs < sriov->num_vfs) {
            // 销毁 VF
            for (int i = num_vfs; i < sriov->num_vfs; i++) {
                destroy_vf(dev, i);
            }
        }
    }
    
    sriov->num_vfs = num_vfs;
    pci_set_word(dev->config + addr, num_vfs);
}
```

### NVMe SR-IOV 示例

```c
// hw/femu/nvme.c - NVMe 设备的 SR-IOV 支持

static void nvme_init_pci(NvmeCtrl *n)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    
    // 初始化标准 PCI 配置
    // ...
    
    // 如果支持 SR-IOV
    if (n->params.sriov_max_vfs) {
        uint16_t offset = 0x160;
        
        // 初始化 SR-IOV
        pcie_sriov_pf_init(pci_dev, offset,
                          "nvme-vf",               // VF 名称
                          n->params.vf_device_id,  // VF Device ID
                          n->params.sriov_max_vfs, // Initial VFs
                          n->params.sriov_max_vfs, // Total VFs
                          1,                       // VF Offset
                          1);                      // VF Stride
        
        // 配置 VF BARs
        for (int i = 0; i < PCI_NUM_REGIONS; i++) {
            pcie_sriov_pf_init_vf_bar(pci_dev, i, ...);
        }
    }
}
```

---

## 实用技巧

### 1. 性能优化

**选择合适的 VF 数量**：
```
考虑因素：
- CPU 核心数（每个 VF 需要 CPU 资源）
- 内存带宽（每个 VF 需要 DMA 通道）
- 设备硬件队列数量
- VM 实际需求

建议：
- 高性能场景：1-2 VF per VM
- 密集整合：4-8 VF per VM
- 不要过度分配：VF 数量 ≤ 物理资源
```

**中断亲和性**：
```bash
# 绑定 VF 中断到特定 CPU
echo 2 > /proc/irq/125/smp_affinity_list  # CPU 2

# 与 VM vCPU 对齐
VM vCPU 0 → Host CPU 2 → VF IRQ on CPU 2
```

### 2. 故障排查

**VF 无法启用**：
```bash
# 检查 IOMMU 是否启用
dmesg | grep -i iommu

# 检查内核参数
cat /proc/cmdline | grep iommu
# 应包含：intel_iommu=on iommu=pt

# 检查 ACS (Access Control Services)
lspci -vvv -s 01:00.0 | grep ACSCtl
```

**性能问题**：
```bash
# 检查 VF 链路速度
lspci -vvv -s 01:00.1 | grep "LnkSta:"
LnkSta: Speed 8GT/s, Width x8

# 检查 MSI-X 中断
cat /proc/interrupts | grep vfio

# 监控 DMA 活动
perf stat -e dma:* -p <qemu_pid>
```

### 3. 安全考虑

**隔离**：
```
SR-IOV 提供的隔离：
✓ DMA 隔离 (IOMMU)
✓ 中断隔离 (MSI-X)
✓ 配置空间隔离

不提供的隔离：
✗ 物理链路共享（带宽竞争）
✗ 某些硬件资源共享
✗ 侧信道攻击防护
```

**权限控制**：
```bash
# 限制 VF 访问
chmod 600 /sys/bus/pci/devices/0000:01:00.1/

# 使用 AppArmor/SELinux 限制 QEMU
```

---

## 总结

### 关键要点

1. ✅ **SR-IOV 实现硬件级虚拟化**，接近原生性能
2. ✅ **PF 管理，VF 数据传输**，职责分离
3. ✅ **每个 VF 有独立的 BARs 和中断**，完全隔离
4. ✅ **通过 sysfs 启用 VF**，简单易用
5. ✅ **VFIO 提供安全的设备直通**，现代推荐方法
6. ✅ **性能优化需要考虑资源分配**，避免过度订阅

### SR-IOV vs 其他虚拟化技术

| 技术 | 性能 | 隔离性 | 灵活性 | 复杂度 |
|------|------|--------|--------|--------|
| SR-IOV | 极高 | 高 | 中 | 中 |
| 软件模拟 | 低 | 高 | 高 | 低 |
| 直通 (Passthrough) | 极高 | 高 | 低 | 低 |
| Para-virtualization | 高 | 中 | 高 | 高 |

### 使用场景

**适合 SR-IOV**：
- 高性能网络 (NFV, SDN)
- GPU 虚拟化 (vGPU)
- 存储虚拟化 (NVMe)
- HPC 集群

**不适合 SR-IOV**：
- VM 频繁迁移场景
- 需要细粒度 QoS
- 设备不支持 SR-IOV
- 成本敏感场景

---

## 相关链接

- [PCIe 能力结构](../configuration/config-space.md) - 配置空间详解
- [BARs 详解](../configuration/bars.md) - 地址映射
- [MSI-X 中断](../interrupts/msix.md) - VF 中断机制
- [← 返回高级特性](../advanced-features/)
- [返回首页](../README.md)

---

## 参考资料

- **规范**：PCIe Base Spec Appendix (SR-IOV)
- **实现**：FEMU `/hw/pci/pcie_sriov.c`
- **文档**：Intel VT-d Specification
- **工具**：`lspci`, `dmesg`, VFIO documentation

---

*最后更新：2026-07-06*
