# L1 子状态详解（L1 PM Substates：L1.1 与 L1.2）

**L1 子状态（L1 PM Substates，简称 L1SS）** 是 PCIe 3.1+ 引入的精细化深度节能机制，在传统 `L1` 状态的基础上进一步细分出 **L1.1** 和 **L1.2**。如果把普通 `L1` 比作电脑“合盖息屏但风扇还在转”，那么 **L1.1** 就是“停掉内部高频 PLL 引擎”，而 **L1.2** 则是“连外部 100 MHz 参考时钟（REFCLK）也一并关停”，从而将空闲功耗压降到极致。

---

## L1.0 / L1.1 / L1.2 核心对比

| 状态 | PLL（锁相环） | 100 MHz REFCLK | 共模电压维持 | 典型功耗占比 | 典型恢复延迟 | 核心依赖 |
|-----|--------------|---------------|-------------|------------|------------|---------|
| **L0** | 运行 | 运行 | 维持 | 100%（基准） | 0 | — |
| **L1.0**（传统 L1） | 运行或部分门控 | 运行 | 维持 | ~10%–20% | < 4 μs | ASPM L1 或 D3hot |
| **L1.1** | **关闭** | 运行 | 维持（或快速恢复） | ~5%–10% | ~8–16 μs | L1SS 能力 |
| **L1.2** | **关闭** | **关闭** | **关闭**（需 `T_CommonMode` 恢复） | ~1%–5% | ~16–100 μs | **CLKREQ#** + **LTR** |

> **两种触发路径（ASPM vs PCI-PM）**：  
> L1.1 和 L1.2 均支持两种进入方式——**ASPM L1.1/L1.2**（设备在 `D0` 工作态下由硬件检测空闲自动进入）和 **PCI-PM L1.1/L1.2**（软件将设备设为 `D3hot` 进入 L1 后，再进一步滑入 L1.1/L1.2）。

---

## L1 PM Substates 扩展能力结构（Cap ID `0x001E`）

L1SS 位于 PCIe 扩展配置空间（`0x100+`），Capability ID 为 **`0x001E`**，包含 4 个 32-bit 寄存器：

| 偏移 | 寄存器名称 | 关键位域与功能 |
|-----|-----------|--------------|
| `+0x00` | Extended Capability Header | Cap ID = `0x001E`，Version = `0x1`，Next Capability Offset |
| `+0x04` | **L1 PM Substates Capabilities** | `[4:0]` 分别声明是否支持 L1SS、ASPM L1.1/L1.2、PCI-PM L1.1/L1.2；<br>`[15:8]` `Port Common_Mode_Restore_Time`（共模电压恢复时间，单位 μs）；<br>`[17:16]` 与 `[23:19]` 声明端口所需的 `T_POWER_ON` Scale/Value |
| `+0x08` | **L1 PM Substates Control 1** | `[3:0]` 分别使能 ASPM L1.1/L1.2 与 PCI-PM L1.1/L1.2；<br>`[15:8]` 配置生效的 `Common_Mode_Restore_Time`；<br>`[28:16]` 配置 `LTR L1.2 Threshold`（Scale + Value） |
| `+0x0C` | **L1 PM Substates Control 2** | `[1:0]` `T_POWER_ON Scale`；`[7:3]` `T_POWER_ON Value` |

### 关键时序参数计算

1. **`Common_Mode_Restore_Time`（共模电压恢复时间）**  
   在 `L1.2` 中，发送器和接收器的高速差分共模偏置电路也会被关闭。退出 `L1.2` 时，必须等待该时间（取链路两端设备声明值的**最大值**），确保信号电平回到合法共模范围后再开始链路训练。
2. **`T_POWER_ON`（时钟上电稳定时间）**  
   控制设备拉低 **CLKREQ#** 申请恢复 100 MHz REFCLK 后，必须等待的稳定时间：
   - 计算公式：`T_POWER_ON = Value × Scale`
   - `Scale`（`[1:0]`）：`00b = 2 μs`、`01b = 10 μs`、`10b = 100 μs`、`11b = Reserved`
   - 例如：`Value = 5, Scale = 01b` → `5 × 10 μs = 50 μs`。

---

## LTR 集成与准入控制

由于 `L1.2` 的唤醒延迟可达数十至上百微秒，为了防止高频延迟敏感型设备误入 `L1.2` 导致性能暴跌，PCIe 规定进入 **ASPM L1.2** 必须配合 **LTR（Latency Tolerance Reporting，延迟容忍报告）**：

```
设备发送 LTR Message（声明当前可容忍的 Max Snoop / No-Snoop Latency）
                         │
                         ▼
       比较：设备当前 LTR 容忍度  vs  LTR_L1.2_Threshold
         │                                       │
   [LTR < Threshold]                       [LTR >= Threshold]
   （业务对延迟敏感，不能睡太深）           （允许较高唤醒延迟）
         │                                       │
         ▼                                       ▼
   最高只进入 L1.0 / L1.1                  允许拉高 CLKREQ# 进入 L1.2
```

> **注意**：使能 LTR 不仅需要在 L1SS Control 1 中设置阈值，还必须在标准 PCIe Capability 的 **Device Control 2（`+0x28`）** 中开启 `LTR Mechanism Enable` 位，否则 LTR 报文无法上报。

---

## Linux 查看与验证命令

```bash
# 查看设备 L1 PM Substates 能力、LTR 阈值及当前 T_POWER_ON 配置
lspci -vvv -s 01:00.0 | grep -A 6 "L1 PM Substates"
# 输出解读：
#   L1SubCap : 硬件支持的子状态及端口声明的恢复时间
#   L1SubCtl1: 当前 OS 启用的 L1.1/L1.2 位与 LTR1.2_Threshold
#   L1SubCtl2: 当前生效的 T_PwrOn 时间

# 启用系统级 L1 子状态节能（需主板 BIOS 与设备均支持）
echo powersupersave > /sys/module/pcie_aspm/parameters/policy
```

---

## 关联与引申概念

- **[ASPM 电源管理](aspm.md)**：了解 `L1.1` / `L1.2` 在整个 ASPM 自动节能体系中的定位。
- **[CLKREQ# 信号与时钟管理](clkreq.md)**：深入 `L1.2` 关断 100 MHz 参考时钟的硬件边带握手协议。
- **[PCIe 电源状态](power-states.md)**：区分 ASPM L1.x 与 D3hot/L2/L3 等设备级电源状态。
- **[Extended Capabilities](../configuration/extended-capabilities.md)**：了解 `0x001E` 等扩展能力寄存器在配置空间中的组织方式。
- **[PCIe 时钟架构](../architecture/clock-architecture.md)**：理解参考时钟（REFCLK）与共模电路在电源管理中的行为。
