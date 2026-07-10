# PCIe 拓扑结构

## 概述

PCIe 采用**树形拓扑结构**，由 Root Complex 作为根节点，通过 Switch 扩展，连接多个 Endpoint 设备。每个连接都是**点对点**的，没有共享总线。

---

## 核心组件

### 1. Root Complex (根复合体)

**定义**：PCIe 拓扑的根节点，通常集成在 CPU 或芯片组中。

**职责**：
- 连接 CPU 和内存到 PCIe 树
- 生成配置事务
- 处理中断和错误
- 提供多个 Root Port（根端口）

**典型配置**：
```
┌─────────────────────────────┐
│          CPU/SOC            │
│                             │
│  ┌────────────────────────┐ │
│  │   Root Complex (RC)    │ │
│  │                        │ │
│  │  ┌──┐  ┌──┐  ┌──┐     │ │
│  │  │RP│  │RP│  │RP│ ... │ │  RP = Root Port
│  └──┴──┴──┴──┴──┴──┴─────┘ │
└─────┬──────┬──────┬─────────┘
      │      │      │
   (PCIe)  (PCIe) (PCIe)
      │      │      │
     设备  Switch  设备
```

**实现示例**：
- Intel CPU：内置 PCIe Root Complex，提供 16-24 条 Lane
- AMD Ryzen：内置 PCIe 控制器
- 服务器平台：可能有多个 Root Complex

---

### 2. Switch (交换机)

**定义**：扩展 PCIe 连接数量的桥接设备。

**结构**：
- **1 个上行端口 (Upstream Port)**：连接到 Root Complex 或上级 Switch
- **多个下行端口 (Downstream Port)**：连接到下级设备或 Switch

**工作原理**：
```
            上行端口 (x8)
                ▲
                │
        ┌───────┴───────┐
        │    Switch     │
        │   (内部路由)   │
        └───┬───┬───┬───┘
            │   │   │
         下行端口 (x4 each)
            │   │   │
          设备 设备 设备
```

**Lane 分配示例**：
- 上行链路：x8 (连接到 Root Complex)
- 下行链路 1：x4 (显卡)
- 下行链路 2：x2 (NVMe SSD)
- 下行链路 3：x2 (网卡)

**透明 vs 非透明**：
- **透明 Switch**：下游设备对软件完全可见（标准模式）
- **非透明 Switch (NTB)**：隔离不同的地址空间，用于多主机系统

---

### 3. Endpoint (端点设备)

**定义**：PCIe 拓扑的叶子节点，实际的功能设备。

**分类**：

#### Type 0 Endpoint (标准端点)
- 普通 PCIe 设备
- 有一个上行端口连接到系统
- 示例：显卡、网卡、NVMe SSD

#### Type 1 Endpoint (桥接设备)
- 可以桥接到其他总线
- 示例：PCIe-to-PCI 桥接器

#### Root Complex Integrated Endpoint
- 集成在 Root Complex 内部的设备
- 无需物理 PCIe 链路
- 示例：集成显卡、集成网卡

**配置示例**：
```c
// FEMU 中的 NVMe Endpoint 定义
PCIDevice *pci_dev = PCI_DEVICE(n);
pci_config_set_class(pci_dev->config, PCI_CLASS_STORAGE_EXPRESS);

// 注册 BAR0 (内存映射寄存器)
memory_region_init_io(&n->iomem, ...);
pci_register_bar(pci_dev, 0, PCI_BASE_ADDRESS_SPACE_MEMORY, &n->iomem);
```

---

### 4. Bridge (桥接器)

**定义**：连接两个 PCIe 总线段的设备。

**类型**：
- **Root Port**：Root Complex 的下行端口（Type 1 配置头）
- **Switch Port**：Switch 的上行/下行端口
- **PCIe-to-PCI Bridge**：连接传统 PCI 设备

**配置空间特点**：
- 使用 Type 1 配置头
- 包含总线编号（Primary/Secondary/Subordinate）
- 管理地址窗口（内存/IO 地址范围）

---

## 完整拓扑示例

### 典型桌面系统

```
                    ┌───────────────────┐
                    │       CPU         │
                    │  ┌──────────────┐ │
                    │  │Root Complex  │ │
                    │  └──┬────┬────┬─┘ │
                    └─────┼────┼────┼───┘
                          │    │    │
                ┌─────────┘    │    └──────────┐
                │              │               │
                │ x16          │ x4            │ x1
                │              │               │
          ┌─────▼─────┐  ┌────▼────┐    ┌────▼─────┐
          │    GPU    │  │  NVMe   │    │  Wi-Fi   │
          │ (Endpoint)│  │   SSD   │    │  (EP)    │
          └───────────┘  │  (EP)   │    └──────────┘
                         └─────────┘
```

### 服务器系统（带 Switch）

```
                ┌────────────────────────────┐
                │          CPU               │
                │  ┌───────────────────────┐ │
                │  │   Root Complex        │ │
                │  └───┬───────────┬───────┘ │
                └──────┼───────────┼─────────┘
                       │ x16       │ x8
                       │           │
          ┌────────────▼──┐   ┌───▼──────────────┐
          │     GPU       │   │   PCIe Switch    │
          │  (Endpoint)   │   │  ┌────────────┐  │
          └───────────────┘   │  │  Internal  │  │
                              │  │   Router   │  │
                              │  └─┬──┬──┬──┬─┘  │
                              └────┼──┼──┼──┼────┘
                                   │  │  │  │
                      ┌────────────┘  │  │  └──────────┐
                      │               │  │             │
                  ┌───▼───┐      ┌───▼──▼───┐    ┌───▼────┐
                  │ NVMe  │      │  NVMe    │    │ 10GbE  │
                  │  SSD  │      │   SSD    │    │  NIC   │
                  └───────┘      └──────────┘    └────────┘
```

### 多 Root Complex 系统

```
┌──────────────┐              ┌──────────────┐
│   CPU 0      │              │   CPU 1      │
│  ┌────────┐  │              │  ┌────────┐  │
│  │  RC 0  │  │              │  │  RC 1  │  │
│  └───┬────┘  │              │  └───┬────┘  │
└──────┼───────┘              └──────┼───────┘
       │                             │
       │                             │
   ┌───▼────┐                    ┌───▼────┐
   │  GPU   │                    │  GPU   │
   └────────┘                    └────────┘

   独立的 PCIe 域                独立的 PCIe 域
```

---

## 设备枚举

系统启动时如何发现 PCIe 设备？

### 枚举过程

```
1. BIOS/UEFI 启动
   │
   ├─> 扫描 Root Complex 的所有 Root Port
   │
2. 对每个 Port：
   │
   ├─> 检查链路是否已训练 (Link Training)
   │   │
   │   ├─> 未训练 → 跳过
   │   └─> 已训练 → 继续
   │
   ├─> 发送 Configuration Read (Type 0)
   │   │
   │   ├─> 无响应 → 无设备
   │   └─> 有响应 → 发现设备
   │
   ├─> 读取 Vendor ID, Device ID, Class Code
   │
   ├─> 分配总线编号 (Bus Number)
   │
   ├─> 配置 BAR (Base Address Register)
   │   └─> 分配内存/IO 地址空间
   │
   ├─> 如果是 Bridge (Switch Port)：
   │   └─> 递归扫描下游总线
   │
   └─> 如果是 Endpoint：
       └─> 枚举完成，加载驱动
```

### 总线编号分配示例

```
Bus 0 (Root Bus)
 ├─ Device 0: Root Port 0
 │   └─ Bus 1
 │       └─ Device 0: GPU (Endpoint)
 │
 ├─ Device 1: Root Port 1
 │   └─ Bus 2
 │       └─ Device 0: Switch
 │           ├─ Bus 3 (下行端口 0)
 │           │   └─ Device 0: NVMe SSD
 │           │
 │           └─ Bus 4 (下行端口 1)
 │               └─ Device 0: Network Card
 │
 └─ Device 2: Root Port 2
     └─ Bus 5
         └─ Device 0: Wi-Fi Card
```

**BDF 地址格式**：`Bus:Device.Function`
- GPU: `01:00.0`
- NVMe SSD: `03:00.0`
- Network Card: `04:00.0`

---

## 拓扑发现工具

### Linux: lspci

```bash
# 查看树形拓扑
$ lspci -tv

-[0000:00]-+-00.0  Intel Corporation Host Bridge
           +-01.0-[01]----00.0  NVIDIA Corporation GPU
           +-02.0-[02-04]----00.0  Switch Upstream Port
           |                 +-00.0-[03]----00.0  Samsung NVMe SSD
           |                 \-01.0-[04]----00.0  Intel 10GbE NIC
           \-03.0-[05]----00.0  Intel Wi-Fi Adapter

# 详细信息
$ lspci -vvv -s 01:00.0  # 查看 GPU 的详细配置
```

### Windows: Device Manager

- 设备管理器 → 查看 → 按连接排序设备
- 显示设备的物理连接关系

---

## 拓扑限制和约束

### 最大深度
- **规范限制**：无明确限制，但实际受总线编号约束
- **总线编号范围**：0-255（8 位）
- **实际深度**：通常不超过 5-7 层

### 最大设备数
- **每条总线**：最多 32 个设备
- **每个设备**：最多 8 个功能 (Function 0-7)
- **ARI 模式**：256 个功能（扩展编号）

### Lane 分配约束
- **总 Lane 数**：受 Root Complex 限制（通常 16-48 条）
- **分配示例**：
  - GPU: x16
  - NVMe: x4
  - 其他设备：共享剩余 Lane

---

## 拓扑对性能的影响

### 直连 vs 通过 Switch

**直连到 Root Complex**（最佳）：
```
CPU ←→ GPU  (最低延迟，全带宽)
```

**通过 Switch**：
```
CPU ←→ Switch ←→ GPU  (增加延迟，可能带宽受限)
```

### Switch 带宽分配

如果 Switch 的上行链路 < 所有下行链路之和：
```
上行: x8 (8 GB/s)
下行: x4 + x4 + x4 = x12 (12 GB/s 潜在需求)

结果：设备同时传输时会竞争带宽
```

### P2P (Peer-to-Peer) 通信

**同一 Switch 下的设备**：
- 可以直接通信，无需经过 Root Complex
- 需要 ACS (Access Control Services) 支持

**不同 Switch 下的设备**：
- 必须经过共同父节点（通常是 Root Complex）

---

## FEMU 中的拓扑实现

### Root Complex 模拟

```c
// QEMU 中创建 Root Complex
PCIHostState *phb = PCI_HOST_BRIDGE(dev);
PCIBus *bus = pci_register_root_bus(...);
```

### Switch 模拟

```c
// 创建 Switch 上行端口
pcie_switch_upstream_port_realize(...);

// 创建 Switch 下行端口
pcie_switch_downstream_port_realize(...);
```

### Endpoint 模拟

```c
// NVMe Endpoint
PCIDevice *pci_dev = PCI_DEVICE(nvme);
pcie_endpoint_cap_init(pci_dev, ...);
```

---

## 总结

### 关键要点

1. ✅ **树形拓扑**：Root Complex 为根，Endpoint 为叶子
2. ✅ **点对点连接**：每个链路独享带宽
3. ✅ **三种核心组件**：Root Complex、Switch、Endpoint
4. ✅ **总线枚举**：系统启动时自动发现设备
5. ✅ **拓扑影响性能**：直连优于 Switch，注意带宽瓶颈

---

## 下一步学习

- [三层协议模型](../architecture/layering.md) - 了解数据如何在拓扑中传输
- [设备枚举详解](../architecture/enumeration.md) - 深入枚举过程
- [配置空间](../configuration/config-space.md) - 如何识别和配置设备

---

## 参考资料

- **规范**：PCIe Base Spec Chapter 1 (Figure 1-2, 1-3)
- **实现**：`/hw/pci/pcie_port.c` (FEMU/QEMU)

---

**相关页面**：
- [← PCIe 简介](overview.md)
- [三层协议模型 →](../architecture/layering.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
