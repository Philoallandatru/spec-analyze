# PCIe 原子操作（Atomic Operations）

> **一句话总结**：PCIe 原子操作（AtomicOps）允许外部设备（如 GPU、网卡、加速器）直接向系统内存发起硬件级、不可分割的"读-修改-写"事务，从而在多设备、多处理器并发场景下实现无锁同步，避免数据竞争。

如果把共享内存里的计数器比作一个银行账户余额：没有原子操作时，设备 A 和设备 B 同时读取余额 100、各自加 1 后写回，最终结果变成 101，丢掉了一次更新；而 PCIe AtomicOp 就像银行柜台的"原子转账指令"——内存控制器在底层直接锁定这笔操作，按顺序执行完读、加、写并返回旧值，任何人都无法插队。

---

## 三种标准原子操作语义

PCIe 规范在 TLP（事务层数据包）层面定义了三种基本原子操作，支持 **32 位** 或 **64 位** 操作数（部分平台可选支持 128 位 CAS）：

| 操作类型 | AtomicOp 语义（不可分割执行） | 参数数量 | 典型用途 |
| :--- | :--- | :--- | :--- |
| **FetchAdd**（取值并加） | `old = *addr; *addr = old + operand; return old;` | 1 个（增量，可为负） | 全局计数器、引用计数、队列索引递增 |
| **Swap**（无条件交换） | `old = *addr; *addr = operand; return old;` | 1 个（新值） | 自旋锁（Spinlock）、所有权切换 |
| **CAS**（比较并交换） | `old = *addr; if (old == cmp) *addr = swap; return old;` | 2 个（Compare + Swap） | 无锁队列/栈/哈希表、乐观并发控制 |

> **关键特点**：与普通 Memory Write（Posted 事务，无返回值）不同，**AtomicOp 是 Non-Posted 事务**——发起方（Requester）会收到一个标准的 **Completion with Data**，里面携带操作执行**之前**的原始内存值，软件/设备硬件据此判断 Swap 或 CAS 是否成功。

---

## 工作原理：TLP 格式、路由与缓存一致性

### 1. AtomicOp Request TLP 结构

AtomicOp 使用特殊的 Memory Request TLP 格式（`Type = 01100b` / `01101b` / `01110b` 分别对应 FetchAdd、Swap、CAS 的 32/64 位编码，携带 3DW 或 4DW Header + Data Payload）：

- **Requester ID & Tag**：标识发起设备与事务序号，用于接收 Completion 返回值。
- **Address**：目标系统内存物理地址（32 位操作需 4 字节对齐，64 位操作需 8 字节对齐）。
- **Data Payload**：
  - `FetchAdd` / `Swap`：携带 1 个操作数（4B 或 8B）。
  - `CAS`：携带 2 个操作数（先 `Compare Value`，后 `Swap Value`，共 8B 或 16B；128-bit CAS 为 32B）。

### 2. 端到端执行与缓存一致性流程

```text
Endpoint (Requester)
  │  1. 发起 AtomicOp Request TLP（如 FetchAdd +1）
  ▼
PCIe Switch（需支持 AtomicOp Routing）
  │  2. 按地址向上游转发（若启用 ACS ReqRedir 则强制上行）
  ▼
Root Complex (Completer)
  │  3. 发起缓存侦听（Snoop）：若目标地址在 CPU Cache 中，
  │     先将脏缓存行写回或置为无效（MESI/MESIF 一致性协议）
  ▼
Memory Controller（原子执行单元）
  │  4. 原子执行 Read-Modify-Write，更新内存
  ▼
Root Complex ──▶ Completion with Data (返回操作前的旧值) ──▶ Endpoint
```

---

## 硬件能力要求与支持链（Capability Chain）

PCIe 原子操作需要从发起端到目标端**整条路径上所有组件**共同支持，任何一环缺失都会导致设备收到 `Unsupported Request (UR)` 错误。

能力与控制位位于 **PCIe Capability 结构的 Device Capabilities 2（偏移 `0x24`）和 Device Control 2（偏移 `0x28`）**：

| 组件角色 | 关键寄存器字段 | 职责 |
| :--- | :--- | :--- |
| **Endpoint（Requester）** | `DevCtl2` Bit 6：**AtomicOp Requester Enable** | 软件置 `1` 后，允许设备主动发起 AtomicOp TLP |
| **Switch（Router）** | `DevCap2` Bit 6：**AtomicOp Routing Supported**<br>`DevCtl2` Bit 7：**AtomicOp Egress Blocking** | 支持将 AtomicOp TLP 正确转发；可选阻塞下游出口的 AtomicOp |
| **Root Port / RC（Completer）** | `DevCap2` Bits 7–9：**32-bit / 64-bit / 128-bit CAS Completer Supported** | 最终接收并原子执行内存读-改-写，返回 Completion |

### 检查硬件支持状态

```bash
# 查看端点设备与 Root Port 的 AtomicOps 能力及启用状态
lspci -vvv -s 01:00.0 | grep -i "AtomicOps"
lspci -vvv -s 00:01.0 | grep -i "AtomicOps"
# 正常输出示例：
# DevCap2: ... AtomicOpsCap: Routing+ 32bit+ 64bit+ 128bitCAS-
# DevCtl2: ... AtomicOpsCtl: ReqEn+
```

---

## 性能特征与使用限制

### 延迟与吞吐权衡

- **典型延迟**：单次 PCIe AtomicOp 往返约 **300–600 ns**（包含链路传输、RC 缓存侦听、内存控制器原子锁与 Completion 返回），显著高于 CPU 本地原子指令（10–50 ns），但远低于跨设备软件中断/锁协同开销。
- **优化建议**：
  1. **批量合并（Batching）**：设备内部先在本地寄存器/SRAM 累加，达到阈值后再发起一次 `FetchAdd` 更新主存，避免高频细粒度争抢。
  2. **缓存行对齐**：不同设备的原子变量按 64 字节（Cache Line）对齐，避免伪共享（False Sharing）引发频繁 Snoop。
  3. **操作降级**：能用 `FetchAdd` 解决的场景不要用 `CAS` 循环，减少重试与总线上 Payload 长度。

### 核心限制

1. **严格地址对齐**：32 位操作必须 4 字节对齐，64 位操作必须 8 字节对齐，否则触发错误。
2. **仅限支持的目标内存**：通常仅 **系统主内存（DRAM）** 支持作为 AtomicOp Completer；普通设备 BAR（MMIO 空间）和 ROM 通常不支持（除非设备显式声明支持 P2P AtomicOp Completer）。
3. **Outstanding Tag 限制**：因为是 Non-Posted 事务，并发数受限于设备允许的 Outstanding Tag 数量。

---

## 关联与引申概念

- **ACS（Access Control Services）**（[acs.md](acs.md)）：ACS 的 `P2P Request Redirect` 同样作用于 AtomicOp TLP；在开启 ACS 的虚拟化环境中，P2P AtomicOp 会被重定向至 Root Complex 接受 IOMMU 校验。
- **TLP 事务类型与 Non-Posted 机制**（[../transaction-layer/transaction-types.md](../transaction-layer/transaction-types.md)）：AtomicOp 属于 Non-Posted 事务，需要消耗 Tag 并等待 Completion 返回旧值。
- **CXL（Compute Express Link）**：CXL.mem / CXL.cache 在 PCIe 物理层之上进一步提供了硬件级缓存一致性原子语义，适合更细粒度的 CPU-加速器内存共享。
- **GPU Direct / RoCE / 多加速器协同**：多 GPU 梯度聚合、RDMA 网卡无锁队列计数等场景广泛依赖 PCIe AtomicOps 消除主机 CPU 轮询介入。
