# EDSFF（E1.S / E3.S）企业级架构与 1C/2C 连接器

过去服务器前部硬盘槽长期沿用为机械盘设计的 U.2（SFF-8639），在 PCIe Gen 5/6 高速信号完整性、高密度散热与 40W–70W 高功耗面前已力不从心。**EDSFF（Enterprise and Datacenter Standard Form Factor）** 基于统一的 **SFF-TA-1002** 高速连接器标准，成为现代全闪存服务器的绝对主流 `[EDSFF Spec §1–2]`。

---

## E1.S / E1.L vs E3.S / E3.L 家族定位

| 形态 | 典型尺寸与机箱适配 | 常见厚度档位 | 典型最大功耗 | 核心定位 |
| :--- | :--- | :--- | :--- | :--- |
| **`E1.S`** | 短尺（约 111.5mm 长），**1U 服务器垂直密插** | 5.9mm / 8.0mm / 9.5mm / 15mm / 25mm | **12W – 40W** | 超大规模数据中心主力计算/存储盘（1U 可插 32 盘） |
| **`E1.L`** | 长尺（约 318.75mm 长），1U 高容量 | 9.5mm / 18mm | 25W – 40W+ | 超大容量冷/温数据存储（QLC 高密盘） |
| **`E3.S`** | 2U 替代 U.2（约 112.75mm 长） | 7.5mm (1T) / 16.8mm (2T) | **25W – 70W+** | 高性能 PCIe Gen5/Gen6 x4/x8、**双活（Dual-Port）**与 CXL 设备 |
| **`E3.L`** | 2U 加长版（约 142.2mm 长） | 7.5mm / 16.8mm | 40W – 70W+ | 超大容量或高功耗计算存储加速卡 |

---

## SFF-TA-1002 连接器：1C / 2C / 4C 与供电设计

EDSFF 最大的工程优雅在于**全家族共用同一种金手指接口定义（SFF-TA-1002）**，只需按需增加宽度即可扩展通道与供电 `[SFF-TA-1002 §4]`：

- **`1C` 连接器（56 Pin，每面 28 Pin）**：支持 **PCIe x4**（单端口或双端口 x2+x2），`E1.S` 标配。
- **`2C` 连接器（84 Pin，每面 42 Pin）**：支持 **PCIe x8**（或完整双活 x4+x4），`E3.S` / `E3.L` 常用。
- **`4C` / `4C+` 连接器（140/168 Pin）**：支持 **PCIe x16**，用于 CXL 内存扩展卡或高算力加速器。

### 为什么放弃 3.3V 主供电，全面拥抱 `12V` + `3.3Vaux`？

1. **主电源纯 `12V`（$12\text{V} \pm 10\%$）**：与 M.2 使用 `3.3V` 不同，同样 40W 功耗下，用 `12V` 电流只需 `3.3A`，而用 `3.3V` 电流高达 `12A`（铜损与压降巨大）。`1C`/`2C` 分配了多组大电流 `12V` 引脚，轻松支撑 25W–80W 级设备。
2. **`3.3Vaux`（$3.3\text{V} \pm 5\%$）**：专供 **SMBus/I3C 管理芯片、MCTP/NVMe-MI 带外监控与 `PRSNT#` 在位逻辑**，即使主板还没给该盘槽上主 12V 电，BMC 也能读出盘的 VPD 资产信息。

---

## 关键企业级边带引脚

除标准 `PERST#`、`CLKREQ#`、`PRSNT0#/PRSNT1#`、`SMCLK/SMDAT` 外，EDSFF 还标准化了企业级运维必备引脚：
- **`PWRDIS`（Power Disable）**：允许背板 CPLD/BMC 远程对单个盘槽强制断电重启（Hard Power Cycle），挽救彻底卡死的固件，无需人工去机房拔盘。
- **`PLN#`（Power Loss Notification）**：主板/电源掉电瞬间提前数毫秒拉低通知 SSD，触发板载电容 **PLP（掉电保护）** 将缓存紧急刷入 NAND。
- **`DUALPORTEN#`**：指示该盘工作在单端口 x4 还是双活（Dual-Port）x2+x2 模式。
- **LED 指示规范**：由背板或盘控结合管理信号驱动 **绿灯（Activity/Power）** 与 **琥珀灯（Fault/Locate）**。

---

## 关联与引申概念

- **带外管理通道**：BMC 如何通过 `3.3Vaux` 与 `SMCLK/SMDAT` 轮询 EDSFF 盘健康，详见 [NVMe-MI 架构、带内/带外通道与消息类型](../nvme-mi/nvme-mi-architecture-and-messages.md)。
- **双活与多路径**：EDSFF `2C` 支持的 Dual-Port 在 NVMe 协议层的访问模型，详见 [命名空间访问模型](../nvme/concepts/namespace-access-models.md)。
- **CXL 复用**：E3.S 2T 形态是 CXL 3.x 内存扩展模块的主流载体，详见 [CXL 3.x 架构与协议复用](../host-system/cxl-architecture-overview.md)。
