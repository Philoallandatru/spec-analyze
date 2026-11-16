# PCIe 术语表

本术语表按字母顺序列出 PCI Express 核心协议、物理层、配置空间及安全/虚拟化扩展中的关键术语与缩写，并提供直达对应专题文档的链接。

---

## A

### ACK (Acknowledge)
**确认** - 数据链路层的 DLLP，接收方用来向发送方确认已无误收到指定序列号之前的 TLP，从而释放发送方的 Retry Buffer。参见：[Ack/Nak 协议](../data-link-layer/ack-nak.md)。

### ACS (Access Control Services)
**访问控制服务** - PCIe 扩展能力，用于控制设备间的点对点（P2P）流量路由并增强虚拟化隔离安全性，强制可疑流量经由 Root Complex / IOMMU 检查。参见：[ACS](../advanced-features/acs.md)。
- **Source Validation**：源 ID 验证，防止设备伪造 Requester ID
- **Translation Blocking**：阻止未受信任设备绕过 IOMMU 直接使用已翻译地址
- **P2P Request / Completion Redirect**：将 P2P 流量重定向到上游 Root Complex

### AER (Advanced Error Reporting)
**高级错误报告** - PCIe 扩展能力（Cap ID `0x0001`），提供细粒度的可纠正（Correctable）与不可纠正（Uncorrectable）错误检测、掩码、严重性配置及 TLP Header 日志记录。参见：[AER](../error-handling/aer.md)。

### ARI (Alternative Routing-ID Interpretation)
**备选路由 ID 解释** - 扩展能力，将传统 5 位 Device 号与 3 位 Function 号合并为 8 位 Function 号，使单个下游设备最多支持 256 个 Function（而非标准的 8 个），常与 SR-IOV 配合使用。参见：[ARI](../advanced-features/ari.md)。

### ASPM (Active State Power Management)
**主动状态电源管理** - 链路在 L0 正常工作状态下，硬件自动切入低功耗链路状态（L0s、L1）以节省功耗的机制，无需软件干预。参见：[ASPM](../power-management/aspm.md)。

### AtomicOp (Atomic Operation)
**原子操作** - PCIe 事务类型，保证跨总线的读-修改-写（FetchAdd、Swap、CAS）操作不可被打断，用于多处理器或加速器同步。参见：[Atomic Operations](../advanced-features/atomic-ops.md)。

### ATS (Address Translation Services)
**地址翻译服务** - 允许 PCIe 设备向系统 **IOMMU**（输入输出内存管理单元，负责将设备 DMA 虚拟地址映射为物理地址并做隔离）查询并缓存地址翻译结果（ATC），降低高并发 DMA 翻译延迟。

---

## B

### BAR (Base Address Register)
**基地址寄存器** - 配置空间偏移 `0x10–0x24` 的寄存器，由固件/OS 在枚举时写入物理基地址，用于将设备的寄存器或板载显存/缓冲区映射到系统内存空间（MMIO）或 I/O 空间。支持 32 位或 64 位地址、可预取（Prefetchable）或不可预取。参见：[BAR 详解](../configuration/bars.md)。

### BDF (Bus:Device.Function)
**总线:设备.功能** - PCIe 功能的唯一拓扑定位符（共 16 位路由 ID）：
- **Bus**：8 位（0–255）
- **Device**：5 位（0–31，启用 ARI 时并入 Function）
- **Function**：3 位（0–7，启用 ARI 时扩展为 8 位 0–255）
- 示例：`03:00.0`

### Bridge
**桥接器** - 连接两个 PCIe 总线段的设备，使用 Type 1 配置空间头部，包含 **Root Port**（根端口）、**Switch Port**（交换机上下行端口）和传统 PCIe-to-PCI 桥。参见：[Header 类型](../configuration/header-types.md)。

---

## C

### CAS (Compare and Swap)
**比较并交换** - 一种 PCIe AtomicOp 事务，比较目标内存值与预期值，相等则写入新值，用于无锁并发同步。

### CMA (Component Measurement and Authentication)
**组件测量和认证** - PCIe 安全架构特性，利用 **DOE** 邮箱通道和 **SPDM** 协议对设备固件、配置摘要进行密码学度量与身份认证，防止硬件/固件篡改。

### COM (Comma)
**逗号符号** - 8b/10b 编码（Gen 1/2）中的特殊控制字符（`K28.5`），用于接收端识别字节边界（比特对齐）及组成训练序列。

### Completion (Cpl)
**完成包** - TLP 类型，作为对 Non-Posted 请求的响应：
- **Cpl**：不带数据的完成（如写 I/O 或配置写响应、读出错响应）
- **CplD**：带数据的完成（如 Memory Read 返回的数据）
- **CplLk / CplDLk**：锁定完成（传统 PCI 兼容用途，现代 PCIe 已基本废弃）

### Configuration Space
**配置空间** - 每个 PCIe Function 独立的寄存器空间，包括 256 字节标准 PCI 兼容空间（`0x00–0xFF`）和 3840 字节 PCIe 扩展空间（`0x100–0xFFF`，共 4 KB），用于设备识别、BAR 分配与能力控制。参见：[配置空间](../configuration/config-space.md)。

### Credit
**信用** - 基于信用的流控单位，代表接收端缓冲区的可用空间，分为 **Header Credit (HDR)**（TLP 头部槽位）和 **Data Credit (DATA)**（通常以 4 DW = 16 字节为单位）。参见：[流量控制](../transaction-layer/flow-control.md)。

---

## D

### DLL (Data Link Layer)
**数据链路层** - PCIe 协议栈中间层，位于事务层与物理层之间，通过序列号、LCRC 与 Ack/Nak 重传保证单跳链路上的可靠传输，并负责流控信用交换。参见：[三层协议模型](../architecture/layering.md)。

### DLLP (Data Link Layer Packet)
**数据链路层包** - 仅在相邻两个物理端口间传递的 6 字节（含 16 位 CRC）控制包，不跨越 Switch 路由，用于 Ack/Nak、流控（InitFC/UpdateFC）和电源管理握手。参见：[DLLP 格式](../data-link-layer/dllp-format.md)。

### DMA (Direct Memory Access)
**直接内存访问** - 允许 PCIe 设备（在配置空间 Command 寄存器开启 Bus Master Enable 后）不经过 CPU 搬运，直接发起 Memory Read/Write TLP 读写主机系统内存的机制。

### DOE (Data Object Exchange)
**数据对象交换** - PCIe 扩展能力（Cap ID `0x002E`），在配置空间中提供标准化的邮箱（Mailbox）接口，用于主机与设备交换结构化安全数据对象（如 SPDM 认证报文、IDE 密钥协商、CXL CDAT）。参见：[DOE](../advanced-features/doe.md)。

### Downstream Port
**下行端口** - Root Complex 或 Switch 面向下游设备（远离 CPU 方向）的端口，如 Root Port 和 Switch Downstream Port。

### DPC (Downstream Port Containment)
**下游端口隔离** - PCIe 扩展能力，当下游链路发生严重不可纠正错误时，由下行端口自动切断链路并隔离故障，防止错误扩散导致整机宕机，并支持热恢复。

### D-State (Device Power State)
**设备电源状态** - 设备级功耗状态：
- **D0**：全速工作状态
- **D1 / D2**：可选中间省电状态
- **D3Hot**：深度省电，主电源 VCC 仍在，可通过配置空间唤醒
- **D3Cold**：最深省电，主电源切断，仅保留辅助电源 VAUX（若支持 PME 唤醒）

### DW (Double Word)
**双字** - 32 位（4 字节）数据单位，PCIe TLP 头部与长度字段均以 DW 为基本计量单位。

---

## E

### ECAM (Enhanced Configuration Access Mechanism)
**增强配置访问机制** - 将整个 PCIe 拓扑中每个 BDF 的 4 KB 配置空间线性映射到主机物理内存空间的机制，CPU 通过普通内存读写即可访问 `0x100–0xFFF` 扩展配置空间。

### ECRC (End-to-End CRC)
**端到端 CRC** - 可选的 32 位校验码，由发送端事务层生成并附在 TLP 末尾，跨越整个路径（包括所有 Switch）直达最终接收端，用于检测交换机内部或数据通路损坏。参见：[ECRC](../transaction-layer/ecrc.md)。

### EDB (End Bad)
**错误结束** - 8b/10b 编码中的特殊定界符，标记一个在传输途中被截断或已知无效的 TLP。

### EIEOS (Electrical Idle Exit Ordered Set)
**电气空闲退出有序集** - 物理层发送的特定脉冲序列，用于通知对端接收器链路即将退出电气空闲（Electrical Idle）状态并准备同步。

### Endpoint (EP)
**端点** - PCIe 拓扑树的叶子节点（如 NVMe SSD、GPU、NIC），使用 Type 0 配置空间头部，包含标准 Endpoint 和集成在 Root Complex 内部的 RCiEP。

### Equalization
**均衡** - Gen 3（8 GT/s）及以上高速链路必需的物理层信号调理技术，通过调整发送端预加重/去加重（Tx Preset）和接收端 CTLE/DFE 滤波器补偿高频信道损耗。参见：[链路均衡](../physical-layer/link-equalization.md)。

---

## F

### FCP (Flow Control Protocol Error)
**流控制协议错误** - AER 中的不可纠正错误，表示接收到的流控更新违背了协议约定。

### FEC (Forward Error Correction)
**前向纠错** - PCIe 6.0 在 PAM4 调制下引入的轻量级低延迟纠错码，可在无需重传的情况下自动纠正链路传输中的随机比特误码。

### Flit (Flow Control Unit)
**流控单元** - PCIe 6.0 引入的固定长度（256 字节，含 242B 负载 + 8B CRC + 6B FEC）物理/链路传输帧，取代以往变长 TLP 成帧方式以适配 PAM4 与低延迟 FEC。

### FLR (Function Level Reset)
**功能级复位** - 通过写入 Device Control 寄存器 bit 15 触发的软复位，仅复位单个 Function（如单个 VF 或多功能卡中的一个功能），不影响同设备其他 Function 或链路状态。

### Flow Control (FC)
**流控制** - 基于 Credit 的点对点流量控制机制，确保发送方仅在确认对端有足够缓冲区时才发送 TLP，避免硬件丢包。参见：[流量控制](../transaction-layer/flow-control.md)。

### Fmt (Format)
**格式** - TLP 头部字段，指示头部长度（3 DW 或 4 DW）及是否携带数据负载。

---

## G

### Gen (Generation)
**代数** - PCIe 协议代际与单 Lane 速率等级：Gen 1 (2.5 GT/s)、Gen 2 (5.0 GT/s)、Gen 3 (8.0 GT/s)、Gen 4 (16.0 GT/s)、Gen 5 (32.0 GT/s)、Gen 6 (64.0 GT/s)。参见：[速率与编码](../physical-layer/speed-encoding.md)。

### GT/s (Giga Transfers per second)
**千兆传输/秒** - 衡量物理层每秒信号跳变/符号传输次数的速率单位（含编码与纠错开销，不等同于有效数据吞吐率 GB/s）。

---

## H

### Hot-Plug
**热插拔** - 在系统不断电运行的情况下动态插入或拔出 PCIe 设备的能力，包括原生 PCIe Hot-Plug 与基于 ACPI 的热插拔。参见：[热插拔](../advanced-features/hot-plug.md)。

---

## I

### IDE (Integrity and Data Encryption)
**完整性和数据加密** - PCIe 5.0 ECN / 6.0 规范引入的硬件级安全特性（Cap ID `0x0030`），对链路上的 TLP 进行实时 AES-GCM 加密、完整性校验与防重放保护，抵御物理总线窃听与篡改。参见：[IDE](../advanced-features/ide.md)。

### InitFC (Initialize Flow Control)
**初始化流控制** - 链路进入 L0 前数据链路层发送的 DLLP（分 InitFC1 和 InitFC2 两阶段），用于通告各虚拟通道的初始 Credit 配额。参见：[流控初始化](../data-link-layer/fc-init.md)。

### INTx
**传统中断** - 模拟传统 PCI 四条共享中断线（INTA#–INTD#）的虚拟边带机制（通过 Assert/Deassert Message TLP 传递），已被 MSI/MSI-X 取代。参见：[INTx](../interrupts/intx.md)。

### IOMMU (Input-Output Memory Management Unit)
**输入输出内存管理单元** - 位于 Root Complex 中的硬件单元（如 Intel VT-d、AMD-Vi、ARM SMMU），负责将设备发起的 DMA 地址翻译为宿主物理内存地址，并实现虚拟机/进程间的内存访问隔离。

---

## L

### Lane
**通道** - PCIe 最基本的物理传输单元，由一组发送差分对（TX+/TX-）和一组接收差分对（RX+/RX-）共 4 根信号线组成，支持全双工通信。

### LCRC (Link CRC)
**链路 CRC** - 数据链路层在每个 TLP 末尾附加的 32 位校验码，用于逐跳检测链路传输误码；若校验失败接收方将回复 NAK 触发重传。参见：[LCRC](../data-link-layer/lcrc.md)。

### Link & Link Width
**链路与链路宽度** - 由 1 至 32 条并行 Lane 组成的点对点物理连接；常见宽度为 x1、x2、x4、x8、x16（规范还定义 x12、x32）。参见：[链路宽度](../physical-layer/link-width.md)。

### L-State (Link Power State)
**链路电源状态** - 物理链路功耗状态：
- **L0**：正常工作状态
- **L0s**：单向快速省电状态（仅 Tx 进入电气空闲）
- **L1**：双向深度省电状态（含 **L1.1 / L1.2** 子状态进一步关闭时钟与模拟电路）
- **L2**：辅助电源待机状态（主电源可关闭）
- **L3**：完全断电关闭状态

### LTR (Latency Tolerance Reporting)
**延迟容忍报告** - PCIe 扩展能力（Cap ID `0x0018`），允许 Endpoint 向 Root Complex 通告自身可容忍的最大读写延迟，帮助平台协调更深度的 L1.2 节能或 CPU 低功耗状态。

### LTSSM (Link Training and Status State Machine)
**链路训练和状态状态机** - 物理层核心控制状态机，管理链路从上电探测（Detect）、极性与速率协商（Polling/Configuration）、正常工作（L0）、错误恢复（Recovery）到低功耗及复位的全生命周期。参见：[LTSSM](../physical-layer/ltssm.md)。

---

## M

### Max Payload Size (MPS)
**最大负载大小** - 单个 TLP 允许携带的最大数据字节数（128、256、512、1024、2048 或 4096 字节），由路径上所有设备的最小支持值决定。

### Max Read Request Size (MRRS)
**最大读请求大小** - 单个 Memory Read 请求一次最多可读取的字节数（128–4096 字节），设备可将大读请求拆分为多个不超过 MPS 的 Completion 返回。

### MMIO (Memory-Mapped I/O)
**内存映射 I/O** - CPU 通过读写映射到 BAR 物理地址的内存区域来访问 PCIe 设备内部控制寄存器或片上缓冲区的机制（相对于传统 Port I/O）。

### MRd / MWr (Memory Read / Memory Write)
**内存读 / 内存写** - 最核心的两类 TLP：MRd 为 Non-Posted（需返回 CplD），MWr 为 Posted（无需响应）。参见：[事务类型](../transaction-layer/transaction-types.md)。

### MSI / MSI-X (Message Signaled Interrupts)
**消息信号中断** - 设备通过向特定内存地址发送 Memory Write TLP 来触发 CPU 中断的现代中断机制；MSI 最多支持 32 个向量，**MSI-X** 最多支持 2048 个独立配置的向量。参见：[MSI](../interrupts/msi.md) · [MSI-X](../interrupts/msix.md)。

---

## N

### NAK (Negative Acknowledge)
**否定确认** - 数据链路层 DLLP，当接收端检测到 LCRC 错误或序列号不连续时发送，通知发送端从 Retry Buffer 重传未确认的 TLP。

### Non-Posted / Posted
**非 Posted / Posted 事务** - **Posted**（如 MWr、Msg）发完即止，无需应答；**Non-Posted**（如 MRd、I/O、Config 读写）必须等待对端返回 Completion。

### NTB (Non-Transparent Bridge)
**非透明桥** - 隔离两个独立 Root Complex 地址域的特殊 PCIe 桥接器，用于多主机系统（如双控存储服务器）之间高速共享内存与通信。

---

## O

### Ordered Set (OS)
**有序集** - 物理层生成的特殊控制符号序列（不经数据链路层），用于链路训练（**TS1/TS2**）、时钟频偏补偿（**SKP**）以及退出电气空闲（**EIEOS**）。

---

## P

### P2P (Peer-to-Peer)
**点对点** - 同一个 PCIe 拓扑下两个 Endpoint（如 GPU 与 NVMe SSD）不经过主机内存直接通过 Switch 或 Root Complex 互传数据的通信方式。

### PAM4 (Pulse Amplitude Modulation 4-level)
**4 电平脉冲幅度调制** - PCIe 6.0 采用的信号调制方式，用 4 个电压电平在一个符号周期内编码 2 个比特（相比 Gen 1–5 的 NRZ 翻倍比特率）。

### PASID (Process Address Space ID)
**进程地址空间 ID** - TLP 前缀中携带的 20 位进程标识，配合 ATS 和 PRI 使多个用户态进程可直接共享同一个物理 PCIe 设备（Shared Virtual Memory, SVM）。

### PF / VF (Physical Function / Virtual Function)
**物理功能 / 虚拟功能** - SR-IOV 架构下的两类功能：PF 是具备完整配置管理权限的物理设备；VF 是由 PF 派生、可直接透传给虚拟机使用的轻量级虚拟设备。

### PHY (Physical Layer)
**物理层** - PCIe 三层协议栈最底层，负责差分信号收发、串并转换、编码（8b/10b、128b/130b、PAM4）、加扰、时钟补偿及 LTSSM 链路训练。参见：[物理层概述](../physical-layer/overview.md)。

### Poisoned TLP
**中毒 TLP** - 头部 EP（Error Poisoned）位置 1 的 TLP，表示其携带的数据已损坏（如端到端 ECC 错误），供接收端隔离处理而非直接静默使用。

### PRI (Page Request Interface)
**页请求接口** - 允许支持 ATS 的 PCIe 设备在遇到 IOMMU 缺页时主动向主机发送 Page Request，触发操作系统按需分配并映射内存页（类似设备的缺页中断）。

### PTM (Precision Time Measurement)
**精确时间测量** - 跨 Root Complex、Switch 和 Endpoint 硬件同步纳秒级时间基准的协议，消除软件时间戳和链路传播延迟抖动。参见：[PTM](../advanced-features/ptm.md)。

---

## R

### RBAR (Resizable BAR)
**可调整大小 BAR** - 扩展能力（Cap ID `0x0015`），允许设备向系统声明多种可选 BAR 空间大小（如 GPU 将 256 MB BAR 扩展为 16 GB+ 以直接映射全部显存）。

### RC (Root Complex) / RP (Root Port)
**根复合体 / 根端口** - **RC** 是 PCIe 拓扑的根节点，连接 CPU、内存与 PCIe 子系统；**RP** 是 RC 上向下连接 Switch 或 Endpoint 的下行端口。参见：[拓扑结构](../introduction/topology.md)。

### Relaxed Ordering (RO)
**宽松排序** - TLP 属性位，允许无强顺序依赖的请求或完成包超越前面的包，以提升高并发吞吐性能。参见：[排序规则](../transaction-layer/ordering.md)。

### Requester ID (RID)
**请求者 ID** - TLP 头部中的 16 位 BDF 标识符，用于标明请求发起者，使 Completion 能够准确路由回原设备，并在虚拟化中供 IOMMU 识别设备身份。

### Retimer
**重定时器** - 主动式物理层信号中继芯片（PCIe 4.0/5.0 引入），内置时钟数据恢复（CDR）与均衡电路，用于延长长走线或背板链路距离，对上层协议完全透明。参见：[Retimer](../physical-layer/retimer.md)。

### Retry Buffer
**重试缓冲区** - 发送端数据链路层的缓存队列，暂存所有已发送但尚未收到 ACK 的 TLP；收到 NAK 或超时后即从此缓冲区重传。参见：[重传缓冲区](../data-link-layer/retry-buffer.md)。

---

## S

### Scrambling
**加扰** - 物理层在发送前对数据流进行的伪随机化处理（通过 LFSR 异或），用于打散长连 0/1 序列、降低电磁干扰（EMI）并维持时钟恢复所需的跳变密度。

### Sequence Number
**序列号** - 数据链路层为每个 TLP 分配的 12 位递增编号（0–4095），用于接收端检错排序和 Ack/Nak 确认。

### SKP (Skip)
**跳过符号** - 物理层周期性插入的有序集，用于吸收发送端与接收端参考时钟之间微小频偏（如 ±300 ppm）引起的缓冲区溢出或下溢。

### SPDM (Security Protocol and Data Model)
**安全协议和数据模型** - 由 DMTF 制定的设备安全认证标准，在 PCIe 中通常运行在 DOE 之上，负责设备身份证书验证、固件度量与密钥交换。

### SR-IOV (Single Root I/O Virtualization)
**单根 I/O 虚拟化** - 允许单个物理 PCIe 设备在硬件层面虚拟出多个 VF 并直接分配给虚拟机，实现接近原生的 I/O 性能。参见：[SR-IOV](../advanced-features/sriov.md)。

### Switch
**交换机** - 扇出 PCIe 端口的桥接设备，拥有 1 个连接 Root Complex 的上行端口（Upstream Port）和多个连接下级设备的下行端口（Downstream Port），支持设备间 P2P 路由。

---

## T

### Tag
**标签** - Non-Posted 请求头部中的 8 位（或启用 Extended Tag 后的 10/14 位）字段，用于设备并发发出多个未完成请求时，将返回的 Completion 与原始请求一一匹配。

### TC (Traffic Class) / VC (Virtual Channel)
**流量类别 / 虚拟通道** - **TC**（TC0–TC7）是 TLP 携带的服务优先级标签；**VC**（VC0–VC7，VC0 必选）是链路上拥有独立流控 Credit 与缓冲区的逻辑通道，两者映射实现 QoS 隔离。参见：[虚拟通道](../transaction-layer/virtual-channels.md)。

### TL (Transaction Layer) / TLP (Transaction Layer Packet)
**事务层 / 事务层包** - PCIe 协议栈顶层及它构建的数据包，负责将上层读写/中断请求封装为带有路由、排序与流控属性的 TLP（含 Header、可选 Data Payload 和可选 ECRC）。参见：[TLP 格式](../transaction-layer/tlp-format.md)。

### TPH (TLP Processing Hints)
**TLP 处理提示** - 扩展能力（Cap ID `0x0017`），允许设备在写内存 TLP 中附带 Steering Tag，提示主机将 DMA 数据直接预填充至指定 CPU 核心的 Cache（DDIO），降低访存延迟。参见：[TPH](../advanced-features/tph.md)。

### Type 0 / Type 1 Header
**类型 0 / 类型 1 配置头** - Endpoint 设备使用 **Type 0**（含 6 个 BAR）；Bridge/Switch/Root Port 设备使用 **Type 1**（含 2 个 BAR 及 Primary/Secondary/Subordinate Bus 号）。

---

## U

### UpdateFC
**更新流控制** - 链路正常工作期间数据链路层周期性发送的 DLLP，用于向发送端归还已释放的缓冲区 Credit。

### UR (Unsupported Request)
**不支持的请求** - 当设备收到无法识别或地址越界的 Non-Posted 请求时，在 Completion 状态字段返回的错误码，同时会记录到 AER。

---

## V

### VPD (Vital Product Data)
**重要产品数据** - 存储在设备配置空间 VPD Capability 指向区域中的只读/可读写信息（如序列号、部件号、厂商自定义资产标签）。

---

## 常用缩写速查

| 缩写 | 全称 | 中文 |
|------|------|------|
| ACK / NAK | Acknowledge / Negative Acknowledge | 确认 / 否定确认 |
| ACS | Access Control Services | 访问控制服务 |
| AER | Advanced Error Reporting | 高级错误报告 |
| ARI | Alternative Routing-ID Interpretation | 备选路由 ID 解释 |
| ASPM | Active State Power Management | 主动状态电源管理 |
| ATS | Address Translation Services | 地址翻译服务 |
| BAR | Base Address Register | 基地址寄存器 |
| BDF | Bus:Device.Function | 总线:设备.功能 |
| CMA | Component Measurement and Authentication | 组件测量与认证 |
| DLL / DLLP | Data Link Layer / Packet | 数据链路层 / 数据链路层包 |
| DMA | Direct Memory Access | 直接内存访问 |
| DOE | Data Object Exchange | 数据对象交换 |
| DPC | Downstream Port Containment | 下游端口隔离 |
| DW | Double Word | 双字（32 位） |
| ECAM | Enhanced Configuration Access Mechanism | 增强配置访问机制 |
| ECRC / LCRC | End-to-End CRC / Link CRC | 端到端 CRC / 链路 CRC |
| EP | Endpoint | 端点 |
| FEC | Forward Error Correction | 前向纠错 |
| FLR | Function Level Reset | 功能级复位 |
| GT/s | Giga Transfers per second | 千兆传输/秒 |
| IDE | Integrity and Data Encryption | 完整性和数据加密 |
| IOMMU | Input-Output Memory Management Unit | 输入输出内存管理单元 |
| LTR | Latency Tolerance Reporting | 延迟容忍报告 |
| LTSSM | Link Training and Status State Machine | 链路训练和状态状态机 |
| MMIO | Memory-Mapped I/O | 内存映射 I/O |
| MPS / MRRS | Max Payload Size / Max Read Request Size | 最大负载大小 / 最大读请求大小 |
| MSI / MSI-X | Message Signaled Interrupts (Extended) | 消息信号中断（扩展） |
| NTB | Non-Transparent Bridge | 非透明桥 |
| P2P | Peer-to-Peer | 点对点 |
| PAM4 | Pulse Amplitude Modulation 4-level | 4 电平脉冲幅度调制 |
| PASID | Process Address Space ID | 进程地址空间 ID |
| PF / VF | Physical Function / Virtual Function | 物理功能 / 虚拟功能 |
| PHY | Physical Layer | 物理层 |
| PRI | Page Request Interface | 页请求接口 |
| PTM | Precision Time Measurement | 精确时间测量 |
| RC / RP | Root Complex / Root Port | 根复合体 / 根端口 |
| RID | Requester ID | 请求者 ID |
| SPDM | Security Protocol and Data Model | 安全协议与数据模型 |
| SR-IOV | Single Root I/O Virtualization | 单根 I/O 虚拟化 |
| TC / VC | Traffic Class / Virtual Channel | 流量类别 / 虚拟通道 |
| TL / TLP | Transaction Layer / Packet | 事务层 / 事务层包 |
| TPH | TLP Processing Hints | TLP 处理提示 |
| VPD | Vital Product Data | 重要产品数据 |

---

## 关联与引申概念

- **入门与全局导航**：[Wiki 主页](../README.md) · [PCIe 简介](../introduction/overview.md) · [常见问题 (FAQ)](faq.md)
- **寄存器与规范查询**：[寄存器速查表](register-quick-ref.md) · [规范索引](spec-index.md) · [配置空间详解](../configuration/config-space.md)
- **三层协议核心**：[三层协议模型](../architecture/layering.md) · [TLP 格式](../transaction-layer/tlp-format.md) · [LTSSM 状态机](../physical-layer/ltssm.md)
