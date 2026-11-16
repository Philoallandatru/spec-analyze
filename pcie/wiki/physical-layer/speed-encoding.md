# PCIe 速度与编码 (Speed and Encoding)

PCIe 每一代升级不仅把物理线路的传输速率翻倍，还通过改进编码与调制方式压榨更高的有效带宽。如果把链路比作快递传送带，**信号速率（GT/s）**是传送带的移动速度，**编码效率**是货箱空间利用率，两者相乘才是真正送达的**有效数据带宽（Gbps / GB/s）**。

---

## 各代速度与带宽总览

| 代数 | 信号速率 | 调制方式 | 编码方案 | 编码/载荷效率 | 单向单 Lane 有效带宽 | x16 单向带宽 | 典型应用 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Gen 1** | 2.5 GT/s | NRZ | 8b/10b | 80% | 2.0 Gb/s (250 MB/s) | 4.0 GB/s | 早期外设、低速扩展 |
| **Gen 2** | 5.0 GT/s | NRZ | 8b/10b | 80% | 4.0 Gb/s (500 MB/s) | 8.0 GB/s | 声卡、千兆网卡、芯片组通道 |
| **Gen 3** | 8.0 GT/s | NRZ | 128b/130b | 98.46% | 7.877 Gb/s (~985 MB/s) | ~15.75 GB/s | 消费级 NVMe SSD、万兆网卡 |
| **Gen 4** | 16.0 GT/s | NRZ | 128b/130b | 98.46% | 15.754 Gb/s (~1.97 GB/s) | ~31.5 GB/s | 主流 GPU、高速 NVMe SSD |
| **Gen 5** | 32.0 GT/s | NRZ | 128b/130b | 98.46% | 31.508 Gb/s (~3.94 GB/s) | ~63.0 GB/s | AI 加速卡、旗舰 SSD、CXL 2.0/3.0 |
| **Gen 6** | 64.0 GT/s | PAM4 | 1b/1b + 256B FLIT | ~92.2% (TLP 载荷) | ~59.0 Gb/s (~7.38 GB/s) | ~118 GB/s | 超大规模数据中心、AI 集群 |

> **关键术语**：
> - **GT/s（Giga Transfers per second）**：每秒十亿次信号传输（物理层原始比特/符号速率）。
> - **Gb/s（Gigabits per second）**：扣除物理层编码开销后的有效比特率。
> - **全双工**：PCIe 发送与接收独立成对，上述均为**单向带宽**，双向总吞吐翻倍。

---

## 为什么编码和调制会演进？

### 1. Gen 2 → Gen 3：为什么改用 128b/130b？

Gen 1/2 的 8b/10b 有 **20% 开销**。若 Gen 3 想让单 Lane 带宽翻倍到 ~8 Gb/s，继续用 8b/10b 就必须把信号速率做到 10 GT/s；而改用 **128b/130b**（仅 1.54% 开销）后，只需 **8.0 GT/s** 就能实现带宽翻倍，大幅降低了高速模拟前端的设计难度和功耗。

### 2. Gen 5 → Gen 6：为什么引入 PAM4 与 256B FLIT？

到 32 GT/s（Gen 5）时，传统 **NRZ（非归零码，每符号 1 bit）**的单位间隔（UI）已缩短至 **31.25 ps**，铜线信道衰减逼近物理极限。Gen 6 采取两项革新：
- **PAM4（4 电平脉冲幅度调制）**：用 4 个电压电平分别表示 `00 / 01 / 10 / 11`，**每个符号传 2 bit**，因此 64 GT/s 的物理波特率依然只有 **32 GBaud**，无需把信号频率翻倍。
- **256 字节固定 FLIT 模式**：PAM4 电平间距缩小为原来的 1/3，抗噪能力下降。PCIe 6.0 引入定长 256B **FLIT（Flow Control Unit）**，通过轻量级 **FEC（前向纠错）** 在纳秒级内自动修复小突发错误，配合 8B CRC 兜底：

```text
PCIe 6.0 256-Byte FLIT 结构：
+------------------------+---------+---------+---------+
|      TLP Region        |   DLP   |   CRC   |   FEC   |
|       236 Bytes        | 6 Bytes | 8 Bytes | 6 Bytes |
+------------------------+---------+---------+---------+
  多个 TLP 可紧凑打包      流控/Seq   错误检测   前向纠错
```

---

## 速度协商与动态升降速流程

PCIe 链路总是先在最稳妥的 **Gen 1（2.5 GT/s）** 下完成初始建链，再由 LTSSM 协商并切换到双方都支持的最高速率：

```text
上电 / 复位
   │
   ▼
[Detect / Polling / Configuration] ── 在 Gen 1 (2.5 GT/s) 下完成基础建链
   │
   ▼
[L0 (Gen 1)] ── 交换 TS1/TS2 中的 Supported Link Speeds 能力
   │
   ▼
[Recovery.Speed] ── 双方切换 PLL 速率至目标速度 (如 Gen 4)
   │
   ▼
[Recovery.Equalization] ── Gen 3+ 必须执行发送/接收链路均衡 (Phase 0-3)
   │
   ▼
[L0 (目标速率)] ── 正常高速通信（若误码率过高，可降速回退）
```

---

## 理论带宽 vs 实际有效带宽

物理层算出的带宽是“传送带总载重”，上层协议还会占用一部分空间：
1. **TLP 头部开销**：每个 TLP 包含 12–20 字节头部（3DW/4DW Header + Prefix）以及 4 字节 LCRC。
2. **DLLP 开销**：链路层的 Ack/Nak 确认包与流控（Flow Control）信用更新包。
3. **载荷大小影响**：
   - **大块顺序读写（如 4KB / 512B MPS）**：协议开销占比低，实际有效吞吐可达理论值的 **85%–92%**（例如 Gen 4 x4 理论 7.88 GB/s，NVMe SSD 实测约 6.8–7.4 GB/s）。
   - **小包随机读写（如 64B Payload）**：头部占比显著增大，有效带宽效率会明显下降。

---

## 常用观测与调速命令

```bash
# 1. 查看设备最大支持速率与当前协商速率
lspci -vvv -s 01:00.0 | grep -E "LnkCap|LnkSta"

# 2. 手动限制目标速率并触发重训练（示例：强制降速至 Gen 3）
# Link Control 2 寄存器位于 PCIe Cap + 0x30，低 4 位为 Target Link Speed (3 = Gen3)
sudo setpci -s 01:00.0 CAP_EXP+30.w=0003
# Link Control 寄存器位于 PCIe Cap + 0x10，Bit 5 (0x20) 为 Retrain Link
sudo setpci -s 01:00.0 CAP_EXP+10.w=0020

# 3. 检查是否因信号质量问题发生过降速或存在可纠正错误
lspci -vvv -s 01:00.0 | grep -E "LnkSta:|CESta:|UESta:"
```

---

## 关联与引申概念

- **8b/10b 与 128b/130b 细节**：K-codes、Sync Header 与 LFSR 扰码器工作原理，详见 [编码方案](encoding.md)。
- **链路均衡（Link Equalization）**：Gen 3+ 切换高速率时必须执行的 TX FFE 与 RX CTLE/DFE 调优，详见 [链路均衡](link-equalization.md)。
- **LTSSM 状态机**：控制 `Recovery.Speed` 与速率回退的状态转换逻辑，详见 [LTSSM 状态机](ltssm.md)。
- **信号完整性与损耗预算**：高频信号在 PCB 上的衰减与眼图规范，详见 [电气规范](electrical-spec.md)。
