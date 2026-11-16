# PCIe 链路均衡 (Link Equalization)

**链路均衡（Link Equalization）** 是 PCIe Gen 3（8 GT/s）及以上高速链路必需的信号调优技术：发送端和接收端在链路训练期间互相配合，调节滤波器和放大器参数，抵消高速信号在 PCB 铜线和连接器上的高频衰减。如果把高速传输比作在长走廊里喊话，**发送端均衡**就像说话时刻意加重辅音（高频），**接收端均衡**就像戴上助听器放大微弱的高音并消除回声（码间干扰 ISI）。

---

## 为什么 Gen 3+ 必须做链路均衡？

随着速率从 5 GT/s 升至 8 / 16 / 32 GT/s，单位比特时间（UI）缩短到 125 ps 至 31.25 ps，信号面临两大物理瓶颈：
1. **频率相关插入损耗**：PCB 介质损耗和铜箔趋肤效应会严重削弱高频分量，导致方波变成圆钝的“小山包”。
2. **码间干扰（ISI, Inter-Symbol Interference）**：前一个比特的能量拖尾到后一个比特的时间窗口里，使接收端**眼图（Eye Diagram）闭合**，误码率（BER）飙升。

链路均衡的目标，就是把闭合的眼图重新“撑开”，使链路满足 **BER < $10^{-12}$** 的规范要求。

---

## 发送端与接收端均衡技术

```text
发送端 (TX)                       信道 (PCB/连接器)              接收端 (RX)
+------------------+              (高频严重衰减)             +--------+     +--------+
|  3-Tap FFE       | ── 预加重 ──────────────────────────▶  |  CTLE  | ──▶ |  DFE   | ──▶ 判决器
| (C-1, C0, C+1)   |    压低连续平直电平，突出跳变沿          | 线性放大|     | 消回声 |
+------------------+                                        +--------+     +--------+
```

### 1. 发送端：De-emphasis vs 3-Tap FFE（前馈均衡器）

- **Gen 1 / Gen 2（去加重 De-emphasis）**：仅有 Post-cursor（`-3.5 dB` 或 `-6 dB`），做法是**不抬高跳变沿，而是压低连续相同电平的后续比特**，避免发送端电压超出芯片供电摆幅限制。
- **Gen 3+（3-Tap FFE）**：升级为 **3 抽头有限脉冲响应滤波器**，同时修正跳变前后的信号畸变：
  - **Pre-cursor ($C_{-1}$)**：在比特跳变**前**预先微调幅度（Pre-shoot），抵消前向码间干扰。
  - **Main cursor ($C_0$)**：当前比特的主驱动幅度。
  - **Post-cursor ($C_{+1}$)**：在比特跳变**后**压低连续相同电平的幅度（De-emphasis），抵消后向拖尾。

PCIe 规范预定义了 **11 组标准预设（Preset P0–P10）**（例如 P4 仅带去加重、P7 同时带 Pre-shoot 和去加重），方便训练初期快速试探。

### 2. 接收端：CTLE + DFE（为什么必须两者配合？）

- **CTLE（连续时间线性均衡器）**：模拟前端的高频增益放大器，像“高音增强旋钮”把被 PCB 衰减的高频分量拉回来；缺点是**无差别放大高频噪声与串扰**。
- **DFE（判决反馈均衡器）**：非线性数字/混合信号滤波器。它把**已经正确判决出的历史比特**乘以反馈抽头权重（Tap 1–Tap N），直接从当前采样电压中减掉历史拖尾。**最大优势是只消 ISI，完全不放大噪声**；但在 Gen 6 PAM4 四电平下，一旦前面判错一个符号，可能引发后续连续误判（**错误传播**），因此需要配合 Precoding 与 FEC。

---

## Gen 3+ 链路均衡 4 阶段流程

当链路从 Gen 1（2.5 GT/s）切换到 Gen 3/4/5 时，LTSSM 会在 `Recovery.Equalization` 中执行最多 4 个阶段（Phase 0–3）：

```text
Phase 0: 下游端 (Upstream/EP) 先切入高速，使用默认/初始 Preset
   │
   ▼
Phase 1: 粗调建立基本通信 (Gen 3+ 必需)
   │     双方使用 TS1 交换各自推荐的初始 TX Preset，确保能稳定解调 TS1
   ▼
Phase 2: 上游端 (RC/Switch) 微调下游端 (EP) 的 TX 系数
   │     RC 评估接收眼图质量 → 通过 TS1 发送 Preset 或 (+1/-1) 系数微调请求
   │     → EP 调整 TX FFE 并回传确认 → 直到 RC 发送 "Reject/No More Changes" 收敛
   ▼
Phase 3: 下游端 (EP) 微调上游端 (RC/Switch) 的 TX 系数
   │     流程与 Phase 2 对称，EP 优化 RC 的发送波形，同时微调自身 RX CTLE/DFE
   ▼
完成均衡 → 进入 L0 高速运行
```

> **注**：在部分短距离、损耗极低的链路上，如果 Phase 1 后信号质量已足够好，系统可配置跳过或简化 Phase 2/3 以缩短训练时间。

---

## Lane Margining（接收端裕量测试，PCIe 4.0+）

PCIe 4.0 引入了内建的 **Lane Margining at the Receiver（LMR）** 扩展能力，无需昂贵的外部高速示波器，芯片自身就能测量工作状态下的眼图余量：
- **时序裕量（Timing Margin）**：在左右方向偏移采样时钟，测试多宽的窗口内仍能保持无误码。
- **电压裕量（Voltage Margin）**：在上下方向偏移判决阈值，测试多高的电压余量内仍能保持无误码。

---

## 观测与调试命令

```bash
# 1. 查看链路均衡状态（关注 LinkSta2 中的 Equalization Complete 及 Phase 1/2/3 完成标志）
lspci -vvv -s 01:00.0 | grep -A5 "LnkSta2"

# 2. 直接读取 Link Control 2 (CAP_EXP+0x30) 与 Link Status 2 (CAP_EXP+0x32) 寄存器
sudo setpci -s 01:00.0 CAP_EXP+30.w
sudo setpci -s 01:00.0 CAP_EXP+32.w

# 3. 检查是否支持 Lane Margining 扩展能力 (Extended Cap ID 0x0027)
lspci -vvv -s 01:00.0 | grep -i "Lane Margining"
```

---

## 关联与引申概念

- **链路训练（Link Training）**：均衡在整体建链流程中所处的位置与 TS1/TS2 交互，详见 [链路训练过程](link-training.md)。
- **LTSSM 状态机**：负责调度 `Recovery.Equalization` 的状态机机制，详见 [LTSSM 状态机](ltssm.md)。
- **电气规范与眼图**：插入损耗预算、UI（单位间隔）及眼高/眼宽合规标准，详见 [电气规范](electrical-spec.md)。
- **Retimer**：当均衡已无法弥补过长 PCB 走线损耗时，需要用 Retimer 分段再生信号，详见 [Retimer 详解](retimer.md)。
