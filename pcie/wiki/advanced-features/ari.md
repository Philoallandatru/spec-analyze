# PCIe 备选路由 ID（ARI - Alternative Routing-ID Interpretation）

> **一句话总结**：ARI 是 PCIe 2.0 引入的"门牌号扩编方案"，它把传统 BDF 寻址里的 5 位 Device 号和 3 位 Function 号合并成一个 8 位 Function 号，让单个物理设备能挂载的功能从最多 8 个暴增到 256 个。

在传统 PCIe 规则里，一条总线就像一条街道，最多有 32 栋楼（Device 0–31），每栋楼最多有 8 个房间（Function 0–7）。但像现代 SR-IOV 网卡或 NVMe SSD 这样的"超级大厦"，往往只占 1 栋楼的位置，却需要上百个虚拟功能（VF）房间。ARI 的做法很简单：不再区分楼栋号和房间号，把原本的 `5 + 3 = 8` 位直接合并成 `8` 位房间号，于是这一栋楼里就能编出 `0–255` 共 256 个功能。

---

## 从传统 BDF 到 ARI 寻址

PCIe 的 Requester ID / Completer ID 始终是 **16 位**，ARI 并没有改变 TLP 包头格式，只是改变了上下游对低 8 位的**解释方式**：

```text
传统 BDF 模型（每设备最多 8 个功能，Func 0-7）：
 15           8 7        3 2      0
┌──────────────┬──────────┬────────┐
│  Bus Number  │  Device  │  Func  │
│   (8 bits)   │ (5 bits) │(3 bits)│
└──────────────┴──────────┴────────┘
     0-255         0-31       0-7

ARI 寻址模型（设备固定为 Dev 0，最多 256 个功能，Func 0-255）：
 15           8 7                 0
┌──────────────┬───────────────────┐
│  Bus Number  │ ARI Function Num  │
│   (8 bits)   │     (8 bits)      │
└──────────────┴───────────────────┘
     0-255            0-255
```

> **提示**：在 `lspci` 等工具输出中，高编号的 ARI Function 仍会按传统 `Bus:Dev.Func` 格式显示（例如 Function 255 会显示为 `01:1f.7`，因为 `0x1F × 8 + 7 = 255`），但在硬件内部它们都属于同一台物理设备的连续 8 位 Function 编号。

---

## ARI Capability 与上游转发（ARI Forwarding）

要让 ARI 生效，**端点设备**和**直接相连的上游端口（Root Port 或 Switch Downstream Port）**必须同时支持并启用 ARI：

```text
CPU / Root Complex
  │
Root Port（必须开启 Device Control 2 的 ARI Forwarding Enable）
  │
[Switch Downstream Port]（若经过 Switch，同样必须开启 ARI Forwarding Enable）
  │
Endpoint 设备（必须位于该总线的 Device 0，且实现 ARI Extended Capability 0x000E）
```

### 1. 端点侧：ARI Extended Capability（ID `0x000E`）

位于设备每个 Function 的扩展配置空间中：

| 偏移 | 寄存器 | 关键字段与作用 |
| :--- | :--- | :--- |
| `0x00` | Extended Capability Header | Capability ID = `0x000E` |
| `0x04` | **ARI Capability Register** (16-bit) | Bits `15:8` 为 **Next Function Number**（指向下一个有效 Function 编号，`0` 表示链表结束）；Bit 0/1 分别表示是否支持 MFVC / ACS Function Groups |
| `0x06` | **ARI Control Register** (16-bit) | 控制 MFVC / ACS Function Groups 启用及只读 Function Group 字段 |

**链表式枚举过程**：传统设备需要软件逐个探测 Function 0–7；ARI 设备则通过 `Next Function Number` 构成单向链表，操作系统从 Function 0 开始读取该字段直接跳到下一个有效功能，直到 `Next Function Number == 0` 结束，大幅加速大量 VF 的枚举。

### 2. 上游端口侧：ARI Forwarding Enable

位于 Root Port 或 Switch Downstream Port 的 **PCIe Capability** 中：

- **Device Capabilities 2（偏移 `0x24`）Bit 5**：`ARI Forwarding Supported`（只读，硬件是否支持将低 8 位视为 8 位 Function 号）。
- **Device Control 2（偏移 `0x28`）Bit 5**：`ARI Forwarding Enable`（软件置 `1` 后，上游端口才不会把 Dev != 0 的传统配置请求或 TLP 当成不存在的设备丢弃）。

---

## ARI 与 SR-IOV 的协同

SR-IOV（单根 I/O 虚拟化）是 ARI 最主要的使用场景：

| 对比维度 | 未启用 ARI | 启用 ARI |
| :--- | :--- | :--- |
| **单个 PF 可创建的最大 VF 数** | 最多 **7 个**（Func 0 为 PF，Func 1–7 为 VF） | 最多 **255 个**（Func 0 为 PF，Func 1–255 为 VF） |
| **SR-IOV Capability 交互** | 传统 3 位 Function 编号 | 需软件置位 SR-IOV Control Register（偏移 `0x08`）Bit 4 **`ARI Capable Hierarchy`** |
| **VF 编号公式** | `VF[i] = PF_BDF + First_VF_Offset + (i × VF_Stride)` | 公式相同，但结果跨过 7 后直接延续到 `8..255`（无需占用其他 Device 号） |

### 典型初始化流程

1. 系统枚举阶段，操作系统发现 Root Port / Switch Downstream Port 支持 `ARI Forwarding`，且设备位于 `Device 0` 并带有 ARI Capability（`0x000E`）。
2. 软件在上游端口置位 `ARI Forwarding Enable`，在 PF 的 SR-IOV Control 中置位 `ARI Capable Hierarchy`。
3. 管理员通过 sysfs（如 `echo 64 > /sys/bus/pci/devices/0000:01:00.0/sriov_numvfs`）创建 VF。
4. 硬件激活对应数量的 VF，总线重新扫描后枚举出 `01:00.1` 至 `01:08.0`（共 64 个 VF），各自拥有独立 BAR 与 MSI-X 中断。

---

## 常用检查命令与常见限制

```bash
# 1. 检查端点设备是否支持 ARI 及 Next Function 链表
lspci -vvv -s 01:00.0 | grep -A 2 "Alternative Routing"

# 2. 检查上游 Root Port 是否支持并已启用 ARI Forwarding (ARIFwd+)
lspci -vvv -s 00:01.0 | grep -E "DevCap2|DevCtl2" | grep ARIFwd

# 3. 查看 SR-IOV PF 支持的最大 VF 数（若受限于 7，通常说明链路中某处未开启 ARI）
cat /sys/bus/pci/devices/0000:01:00.0/sriov_totalvfs
```

**常见踩坑点**：
- **链路木桶效应**：Root Port、中间所有 Switch Downstream Port、以及 Endpoint 必须**全部**支持 ARI；任意一级不支持，系统就无法启用 ARI，VF 上限会被锁死在 7 个或无法枚举高编号 VF。
- **拓扑位置要求**：ARI 设备必须独占该总线的 **Device 0**，不能与同总线上的 Device 1 等其他设备混用。
- **资源需求暴涨**：256 个功能意味着需要大量 MMIO（BAR）空间和 MSI-X 中断向量，需确保 BIOS 开启 `Above 4G Decoding` 并分配充足的 Bus/MMIO 资源。

---

## 关联与引申概念

- **SR-IOV（Single Root I/O Virtualization）**（[sriov.md](sriov.md)）：ARI 的核心搭档，ARI 解除了 SR-IOV 单设备 7 个 VF 的数量瓶颈。
- **PCIe 设备枚举（Enumeration）**（[../architecture/enumeration.md](../architecture/enumeration.md)）：BIOS/OS 扫描 PCIe 树并分配总线号与资源的标准流程，ARI 通过 `Next Function Number` 扩展了单设备的扫描范围。
- **ACS（Access Control Services）**（[acs.md](acs.md)）：当一个 ARI 设备虚拟出数十上百个 VF 并直通给不同虚拟机时，需要 ACS（含 ACS Function Groups）配合 IOMMU 保证各个 VF 之间的隔离性。
- **BDF（Bus:Device.Function）**：PCIe 16 位逻辑地址标识，也是路由完成包（Completion TLP）和中断映射（IOMMU Remapping）的核心索引。
