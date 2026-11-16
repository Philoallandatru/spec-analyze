# PCIe 拓扑结构

**一句话总结**：PCIe 拓扑是一棵以 CPU 内的 **Root Complex（根复合体）** 为树根、以 **Switch（交换机）** 为分叉节点、以各类外设 **Endpoint（端点）** 为树叶的树形网络，树上每条连线都是点对点专属链路。

> **直观类比**：PCIe 拓扑就像公司的组织架构或快递分拨网络——**Root Complex** 是总公司总部（对接 CPU 与内存），**Root Port** 是总部的对外窗口，**Switch** 是区域分拨中心（把一条主干道拆分成多条支线），而 **Endpoint**（显卡、SSD、网卡）则是最终干活的末端网点。

---

## 拓扑中的四大核心角色

### 1. Root Complex（根复合体，简称 RC）

RC 是整个 PCIe 树形拓扑的起点，通常直接集成在 CPU（或 SoC）内部，负责在 **CPU/系统内存域** 与 **PCIe I/O 域** 之间充当桥梁：
- 生成配置请求（扫描和初始化下游设备）、内存读写请求与 I/O 请求。
- 接收来自下游设备的 DMA（直接内存访问）请求和中断消息。
- 内部包含多个 **Root Port（根端口）**，每个 Root Port 引出一条独立的 PCIe 链路。

```
┌─────────────────────────────────┐
│             CPU / SoC           │
│  ┌───────────────────────────┐  │
│  │     Root Complex (RC)     │  │
│  │   ┌────┐  ┌────┐  ┌────┐  │  │
│  │   │ RP │  │ RP │  │ RP │  │  │  RP = Root Port
│  └───┴─┬──┴──┴─┬──┴──┴─┬──┴──┘  │
└────────┼───────┼───────┼────────┘
         │       │       │
      ┌──▼──┐ ┌──▼──┐ ┌──▼───┐
      │ GPU │ │ SSD │ │Switch│
      └─────┘ └─────┘ └──────┘
```

### 2. Switch（交换机）

当主板需要连接比 CPU 原生 Lane 更多的设备时，就需要 **PCIe Switch** 来扩展扇出（Fan-out）：
- **结构**：内部由 1 个 **Upstream Port（上行端口，朝向 RC）** 和多个 **Downstream Port（下行端口，朝向下游设备）** 组成。在软件视角下，Switch 表现为一组相互连接的虚拟 PCI-to-PCI Bridge。
- **路由与转发**：根据 TLP 包头中的地址或 BDF 路由，将上行/下行数据包精准转发到目标端口。
- **透明 vs 非透明（NTB）**：标准透明 Switch 对操作系统完全可见；**NTB（Non-Transparent Bridge，非透明桥）** 则用于隔离两个独立主机（如双控存储服务器之间互连），各自拥有独立的地址空间。

### 3. Endpoint（端点设备，简称 EP）

Endpoint 是 PCIe 树形拓扑的叶子节点，即真正提供业务功能的终端设备（显卡、NVMe SSD、网卡等）。它只能发起或响应事务，**不能**将事务转发给其他下游链路。常见分类如下：
- **PCIe Endpoint（Type 0）**：最常见的标准设备（GPU、NVMe SSD）。
- **Root Complex Integrated Endpoint（RCiEP）**：直接集成在 CPU/SoC 内部的设备（如核显、内置加密引擎），无需经过物理 PCIe 插槽链路。
- **Legacy Endpoint**：为兼容早期 PCI 软件（如使用 I/O 空间或传统中断）保留的过渡型端点。

### 4. Bridge（桥接器）

Bridge 用于连接两条不同的 PCI/PCIe 总线段，并在配置空间中使用 **Type 1 配置头**（记录上游总线号 `Primary`、直连下游总线号 `Secondary` 以及下游子树最大总线号 `Subordinate`）。Root Port、Switch 的各个端口、以及 PCIe-to-PCI 转换芯片，本质上都属于 Bridge。

---

## 典型系统拓扑示例

### 桌面 / 工作站拓扑（直连为主）

```
                ┌─────────────────────┐
                │     Root Complex    │
                └──┬──────┬───────┬───┘
              x16  │      │ x4    │ x1
                   ▼      ▼       ▼
                ┌─────┐ ┌────┐ ┌─────┐
                │ GPU │ │NVMe│ │Wi-Fi│
                └─────┘ └────┘ └─────┘
```

### 服务器拓扑（带 Switch 扩展与多路 RC）

高端服务器通常有多个 CPU（每个 CPU 自带一个或多个 Root Complex，形成独立 PCIe 域），并通过 Switch 挂载大量 NVMe SSD 或加速器：

```
        ┌───────────────────┐
        │   CPU (RC 0)      │
        └───┬──────────┬────┘
        x16 │          │ x8 (上行带宽)
            ▼          ▼
        ┌───────┐  ┌──────────────────────────┐
        │  GPU  │  │       PCIe Switch        │
        └───────┘  └──┬─────────┬─────────┬───┘
                  x4  │     x4  │     x4  │
                      ▼         ▼         ▼
                  ┌──────┐  ┌──────┐  ┌───────┐
                  │NVMe 1│  │NVMe 2│  │ 网卡  │
                  └──────┘  └──────┘  └───────┘
```

> **带宽超卖（Oversubscription）注意**：上图中 Switch 上行仅 `x8`，而下游 3 个设备合计 `x12`。若三者同时满载向 CPU 写数据，就会在上行链路发生带宽竞争。

---

## 设备如何被发现与编号？（BDF 与枚举简述）

系统上电时，固件（BIOS/UEFI）或操作系统通过**深度优先遍历**扫描整棵 PCIe 树，并为每个设备分配唯一的 **BDF 地址（Bus:Device.Function）**：
- **Bus（总线号，8 位，0–255）**：每经过一个 Bridge（Root Port 或 Switch Port），就开启一条新的逻辑总线号。
- **Device（设备号，5 位，0–31）**：PCIe 是点对点连接，一条链路上通常只有 1 个设备，因此下游总线上的 Device 号几乎总是 `0`。
- **Function（功能号，3 位，0–7）**：单张物理网卡或显卡内部可包含多个逻辑功能（如 `00.0` 是显卡核心、`00.1` 是 HDMI 音频控制器）。若启用 **ARI（Alternative Routing-ID Interpretation）**，Device 与 Function 可合并为 8 位，支持单设备最多 256 个 Function。

在 Linux 中，用一条命令即可直观看到整棵拓扑树：

```bash
$ lspci -tv
-[0000:00]-+-00.0  Intel Corporation Host Bridge
           +-01.0-[01]----00.0  NVIDIA Corporation GPU          # BDF = 01:00.0
           +-02.0-[02-04]----00.0  Switch Upstream Port
           |                 +-00.0-[03]----00.0  NVMe SSD      # BDF = 03:00.0
           |                 \-01.0-[04]----00.0  10GbE NIC     # BDF = 04:00.0
           \-03.0-[05]----00.0  Intel Wi-Fi Adapter             # BDF = 05:00.0
```

---

## 拓扑约束与性能影响

1. **深度与数量上限**：单域总线号为 0–255（最多 256 条总线）；多 CPU 服务器会引入多个 PCIe Domain（如 `0000:`、`0001:`）突破 256 条总线限制。
2. **直连 vs Switch 跳数**：直连 Root Port 延迟最低；每多经过一层 Switch，都会增加微小的转发延迟（百纳秒级），且共享上行带宽。
3. **P2P（Peer-to-Peer）点对点直通**：同一 Switch 下的两块设备（如两块 GPU 或 GPU 与 NVMe SSD）可以直接通过 Switch 内部交换数据，无需绕道 CPU 内存；但若启用了虚拟化隔离，通常需要 **ACS（Access Control Services）** 控制 P2P 流量必须经过 RC/IOMMU 校验。
4. **NUMA 亲和性**：在多路 CPU 服务器中，设备直连到哪颗 CPU 的 Root Complex，该 CPU 访问它就最快；跨 CPU 访问需走处理器间互联总线（如 UPI/Infinity Fabric），会显著增加延迟。

---

## 关联与引申概念

- **[PCIe 简介](overview.md)**：回顾 PCIe 的发展历程、代际速率与核心优势。
- **[设备枚举](../architecture/enumeration.md)**：深入了解系统启动时如何递归扫描总线、配置 Bridge 的 Primary/Secondary/Subordinate 寄存器以及分配 BAR 资源。
- **[地址空间管理](../architecture/address-space.md)**：理解 Root Complex 与 Switch 如何通过地址窗口（Memory Window）将 MMIO 流量精准路由到下游设备。
- **[三层协议模型](../architecture/layering.md)**：查看数据包（TLP）在经过 Switch 每一跳时，数据链路层与物理层是如何逐跳保护传输的。
