# 链路训练过程 (Link Training Process)

**链路训练（Link Training）** 是 PCIe 设备上电或复位后，物理层自动完成“认出对端 → 对齐节奏 → 商量车道与车速 → 调优信号质量”的初始化过程。如果把两个 PCIe 设备比作在嘈杂房间里打电话的两个人，链路训练就是：**先确认对方拿起了听筒（Detect），再通过“喂、喂”对上语速（Polling），接着商量好用几条线路、多快语速（Configuration），最后在提速后微调麦克风音量避免破音（Equalization），正式开始通话（L0）**。

---

## 完整训练时间线

无论设备最终支持 Gen 3、Gen 4 还是 Gen 5，PCIe 链路**总是先在最稳妥的 Gen 1（2.5 GT/s）下完成基础建链**，再逐步提速：

```text
上电 / PERST# 释放
   │
   ▼  1. Detect (检测对端)
   │     发送端检测 RX 端接阻抗 (50Ω)，确认对端芯片在位
   ▼  2. Polling (位锁定 & 符号锁定) — 运行于 2.5 GT/s
   │     互发 TS1 → CDR 锁定比特流与 K28.5 边界 → 互发 TS2 确认
   ▼  3. Configuration (宽度与极性协商) — 运行于 2.5 GT/s
   │     确定 Link/Lane 编号、协商最大公共宽度 (如 x4/x16)
   │     自动纠正 Lane 反转、差分极性反转，完成多 Lane 去偏斜 (Deskew)
   ▼  4. L0 (Gen 1 初始运行)
   │     物理层就绪，DLLP 开始流控初始化
   ▼  5. Recovery.Speed + Equalization (Gen 2/3/4/5 提速)
   │     若双方支持更高速率，进入 Recovery 切换 PLL 速率
   │     Gen 3+ 必须执行 4 阶段链路均衡 (TX FFE + RX CTLE/DFE)
   ▼  6. L0 (目标高速率运行)
         以协商后的最高稳定速率传输 TLP / DLLP
```

---

## 关键训练序列：TS1 与 TS2 Ordered Set

在 `Polling`、`Configuration` 和 `Recovery` 阶段，物理层不发数据包，而是持续发送固定长度的 **TS1 / TS2 训练序列（Training Sequence）** 来传递握手信息：

| 符号位置 | Gen 1/2 (8b/10b) 内容 | Gen 3+ (128b/130b) 内容 | 作用说明 |
| :--- | :--- | :--- | :--- |
| **Symbol 0** | `K28.5` (COM) | Ordered Set 标识 (`1E` / `2D` 等) | 标记训练序列起始边界，用于符号对齐 |
| **Symbol 1** | Link Number (`PAD` 或编号) | Link Number | 协商链路编号（通常为 0） |
| **Symbol 2** | Lane Number (`PAD` 或编号) | Lane Number | 协商物理 Lane 编号（用于去偏斜与 Lane 反转检测） |
| **Symbol 3** | `N_FTS` | `N_FTS` | 退出 `L0s` 省电状态所需的 FTS 序列数量 |
| **Symbol 4** | Data Rate Identifier | Data Rate Identifier | 声明双方各自支持的速率（2.5 / 5 / 8 / 16 / 32 / 64 GT/s） |
| **Symbol 5** | Training Control | Training Control | 控制位：Hot Reset、Disable Link、Loopback、Disable Scrambling 等 |
| **Symbol 6–15** | TS1 (`0x4A`) / TS2 (`0x45`) 重复 | TS1 / TS2 标识与**均衡系数/Preset** | Gen 3+ 在此处携带 TX 均衡请求与反馈 |

> **极性反转（Polarity Inversion）如何自动解决？**
> PCB 布线时如果把 `RX+` 和 `RX-` 接反了，收到的 TS1 标识字节（`0x4A`）会变成按位取反的 `0xB5`。接收端检测到连续反相的 TS1/TS2 后，会自动在内部将差分信号极性翻转回来，无需软件干预。

---

## 常见训练失败现象与排查

| 现象 | 常见原因 | 排查思路 |
| :--- | :--- | :--- |
| **设备完全识别不到（`lspci` 无显示）** | 卡在 `Detect` 或 `Polling`：供电/参考时钟缺失、`PERST#` 未释放、金手指接触不良 | 检查插槽供电、100 MHz 时钟及 `dmesg` 热插拔日志 |
| **速率降级（如 Gen 4 降为 Gen 3/Gen 1）** | `Recovery.Speed` 或均衡失败：PCB 走线过长、损耗过大、BIOS 限制了 Max Link Speed | 用 `lspci -vvv` 对比 `LnkCap` 与 `LnkSta`；检查 BIOS 设置 |
| **宽度降级（如 x16 降为 x8/x4）** | `Configuration` 阶段部分 Lane 无法锁定：主板插槽电气仅接通 x8、个别引脚虚焊或损坏 | 检查主板通道分配表，清洁金手指或更换插槽测试 |

```bash
# 查看当前设备链路训练结果
lspci -vvv -s 01:00.0 | grep -E "LnkCap:|LnkSta:|LnkSta2:"
```

---

## 关联与引申概念

- **LTSSM 状态机**：链路训练所遵循的 11 个主状态与转换条件，详见 [LTSSM 状态机](ltssm.md)。
- **链路均衡（Link Equalization）**：Gen 3+ 提速时 TS1/TS2 中交换均衡系数的具体流程，详见 [链路均衡](link-equalization.md)。
- **链路宽度与 Lane 反转**：`Configuration` 阶段如何协商 x1–x32 及自动纠正倒序布线，详见 [链路宽度](link-width.md)。
- **CDR（时钟数据恢复）**：`Polling` 阶段依赖跳变沿完成位锁定的底层电路，详见 [物理层概述](overview.md)。
- **Retimer**：当走线过长导致训练失败时，如何在链路中间重定时信号，详见 [Retimer 详解](retimer.md)。
