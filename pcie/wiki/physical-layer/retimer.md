# PCIe Retimer（重定时器）

**Retimer（重定时器）** 是 PCIe Gen 3（8 GT/s）及以上高速链路中用来延长传输距离的物理层信号再生芯片。如果把长距离高速信号传输比作长跑接力，**Retimer 就像中途的接力选手**：它把跑得气喘吁吁、波形变形的信号先接住，在内部用精准时钟重新采样复原（去除累积抖动和噪声），再以满格电量重新起跑送往下一程。

---

## 为什么需要 Retimer？Retimer vs Redriver

到了 Gen 4（16 GT/s）和 Gen 5（32 GT/s），信号在普通主板 PCB 上跑过 **7–15 cm** 或经过连接器/背板后，高频衰减和抖动就会让眼图彻底闭合。此时需要中继芯片来“续命”，业界主要有 **Redriver** 和 **Retimer** 两种方案：

| 对比维度 | Redriver（模拟中继器） | Retimer（数字重定时器） |
| :--- | :--- | :--- |
| **核心原理** | 纯模拟放大 + 线性均衡（类似扩音器） | **CDR 时钟恢复 + 数字重定时 + 重新驱动**（类似信号再生站） |
| **抖动清除能力** | 无（放大信号的同时也会放大噪声与抖动） | **完全清除**（用本地纯净时钟重新采样发送） |
| **协议感知** | 完全透明，不识别训练序列 | 参与物理层 **LTSSM** 与 **链路均衡**（对上层 TLP/DLLP 透明） |
| **典型延迟** | < 1 ns | **3–5 ns**（CDR + 弹性缓冲 FIFO + 串并转换） |
| **功耗与成本** | 低 | 较高 |
| **适用代数** | Gen 1 / Gen 2 / 短距离 Gen 3 | **Gen 4 / Gen 5 / Gen 6 长距离或跨背板拓扑必需** |

---

## Retimer 内部架构与工作原理

Retimer 内部为双向对称结构，每个方向包含完整的 RX 与 TX 模拟/数字前端，以及跨时钟域的 **弹性缓冲（Elastic FIFO）**：

```text
上游端口                                                        下游端口
──────▶ [AFE / CTLE / DFE] ──▶ [CDR & 解串] ──▶ [Elastic FIFO] ──▶ [串行化 & TX FFE] ──────▶
                                    │                  ▲
                                    ▼                  │
                            ┌──────────────────────────────────┐
                            │   LTSSM 监听 & 均衡协同控制器     │
                            │  - 参与 TS1/TS2 协商与链路均衡    │
                            │  - 依据 FIFO 水位插入/删除 SKP   │
                            └──────────────────────────────────┘
```

1. **隔离信道损耗**：原本 20 cm 的高损耗链路被 Retimer 切分为两段 10 cm 的独立信道，每段分别与 Retimer 独立完成**链路均衡（Link Equalization）**，眼图质量大幅提升。
2. **时钟域隔离与 SKP 处理**：Retimer 拥有独立的参考时钟 PLL。当上下游存在微小频偏（或开启 SSC 扩频时钟）时，Retimer 会根据 Elastic FIFO 水位自动插入或删除 **SKP Ordered Set**，防止缓冲区溢出或读空。
3. **协议透明与级联限制**：
   - Retimer **不解析 TLP/DLLP**，也不占用 PCIe 总线号（Bus/Device/Function），对操作系统和驱动几乎透明。
   - PCIe 规范允许一条链路上**最多级联 2 个 Retimer**（`RC ─[R1]─[R2]─ EP`），否则累积延迟和训练复杂度将超出重传超时预算。

---

## 性能影响与日常排查

- **带宽**：**零损失**。Retimer 工作在物理层，不改变编码效率和有效数据率（例如 Gen 4 x4 实测吞吐依然可达 ~6.8–7.4 GB/s）。
- **延迟**：每个 Retimer 增加约 **3–5 ns** 传播延迟。对于单次耗时数微秒（μs）的 NVMe SSD 读写而言，千分之几的延迟增量几乎可忽略；仅超低延迟高频交易网络或 CXL 内存扩展场景需重点评估。

```bash
# 1. 查看链路是否因 Retimer 协同训练异常发生降速
lspci -vvv -s 01:00.0 | grep -E "LnkCap:|LnkSta:|LnkSta2:"

# 2. 查看 SMBIOS/DMI 信息中平台是否标注了板载 Retimer
sudo dmidecode -t 9 | grep -i -E "Designation|Type|Retimer"

# 3. 监控 AER 可纠正错误（若 Retimer SKP 处理或时钟异常，常伴随 Receiver Error）
lspci -vvv -s 01:00.0 | grep -A4 "CESta:"
```

---

## 关联与引申概念

- **CDR（时钟数据恢复）**：Retimer 能够清除上游抖动并重定时数据的核心电路，详见 [物理层概述](overview.md)。
- **链路均衡（Link Equalization）**：Retimer 将长链路分段后，分别与两端独立完成 TX FFE 和 RX CTLE/DFE 调优，详见 [链路均衡](link-equalization.md)。
- **SKP Ordered Set**：Retimer 用来吸收上下游时钟频偏的物理层填充符，详见 [编码方案](encoding.md)。
- **LTSSM 状态机**：Retimer 在链路训练期间如何转发并协同 TS1/TS2 序列，详见 [LTSSM 状态机](ltssm.md)。
- **CXL（Compute Express Link）**：基于 PCIe 5.0/6.0 物理层的内存扩展与一致性互连协议，对 Retimer 的低延迟和 FLIT 模式有更严苛的要求。
