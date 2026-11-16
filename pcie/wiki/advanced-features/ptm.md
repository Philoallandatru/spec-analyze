# 精确时间测量（Precision Time Measurement, PTM）

> **一句话总结**：PTM 是 PCIe 3.1 引入的硬件级"对表"协议，让主机（Root Complex）与各个 PCIe 扩展卡（网卡、NVMe、采集卡）之间以亚微秒甚至纳秒级精度同步时钟，同时自动扣除信号在链路上的传输延迟。

如果把主板上的 CPU、Switch 和网卡比作不同房间里的时钟：传统软件对表（比如 CPU 读一下网卡寄存器）就像打电话问对方几点——等声音传过来、系统调度完，时间早就过了好几微秒，而且根本不知道信号在路上跑了多久。PTM 的做法是在**物理层/数据链路层收发瞬间**由硬件打下精准时间戳，通过"一来一回"四次打点（$t_1, t_2, t_3, t_4$）算出链路单程传播延迟，从而把误差压缩到纳秒级。

---

## PTM 层次架构与角色

PTM 采用自顶向下的树状同步架构，沿着 PCIe 拓扑逐跳传递时间基准：

```text
┌─────────────────────────┐
│    PTM Root (Master)    │  ← 全拓扑唯一主时钟源（通常位于 Root Complex）
└────────────┬────────────┘
             │  PTM Dialog (T1/T2/T3/T4)
    ┌────────┴────────┐
    ▼                 ▼
┌────────┐      ┌───────────┐
│ EP 1   │      │  Switch   │  ← 对上游是 PTM Requester（对表）
│ (Req)  │      │(Req + Rsp)│  ← 对下游是 PTM Responder（传递校准后的主时钟）
└────────┘      └─────┬─────┘
                      │  PTM Dialog
                      ▼
                ┌───────────┐
                │   EP 2    │  ← PTM Requester
                │   (Req)   │
                └───────────┘
```

| 角色 | 所在组件 | 职责 |
| :--- | :--- | :--- |
| **PTM Root** | Root Complex（或根端口） | 维护整个 PCIe 域的权威主时钟（Master Time），响应下游请求 |
| **PTM Requester** | Endpoint 或 Switch Upstream Port | 周期性向上游发起 PTM Request 消息，根据返回时间戳校准本地 PTM Context |
| **PTM Responder** | Root Complex、Switch Downstream Port | 接收下游 PTM Request，记录收发时间戳并返回主时钟信息 |

---

## 核心机制：PTM Dialog 与链路延迟计算

PTM 通过专门的 **PTM Request** 和 **PTM Response / ResponseD** Message TLP 完成对表，**不需要消耗 CPU 周期或软件干预**。一次完整的 PTM 对话（PTM Dialog）在硬件层记录四个关键时间戳：

```text
Upstream (Responder)                          Downstream (Requester)
        │                                              │
        │◀─────────── 1. PTM Request TLP ──────────────┤  发送瞬间记录 t1（本地时钟）
  接收瞬间记录 t2                                       │
  (Master Time)                                        │
        │                                              │
        ├──────────── 2. PTM Response TLP ────────────▶│  接收瞬间记录 t4（本地时钟）
  发送瞬间记录 t3                                       │
        │                                              │
        ├──────────── 3. PTM ResponseD TLP ───────────▶│  携带上次对话的 t2 与 (t3 - t2)
        │                (含主时间与停留时间)            │
```

由于硬件在发送 `PTM Response` 的当下无法把"此刻自己的发送时间 $t_3$"实时写进正在发出的包里，因此需要后续的 **`PTM ResponseD`** 携带上一轮的精准时间差：

1. **计算单程链路传播延迟（Link Propagation Delay）**：
   $$\text{Link Delay} = \frac{(t_4 - t_1) - (t_3 - t_2)}{2}$$
   - $(t_4 - t_1)$：Requester 侧感知的往返总耗时。
   - $(t_3 - t_2)$：Responder 内部处理停留时间。
   - 相减除以 2 即剔除了对端处理耗时，得到纯净的物理链路单向传输延迟。

2. **还原对端接收瞬间的精确主时钟**：
   $$\text{Master Time at } t_4 = t_2 + (t_3 - t_2) + \text{Link Delay}$$
   Requester 由此将本地时钟与 PTM Root 对齐，并计算本地时钟与主时钟的偏移量与频率漂移率（Skew）。

---

## PTM Extended Capability 寄存器（ID `0x001F`）

PTM 的标准配置空间结构非常精简，仅包含头部与两个 32 位寄存器（`t1–t4` 与 Master Time 均在硬件内部 PTM Context 与 TLP 中流转，不映射到配置空间）：

| 偏移 | 寄存器名称 | 关键字段与作用 |
| :--- | :--- | :--- |
| `0x00` | Extended Capability Header | Capability ID = `0x001F` |
| `0x04` | **PTM Capability Register** | Bit 0：`PTM Requester Capable`<br>Bit 1：`PTM Responder Capable`<br>Bit 2：`PTM Root Capable`<br>Bits 15:8：`Local Clock Granularity`（本地时钟粒度，单位 ns） |
| `0x08` | **PTM Control Register** | Bit 0：`PTM Enable`（软件置 1 启用 PTM）<br>Bit 1：`Root Select`（置 1 选择该端口作为 PTM Root，通常仅 Root Complex 置位）<br>Bits 15:8：`Effective Granularity`（软件写入全路径最粗时钟粒度） |

### 检查 PTM 状态

```bash
# 查看 Root Port 与设备的 PTM 能力及开启状态
lspci -vvv | grep -A 4 "Precision Time Measurement"
```

---

## PTM 与 IEEE 1588 PTP / NTP 的关系

在数据中心、5G 基站和高频交易（HFT）中，PTM 并不是要取代网络侧的 **IEEE 1588 PTP**，而是**打通"网口到 CPU"的最后一米**：

| 对比维度 | PCIe PTM | IEEE 1588 PTP | NTP |
| :--- | :--- | :--- | :--- |
| **适用范围** | 单台主机内部 PCIe 拓扑 | 局域网 / 数据中心跨主机网络 | 广域网 / 互联网 |
| **典型同步精度** | **< 10–100 ns**（亚微秒/纳秒级） | **< 1 μs**（硬件时间戳可达数十 ns） | **1–10 ms** |
| **实现方式** | PCIe 硬件链路层自动对表 | 网卡 PHC + 交换机 Boundary Clock | 纯软件网络协议 |
| **协同方式** | 网卡先通过 PTP 与外部网络授时服务器对齐 **PHC（PTP Hardware Clock）**，再通过 **PCIe PTM** 把精确时间同步给主机 CPU（TSC / 系统时钟）以及同机其他 GPU/NVMe 设备，消除 PCIe 总线读取 PHC 的延迟抖动。 | ← 跨主机时间基准来源 | 粗粒度系统时间 |

---

## 关联与引申概念

- **IEEE 1588 PTP（Precision Time Protocol）**：网络级精确时间协议；Linux 内核通过 `PTP_SYS_OFFSET_PRECISE` ioctl 利用 PCIe PTM 硬件交叉时间戳实现网卡 PHC 与系统时钟的零抖动同步。
- **PCIe 时钟架构（Clock Architecture）**（[../architecture/clock-architecture.md](../architecture/clock-architecture.md)）：Common Clock、SRNS、SRIS 等物理参考时钟架构决定链路两端的时钟同源性，而 PTM 则在逻辑层提供统一的绝对时间参考。
- **TLP 消息类型（Message TLP）**（[../transaction-layer/transaction-types.md](../transaction-layer/transaction-types.md)）：PTM Request / Response / ResponseD 均以 PCIe Message TLP 形式在事务层与数据链路层协同传输。
