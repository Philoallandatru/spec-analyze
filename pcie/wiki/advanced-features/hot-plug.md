# PCIe 热插拔（Hot-Plug）

> **一句话总结**：PCIe 热插拔允许服务器在不断电、不重启的情况下动态插拔扩展卡或 U.2/E1.S NVMe 固态硬盘，通过硬件在位检测、受控上下电时序与内核 `pciehp` 驱动协同完成设备枚举与卸载。

如果把系统比作一辆正在高速行驶的汽车，热插拔就像"不停机更换轮胎"——不仅要有机械卡扣和独立供电开关确保安全，还要提前通知行车电脑（操作系统）停止使用这个轮子、切断动力、换上新胎后再重新校准接入。

---

## 两种热插拔机制对比

| 机制 | 控制主体 | 典型场景 | 特点 |
| :--- | :--- | :--- | :--- |
| **Native Hot-Plug**（原生热插拔） | PCIe 规范标准寄存器 + OS 原生驱动（Linux `pciehp`） | 现代服务器 NVMe U.2/E1.S 背板、PCIe 热插拔插槽 | 标准化程度高、响应快、不依赖复杂 BIOS ACPI ASL 代码 |
| **ACPI-based Hot-Plug** | 平台固件（BIOS/BMC）通过 ACPI AML/事件通知 OS | 旧式服务器、部分虚拟化/特定 OEM 定制背板 | 由固件封装底层电源/I2C 控制细节 |

本文重点介绍 PCIe 规范定义的 **Native Hot-Plug** 机制。

---

## 硬件组件与标准寄存器

Native Hot-Plug 控制器集成在 **Root Port** 或 **Switch Downstream Port** 的 PCIe Capability 结构中，与物理插槽上的传感器和指示灯联动：

| 物理插槽组件 | 作用 |
| :--- | :--- |
| **Presence Detect（在位检测）** | 通过专用的 `PRSNT1#`/`PRSNT2#` 引脚（带内/带外）感知卡是否已完全插入 |
| **Power Controller（电源控制器）** | 独立控制插槽的主电源（12V / 3.3V）和辅助电源（3.3Vaux）通断，防浪涌与过流保护 |
| **Attention Button（注意按钮）** | 供运维人员按下的物理请求按钮，触发有序热移除/热插入流程 |
| **MRL Sensor（机械锁存传感器）** | 检测固定卡扣（Latch）是否扣紧，防止带电状态下设备被意外拉出 |
| **Attention / Power Indicator** | LED 指示灯，用常亮、闪烁、熄灭告知运维人员当前是否可安全插拔 |

### 核心寄存器一览（位于 PCIe Capability 结构内）

| 寄存器 | 关键字段 | 作用 |
| :--- | :--- | :--- |
| **Slot Capabilities** | `Hot-Plug Capable`、`Hot-Plug Surprise`、`Power Controller Present`、`Attention Button Present`、`MRL Sensor Present`、`Physical Slot Number`、`Slot Power Limit` | 只读，向 OS 声明该插槽支持哪些热插拔硬件特性及最大供电功率 |
| **Slot Control** | `Power Controller Control`（0=上电, 1=断电）、`Attention/Power Indicator Control`（On/Blink/Off）、各类事件中断使能位、`Hot-Plug Interrupt Enable` | 读写，OS 下发上电/断电、开关指示灯命令及开启中断 |
| **Slot Status** | `Presence Detect State`、`MRL Sensor State`、`Data Link Layer State Changed (DLLSC)`、`Presence Detect Changed (PDC)`、`Attention Button Pressed`、`Power Fault Detected`、`Command Completed` | 读/写1清零（RW1C），报告插槽实时物理状态与触发中断的事件标志 |

---

## 热插拔标准工作流程

### 1. 热插入（Hot-Add）流程

```text
物理插入设备 ──▶ Presence Detect 触发 PDC 中断
                      │
                      ▼
内核 pciehp 响应 ──▶ 闪烁指示灯 ──▶ 写 Slot Control 给插槽上电
                                          │
                                          ▼
                   供电稳定 + 释放 PERST# ──▶ 物理层链路训练 (LTSSM → L0)
                                          │
                                          ▼
                   Data Link Layer Active ──▶ OS 枚举配置空间、分配 Bus/BAR/MSI-X
                                          │
                                          ▼
                   加载设备驱动（如 nvme）──▶ 电源指示灯常亮，设备就绪
```

### 2. 有序热移除（Orderly Hot-Remove）与意外热移除（Surprise Removal）

```text
【有序移除（推荐）】
按下 Attention Button（或 echo 1 > /sys/bus/pci/slots/<N>/power）
  ──▶ 驱动刷新缓存、停止下发新 I/O、卸载驱动并释放 BAR/IRQ
  ──▶ 写 Slot Control 切断插槽电源（断言 PERST# → 关主电源）
  ──▶ 指示灯熄灭，提示可安全拔出设备

【意外移除（Surprise Removal，需 Hot-Plug Surprise 支持）】
用户未通知直接拔出设备
  ──▶ 链路瞬间掉线（Link Down / PDC 事件）
  ──▶ 驱动立即中止所有在途 DMA/TLP（避免阻塞总线或触发系统 MCE）
  ──▶ 向上层返回 I/O 错误，清理设备节点与资源
```

### 关键上下电电气时序

- **上电时序**：辅助电源（3.3Vaux）就绪 → 主电源（3.3V/12V）按序上电 → `PERST#` 复位信号保持低电平至少 **100 ms** 等待电源与参考时钟稳定 → 释放 `PERST#` → 启动 LTSSM 链路训练。
- **断电时序**：软件停止所有 DMA 与事务 → 断言 `PERST#`（拉低）→ 等待至少 **1 ms** → 关闭主电源 →（若需）关闭 Aux 电源，等待电容放电。

---

## 常用 Linux 热插拔管理与排查命令

```bash
# 查看当前系统所有 PCIe 热插拔插槽及供电状态
grep -H . /sys/bus/pci/slots/*/power

# 查看指定 Root Port 的 Slot Capabilities / Control / Status
lspci -vvv -s 00:01.0 | grep -A 8 "SltCap:"

# 手动触发有序断电（安全移除前）与重新上电扫描
echo 0 > /sys/bus/pci/slots/1/power   # 安全断电并卸载设备
echo 1 > /sys/bus/pci/slots/1/power   # 重新上电并枚举
```

**常见问题排查**：
- **插入后无反应**：检查 BIOS 是否开启 Native Hot-Plug / `pciehp` 驱动是否加载；确认 `SltCap` 中 `HotPlug+` 已声明，以及 `Presence Detect` 引脚状态是否变化。
- **拔出导致系统宕机（Kernel Panic / AER Fatal）**：通常是未启用 `Surprise Removal` 或设备仍有悬空 DMA 请求未完成时被强行断电，应先执行软件卸载（`echo 0 > power`）或确保驱动正确处理 Link Down。

---

## 关联与引申概念

- **LTSSM（链路训练与状态机）**（[../physical-layer/ltssm.md](../physical-layer/ltssm.md)）：插槽上电释放 `PERST#` 后，物理层通过 `Detect → Polling → Configuration → L0` 完成链路训练，Data Link Layer 激活后触发 `DLLSC` 事件通知热插拔驱动开始枚举。
- **PCIe 设备枚举（Enumeration）**（[../architecture/enumeration.md](../architecture/enumeration.md)）：热插入时，内核需要为新设备动态分配预留的 Bus 号与 MMIO/BAR 地址窗口（常需 BIOS 预留 Hot-Plug Bus/Memory Padding）。
- **电源管理与复位（Power States & PERST#）**（[../power-management/power-states.md](../power-management/power-states.md)）：热插拔的受控上下电严格依赖电源状态转换与 `PERST#` 复位时序规范。
- **AER（高级错误报告）**（[../error-handling/aer.md](../error-handling/aer.md)）：意外热拔出时可能伴随 Surprise Down / Completion Timeout 等错误，热插拔驱动需与 AER 协同屏蔽预期内的 Link Down 误报。
