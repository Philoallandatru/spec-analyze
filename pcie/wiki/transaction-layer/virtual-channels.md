# PCIe 虚拟通道 (Virtual Channel, VC)

**虚拟通道（Virtual Channel, VC）** 是 PCIe 在同一条物理链路上划分出的多条"逻辑独立车道"（最多支持 8 条：`VC0 ~ VC7`）。每个 VC 拥有完全独立的接收缓冲区、流控制信用池和排序队列，从而让高优先级或实时性流量（如音视频、控制信令）免受大流量普通数据的拥堵干扰，实现硬件级 **QoS（服务质量）** 隔离。

> **直观类比**：一条双向八车道的高速公路（物理链路）上，划出了"普通车道（`VC0`）"和"应急/VIP 专用道（`VC1`）"。哪怕普通车道因为大货车（大块 DMA 传输）排起长队甚至暂时没车位（Credit 耗尽），VIP 车道依然有自己独立的收费站、独立车位（独立 Credit）和专属优先通行权（VC 仲裁），绝不会被普通车道堵死。

---

## 1. TC 与 VC 的映射关系

为了让同一套硬件机制灵活适配不同设备，PCIe 把"业务打标"和"车道分配"拆成两层：

1. **TC（Traffic Class，流量类别，3 bits：`TC0 ~ TC7`）**：写在每个 **TLP Header** 里，由发起方或软件为数据包打上"服务等级标签"（端到端携带）。
2. **VC（Virtual Channel，虚拟通道，`VC0 ~ VC7`）**：每段链路两端端口上的**硬件独立缓冲与流控队列**。

```
端到端 TLP 标签 (Header 中携带)          端口硬件映射 (TC/VC Map 寄存器)
         TC0  ──────────────────────────▶  VC0 (强制支持，默认通道)
         TC1, TC2 ──────────────────────▶  VC1 (可选，多对一合并)
         TC3  ──────────────────────────▶  VC2 (可选，一对一独占)
```

**核心映射规则**：
- **VC0 强制存在**，且 `TC0` 必须映射到 `VC0`。
- **多对一允许**：多个 `TC` 可以映射到同一个 `VC`（例如默认情况下所有 `TC0~TC7` 全映射到 `VC0`）。
- **一对多禁止**：同一个 `TC` 在同一段链路上**绝对不能**同时映射到两个不同的 `VC`。
- **独立排序**：PCIe 排序规则**仅在同一 TC 内生效**，不同 VC/TC 之间互不阻塞、无排序约束。

---

## 2. 独立流控制与仲裁机制

### 2.1 每个 VC 独立的 6 组信用池
每个启用的 VC 都在数据链路层通过独立的 `InitFC` / `UpdateFC` DLLP（Byte 0 低 3 位携带 `VC ID`）管理自己的 **Posted / Non-Posted / Completion**（各自含 Header 与 Data）共 6 组信用。因此即使 `VC0` 信用耗尽阻塞，`VC1` 依然可以全速收发。

### 2.2 VC 仲裁（决定谁先上物理链路）
当多个 VC 同时有 TLP 准备发送时，端口通过 **Port VC Capability** 配置的仲裁策略决定发送顺序：
- **固定优先级（Fixed Priority）**：高编号 VC 严格优先于低编号 VC（如 `VC7 > ... > VC1 > VC0`），适合强实时控制流量。
- **加权轮询（WRR / Time-based WRR）**：通过仲裁表为各 VC 分配时间片权重（如 `VC1` 占 60%、`VC0` 占 40%），在保证高优先级带宽的同时防止低优先级完全饿死。

---

## 3. 实战观测（Linux `lspci`）

绝大多数普通消费级/通用服务器设备只实现默认的 `VC0`；支持多 VC 的高端交换机或专用硬件会在扩展配置空间中暴露 `Virtual Channel` Capability：

```bash
lspci -vvv -s 01:00.0 | grep -A 8 "Virtual Channel"
# 输出示例：
# Capabilities: [100] Virtual Channel
#     Caps:   LPEVC=0 RefClk=100ns PATEntryBits=1
#     VC0:    Enable+ ID=0 ArbSelect=Fixed TC/VC=ff
# (TC/VC=ff 表示 TC0~TC7 全部映射到 VC0)
```

---

## 4. 关联与引申概念

- **流控制（Flow Control）**：每个 VC 独立维护的信用防溢出机制，详见 [流控制机制](flow-control.md) 与 [流控制初始化](../data-link-layer/fc-init.md)。
- **排序规则（Ordering）**：理解为什么不同 TC/VC 之间没有排序约束，以及同 TC 内的超越规则，详见 [排序规则](ordering.md)。
- **TLP 格式详解**：查看 `TC[2:0]` 字段在 TLP Header DW0 中的位置，详见 [TLP 格式详解](tlp-format.md)。
- **CXL（Compute Express Link）**：在 PCIe 物理层之上演进的现代互连协议，其 `CXL.io`、`CXL.cache`、`CXL.mem` 多协议复用与隔离的设计思想便与多 VC 硬件隔离一脉相承。
