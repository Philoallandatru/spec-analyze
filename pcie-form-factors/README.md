# PCIe 机械形态、供电与边带信号（CEM / EDSFF / M.2）

本模块基于 **PCIe Card Electromechanical (CEM)**、**EDSFF（SFF-TA-1002 / E1.S / E3.S）** 与 **PCIe M.2 Specification**，拆解软件工程师最容易忽略的**物理边界：插槽供电上限、热插拔短针（Staggered Pin）探测、以及决定能否上电和省电的边带信号（`PERST#`、`PRSNT#`、`CLKREQ#`、`PWRBRK#`、`PLN#`）**。

> **20/80 裁剪原则**：聚焦供电轨与功耗上限、边带信号时序逻辑、1C/2C/4C 与 M.2 Key M 引脚差异，不展开具体机械公差与 CAD 尺寸图。

---

## 文档导航

1. **[PCIe CEM 插槽供电与边带信号时序](pcie-cem-and-sideband.md)**
   - AIC 插卡 10W/25W/75W 供电上限、`PRSNT1#/PRSNT2#` 短针热插拔防拉弧、`PERST#`/`WAKE#`/`CLKREQ#`/`PWRBRK#` 时序。
2. **[EDSFF（E1.S / E3.S）企业级架构与 1C/2C 连接器](edsff-e1-e3-architecture.md)**
   - SFF-TA-1002 1C/2C/4C 引脚扩展、12V 主供电（25W–80W+）+ `3.3Vaux`、Dual-Port 双活、热插拔与 LED 指示。
3. **[M.2 Key M（NVMe）引脚、3.3V 单轨与 L1.2 功耗](m2-key-m-and-power.md)**
   - M.2 Key M（PCIe x4）引脚定义、`3.3V` 单轨限制、`PEDET`/`SUSCLK`/`PLN#` 与 `CLKREQ#` 控制的 L1.2 省电陷阱。
4. **[U.2 / U.3 三模背板、MCIO 线缆与 PCIe Retimer](u2-u3-mcio-and-retimer.md)**
   - `SFF-8639` vs `SFF-TA-1001` 互插兼容规则、MCIO 高密铜缆与 Gen5/6 Retimer 信号重定时。

---

## 关联与引申概念

- **PCIe 电气层基础**：`PERST#`、`WAKE#`、`CLKREQ#` 在 PHY 层的定义，详见 [PCIe 电气规范](../pcie/wiki/physical-layer/electrical-spec.md)。
- **上电枚举流程**：从 `Power Good` 与 `PERST#` 释放到 UEFI 枚举，详见 [上电、UEFI/ACPI 与 PCIe 枚举](../host-system/boot-and-enumeration.md)。
- **带外管理与 3.3Vaux**：BMC 如何利用辅助供电在关机时读写 SSD，详见 [MCTP 传输层与 PCIe VDM / SMBus 封装](../nvme-mi/mctp-transport-and-vdm.md)。
