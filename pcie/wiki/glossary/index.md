# PCIe 术语表

本术语表涵盖 PCI Express 技术中的所有关键术语、缩写和概念。

---

## A

### ACK (Acknowledge)
**确认** - 数据链路层的 DLLP，用于确认成功接收 TLP。

### ACS (Access Control Services)
**访问控制服务** - PCIe 扩展能力，用于控制点对点流量和增强虚拟化安全性。
- 源验证 (Source Validation)
- 翻译阻止 (Translation Blocking)
- P2P 请求重定向
- P2P 完成重定向

### AER (Advanced Error Reporting)
**高级错误报告** - PCIe 扩展能力，提供详细的错误检测、日志记录和报告机制。
- 支持可纠正和不可纠正错误
- TLP 前缀日志
- 错误源识别

### ARI (Alternative Routing-ID Interpretation)
**备选路由 ID 解释** - 扩展能力，将设备号和功能号合并为 8 位功能号，支持最多 256 个功能（而非标准的 8 个）。

### ASPM (Active State Power Management)
**主动状态电源管理** - 在 L0 状态下动态进入低功耗链路状态（L0s、L1）以节省电力。

### AtomicOp (Atomic Operation)
**原子操作** - PCIe 事务，保证读-修改-写操作的原子性，用于多处理器同步。
- Fetch and Add
- Swap
- Compare and Swap (CAS)

### ATS (Address Translation Services)
**地址翻译服务** - 允许设备缓存 IOMMU 地址翻译，减少翻译延迟。

---

## B

### BAR (Base Address Register)
**基地址寄存器** - 配置空间中的寄存器（偏移 0x10-0x24），用于映射设备的内存或 IO 地址空间。
- 32 位或 64 位地址
- 内存或 IO 空间
- 可预取或不可预取

### BDF (Bus:Device.Function)
**总线:设备.功能** - PCIe 设备的三段地址标识符。
- Bus: 0-255 (8 位)
- Device: 0-31 (5 位)
- Function: 0-7 (3 位，标准) 或 0-255 (8 位，ARI)
- 示例：`03:00.0`

### Bridge
**桥接器** - 连接两个 PCIe 总线段的设备，使用 Type 1 配置头。
- Root Port (根端口)
- Switch Port (交换机端口)
- PCIe-to-PCI Bridge

---

## C

### CAS (Compare and Swap)
**比较并交换** - 原子操作类型，用于无锁编程。

### CMA (Component Measurement and Authentication)
**组件测量和认证** - 通过 DOE 实现的安全特性。

### COM (Comma)
**逗号符号** - 8b/10b 编码中的特殊符号 (K28.5)，用于比特对齐和链路训练。

### Completion (Cpl)
**完成** - TLP 类型，对 Non-Posted 请求的响应。
- Cpl: 无数据的完成
- CplD: 带数据的完成
- CplLk: 锁定完成（已废弃）

### Configuration Space
**配置空间** - 每个 PCIe 功能的 256 字节（标准 PCI）或 4096 字节（PCIe 扩展）地址空间，用于设备识别和配置。
- Type 0 Header: Endpoint 设备
- Type 1 Header: Bridge 设备

### Credit
**信用** - 流控制机制中的单位，表示接收缓冲区的可用空间。
- Header Credit (HDR): TLP 头部数量
- Data Credit (DATA): 数据负载大小（DW）

---

## D

### D-State (Device Power State)
**设备电源状态** - PCI 设备的电源管理状态。
- D0: 完全工作
- D1/D2: 中间省电状态（可选）
- D3Hot: 深度省电，主电源开启
- D3Cold: 最深省电，主电源关闭

### DLL (Data Link Layer)
**数据链路层** - PCIe 三层协议栈的中间层，负责可靠传输。

### DLLP (Data Link Layer Packet)
**数据链路层包** - 控制包，用于 ACK/NAK、流控制、电源管理。

### DOE (Data Object Exchange)
**数据对象交换** - PCIe 扩展能力，提供安全的数据交换协议。
- 支持 CMA、SPDM 等安全协议

### Downstream Port
**下行端口** - Switch 或 Root Complex 向下连接到设备的端口。

### DPC (Downstream Port Containment)
**下游端口隔离** - 错误隔离机制，防止错误传播。

### DW (Double Word)
**双字** - 32 位（4 字节）数据单位。

---

## E

### ECAM (Enhanced Configuration Access Mechanism)
**增强配置访问机制** - 通过内存映射访问 PCIe 扩展配置空间的机制。

### ECRC (End-to-End CRC)
**端到端 CRC** - 可选的 32 位 CRC，覆盖整个 TLP 的端到端路径，检测 Switch 内部和内存错误。

### EDB (End Bad)
**错误结束** - 8b/10b 编码中的特殊符号，表示 TLP 以错误结束。

### Endpoint (EP)
**端点** - PCIe 拓扑的叶子节点，实际的功能设备。
- Type 0 Endpoint: 标准设备
- Root Complex Integrated Endpoint: 集成在 RC 内部的设备

### Equalization
**均衡** - 物理层技术，补偿信号损耗和失真，用于高速率（Gen 3+）。

---

## F

### FCP (Flow Control Protocol Error)
**流控制协议错误** - AER 不可纠正错误类型。

### FLR (Function Level Reset)
**功能级复位** - 复位单个 PCIe 功能而不影响其他功能。

### Flow Control (FC)
**流控制** - 基于信用的机制，防止发送方超出接收方缓冲区容量。

### Fmt (Format)
**格式** - TLP 头部字段，指示 TLP 的基本格式（3 DW 或 4 DW 头部，是否有数据）。

### Function
**功能** - PCIe 设备的独立功能单元，每个设备最多 8 个功能（标准）或 256 个（ARI）。

---

## G

### Gen (Generation)
**代** - PCIe 版本的速度等级。
- Gen 1: 2.5 GT/s
- Gen 2: 5.0 GT/s
- Gen 3: 8.0 GT/s
- Gen 4: 16.0 GT/s
- Gen 5: 32.0 GT/s
- Gen 6: 64.0 GT/s

### GT/s (Giga Transfers per second)
**千兆传输/秒** - PCIe 信号速率单位，表示每秒传输的符号数（不等于数据速率，因编码而异）。

---

## H

### HDR (Header)
**头部** - TLP 或 DLLP 的控制信息部分。

### Hot-Plug
**热插拔** - 在系统运行时插入或移除设备的能力。
- Native Hot-Plug: PCIe 原生支持
- ACPI Hot-Plug: 通过 ACPI 实现

---

## I

### IDE (Integrity and Data Encryption)
**完整性和数据加密** - PCIe 6.0 引入的安全特性，保护 TLP 数据的完整性和机密性。

### InitFC (Initialize Flow Control)
**初始化流控制** - 数据链路层 DLLP，用于链路初始化时交换流控制信用信息。
- InitFC1: 第一阶段
- InitFC2: 第二阶段

### INTx
**传统中断** - PCI 兼容的共享中断线（INTA#-INTD#）。

### IO Space
**IO 空间** - x86 架构的独立 IO 地址空间（0-64K），通过 IN/OUT 指令访问。

---

## L

### Lane
**通道** - PCIe 的最小传输单元，由两对差分信号线组成（TX+/TX-, RX+/RX-）。

### LCRC (Link CRC)
**链路 CRC** - 32 位 CRC，由数据链路层添加，覆盖 TLP，用于逐跳错误检测。

### Link
**链路** - 由 1-32 条 Lane 组成的点对点连接。

### Link Width
**链路宽度** - 链路中并行 Lane 的数量：x1, x2, x4, x8, x12, x16, x32。

### L-State (Link Power State)
**链路电源状态** - PCIe 链路的电源管理状态。
- L0: 正常工作状态
- L0s: 快速恢复省电状态
- L1: 较深省电状态
- L1.1/L1.2: L1 子状态（更省电）
- L2: 辅助电源状态
- L3: 完全关闭

### LTSSM (Link Training and Status State Machine)
**链路训练和状态状态机** - 管理物理层链路生命周期的有限状态机。
- Detect, Polling, Configuration, L0, Recovery, Disabled 等状态

---

## M

### Max Payload Size (MPS)
**最大负载大小** - TLP 数据负载的最大字节数（128, 256, 512, 1024, 2048, 4096）。

### Max Read Request Size (MRRS)
**最大读请求大小** - 单个读请求的最大字节数（128-4096）。

### Memory Space
**内存空间** - 系统主内存地址空间，PCIe 设备的 BAR 映射到此空间。

### Message (Msg)
**消息** - TLP 类型，用于事件通知（中断、电源管理、错误报告等）。
- Msg: 无数据消息
- MsgD: 带数据消息

### MPS (Max Payload Size)
见 Max Payload Size

### MRd (Memory Read)
**内存读** - TLP 类型，从内存地址读取数据（Non-Posted）。

### MRRS (Max Read Request Size)
见 Max Read Request Size

### MSI (Message Signaled Interrupt)
**消息信号中断** - 通过内存写事务发送中断，最多 32 个中断向量。

### MSI-X (MSI Extended)
**扩展消息信号中断** - 更灵活的 MSI 版本，支持最多 2048 个向量，每个向量独立配置。

### MWr (Memory Write)
**内存写** - TLP 类型，向内存地址写入数据（Posted）。

---

## N

### NAK (Negative Acknowledge)
**否定确认** - 数据链路层 DLLP，请求重传 TLP（LCRC 错误或序列号错误）。

### Non-Posted
**非 Posted** - 需要 Completion 响应的事务类型（Memory Read, IO R/W, Config R/W）。

### NTB (Non-Transparent Bridge)
**非透明桥** - 隔离不同地址空间的桥接器，用于多主机系统。

---

## O

### Ordered Set (OS)
**有序集** - 物理层的特殊符号序列，用于链路训练、时钟补偿、电气空闲等。
- TS1/TS2: 训练序列
- SKP: 时钟补偿
- EIEOS: 电气空闲退出

---

## P

### P2P (Peer-to-Peer)
**点对点** - 设备之间直接通信，不经过 Root Complex。

### PAM4 (Pulse Amplitude Modulation 4-level)
**4 电平脉冲幅度调制** - PCIe 6.0 使用的编码，每个符号传输 2 位。

### PASID (Process Address Space ID)
**进程地址空间 ID** - 用于设备支持多个虚拟地址空间。

### PCIe (PCI Express)
**PCI 快速总线** - 高速串行计算机扩展总线标准。

### PF (Physical Function)
**物理功能** - SR-IOV 中的物理设备功能，可创建虚拟功能。

### PHY (Physical Layer)
**物理层** - PCIe 三层协议栈的最底层。

### PM (Power Management)
**电源管理** - 管理设备和链路的电源状态以节省能源。

### PMUX (Protocol Multiplexing)
**协议复用** - 在单个 PCIe 链路上复用多种协议。

### Poisoned TLP
**中毒 TLP** - 带有错误标记的 TLP，表示数据可能不可靠。

### Posted
**Posted** - 无需响应的事务类型（Memory Write, Message）。

### PRI (Page Request Interface)
**页请求接口** - 允许设备请求操作系统分配内存页。

### PTM (Precision Time Measurement)
**精确时间测量** - 在 PCIe 拓扑中同步时间的机制。

---

## Q

### QoS (Quality of Service)
**服务质量** - 通过虚拟通道实现的流量优先级管理。

---

## R

### RC (Root Complex)
**根复合体** - PCIe 拓扑的根节点，通常集成在 CPU 或芯片组中。

### Relaxed Ordering (RO)
**宽松排序** - TLP 属性，允许放宽某些排序约束以提高性能。

### Requester ID (RID)
**请求者 ID** - TLP 头部字段，标识请求的来源（Bus:Device.Function）。

### Retimer
**重定时器** - PCIe 4.0/5.0 引入的设备，重新生成信号以延长链路距离。

### Retry Buffer
**重试缓冲区** - 数据链路层缓冲区，存储已发送但未确认的 TLP，用于重传。

### Root Port (RP)
**根端口** - Root Complex 的下行端口，连接到 PCIe 设备或 Switch。

### RP (Root Port)
见 Root Port

---

## S

### Scrambling
**加扰** - 物理层技术，将数据随机化以减少 EMI 和改善信号特性。

### Sequence Number
**序列号** - 数据链路层添加的 12 位编号（0-4095），用于跟踪和重传 TLP。

### SKP (Skip)
**跳过符号** - 时钟补偿符号，用于调整发送和接收时钟差异。

### SPDM (Security Protocol and Data Model)
**安全协议和数据模型** - 通过 DOE 实现的设备认证协议。

### SR-IOV (Single Root I/O Virtualization)
**单根 IO 虚拟化** - 允许单个物理设备虚拟为多个虚拟功能的技术。
- PF (Physical Function): 物理功能
- VF (Virtual Function): 虚拟功能

### STP (Start TLP)
**TLP 开始** - 物理层成帧符号，标记 TLP 的开始。

### Switch
**交换机** - 扩展 PCIe 连接数量的桥接设备，有 1 个上行端口和多个下行端口。

---

## T

### Tag
**标签** - TLP 头部字段，用于匹配 Non-Posted 请求和其 Completion。

### TC (Traffic Class)
**流量类别** - TLP 属性，用于虚拟通道映射和 QoS。

### TL (Transaction Layer)
**事务层** - PCIe 三层协议栈的最高层。

### TLP (Transaction Layer Packet)
**事务层包** - PCIe 的数据包，携带实际的读写事务和消息。

### TPH (TLP Processing Hints)
**TLP 处理提示** - 扩展能力，提供处理器缓存提示以优化性能。

### Type (TLP Type)
**类型** - TLP 头部字段，指示事务类型（Memory, IO, Config, Message, Completion）。

### Type 0 Header
**类型 0 头部** - Endpoint 设备使用的配置空间头部格式。

### Type 1 Header
**类型 1 头部** - Bridge 设备使用的配置空间头部格式。

---

## U

### UpdateFC
**更新流控制** - 数据链路层 DLLP，在运行时更新流控制信用。

### Upstream Port
**上行端口** - Switch 或 Endpoint 向上连接到 Root Complex 的端口。

### UR (Unsupported Request)
**不支持的请求** - AER 错误类型，表示设备不支持该请求。

---

## V

### VC (Virtual Channel)
**虚拟通道** - 在单个物理链路上实现流量分类的逻辑通道。
- VC0: 默认虚拟通道（必须）
- VC1-VC7: 可选

### Vendor ID
**厂商 ID** - 配置空间偏移 0x00 的 16 位字段，由 PCI-SIG 分配，标识设备制造商。

### VF (Virtual Function)
**虚拟功能** - SR-IOV 中由物理功能创建的轻量级功能，可独立分配给虚拟机。

### VPD (Vital Product Data)
**重要产品数据** - 存储设备信息（序列号、部件号等）的只读或可读写数据区域。

---

## 常用缩写速查

| 缩写 | 全称 | 中文 |
|------|------|------|
| ACK | Acknowledge | 确认 |
| ACS | Access Control Services | 访问控制服务 |
| AER | Advanced Error Reporting | 高级错误报告 |
| ARI | Alternative Routing-ID Interpretation | 备选路由 ID 解释 |
| ASPM | Active State Power Management | 主动状态电源管理 |
| BAR | Base Address Register | 基地址寄存器 |
| BDF | Bus:Device.Function | 总线:设备.功能 |
| CRC | Cyclic Redundancy Check | 循环冗余校验 |
| DLL | Data Link Layer | 数据链路层 |
| DLLP | Data Link Layer Packet | 数据链路层包 |
| DOE | Data Object Exchange | 数据对象交换 |
| DW | Double Word | 双字（32位） |
| ECRC | End-to-End CRC | 端到端 CRC |
| EP | Endpoint | 端点 |
| FC | Flow Control | 流控制 |
| GT/s | Giga Transfers per second | 千兆传输/秒 |
| LCRC | Link CRC | 链路 CRC |
| LTSSM | Link Training and Status State Machine | 链路训练和状态状态机 |
| MPS | Max Payload Size | 最大负载大小 |
| MRRS | Max Read Request Size | 最大读请求大小 |
| MSI | Message Signaled Interrupt | 消息信号中断 |
| MSI-X | MSI Extended | 扩展消息信号中断 |
| NAK | Negative Acknowledge | 否定确认 |
| NTB | Non-Transparent Bridge | 非透明桥 |
| P2P | Peer-to-Peer | 点对点 |
| PCIe | PCI Express | PCI 快速总线 |
| PF | Physical Function | 物理功能 |
| PHY | Physical Layer | 物理层 |
| PM | Power Management | 电源管理 |
| PTM | Precision Time Measurement | 精确时间测量 |
| RC | Root Complex | 根复合体 |
| RID | Requester ID | 请求者 ID |
| RP | Root Port | 根端口 |
| SR-IOV | Single Root I/O Virtualization | 单根 IO 虚拟化 |
| TC | Traffic Class | 流量类别 |
| TL | Transaction Layer | 事务层 |
| TLP | Transaction Layer Packet | 事务层包 |
| TPH | TLP Processing Hints | TLP 处理提示 |
| VC | Virtual Channel | 虚拟通道 |
| VF | Virtual Function | 虚拟功能 |
| VPD | Vital Product Data | 重要产品数据 |

---

## 相关页面

- [返回首页](../README.md)
- [PCIe 简介](../introduction/overview.md)
- [寄存器速查](register-quick-ref.md)

---

*最后更新：2026-07-06*
