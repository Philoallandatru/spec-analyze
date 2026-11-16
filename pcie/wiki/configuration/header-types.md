# PCIe 配置空间头部类型（Type 0 vs Type 1）

**PCIe 配置空间的前 64 字节（`0x00–0x3F`）称为配置空间头部（Header）**，根据设备在总线拓扑中扮演的角色不同，分为 **Type 0（端点设备）** 和 **Type 1（桥接设备）** 两种格式。

> **直观类比**：如果把 PCIe 树比作城市交通网，**Type 0** 就像终点站或商店（NVMe SSD、网卡、显卡），关心的是“自己占多大一块地皮（BAR）”；**Type 1** 就像立交桥或路由器（Root Port、Switch Port），关心的是“下游连着哪几条路（总线号范围）以及哪些地址的车辆该往桥下放行（地址窗口）”。

---

## 1. 通用头部（前 16 字节，0x00–0x0F）

无论 Type 0 还是 Type 1，前 16 字节布局完全一致，核心在于偏移 `0x0E` 的 **Header Type** 字段：

| 偏移 | 位宽 | 字段 | 说明 |
|------|------|------|------|
| `0x00` | 16b | **Vendor ID** | 厂商 ID（全 `0xFFFF` 表示设备不存在） |
| `0x02` | 16b | **Device ID** | 设备型号 ID |
| `0x04` | 16b | **Command** | 设备控制开关（Memory Space、Bus Master、INTx Disable） |
| `0x06` | 16b | **Status** | 设备状态（Bit 4 指示是否存在 Capability 链表） |
| `0x08` | 8b | **Revision ID** | 硬件修订版本号 |
| `0x09` | 24b | **Class Code** | 设备分类（如 `0x010802` 为 NVMe，`0x060400` 为 PCI-to-PCI Bridge） |
| `0x0C` | 8b | **Cache Line Size** | 缓存行大小 |
| `0x0D` | 8b | **Latency Timer** | 传统 PCI 延迟计时器（PCIe 中保留为 0） |
| `0x0E` | 8b | **Header Type** | **Bit 7**：`1`=多功能设备；**Bits 6:0**：`0x00`=Type 0，`0x01`=Type 1 |
| `0x0F` | 8b | **BIST** | 内建自检（Built-In Self Test） |

> **多功能设备（Multi-Function）**：当 `Header Type` 的 **Bit 7 = 1** 时，操作系统会继续扫描同一 Device 下的 `Function 1 ~ 7`（例如一张网卡同时暴露网口 Func 0 和存储 Func 1，或支持 **ARI** 时扩展到 256 个 Function）。

---

## 2. Type 0 Header（端点设备）

所有终端功能设备（Endpoint，包括 NVMe SSD、GPU、网卡）均使用 Type 0 头部，最大特点是拥有 **6 个 BAR 槽位（`0x10–0x27`）**：

```
  0x00 ┌─────────────────┬─────────────────┐
       │   Vendor ID     │    Device ID    │
  0x04 ├─────────────────┼─────────────────┤
       │     Command     │     Status      │
  0x08 ├───────┬─────────┴─────────────────┤
       │Rev ID │        Class Code         │
  0x0C ├───────┼───────┬───────────┬───────┤
       │CacheL │Lat Tmr│Header=0x00│ BIST  │
  0x10 ├───────┴───────┴───────────┴───────┤
       │              BAR 0                │
  0x14 │              BAR 1                │  6 个 BAR（32-bit 占 1 个，
  0x18 │              BAR 2                │  64-bit 占连续 2 个）
  0x1C │              BAR 3                │
  0x20 │              BAR 4                │
  0x24 │              BAR 5                │
  0x28 ├───────────────────────────────────┤
       │        CardBus CIS Pointer        │
  0x2C ├─────────────────┬─────────────────┤
       │Subsys Vendor ID │ Subsys Device ID│  区分不同板卡 OEM 厂商
  0x30 ├─────────────────┴─────────────────┤
       │     Expansion ROM Base Address    │
  0x34 ├───────┬───────────────────────────┤
       │Cap Ptr│         Reserved          │  传统 Capability 链表入口
  0x38 ├───────┴───────────────────────────┤
       │             Reserved              │
  0x3C ├───────┬───────┬─────────┬─────────┤
       │IntLine│IntPin │ Min_Gnt │ Max_Lat │  IntPin: 0=无, 1~4=INTA~INTD
  0x40 └───────┴───────┴─────────┴─────────┘
```

- **BAR0–BAR5**：用于将设备控制寄存器或缓冲区映射到系统内存空间（详见 [BAR 详解](bars.md)）。
- **Subsystem IDs（`0x2C`）**：同一颗主控芯片（Vendor/Device ID 相同），不同板卡厂商（如 Dell、Samsung OEM）通过 Subsystem ID 区分。
- **Interrupt Pin（`0x3D`）**：指示设备使用哪条传统中断线（`1=INTA# ~ 4=INTD#`，`0` 表示不用 INTx）。

---

## 3. Type 1 Header（桥接设备）

Root Port、PCIe Switch 的上下游端口以及 PCIe-to-PCI 桥使用 Type 1 头部。它只有 2 个 BAR（`0x10–0x17`），其余空间用来管理**下游总线号**和**地址转发窗口**：

```
  0x00~0x0F  [通用 16 字节头部，Header Type = 0x01]
  0x10 ├───────────────────────────────────┤
       │              BAR 0                │  桥设备自身寄存器（可选，通常不用）
  0x14 │              BAR 1                │
  0x18 ├────────┬────────┬────────┬────────┤
       │Primary │Secondry│Subordnt│Sec Lat │  总线号三元组（决定拓扑范围）
  0x1C ├────────┼────────┼────────┴────────┤
       │IO Base │IO Limit│   Sec Status    │  I/O 转发窗口
  0x20 ├────────┴────────┼─────────────────┤
       │   Memory Base   │  Memory Limit   │  32-bit 非预取内存窗口
  0x24 ├─────────────────┼─────────────────┤
       │Prefetch Mem Base│Prefetch Mem Lim │  64-bit 可预取内存窗口（低位）
  0x28 ├─────────────────┴─────────────────┤
       │    Prefetch Base Upper 32 bits    │  64-bit 可预取内存窗口（高 32 位）
  0x2C ├───────────────────────────────────┤
       │    Prefetch Limit Upper 32 bits   │
  0x30 ├─────────────────┬─────────────────┤
       │ IO Base Upper   │ IO Limit Upper  │
  0x34 ├────────┬────────┴─────────────────┤
       │Cap Ptr │         Reserved         │
  0x38 ├────────┴──────────────────────────┤
       │    Expansion ROM Base Address     │
  0x3C ├────────┬────────┬─────────────────┤
       │IntLine │IntPin  │ Bridge Control  │  含 Secondary Bus Reset (Bit 6)
  0x40 └────────┴────────┴─────────────────┘
```

### 3.1 总线号三元组（`0x18–0x1A`）

PCIe 拓扑是树形结构，每个 Type 1 桥接器用三个编号界定自己管辖的子树范围：

- **Primary Bus Number（`0x18`）**：桥接器上游连接的总线号。
- **Secondary Bus Number（`0x19`）**：桥接器下游直接连出的总线号。
- **Subordinate Bus Number（`0x1A`）**：该桥下游所有分支中**最大的总线号**（路由配置请求 TLP 时，只要目标 `Bus` 落在 `[Secondary, Subordinate]` 区间内，桥就向下游转发）。

```
                Bus 0 (Root Complex)
                  │
            [Root Port]  Primary=0, Secondary=1, Subordinate=5
                  │
                Bus 1
                  │
             [Switch]    Primary=1, Secondary=2, Subordinate=5
              /    \
          Bus 2    Bus 3~5 (下游还有二级桥)
            │
      [NVMe SSD (Type 0)]
```

### 3.2 地址转发窗口（Base / Limit）

当 CPU 读写下游设备的 BAR 地址时，TLP 必须穿过沿途所有 Type 1 桥接器。桥接器通过 **Base** 和 **Limit** 寄存器判断目标地址是否属于自己管辖的下游区域：

| 窗口类型 | 偏移 | 对齐粒度 | 说明 |
|----------|------|----------|------|
| **Memory Base / Limit** | `0x20–0x23` | 1 MB（高 12 位有效） | 转发 32-bit **Non-Prefetchable** 内存访问（如控制寄存器）。 |
| **Prefetchable Memory** | `0x24–0x2F` | 1 MB + 高 32 位扩展 | 转发 **Prefetchable** 内存访问，支持 64-bit 地址空间（如大显存、CMB）。 |
| **I/O Base / Limit** | `0x1C, 0x30` | 4 KB | 传统 I/O 端口转发（现代 PCIe 设备通常关闭，令 Base > Limit）。 |

> **如何关闭未使用的窗口？** 规范规定当 `Base > Limit` 时该窗口关闭（例如写入 `Base = 0xFFF0, Limit = 0x0000`）。

### 3.3 Bridge Control（`0x3E`）

常用控制位包括 **Bit 6（Secondary Bus Reset）**：向该位写 1 再清 0，会对下游整条总线发起热复位（Hot Reset），常用于恢复卡死的下游设备。

---

## 4. 枚举与资源分配流程（深度优先）

1. **深度优先编号**：固件/OS 扫描到 Type 1 桥接器时，先给它分配 `Secondary Bus`，把 `Subordinate Bus` 临时设为 `0xFF`（确保后续探测包能往下走），继续递归扫描下游所有设备，最后回填实际的 `Subordinate Bus`。
2. **自底向上汇总 BAR**：收集下游所有 Type 0 设备的 BAR 大小需求，合并对齐后写入沿途 Type 1 桥接器的 Memory Base/Limit 窗口。
3. **开启转发**：置位桥接器 Command 寄存器的 `Memory Space Enable` 和 `Bus Master Enable`，下游设备即可被访问。

---

## 5. 实用 CLI 示例

```bash
# 1. 以树形结构查看 PCIe 总线拓扑（直观看到 Type 1 桥与 Type 0 设备）
lspci -tv

# 2. 查看 Type 1 桥接器的总线号范围与转发窗口
lspci -vvv -s 00:1c.0 | grep -E "Bus:|Memory behind|Prefetchable"
# 示例输出：
#   Bus: primary=00, secondary=01, subordinate=01, sec-latency=0
#   Memory behind bridge: f8000000-f80fffff [size=1M]
```

---

## 关联与引申概念

| 概念 | 简要说明 | 相关文档 |
|------|----------|----------|
| **配置空间与 ECAM** | Type 0/1 头部占据配置空间前 64 字节，现代系统通过 ECAM（MMCFG）将其映射为内存访问。 | [配置空间详解](config-space.md) |
| **BAR（基址寄存器）** | Type 0 设备的 6 个核心寄存器，定义设备需要多少 MMIO 地址空间。 | [BAR 详解](bars.md) |
| **Capability Linked List** | 紧随 64 字节头部之后的扩展机制，Type 0/1 都通过 `0x34` 指针串联功能特性。 | [能力结构详解](capabilities.md) |
| **ARI（Alternative Routing-ID）** | 扩展能力之一，打破传统“最多 8 个 Function”限制，将 Device+Function 合并为 8-bit 支持 256 个 Function。 | [扩展能力详解](extended-capabilities.md) |
