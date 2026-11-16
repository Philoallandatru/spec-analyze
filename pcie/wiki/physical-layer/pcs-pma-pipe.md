# PCS、PMA 与 PIPE 接口

在芯片设计（ASIC / SoC / FPGA）中，PCIe 物理层并不是铁板一块，而是被清晰地拆分为**数字逻辑部分（PCS）**与**模拟高速收发部分（PMA / PMD）**，两者之间通过标准的 **PIPE 接口** 通信。如果把物理层比作快递分拣中心，**PCS** 是按条形码打包、对齐、校对的“分拣流水线”，**PMA** 是把包裹装车发上高速公路的“装卸码头”，而 **PIPE** 就是两者之间的标准传送带接口。

---

## 物理层内部子层划分

```text
                    来自数据链路层 (TLP / DLLP)
                                │
┌───────────────────────────────▼───────────────────────────────┐
│  PCS (Physical Coding Sublayer，物理编码子层 - 数字逻辑)         │
│  • 8b/10b 或 128b/130b 编解码、扰码/解扰 (Scrambler)            │
│  • 多 Lane 条带化 (Byte Striping) 与去偏斜 (Lane Deskew)         │
│  • 弹性缓冲 (Elastic Buffer) 与 SKP 频偏补偿                    │
│  • LTSSM 状态机与 TS1/TS2/EIOS/EIEOS 生成检测                  │
└───────────────────────────────┬───────────────────────────────┘
                                │  ◀── PIPE 并行接口 (标准信号总线)
┌───────────────────────────────▼───────────────────────────────┐
│  PMA (Physical Media Attachment，物理介质附加子层 - 混合/模拟)   │
│  • SerDes 串并转换 (PISO / SIPO) 与 PLL 倍频                    │
│  • 时钟数据恢复 (CDR) 与比特流采样                               │
│  • 发送端 FFE 预加重 / 接收端 CTLE + DFE 均衡                   │
│  • 电气空闲 (Electrical Idle) 与接收端在位检测 (Rx Detect)       │
└───────────────────────────────┬───────────────────────────────┘
                                │
┌───────────────────────────────▼───────────────────────────────┐
│  PMD (Physical Media Dependent，物理介质相关子层 - 模拟前端)    │
│  • 差分驱动器/接收器 (TX+/TX-, RX+/RX-)、100 Ω 端接电阻          │
└───────────────────────────────┬───────────────────────────────┘
                                ▼
                         PCB 铜线 / 金手指
```

---

## PIPE 接口（PHY Interface for PCI Express）

**PIPE** 是由 Intel 制定并被业界广泛采纳的事实标准，它把“协议控制器 IP（MAC + PCS）”与“模拟 SerDes IP（PMA）”解耦，使得控制器厂商和模拟 PHY 厂商可以独立研发并无缝对接。

| 关键信号方向 | 典型信号 | 作用说明 |
| :--- | :--- | :--- |
| **数据通道** | `TxData` / `RxData`（含 `TxDataK`） | 并行收发数据（如 8/16/32/64 bit 宽，随速率降频传递） |
| **时钟与速率** | `PCLK`、`Rate` | 由 PHY 输出给 MAC/PCS 的并行参考时钟；`Rate` 指示切换 Gen 1–6 |
| **低功耗控制** | `PowerDown[1:0]` | 控制 PHY 进入 `P0`（正常）、`P0s`（L0s）、`P1`（L1）或 `P2`（L2） |
| **电气空闲** | `TxElecIdle`、`RxElecIdle` | 通知发送端进入高阻静默态，或上报接收端检测到静默 |
| **在位与状态** | `TxDetectRx`、`RxStatus[2:0]` | Detect 状态检测远端端接电阻、上报 SKP 增删或 8b/10b/同步错误 |

---

## 弹性缓冲（Elastic Buffer）与 SKP 补偿

即使系统采用公共 100 MHz 参考时钟，一旦开启 **SSC（扩频时钟）** 或使用 **独立时钟（SRIS/SRNS）**，发送端与接收端的真实时钟频率仍会存在百万分之几（ppm）的微小偏差。

```text
TX 时钟域 (f_tx) ──▶ [ 写入 ] ──▶ [ Elastic Buffer (FIFO) ] ──▶ [ 读出 ] ──▶ RX 本地时钟域 (f_rx)
                                          │
                 • 若 f_tx > f_rx（缓冲趋满）：丢弃 1 个 SKP 符号，防止溢出
                 • 若 f_tx < f_rx（缓冲趋空）：复制 1 个 SKP 符号，防止读空
```

- **SKP Ordered Set**：发送端周期性插入的“无意义填充符”。Elastic Buffer 只增删 SKP，绝不触碰真实业务数据，从而安全吸收时钟频偏。

---

## Lane-to-Lane Deskew（多通道去偏斜）

对于 x4、x8、x16 链路，数据包按字节依次条带化（Byte Striping）分散到各条 Lane 上发送。由于 PCB 走线长度、连接器引脚或过孔不可能做到绝对零误差，不同 Lane 到达接收端的时间会有几个比特周期（UI）甚至数个符号周期的偏移（Skew）。

- **对齐原理**：每条 Lane 的独立缓冲器在收到训练序列（`TS1/TS2`）或低功耗唤醒序列（`FTS` / `EIEOS`）中的特定边界标记后，先暂存到达较快的 Lane，等待最慢的 Lane 到齐后再**同一拍释放**给上层重组，抹平跨通道偏斜。

---

## 电气空闲进出序列（EIOS 与 EIEOS）

当链路进入 `L0s`、`L1` 省电状态或准备切换速率时，发送端会关闭差分驱动进入 **Electrical Idle（电气空闲，共模恒定、差分电压 ≈ 0）**：

- **EIOS（Electrical Idle Ordered Set）**：发送端在静默前发出的“告别信号”，通知对端“我要停发数据了，不要报解码错误”。
- **EIEOS（Electrical Idle Exit Ordered Set）**：唤醒或切速时发送端发出的低频周期方波（如交替的长串 `0` 和 `1`），帮助刚从休眠苏醒的接收端 **CDR 快速重新锁相** 并对齐 128b/130b 块边界。

---

## 关联与引申概念

- **编码与 SKP 机制**：8b/10b K-codes、128b/130b 同步头与扰码细节，详见 [编码方案](encoding.md)。
- **LTSSM 状态机**：`Detect`、`Polling`、`L0s/L1` 与 `Recovery` 如何驱动 PIPE 信号切换，详见 [LTSSM 状态机](ltssm.md)。
- **时钟架构**：Common Clock 与 SRIS/SRNS 独立时钟对 Elastic Buffer 深度与 SKP 频率的要求，详见 [时钟架构](../architecture/clock-architecture.md)。
- **电气规范**：PCB 等长约束（Lane 内 ±5 mil、Lane 间 ±20 mil）如何决定 Deskew 负担，详见 [电气规范](electrical-spec.md)。
