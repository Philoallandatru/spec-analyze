# PCIe CEM 插槽供电与边带信号时序

**PCIe CEM（Card Electromechanical Specification）** 规定了台式机/服务器主板上标准立式 **Add-in Card（AIC）** 插槽的机械尺寸、金手指引脚、供电上限与边带控制信号 `[PCIe CEM §2–4]`。

可以把 CEM 金手指想成**插卡与主板的“安检门 + 电源插座”**：高速差分对负责传数据，而边带引脚负责确保“插稳了再通电、时钟稳了再放行复位、超功率了立刻降频”。

---

## 插槽供电轨与功率上限

| 插槽 / 形态 | $+12\text{V}$ 上限 | $+3.3\text{V}$ 上限 | 总槽位功率上限 | 备注 |
| :--- | :--- | :--- | :--- | :--- |
| **x1 标准卡** | 0.5A (6W) | 3.0A (9.9W) | **10W**（半高可扩展至 25W） | 仅靠槽位供电 |
| **x4 / x8 卡** | 2.1A (25W) | 3.0A (9.9W) | **25W** | 企业级 AIC NVMe 加速卡常见 |
| **x16 全长/高功率卡** | 5.5A (66W) | 3.0A (9.9W) | **75W** | 超出 75W 需外接 6-pin (75W) / 8-pin (150W) / 12V-2x6 (600W) |
| **辅助供电 `3.3Vaux`** | — | 375mA (唤醒启用) / 20mA | **~1.2W** | 系统关机（S3/S5）时维持 `WAKE#` 或 BMC 管理逻辑 |

---

## 核心边带信号（Sideband Signals）与热插拔时序

| 引脚名 | 方向 | 核心作用与工程意义 |
| :--- | :--- | :--- |
| **`PRSNT1#` / `PRSNT2#`** | 卡 → 主板 | **在位检测短针（Staggered Pin）**：金手指刻意做短，插卡时**最后接触**、拔卡时**最先断开**。保证卡完全插稳后主板才上主电，拔出瞬间提前断电防拉弧。 |
| **`PERST#`** | 主板 → 卡 | **基本复位（Fundamental Reset）**：低电平有效。上电时必须等 `12V/3.3V` 稳定且 `100 MHz REFCLK` 稳定至少 **`100ms`（$T_{\text{PVPERL}}$）** 后才允许拉高释放。 |
| **`CLKREQ#`** | 卡 ↔ 主板 | **参考时钟请求**：低电平有效。进入 **ASPM L1.2** 时双方拉高 `CLKREQ#` 关断 100 MHz 时钟省电；任一方需要发包时拉低请求恢复时钟。 |
| **`WAKE#`** | 卡 → 主板 | **带外唤醒**：处于 `D3cold` 的设备拉低此信号，请求主板恢复供电。 |
| **`PWRBRK#`** | 主板 → 卡 | **紧急降功率（Power Break）**：低电平有效。服务器电源（PSU）即将过载或风扇故障时拉低此脚，强制高功耗卡在微秒级内紧急降频/限流。 |
| **`SMCLK` / `SMDAT`** | 双向 | **SMBus 管理总线**：供 BMC/PCH 读取卡上温度、VPD 或运行 MCTP over SMBus。 |

```text
标准上电与热插拔时序：
  PRSNT1#/2# 接通 ──▶ 主板开启 12V/3.3V ──▶ REFCLK 100MHz 稳定 (>=100ms) ──▶ 拉高释放 PERST# ──▶ LTSSM Detect → L0
```

```bash
# 查看 PCIe 设备协商的 Slot Power Limit 与当前 Link 状态
lspci -vvv -s 0000:01:00.0 | grep -E "SlotCap|DevCap|LnkSta:"
```

---

## 关联与引申概念

- **物理层电气参数**：`PERST#` 与参考时钟抖动指标，详见 [PCIe 电气规范](../pcie/wiki/physical-layer/electrical-spec.md)。
- **上电枚举**：固件在 `PERST#` 释放后如何扫描设备，详见 [上电、UEFI/ACPI 与 PCIe 枚举](../host-system/boot-and-enumeration.md)。
- **L1.2 功耗交互**：`CLKREQ#` 与 NVMe APST 配合时的超时坑，详见 [PCIe 复位层级、FLR 与电源状态交互](../nvme-pcie-transport/pcie-reset-power-and-flr.md)。
