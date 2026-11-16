# PCIe 设备枚举

**一句话总结**：设备枚举（Enumeration）是系统开机后由 BIOS/UEFI 或操作系统内核执行的“全网点名与资源分配”过程——通过**深度优先遍历**发现所有 PCIe 设备，为其分配 **BDF 编号**、配置 **Bridge 总线区间**、探测并分配 **BAR 内存地址**，最后激活设备使其可被驱动接管。

> **直观类比**：设备枚举就像新学期开学老师点名分座位：校长（Root Complex）从 0 号教室（Bus 0）开始挨个敲门；每遇到一个走廊转接口（Bridge/Switch），就给走廊后面的房间编上新教室号（Secondary Bus），记下这片区域的最大编号（Subordinate Bus）；等把所有学生（Endpoint）都登记好、发完储物柜钥匙（BAR 地址）和呼叫铃（MSI-X 中断），整栋楼就可以正式上课了。

---

## 枚举完整流程（7 步走）

```
1. 物理层链路训练 ──▶ LTSSM 完成，链路进入 L0 状态
       │
2. 递归扫描总线   ──▶ 从 Bus 0 开始深度优先搜索 (DFS)
       │
3. 读取配置空间   ──▶ 读 Offset 0x00 的 Vendor ID（若为 0xFFFF 则无设备）
       │
4. 配置 Bridge    ──▶ 为下游分配 Secondary Bus，回溯更新 Subordinate Bus
       │
5. 探测并分配 BAR ──▶ 写全 1 测大小，为设备与 Bridge 窗口分配物理内存地址
       │
6. 配置中断       ──▶ 分配 MSI / MSI-X 向量或路由传统 INTx
       │
7. 启用设备       ──▶ 置位配置空间 Command 寄存器的 Memory/Bus Master 使能位
```

---

## 配置空间是如何被访问的？

在枚举阶段，设备还没有 BAR 地址，CPU 只能通过 **配置空间（Configuration Space）** 与其对话。有两种访问机制：

1. **传统 I/O 端口机制（已过时，仅兼容前 256 字节）**：通过向 `0xCF8`（`CONFIG_ADDRESS`）写入目标 Bus/Dev/Func/Offset，再从 `0xCFC`（`CONFIG_DATA`）读写数据。
2. **ECAM（Enhanced Configuration Access Mechanism，PCIe 标准）**：固件通过 ACPI `MCFG` 表向操作系统通告一段连续的物理内存基地址，将全系统所有设备的 **4096 字节扩展配置空间**平铺映射其中：
   $$\text{配置空间物理地址} = \text{ECAM\_Base} + (\text{Bus} \ll 20) + (\text{Device} \ll 15) + (\text{Function} \ll 12) + \text{Offset}$$
   CPU 只需执行一次普通的内存读写指令，Root Complex 就会自动将其转换为 **Configuration Read/Write TLP** 发往目标设备。

### Type 0 vs Type 1 配置请求
- **Config Type 0**：目标设备就在当前直连总线上，直接由该总线的目标设备接收处理。
- **Config Type 1**：目标设备在更下游的总线上，必须经过 **Bridge** 转发；当数据包到达目标总线直连的那个 Bridge 时，由它将其转换为 **Type 0** 交给最终设备。

---

## 深度优先扫描（DFS）与 Bridge 总线号分配

每经过一个 Bridge（包括 Root Port 和 Switch 端口），配置空间（Type 1 Header）中都维护着三个关键总线号寄存器：
- **Primary Bus Number（`0x18`）**：Bridge 上游紧邻的总线号。
- **Secondary Bus Number（`0x19`）**：Bridge 下游直连的总线号。
- **Subordinate Bus Number（`0x1A`）**：该 Bridge 下游整棵子树所包含的**最大**总线号。

```
Root Complex (Bus 0)
  └─ Root Port (00:01.0) [Pri=0, Sec=1, Sub=4]
       └─ Switch Upstream Port (01:00.0) [Pri=1, Sec=2, Sub=4]
            ├─ Downstream Port 1 (02:00.0) [Pri=2, Sec=3, Sub=3]
            │    └─ NVMe SSD (03:00.0)  ← Type 0 Endpoint
            └─ Downstream Port 2 (02:01.0) [Pri=2, Sec=4, Sub=4]
                 └─ 10GbE NIC (04:00.0) ← Type 0 Endpoint
```

**扫描规则要点**：
- 扫描每个设备的 Function 0 时，检查 `Header Type`（偏移 `0x0E`）的 Bit 7：若为 `0` 表示单功能设备，直接跳过 Function 1–7；若为 `1` 表示多功能设备，继续扫描其余 Function。
- 进入 Bridge 时，先把 `Subordinate Bus` 临时设为 `0xFF`（确保后续所有探测请求都能穿透过去），待下游整棵子树递归扫描完毕后，再把实际的最高总线号回填到 `Subordinate Bus`。

---

## 资源分配与设备激活

1. **自底向上统计，自顶向下分配**：系统先探测树上所有 Endpoint 的 **BAR 大小**（写 `0xFFFFFFFF` 读掩码），汇总每个 Bridge 下游所需的总内存空间（按大小对齐），配置好 Bridge 的 `Memory Base/Limit` 窗口，再把具体的物理基地址写入各设备的 BAR。
2. **激活设备（Command Register，偏移 `0x04`）**：地址分配完毕后，内核或驱动必须开启关键使能位，设备才能真正干活：
   - **Memory Space Enable（Bit 1）**：允许设备响应发往其 BAR 的 MMIO 访问。
   - **Bus Master Enable（Bit 2）**：允许设备主动发起 **DMA** 读写和 **MSI/MSI-X** 中断请求（若未置位，设备发出的上游 TLP 会被 Root Complex 拦截丢弃）。

---

## 运行时热插拔（Hot-Plug）与 Linux 调试

热插拔（如服务器不停机更换 U.2 NVMe 盘）本质上是在运行时触发一次**局部枚举**：
插槽上电 → 等待链路训练到 `L0` → 扫描该 Downstream Port 下的新设备 → 从预留资源池中分配 Bus 号与 BAR 地址 → 通知内核加载驱动。

在 Linux 中常用以下命令观察或触发枚举：

```bash
lspci -tv                       # 查看最终枚举出的总线树与 BDF
lspci -vv -s 01:00.0            # 查看 Bus Master 状态、BAR 地址与 MSI-X 向量数
dmesg | grep -i pci             # 查看内核启动阶段的 PCIe 枚举与资源分配日志
echo 1 | sudo tee /sys/bus/pci/rescan   # 手动触发全总线重新扫描
```

---

## 关联与引申概念

- **[地址空间管理](address-space.md)**：深入了解 BAR 探测、64 位 BAR 编码、Bridge 内存窗口与 IOMMU 地址映射细节。
- **[拓扑结构](../introduction/topology.md)**：理解 Root Port、Switch 端口与 Endpoint 在枚举树中的角色分工。
- **[数据包类型](packet-types.md)**：查看枚举阶段使用的 `Config Read/Write Type 0` 与 `Type 1` TLP 格式。
- **[三层协议模型](layering.md)**：理解枚举开始前物理层 `LTSSM` 如何完成链路训练并进入 `L0` 状态。
