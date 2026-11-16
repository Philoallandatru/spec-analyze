# CLKREQ# 信号与时钟管理

**CLKREQ#（Clock Request，时钟请求）** 是 PCIe 插槽或芯片间的一条**低电平有效（Active Low）边带信号（Sideband Signal）**，专门用于动态申请或释放 **100 MHz 参考时钟（REFCLK）**。如果把参考时钟发生器比作大楼里持续耗电的“中央空调鼓风机”，那么 `CLKREQ#` 就是设备手里的**“按需开关呼叫键”**：进入深度睡眠（如 **L1.2**）时，设备释放 `CLKREQ#`（拉高）通知主板关掉鼓风机；需要干活时再拉低 `CLKREQ#`，等时钟稳定后立刻恢复全速通信。

---

## 信号特性与系统连接

与走高速差分线的 PCIe 数据通道（TX/RX）不同，`CLKREQ#` 是一条独立的单端控制线：

| 属性 | 说明 |
|-----|------|
| **信号类型** | 边带信号（Sideband），独立于高速差分对之外（如 M.2 / Mini PCIe 均有专用引脚） |
| **电平逻辑** | **低电平有效（Active Low）**：`Low` = 请求保持/开启时钟；`High` = 释放/不需要时钟 |
| **方向与机制** | 传统单向模式为“设备 → 平台”；在 **L1.2（L1 PM Substates）** 下通常采用开漏双向（Wired-AND）协调 |
| **核心用途** | 控制主板时钟发生器启停 **100 MHz REFCLK**，是实现 **L1.2 超低功耗** 的硬件前提 |

```
┌───────────────────────────────────────────────┐
│              Platform（主板/RC 侧）            │
│   Clock Generator（100 MHz REFCLK 发生器）     │
└───────────┬───────────────────────▲───────────┘
            │ REFCLK (100 MHz)      │ CLKREQ# (Low = 请求, High = 释放)
            ▼                       │
┌───────────────────────────────────┴───────────┐
│               PCIe Device（设备侧）             │
│  • L0 / L0s / L1.0 / L1.1：拉低 CLKREQ#，使用时钟│
│  • L1.2 / L2 / L3：释放 CLKREQ#，允许平台停钟   │
└───────────────────────────────────────────────┘
```

---

## L1.2 中的 CLKREQ# 握手与时序

在普通 `L1` 或 `L1.1` 状态下，设备虽然关闭了内部 **PLL（锁相环）**，但主板仍在持续输出 100 MHz REFCLK。只有进入 **L1.2** 时，才会通过 `CLKREQ#` 彻底关停 REFCLK：

### 进入与退出流程

```
[进入 L1.2]
L0 活动态 ──▶ 链路空闲，协商进入 L1 ──▶ 关闭内部 PLL ──▶ 释放 CLKREQ#（拉高）
                                                              │
                                                              ▼
                                                  主板检测到 High，关闭 REFCLK（进入 L1.2）

[退出 L1.2]
唤醒事件触发 ──▶ 拉低 CLKREQ#（Assert）──▶ 主板重启 REFCLK（耗时 T_clkon）
                                                    │
                                                    ▼
                    等待至少 T_POWER_ON ──▶ 启动 PLL 并重训练链路 ──▶ 恢复 L0
```

### 关键时序参数：`T_POWER_ON`

由于主板时钟发生器从断电到输出稳定的 100 MHz 时钟需要一定启动时间（`T_clkon`，通常 10~100 μs），设备在拉低 `CLKREQ#` 后**不能立刻发包**，必须等待寄存器中协商好的 **`T_POWER_ON`** 时间：

- 该参数位于 **L1 PM Substates Control 2（`+0x0C`）** 寄存器：
  - `T_POWER_ON = Value × Scale`
  - `Scale`（`[1:0]`）：`00b` = `2 μs`，`01b` = `10 μs`，`10b` = `100 μs`，`11b` = 保留
  - `Value`（`[7:3]`）：5-bit 整数乘数（例如 `Value=5, Scale=01b` → `50 μs`）

> **常见故障原因**：如果 `T_POWER_ON` 配得比主板时钟发生器实际稳定时间 `T_clkon` 还短，设备就会在时钟尚未稳定时强行启动链路训练，直接导致唤醒失败、设备掉线或系统卡死。

---

## 平台时钟源架构与多设备共享

主板在设计 `CLKREQ#` 时通常有三种平台策略：**立即关钟**（最省电）、**延迟关钟**（防频繁开关抖动）和 **多设备合并**。

| 时钟架构 | 连接方式 | 优缺点 |
|---------|---------|-------|
| **独立时钟源** | 每个 PCIe 插槽拥有独立的 `CLKREQ#` 与时钟输出通道 | 各设备互不干扰，省电最彻底；但主板 BOM 成本较高 |
| **共享时钟源** | 多个设备的 `CLKREQ#` 通过逻辑门合并（任一设备拉低即请求时钟） | 成本低、布线简单；但只要有一个设备在工作，其余设备就无法关闭 REFCLK |

---

## Linux 检查与调试命令

```bash
# 1. 查看 L1 PM Substates 能力与 T_POWER_ON 配置
lspci -vvv -s 01:00.0 | grep -A 5 "L1 PM Substates"
# 关注输出中的：
#   PortTPowerOnTime=50us  (端口声明的时钟恢复时间)
#   T_PwrOn=50us           (当前生效的 T_POWER_ON)

# 2. 开启深度省电策略以触发 L1.2 与 CLKREQ# 动作
echo powersupersave > /sys/module/pcie_aspm/parameters/policy

# 3. 观察设备运行时电源状态变化
cat /sys/bus/pci/devices/0000:01:00.0/power/runtime_status
```

### 常见问题排查速查

| 现象 | 常见原因 | 排查方向 |
|-----|---------|---------|
| 支持 L1.2 但实际无法进入 | BIOS 禁用了 `CLKREQ#`、主板引脚未连接，或 ASPM 策略未设为 `powersupersave` | 检查 BIOS 电源选项、核对 `policy` 及 `lspci` 中 `ASPM_L1.2+` 是否已使能 |
| 从 L1.2 唤醒后设备无响应 | `T_POWER_ON` 设置过短，REFCLK 尚未稳定 | 核对上下游端口的 `PortTPowerOnTime` 取较大值，或更新 BIOS |
| 随机读写性能明显下降 | 业务间隔恰好频繁触发 L1.2 进出，累加 `T_POWER_ON` 延迟 | 调高 **LTR** 阈值，或退回仅使用 `L1.1`（保留 REFCLK）或 `performance` 策略 |

---

## 关联与引申概念

- **[L1 子状态详解](l1-substates.md)**：深入了解 `L1.1` 与 `L1.2` 的寄存器位域、**LTR** 阈值及 **Common_Mode_Restore_Time**。
- **[ASPM 电源管理](aspm.md)**：从整体上理解 `CLKREQ#` 在自动链路节能中的位置。
- **[PCIe 时钟架构](../architecture/clock-architecture.md)**：理解 Common Clock（共时钟）与 SRNS/SRIS（独立扩频时钟）模式下参考时钟的设计差异。
- **[PCIe 电源状态](power-states.md)**：了解设备在 `L2`/`L3` 深度休眠态下 `CLKREQ#` 与电源轨的状态。
