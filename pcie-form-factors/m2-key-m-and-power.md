# M.2 Key M（NVMe）引脚、3.3V 单轨与 L1.2 功耗

**M.2（前称 NGFF）** 是笔记本、台式机与部分服务器启动盘（Boot Drive）最常见的紧凑型板载形态。其中 **Key M**（第 59–66 引脚位置开缺口防呆）专门用于 **PCIe x4 NVMe SSD**（常见尺寸 `2280`，即宽 22mm × 长 80mm） `[PCIe M.2 Spec §3]`。

---

## M.2 Key M vs EDSFF / CEM 核心差异

| 对比维度 | M.2 Key M | EDSFF（E1.S / E3.S） |
| :--- | :--- | :--- |
| **主供电轨** | **仅 `3.3V` 单轨**（无 12V） | **`12V` 主供电 + `3.3Vaux`** |
| **散热与功耗墙** | 通常 **5W – 12W**（无主动风道极易过热降频） | **25W – 70W+**（带金属外壳与机箱强风道） |
| **热插拔** | **不支持原生热插拔**（需拧螺丝固定，无短针防拉弧设计） | **原生支持前部免工具热插拔** |
| **适用场景** | 消费级 PC、笔记本、服务器内部 OS 启动盘（如 `2280`/`22110`） | 企业级数据中心高密热插拔业务盘 |

---

## M.2 Key M 关键引脚与工程细节

M.2 共有 75 个金手指引脚（双面），除 4 对 PCIe TX/RX 差分线与 1 对 `REFCLK` 外，日常硬件/固件联调最关注以下引脚 `[PCIe M.2 Spec §3.2]`：

- **`3.3V`（多引脚并联）**：因为电压只有 `3.3V`，PCIe Gen 4/5 满载 8–11W 时电流可达 **2.5A–3.5A**，主板供电纹波与接触阻抗稍差就会导致随机掉盘。
- **`PEDET`（PCIe Detect，Pin 69）**：多协议 M.2 插槽用来区分插进来的是 **PCIe 盘**（悬空/上拉）还是 **SATA 盘**（接地），从而决定主板芯片组（PCH）把该接口路由到 PCIe 控制器还是 SATA 控制器。
- **`SUSCLK`（32.768 kHz，Pin 50）**：主板在进入低功耗睡眠（Modern Standby / S0ix）时提供的 32kHz 慢时钟，帮助 SSD 内部定时刷新等后台维持逻辑极低功耗运行。
- **`PERST#`（Pin 50 附近/Pin 52） & `CLKREQ#`（Pin 54）**：控制基本复位与 **ASPM L1.2** 时钟门控。
- **`PLN#`（Power Loss Notification，Pin 12，企业级 `22110` 常用）**：通知具有 PLP 电容的企业级 M.2 启动盘（如双 M.2 RAID1 启动卡）紧急刷盘。

> **排错提示（笔记本 / 桌面级 L1.2 掉盘）**：M.2 盘高度依赖 `CLKREQ#` 与 **L1.2 Substate** 来达成毫瓦级待机功耗，若主板 BIOS 对 `CLKREQ#` 上拉电阻或 $T_{\text{POWER\_ON}}$（电源恢复时间）配置过短，唤醒时极易触发 `NVMe I/O timeout` 进而整个盘从 `/dev/nvme0` 消失。

```bash
# 查看 M.2 NVMe 盘的 L1.2 能力、T_POWER_ON 参数与当前功耗状态
lspci -vvv -s 0000:01:00.0 | grep -A 8 "L1Sub"
nvme get-feature /dev/nvme0 -f 0x02 -H
```

---

## 关联与引申概念

- **L1.2 与复位交互**：`CLKREQ#` 与 NVMe APST 在软件层的协同，详见 [PCIe 复位层级、FLR 与电源状态交互](../nvme-pcie-transport/pcie-reset-power-and-flr.md)。
- **ASPM 原理**：PCIe 数据链路层/物理层 L0s/L1/L1.1/L1.2 转换机制，详见 [ASPM 详解](../pcie/wiki/power-management/aspm.md)。
- **企业级替代方案**：为什么服务器业务盘用 EDSFF 取代 M.2，详见 [EDSFF 架构与连接器](edsff-e1-e3-architecture.md)。
