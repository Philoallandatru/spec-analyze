# PCIe 术语表

**一句话总结**：本表按字母顺序列出 PCIe 规范、硬件设计与内核驱动中最常出现的核心术语与缩写，并用通俗语言解释它们的作用与彼此关联。

---

## A

### ACS (Access Control Services，访问控制服务)
用于控制虚拟化环境中**点对点（P2P）**事务的路由策略。启用 ACS 后，Switch 会强制将下游设备的流量先发往 Root Complex，经 **IOMMU** 校验后再转发，防止虚拟机之间越权互访。

### AER (Advanced Error Reporting，高级错误报告)
PCIe 扩展能力结构之一，用于精细记录并上报链路与事务错误。它将错误分为三类：**可纠正错误**（如硬件重传恢复的 LCRC 错误）、**不可纠正非致命错误**（部分事务失败但链路仍可用）和**不可纠正致命错误**（链路失效需复位）。

### ARI (Alternative Routing-ID Interpretation，替代路由 ID 解释)
传统 BDF 用 5 位表示 Device、3 位表示 Function（单设备最多 8 个 Function）。ARI 将两者合并为 8 位 Function 号，使单个物理设备（常见于 **SR-IOV**）可扩展至 **256 个功能**。

### ASPM (Active State Power Management，主动状态电源管理)
PCIe 链路在设备未彻底断电时的自动节能机制，定义了从全速工作（`L0`）到浅度省电（`L0s`，微秒级唤醒）和深度省电（`L1`，微秒至毫秒级唤醒）的动态切换。可通过 `lspci -vv | grep -i aspm` 查看。

### ATS (Address Translation Services，地址转换服务)
允许 PCIe 设备在自身内部缓存 **IOMMU** 的地址转换结果（Device TLB）。发起 DMA 时可直接使用转换后的物理地址并标记“已转换”，减少对系统 IOMMU 的查询延迟。常与 **PASID**、**PRI** 配合使用。

---

## B

### BAR (Base Address Register，基地址寄存器)
位于设备配置空间（偏移 `0x10`–`0x24`，共 6 个 32 位槽位）中的关键寄存器。系统启动时，固件通过向 BAR 写入全 1 再读回掩码来探测设备所需的内存大小，随后分配一段系统物理地址写入 BAR，从而建立 **MMIO** 映射窗口。

### BDF (Bus:Device.Function，总线:设备:功能)
PCIe 设备在系统中的唯一“门牌号”，格式为 `BB:DD.F`（若包含多域则为 `DDDD:BB:DD.F`）。其中 **Bus** 占 8 位（0–255）、**Device** 占 5 位（0–31）、**Function** 占 3 位（0–7）。

### Bridge (桥接器)
连接上下游两条 PCI/PCIe 总线的转发设备，使用 **Type 1 配置头**。**Root Port**、**Switch** 的上行/下行端口以及 PCIe-to-PCI 转换芯片都属于 Bridge，负责管理总线号范围（Primary/Secondary/Subordinate）和地址转发窗口。

---

## C

### Capability (能力寄存器)
组织在配置空间中的功能链表。标准配置空间（前 256 字节）存放电源管理、**MSI/MSI-X**、**PCIe Capability** 等基础能力；扩展配置空间（256–4095 字节）存放 **AER**、**SR-IOV**、**ATS** 等高级扩展能力。

### CEM (Card Electromechanical Specification，卡电气机械规范)
定义标准 PCIe 插卡的物理尺寸、金手指引脚、供电上限与散热规范（如全高全长 FHFL、半高半长 HHHL）。

### Completion (完成包，Cpl / CplD)
**Non-Posted 事务**（如内存读、配置读写）的响应 **TLP**。常见状态包括：`SC`（成功完成）、`UR`（不支持的请求）、`CRS`（配置重试状态，设备仍在初始化）和 `CA`（完成者中止）。

### CRC (Cyclic Redundancy Check，循环冗余校验)
PCIe 用于检测传输误码的校验机制，分两级：**LCRC**（32 位，由数据链路层添加，逐跳校验）和 **ECRC**（32 位，由事务层可选添加，端到端校验）。

### CXL (Compute Express Link，计算快速链路)
构建在 PCIe 5.0/6.0 物理层之上的高速一致性互连协议。它包含 `CXL.io`（兼容 PCIe）、`CXL.cache`（设备缓存主机内存）和 `CXL.mem`（主机直接访问设备内存池），广泛用于 AI 加速与内存扩展。

---

## D

### DLLP (Data Link Layer Packet，数据链路层包)
长度固定为 6 字节的链路控制包，只在相邻两个端口之间传输，不携带应用数据。主要用于 **Ack/Nak** 确认重传、**Flow Control** 信用更新以及电源管理状态协商。

### DMA (Direct Memory Access，直接内存访问)
外设无需 CPU 逐字节参与，直接向系统内存发起读/写 TLP 的机制。包括常用于命令队列的**一致性 DMA**（Coherent）与常用于大块数据搬运的**流式 DMA**（Streaming）。

### DOE (Data Object Exchange，数据对象交换)
PCIe 5.0/6.0 引入的配置空间邮箱（Mailbox）机制，用于在主机与设备间安全传输结构化数据对象，是 **SPDM** 身份认证与 **IDE** 密钥交换的底层传输通道。

### DPC (Downstream Port Containment，下游端口隔离)
当下游链路发生不可纠正错误时，由 Root Port 或 Switch 下行端口立即切断该端口的转发，防止错误包扩散并导致整机崩溃。

---

## E

### ECAM (Enhanced Configuration Access Mechanism，增强配置访问机制)
PCIe 标准的配置空间访问方式。它将所有设备的 4KB 配置空间平铺映射到一段连续的系统内存空间中，CPU 通过普通内存读写指令即可访问任意设备的配置寄存器（基地址由 ACPI `MCFG` 表提供）。

### ECRC (End-to-End CRC，端到端循环冗余校验)
可选的 32 位事务层校验码，在发送端生成并一直保留到最终接收端才验证，用于捕获中间 Switch 内部转发或缓冲时可能发生的静默数据损坏。

### Endpoint (端点设备，EP)
PCIe 树形拓扑的末端叶子节点（如 GPU、NVMe SSD、网卡），使用 **Type 0 配置头**，不能向其他下游设备转发事务。

### Equalization (均衡)
PCIe Gen3（8 GT/s）及以上高速链路必须执行的信号调理过程。发送端通过预加重/去加重、接收端通过 CTLE/DFE 滤波器补偿 PCB 走线的高频衰减，确保眼图张开。

---

## F

### FLR (Function Level Reset，功能级复位)
允许软件单独复位设备上的某一个 **Function**，而不影响同一物理卡上的其他 Function 或相邻设备，在虚拟化（如 VF 回收）中至关重要。

### Flow Control (流控制)
PCIe 采用**基于信用（Credit）**的链路级流控：接收方通过 `InitFC` 和 `UpdateFC` **DLLP** 告知发送方自己还有多少缓冲空间；发送方只有在信用充足时才能发 **TLP**，从根本上避免接收缓冲区溢出丢包。

---

## G & H

### Gen (Generation) 与 GT/s (GigaTransfers per second)
PCIe 的代际（Gen1 至 Gen6）与物理层速率单位（每秒十亿次符号/比特传输）。计算单向有效带宽的通用公式为：`有效带宽 = GT/s × Lane数 × 编码效率 / 8`。

### Hot-Plug (热插拔)
在系统不停机状态下安全插入或拔出 PCIe 设备的能力（如服务器 U.2/E1.S NVMe 硬盘热插拔）。Linux 下可通过 `/sys/bus/pci/devices/.../remove` 与 `/sys/bus/pci/rescan` 触发软件热插拔。

---

## I

### IDE (Integrity and Data Encryption，完整性和数据加密)
PCIe 6.0 核心安全特性，在链路或端到端路径上使用 AES-GCM 算法对 TLP 进行硬件级加密与防篡改校验，抵御物理探测与中间人攻击。

### INTx (传统引脚中断)
沿袭自传统 PCI 的 4 根电平触发中断线（`INTA#`–`INTD#`）。在 PCIe 中已改为通过 **Message TLP** 模拟，因存在中断共享开销，现代设备均优先使用 **MSI/MSI-X**。

### IOMMU (Input-Output Memory Management Unit)
位于 Root Complex 侧的硬件单元（如 Intel VT-d、AMD-Vi、ARM SMMU），负责将设备 DMA 请求中的 **IOVA（I/O 虚拟地址）** 翻译为真实物理内存地址，并强制隔离不同设备的访问权限。

---

## L

### Lane (通道) 与 Link (链路)
**Lane** 是 PCIe 最小物理传输单元，由一对发送差分线（`TX+/TX-`）和一对接收差分线（`RX+/RX-`）共 4 根线组成。将 1、2、4、8 或 16 条 Lane 捆绑在一起，就构成一条 **Link（链路）**。

### LCRC (Link CRC，链路循环冗余校验)
由数据链路层附加在每个 TLP 末尾的 32 位校验码，仅在相邻两端口之间有效；经过 Switch 每一跳时都会重新校验并重新生成。

### LTSSM (Link Training and Status State Machine，链路训练与状态机)
物理层的核心控制状态机，负责链路从上电探测（`Detect`）、速率与比特锁定（`Polling`）、通道宽度协商（`Configuration`）到正常收发（`L0`）、错误恢复（`Recovery`）及低功耗休眠（`L0s/L1/L2`）的全生命周期管理。

---

## M

### MPS (Max Payload Size，最大有效载荷大小)
单个 TLP 中可携带的数据负载上限（128 字节至 4096 字节）。整条传输路径上的所有设备必须协商一致的 MPS（取路径上最小支持值）。

### MRRS (Max Read Request Size，最大读请求大小)
设备单次发起 Memory Read 请求允许读取的最大字节数（最大 4096 字节）。它不必等于 MPS——若 MRRS 大于 MPS，接收方会自动将数据拆分为多个不超过 MPS 的 **Completion TLP** 返回。

### MSI / MSI-X (Message Signaled Interrupts，消息信号中断)
PCIe 用“向特定内存地址发送 Memory Write TLP”来替代传统物理中断线。**MSI** 最多支持 32 个向量；**MSI-X** 支持最多 **2048 个独立向量**，且每个向量可独立绑定目标 CPU 核心与独立屏蔽，是高性能 NVMe 和网卡的标配。

---

## N & O

### Non-Posted vs Posted Transaction (非提交 vs 提交事务)
- **Posted（提交事务）**：发送后无需等待响应（Fire-and-Forget），如 **Memory Write** 和 **Message**，延迟极低。
- **Non-Posted（非提交事务）**：必须等待接收方返回 **Completion** 确认或数据，如 **Memory Read**、**I/O 读写**和 **Configuration 读写**。

### NTB (Non-Transparent Bridge，非透明桥)
一种隔离两个独立 Root Complex 地址空间的特殊桥接器，允许两台独立主机通过 PCIe 链路进行高带宽、低延迟的内存窗口映射通信，常用于双控高可用存储系统。

### Ordered Set (有序集)
物理层直接发送的特殊控制符号序列（不经过数据链路层和事务层），包括用于训练协商的 **TS1/TS2**、用于时钟频率差补偿的 **SKP**、以及用于进出低功耗电气空闲状态的 **EIOS/EIEOS**。

---

## P & R

### PAM4 (4-level Pulse Amplitude Modulation，4 电平脉冲幅度调制)
PCIe 6.0 采用的信号调制技术。与传统 NRZ（每符号 1 比特，高低 2 电平）不同，PAM4 用 4 个电压等级表示 `00/01/10/11`，使单符号传输能力翻倍（2 bits/symbol），从而实现 64 GT/s。

### PASID (Process Address Space ID，进程地址空间标识符) 与 PRI (Page Request Interface)
**PASID** 是附加在 TLP 前缀中的 20 位进程标识，使同一块加速卡能同时为多个用户进程直接操作各自的虚拟内存（SVM，共享虚拟内存）；**PRI** 则允许设备在页表缺页时向操作系统请求调页。

### PTM (Precision Time Measurement，精确时间测量)
PCIe 3.1 引入的跨链路纳秒级时间同步协议，使分布式网卡、音视频采集卡与 CPU 之间保持高精度时钟同步。

### RCB (Read Completion Boundary，读完成边界)
规定响应端在拆分大块 Memory Read 返回的多个 Completion 时，每次拆分必须对齐到 64 字节（Root Complex 默认）或 128 字节边界，简化接收端重组逻辑。

### Root Complex (RC，根复合体) 与 Root Port (RP，根端口)
**RC** 是集成在 CPU/SoC 内部的 PCIe 根节点，连接 CPU、内存与 PCIe 树；**RP** 是 RC 上引出下游 PCIe 链路的逻辑端口（表现为 Type 1 Bridge）。

---

## S, T, V

### SPDM (Security Protocol and Data Model，安全协议与数据模型)
通过 **DOE** 运行的设备安全认证协议，用于在系统启动或设备接入时验证硬件身份证书并协商 **IDE** 加密密钥，防止恶意硬件植入。

### SR-IOV (Single Root I/O Virtualization，单根 I/O 虚拟化)
允许单个物理 PCIe 设备（**PF**, Physical Function）在硬件层面虚拟出多个轻量级虚拟设备（**VF**, Virtual Function），直接透传给不同虚拟机使用，兼顾接近原生的 I/O 性能与多租户隔离。

### Switch (交换机)
包含 1 个上行端口（Upstream Port）和多个下行端口（Downstream Port）的 PCIe 扩展芯片，用于增加系统可挂载的设备数量。

### TC (Traffic Class) 与 VC (Virtual Channel)
**TC**（TC0–TC7）是写入 TLP 头部的 3 位流量优先级标签；**VC**（VC0–VC7）是链路上拥有独立流控缓冲区的逻辑通道。所有设备强制支持 `VC0`，可将不同 TC 映射到不同 VC 实现硬件级 QoS。

### TLP (Transaction Layer Packet，事务层数据包)
PCIe 携带实际读写请求、响应与中断消息的核心数据包，由 Header（3 或 4 DW）、可选 Data Payload（最多 1024 DW = 4KB）和可选 ECRC 组成。

---

## 关联与引申概念

- **[PCIe 简介](overview.md)** & **[拓扑结构](topology.md)**：将 `RC`、`Switch`、`EP`、`BDF` 放入系统全景图中理解。
- **[三层协议模型](../architecture/layering.md)** & **[数据包类型](../architecture/packet-types.md)**：深入理解 `TLP`、`DLLP`、`Ordered Set`、`LCRC/ECRC` 与 `Flow Control` 的交互。
- **[设备枚举](../architecture/enumeration.md)** & **[地址空间管理](../architecture/address-space.md)**：查看 `BAR`、`ECAM`、`IOMMU` 与 `Bridge` 窗口在启动阶段是如何配置生效的。
