# PCIe 术语表

## 概述

本术语表收录了 PCIe（PCI Express）技术中常用的术语、缩写和概念，按字母顺序排列，方便快速查阅。每个条目包含中英文对照、简要解释和相关链接。

## A

### ACS (Access Control Services)
**访问控制服务**

PCIe 功能，用于控制点对点（Peer-to-Peer）事务的路由。ACS 对于虚拟化环境中的设备隔离至关重要。

```c
// 使能 ACS
pci_enable_acs(pdev, PCI_ACS_SV | PCI_ACS_RR | PCI_ACS_CR | PCI_ACS_UF);
```

**相关术语：** P2P, IOMMU, 虚拟化

### AER (Advanced Error Reporting)
**高级错误报告**

PCIe 扩展能力，提供详细的错误检测和报告机制。AER 包括可纠正错误、不可纠正非致命错误和不可纠正致命错误。

**错误类型：**
- Correctable Errors（可纠正错误）
- Uncorrectable Non-Fatal Errors（不可纠正非致命错误）
- Uncorrectable Fatal Errors（不可纠正致命错误）

### ARI (Alternative Routing-ID Interpretation)
**替代路由 ID 解释**

允许单个 PCIe 设备实现多达 256 个功能（Function），而不是标准的 8 个。常用于 SR-IOV 设备。

**BDF 扩展：**
- 标准：Bus(8) + Device(5) + Function(3) = 256 个功能
- ARI：Bus(8) + Function(8) = 256 个功能

### ASPM (Active State Power Management)
**主动状态电源管理**

PCIe 链路在活动状态下的电源管理机制，定义了 L0、L0s、L1 等链路状态。

**链路状态：**
- L0：全功率，正常运行
- L0s：快速低功率状态（μs 级恢复）
- L1：低功率状态（μs-ms 级恢复）
- L2/L3：更深的睡眠状态

```bash
# 检查 ASPM 状态
lspci -vv | grep -i aspm
```

### ATS (Address Translation Services)
**地址转换服务**

允许 PCIe 设备在本地缓存 IOMMU 地址转换，减少对系统 IOMMU 的访问，提升 DMA 性能。

**相关术语：** IOMMU, PASID, PRI

## B

### BAR (Base Address Register)
**基地址寄存器**

PCIe 配置空间中的寄存器，定义设备的内存或 I/O 地址空间。每个设备最多可以有 6 个 BAR。

```c
// 读取 BAR0
resource_size_t bar0_start = pci_resource_start(pdev, 0);
resource_size_t bar0_len = pci_resource_len(pdev, 0);

// 映射到内核虚拟地址
void __iomem *mmio = pci_iomap(pdev, 0, bar0_len);
```

**BAR 类型：**
- Memory BAR（32-bit 或 64-bit）
- I/O BAR（已过时）
- Prefetchable vs Non-prefetchable

### BDF (Bus:Device:Function)
**总线:设备:功能**

PCIe 设备的唯一标识符，格式为 BB:DD.F。

**格式：**
- Bus（总线号）：8 位（0-255）
- Device（设备号）：5 位（0-31）
- Function（功能号）：3 位（0-7）

**示例：**
```bash
# 显示设备 BDF
lspci
# 输出：01:00.0 Network controller: Intel Corporation
```

### Bridge
**桥接器**

连接两个 PCIe 总线的设备，如 Root Port、Switch 的上游/下游端口。

**类型：**
- PCI-to-PCI Bridge
- PCIe-to-PCI Bridge
- Root Port（一种特殊的 Bridge）

## C

### Capability
**能力寄存器**

PCIe 配置空间中的可选功能结构，通过链表组织。分为 PCI Capabilities（256 字节空间）和 PCIe Extended Capabilities（4096 字节空间）。

**常见 Capabilities：**
- Power Management (0x01)
- MSI (0x05)
- MSI-X (0x11)
- PCIe Capability (0x10)

```c
// 查找 Capability
int pos = pci_find_capability(pdev, PCI_CAP_ID_MSI);
```

### CEM (Card Electromechanical Specification)
**卡电气机械规范**

定义 PCIe 插卡的物理尺寸、连接器、电气特性和热规格。

**标准卡型：**
- Full-Height, Full-Length
- Half-Height, Half-Length
- Low-Profile

### Completion
**完成包**

Non-Posted 事务的响应 TLP，包含读取的数据或写入的状态。

**Completion 状态：**
- Successful Completion (SC)
- Unsupported Request (UR)
- Configuration Request Retry Status (CRS)
- Completer Abort (CA)

### CRC (Cyclic Redundancy Check)
**循环冗余校验**

PCIe 使用的错误检测机制：
- LCRC（Link CRC）：数据链路层，32 位
- ECRC（End-to-End CRC）：端到端，可选

### CXL (Compute Express Link)
**计算快速链路**

基于 PCIe 物理层的高速互连协议，提供缓存一致性和内存语义，用于 CPU-加速器、CPU-内存扩展等场景。

**CXL 类型：**
- CXL.io：等同于 PCIe
- CXL.cache：设备访问主机内存的缓存一致性
- CXL.mem：主机访问设备内存

## D

### DLLP (Data Link Layer Packet)
**数据链路层包**

用于链路管理的控制包，不包含应用数据。

**DLLP 类型：**
- Ack/Nak：确认/否认 TLP 接收
- Flow Control：流控更新
- Power Management：电源管理

### DMA (Direct Memory Access)
**直接内存访问**

设备无需 CPU 介入直接访问系统内存的机制。

**DMA 类型：**
- Coherent DMA（一致性 DMA）
- Streaming DMA（流式 DMA）
- Scatter-Gather DMA

```c
// 分配 DMA 缓冲区
dma_addr_t handle;
void *buf = dma_alloc_coherent(&pdev->dev, size, &handle, GFP_KERNEL);
```

### DOE (Data Object Exchange)
**数据对象交换**

PCIe 6.0 引入的特性，通过配置空间的 Mailbox 交换数据对象，支持 SPDM、CMA、IDE KM 等安全协议。

### DPC (Downstream Port Containment)
**下游端口隔离**

当检测到严重错误时，自动隔离故障设备，防止错误传播到系统其他部分。

## E

### ECAM (Enhanced Configuration Access Mechanism)
**增强配置访问机制**

通过内存映射方式访问 PCIe 配置空间的标准方法。

**地址计算：**
```
Config_Address = ECAM_Base + (Bus << 20) + (Device << 15) + 
                 (Function << 12) + Offset
```

### ECRC (End-to-End CRC)
**端到端循环冗余校验**

可选的端到端错误检测机制，保护 TLP 从发送端到接收端的完整路径。

### Endpoint
**端点设备**

PCIe 拓扑树的叶节点，不能转发事务到其他设备。

**Endpoint 类型：**
- Legacy Endpoint（传统端点）
- PCI Express Endpoint（PCIe 端点）
- Root Complex Integrated Endpoint（RC 集成端点）

### Equalization
**均衡**

PCIe Gen3 及更高版本使用的信号调整技术，补偿传输损耗。

**均衡阶段：**
- Phase 0：系数交换
- Phase 1：发送端适配
- Phase 2：接收端适配
- Phase 3：精细调整

## F

### FLR (Function Level Reset)
**功能级复位**

复位单个 PCIe 功能而不影响同一设备的其他功能或系统其他部分。

```c
// 执行 FLR
pci_reset_function(pdev);
```

### Flow Control
**流控**

PCIe 使用基于信用（Credit）的流控机制，确保接收端有足够的缓冲空间。

**流控单位：**
- Posted（PH/PD）：Posted Header/Data
- Non-Posted（NPH/NPD）：Non-Posted Header/Data
- Completion（CplH/CplD）：Completion Header/Data

## G

### Gen (Generation)
**代**

PCIe 的版本代际，每一代通常速率翻倍。

| Gen | 速率 | 编码 | 带宽 (x16) |
|-----|------|------|-----------|
| Gen1 | 2.5 GT/s | 8b/10b | 4 GB/s |
| Gen2 | 5.0 GT/s | 8b/10b | 8 GB/s |
| Gen3 | 8.0 GT/s | 128b/130b | ~16 GB/s |
| Gen4 | 16.0 GT/s | 128b/130b | ~32 GB/s |
| Gen5 | 32.0 GT/s | 128b/130b | ~64 GB/s |
| Gen6 | 64.0 GT/s | 242b/256b | ~128 GB/s |

### GT/s (GigaTransfers per second)
**每秒千兆传输次数**

PCIe 链路速率的单位，表示每秒传输的符号数。

**计算带宽：**
```
带宽 = (GT/s × 编码效率 × Lane数量) / 8
例如：PCIe 3.0 x16 = (8 × 128/130 × 16) / 8 ≈ 15.75 GB/s
```

## H

### Hot-Plug
**热插拔**

在系统运行时插入或移除 PCIe 设备的能力。

**要求：**
- 硬件支持（Hot-Plug capable slots）
- 操作系统支持
- 驱动程序支持

```bash
# 热移除设备
echo 1 > /sys/bus/pci/devices/0000:01:00.0/remove

# 重新扫描
echo 1 > /sys/bus/pci/rescan
```

## I

### IDE (Integrity and Data Encryption)
**完整性和数据加密**

PCIe 6.0 引入的安全特性，使用 AES-GCM 加密 TLP，防止数据泄露和篡改。

**IDE 类型：**
- Link IDE：保护单个链路
- Selective IDE：选择性加密特定地址范围
- End-to-End IDE：端到端加密（未来）

### INTx
**传统中断**

PCI 时代的共享中断机制（INTA#-INTD#），PCIe 保留了兼容性，但推荐使用 MSI/MSI-X。

### IOMMU (Input-Output Memory Management Unit)
**输入输出内存管理单元**

硬件单元，提供设备 DMA 地址转换和隔离。

**实现：**
- Intel VT-d
- AMD-Vi (AMD IOMMU)
- ARM SMMU

```bash
# 检查 IOMMU 状态
dmesg | grep -i iommu
```

## L

### Lane
**通道**

PCIe 的基本传输单元，由一对差分信号线（TX+/TX-、RX+/RX-）组成。

**Lane 配置：**
- x1, x2, x4, x8, x16, x32

### LCRC (Link CRC)
**链路循环冗余校验**

数据链路层的 32 位 CRC，保护 TLP 在单个链路上的传输。

### Link
**链路**

一个或多个 Lane 组成的逻辑连接。

**链路状态：**
- DL_Down：数据链路层未激活
- DL_Init：初始化中
- DL_Active：激活
- DL_Down（错误）：错误导致的链路断开

### LTSSM (Link Training and Status State Machine)
**链路训练和状态状态机**

管理 PCIe 链路初始化、训练、电源状态转换的状态机。

**主要状态：**
- Detect：检测链路对端
- Polling：交换训练序列
- Configuration：配置 Lane、速率
- L0：正常运行
- Recovery：错误恢复
- L1/L2：低功耗状态

## M

### MPS (Max Payload Size)
**最大有效载荷大小**

TLP 数据包中数据负载的最大字节数，范围 128-4096 字节。

```bash
# 查看 MPS 设置
lspci -vv | grep MaxPayload
```

**常见值：**
- 128, 256, 512, 1024, 2048, 4096 字节

### MRRS (Max Read Request Size)
**最大读请求大小**

设备可以发起的最大内存读请求大小。

```c
// 设置 MRRS
pcie_set_readrq(pdev, 4096);
```

### MSI (Message Signaled Interrupts)
**消息信号中断**

通过内存写事务发送中断，避免共享中断线。支持 1, 2, 4, 8, 16, 32 个中断向量。

```c
// 使能 MSI
int nvec = pci_alloc_irq_vectors(pdev, 1, 32, PCI_IRQ_MSI);
```

### MSI-X (MSI Extended)
**扩展消息信号中断**

MSI 的扩展版本，支持多达 2048 个中断向量，每个向量可以独立配置目标地址和数据。

**优势：**
- 更多中断向量
- 向量可以独立屏蔽
- 表驱动配置

## N

### Non-Posted Transaction
**非提交事务**

需要 Completion 响应的事务，如内存读、I/O 读写、配置读写。

**特点：**
- 有 Completion 响应
- 保证顺序
- 可能阻塞后续事务

### NTB (Non-Transparent Bridge)
**非透明桥**

特殊的 PCIe Bridge，允许两个独立的 PCIe 域互连，每一侧看到的是独立的地址空间。

**应用场景：**
- 服务器间互连
- 主机-主机通信
- 集群系统

## O

### Ordered Set
**有序集**

物理层使用的特殊符号序列，用于链路训练、时钟补偿等。

**类型：**
- TS1/TS2：训练序列
- SKP：时钟补偿
- EIOS：电气空闲有序集

## P

### PAM4 (4-level Pulse Amplitude Modulation)
**4 电平脉冲幅度调制**

PCIe 6.0 使用的编码方案，每个符号携带 2 位信息，实现 64 GT/s 速率。

### PASID (Process Address Space ID)
**进程地址空间标识符**

允许设备为不同的进程维护独立的 IOMMU 上下文，支持共享虚拟内存。

**PASID 范围：**
- 20 位（最多 1M 个地址空间）

### Posted Transaction
**提交事务**

无需 Completion 响应的事务，如内存写、消息。

**特点：**
- 无 Completion
- 低延迟
- Fire-and-forget

### PRI (Page Request Interface)
**页请求接口**

允许设备在 IOMMU 页表缺失时请求页面映射，支持按需分页。

### PTM (Precision Time Measurement)
**精确时间测量**

PCIe 3.1 引入的时间同步机制，提供纳秒级精度的时间参考。

**应用：**
- 5G 网络
- 金融交易
- 音视频同步

## R

### RCB (Read Completion Boundary)
**读完成边界**

限制读 Completion 不能跨越的地址边界，通常为 64 或 128 字节。

### Relaxed Ordering
**宽松排序**

允许某些事务不严格遵守 PCIe 排序规则，提升性能。

```c
// 使能 Relaxed Ordering
pcie_capability_set_word(pdev, PCI_EXP_DEVCTL, PCI_EXP_DEVCTL_RELAX_EN);
```

### Root Complex (RC)
**根复合体**

PCIe 拓扑的根节点，连接 CPU 和内存系统，通常集成在芯片组中。

**组件：**
- Root Port(s)
- Internal Endpoint(s)（可选）
- Internal Switch（可选）

### Root Port
**根端口**

Root Complex 上的 PCIe Bridge，连接到下游设备或 Switch。

## S

### Slot
**插槽**

物理 PCIe 插槽，可以插入扩展卡。

**规格：**
- x1, x4, x8, x16 机械插槽
- 电气 Lane 数量可能不同

### SPDM (Security Protocol and Data Model)
**安全协议和数据模型**

DMTF 定义的安全协议，通过 DOE 传输，用于设备认证和密钥交换。

### SR-IOV (Single Root I/O Virtualization)
**单根 I/O 虚拟化**

允许单个物理设备虚拟为多个虚拟功能（VF），每个 VM 独占一个 VF。

**功能类型：**
- PF（Physical Function）：物理功能
- VF（Virtual Function）：虚拟功能

```bash
# 使能 SR-IOV
echo 4 > /sys/bus/pci/devices/0000:01:00.0/sriov_numvfs
```

### Switch
**交换机**

PCIe 多端口设备，包含一个上游端口和多个下游端口，用于扩展 PCIe 拓扑。

**结构：**
```
     ┌─────────┐
     │Upstream │
     │  Port   │
     └────┬────┘
    ┌─────┴─────┬─────┐
┌───▼───┐ ┌─────▼──┐ ┌▼────┐
│Down 1 │ │Down 2  │ │Down3│
└───────┘ └────────┘ └─────┘
```

## T

### TC (Traffic Class)
**流量类别**

PCIe 定义的 8 个流量优先级（TC0-TC7），支持 QoS。

### TLP (Transaction Layer Packet)
**事务层包**

PCIe 的核心数据包格式，携带读写请求、完成和消息。

**TLP 格式：**
```
┌────────┬─────────┬────────┬──────┐
│ Header │ Payload │ Digest │ LCRC │
└────────┴─────────┴────────┴──────┘
```

**TLP 类型：**
- Memory Read/Write
- I/O Read/Write
- Configuration Read/Write
- Message
- Completion

## U

### Upstream
**上游**

指向 Root Complex 的方向。

## V

### VC (Virtual Channel)
**虚拟通道**

独立的流控和排序域，每个 VC 有独立的流控缓冲区。

**默认：**
- VC0：必须支持，映射所有 TC
- VC1-VC7：可选

### VF (Virtual Function)
**虚拟功能**

SR-IOV 创建的轻量级 PCIe 功能，可以直接分配给 VM。

## 缩写速查表

| 缩写 | 全称 | 中文 |
|------|------|------|
| ACS | Access Control Services | 访问控制服务 |
| AER | Advanced Error Reporting | 高级错误报告 |
| ARI | Alternative Routing-ID Interpretation | 替代路由 ID 解释 |
| ASPM | Active State Power Management | 主动状态电源管理 |
| ATS | Address Translation Services | 地址转换服务 |
| BAR | Base Address Register | 基地址寄存器 |
| BDF | Bus:Device:Function | 总线:设备:功能 |
| CEM | Card Electromechanical Specification | 卡电气机械规范 |
| CRC | Cyclic Redundancy Check | 循环冗余校验 |
| CXL | Compute Express Link | 计算快速链路 |
| DLLP | Data Link Layer Packet | 数据链路层包 |
| DMA | Direct Memory Access | 直接内存访问 |
| DOE | Data Object Exchange | 数据对象交换 |
| DPC | Downstream Port Containment | 下游端口隔离 |
| ECAM | Enhanced Configuration Access Mechanism | 增强配置访问机制 |
| ECRC | End-to-End CRC | 端到端 CRC |
| FLR | Function Level Reset | 功能级复位 |
| IDE | Integrity and Data Encryption | 完整性和数据加密 |
| IOMMU | Input-Output Memory Management Unit | 输入输出内存管理单元 |
| LCRC | Link CRC | 链路 CRC |
| LTSSM | Link Training and Status State Machine | 链路训练状态机 |
| MPS | Max Payload Size | 最大有效载荷大小 |
| MRRS | Max Read Request Size | 最大读请求大小 |
| MSI | Message Signaled Interrupts | 消息信号中断 |
| MSI-X | MSI Extended | 扩展消息信号中断 |
| NTB | Non-Transparent Bridge | 非透明桥 |
| PAM4 | 4-level Pulse Amplitude Modulation | 4 电平脉冲幅度调制 |
| PASID | Process Address Space ID | 进程地址空间标识符 |
| PRI | Page Request Interface | 页请求接口 |
| PTM | Precision Time Measurement | 精确时间测量 |
| RC | Root Complex | 根复合体 |
| RCB | Read Completion Boundary | 读完成边界 |
| SPDM | Security Protocol and Data Model | 安全协议和数据模型 |
| SR-IOV | Single Root I/O Virtualization | 单根 I/O 虚拟化 |
| TC | Traffic Class | 流量类别 |
| TLP | Transaction Layer Packet | 事务层包 |
| VC | Virtual Channel | 虚拟通道 |
| VF | Virtual Function | 虚拟功能 |

## 延伸阅读

- [PCIe 基础概念](./basics.md)
- [PCIe 学习路径](./learning-path.md)
- [PCI vs PCIe](./pci-vs-pcie.md)
- [PCIe 规范文档](https://pcisig.com/)

## 贡献

欢迎补充和完善本术语表。如有遗漏或错误，请提交 Issue 或 Pull Request。
