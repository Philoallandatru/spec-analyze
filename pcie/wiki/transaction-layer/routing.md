# 路由机制详解 (Routing)

**PCIe 路由机制**决定了 Switch 和 Root Complex 如何把一个 TLP 从源设备准确转发到目标设备。PCIe 根据事务类型的不同，定义了三种互不相同的路由方式：**基于地址（Address-Based）**、**基于 ID（ID-Based）** 和 **隐式路由（Implicit）**。

> **直观类比**：就像寄信有三种投递方式——**基于地址路由**好比按"街道门牌号"分拣（Memory 读写）；**基于 ID 路由**好比按"公司工号 BDF"点对点转交（Config 读写与 Completion 回包）；**隐式路由**好比"发给总部的挂号信"或"全体广播通知"（Message 消息），不用写具体地址或工号，中转站看信封类型就知道该往上游还是下游送。

---

## 1. 三种路由机制总览

| 路由方式 | 适用的 TLP 类型 | 路由依据 | Switch / Bridge 转发判断依据 |
|----------|----------------|----------|------------------------------|
| **Address-Based（基于地址）** | Memory Read / Write, I/O Read / Write | 32-bit 或 64-bit 目标地址 | 配置空间中的 **Memory Base / Limit** 地址窗口 |
| **ID-Based（基于 ID）** | Configuration Read / Write, Completion | 16-bit **BDF** (`Bus:Device.Function`) | 配置空间中的 **Primary / Secondary / Subordinate Bus** 总线号范围 |
| **Implicit（隐式路由）** | 多数 Message (Msg / MsgD) | Header 中的路由子类型编码 (`r[2:0]`) | 固定规则（上送 Root Complex、全下游广播或本地终结） |

---

## 2. Address-Based Routing（基于地址路由）

用于 **Memory** 和 **I/O** 读写请求。系统枚举时，BIOS/OS 会为每个设备的 **BAR（Base Address Register）** 分配一段物理地址空间，并在上游每个 Bridge/Switch 端口配置对应的**地址窗口（Base ~ Limit）**：

```
                   Root Complex
                        │
        ┌───────────────┴───────────────┐
   Port 1 (窗口: 0xF000_0000~0xF0FF_FFFF)  Port 2 (窗口: 0xF100_0000~0xF1FF_FFFF)
        │                               │
   GPU (BAR0: 0xF000_0000)         NVMe SSD (BAR0: 0xF100_0000)
```

**Switch 转发决策流程**：
1. 收到 Memory TLP 后，先检查是否命中本地端口 BAR；若是则本地处理。
2. 否则遍历所有下游端口的地址窗口（Non-Prefetchable Memory Window 与 Prefetchable 64-bit Memory Window）：
   - **命中某下游端口窗口**：直接转发到该下游端口（支持同一 Switch 下设备间的 Peer-to-Peer DMA，除非被 ACS 安全策略拦截）。
   - **未命中任何下游窗口**：说明目标在拓扑上方（例如设备 DMA 访问系统主存 RAM），一律**向上游端口转发**。

---

## 3. ID-Based Routing（基于 ID 路由）

用于 **Configuration Request**（访问设备配置空间）和 **Completion**（Non-Posted 请求的响应回包）。每个 PCIe 功能由唯一的 16 位 **BDF** 标识：`Bus (8-bit) : Device (5-bit) : Function (3-bit)`。

每个 Bridge/Switch 虚拟桥在枚举时都会被分配三个总线号寄存器：
- **Primary Bus Number**：Bridge 上游所连的总线号。
- **Secondary Bus Number**：Bridge 下游**直连**的总线号。
- **Subordinate Bus Number**：Bridge 下游整棵子树中**最大**的总线号。

```
Switch 收到 ID-Based TLP (目标 Bus = N)：
  ├── N == Secondary Bus ──▶ 若是 Type 1 Config TLP，转为 Type 0 后发往直连下游总线
  ├── Secondary < N ≤ Subordinate ──▶ 保持 Type 1，转发给包含 N 的对应下游 Bridge
  └── 不在 [Secondary, Subordinate] 范围内 ──▶ 转发给上游端口（例如 Completion 回传给 RC）
```

> **Completion 的反向路由**：设备 A 发起 Memory Read 时会在 Header 中填入自己的 `Requester ID`（BDF）和 `Tag`；目标设备 B 返回 `Completion` 时，直接把该 `Requester ID` 作为路由目标，沿途 Switch 依据总线范围将其精准送回设备 A。

---

## 4. Implicit Routing（隐式路由）

用于 **Message（Msg / MsgD）** 事务（如传统 `INTx` 模拟中断、电源管理 `PME_Turn_Off`、错误报告 `ERR_COR` 等）。TLP Header 中 `Type[2:0]` 的低 3 位直接编码了路由方向：

| 路由编码 | 路由行为 | 典型应用场景 |
|----------|----------|-------------|
| `000b` | **Routed to Root Complex** | `INTx` 中断、`ERR_COR` / `ERR_FATAL` 错误上报、`PM_PME` |
| `001b` | **Routed by Address** | 少数带地址的消息 |
| `010b` | **Routed by ID** | 少数点对点定向消息（如 Latency Tolerance Reporting / OBFF 等扩展） |
| `011b` | **Broadcast from Root Complex** | `PME_Turn_Off`（RC 广播通知所有设备准备休眠） |
| `100b` | **Local - Terminate at Receiver** | 仅限相邻端口间的本地消息（如 `Set_Slot_Power_Limit`） |
| `101b` | **Gathered and Routed to Root Complex** | `PME_TO_Ack`（Switch 收集所有下游设备的确认后，汇总上报一个给 RC） |

---

## 5. 实战观测（Linux `lspci`）

```bash
# 查看 Bridge 的总线号范围与 MMIO 地址窗口
lspci -vvv -s 00:01.0 | grep -E "Bus:|Memory behind bridge"
# 示例输出：
#   Bus: primary=00, secondary=01, subordinate=03, sec-latency=0
#   Memory behind bridge: f1000000-f1ffffff [size=16M]
```

---

## 6. 关联与引申概念

- **BAR（Base Address Register）**：设备配置空间中用于声明所需 MMIO/I/O 地址空间大小的寄存器，是 Address-Based Routing 的地址分配源头。
- **BDF（Bus:Device.Function）**：PCIe 设备的 16 位逻辑坐标；在启用 **ARI（Alternative Routing-ID Interpretation）** 时，Device 的 5 位与 Function 的 3 位可合并为 8 位 Function 号，使单总线支持多达 256 个 Function（常见于 SR-IOV 虚拟化）。
- **ACS（Access Control Services）**：PCIe 安全扩展机制，用于在 Switch 中限制设备间的直接 Peer-to-Peer 路由，强制流量先经过 Root Complex 的 IOMMU 进行地址隔离检查。
- **TLP 格式详解**：查看 `Fmt`/`Type`、地址字段及 `Requester ID` 在包头中的具体比特位置，详见 [TLP 格式详解](tlp-format.md)。
