# 主机侧性能调优与排错 SOP

在硬件上板联调和线上生产环境中，NVMe SSD 跑不满标称带宽/IOPS、或者突然掉盘超时，绝大多数根因都落在**链路协商、MPS/MRRS 参数、IOMMU/NUMA 配置或中断调度**上。下面整理主机侧最常用的排查定位与调优清单。

---

## 1. 吞吐量跑不满？先查这 4 项物理与协议瓶颈

| 检查项 | 现象与根因 | 排查与优化手段 |
| :--- | :--- | :--- |
| **1. 链路降速/降宽** | `LnkSta` 低于 `LnkCap`（如 Gen4 x4 降为 Gen3 x4 或 x2），带宽直接腰斩。 | `lspci -vvv` 对比 `LnkCap` 与 `LnkSta`，同时查 `CESta` 是否有 `Receiver Error`。 |
| **2. MPS / MRRS 过小** | **MPS（Max Payload Size）** 只有 128B（理想为 256B/512B），导致 TLP 头部开销翻倍；**MRRS** 过小限制单次 DMA 读请求大小。 | 检查整条链路从 Root Port 到 EP 的 `DevCtl` 中 `MaxPayload` 与 `MaxReadReq`（全链路取最小值）。 |
| **3. IOMMU 页表开销** | 高 IOPS 随机读写时大量 CPU 消耗在 `iommu_map/unmap` 与 IOTLB 失效上。 | 物理机非直通场景可在内核启动参数开启 **`iommu=pt`**（透传模式），IOPS 常可提升 10%–30%。 |
| **4. 跨 NUMA / 中断集中** | 所有 NVMe 中断集中在 CPU 0，或跨 Socket 访问远端内存。 | 检查 `/proc/interrupts` 确认中断均匀绑定到所在 NUMA 节点的各核心，压测时用 `numactl` 绑核。 |

---

## 2. 掉盘 / I/O 超时（NVMe Timeout & Reset）排查流程

当系统日志出现 `nvme nvme0: I/O QID 1 timeout, reset controller` 时，按以下由下到上的顺序定位：

```text
Step 1: 看硬件链路还在不在？
   └─ lspci -s 01:00.0 是否变成 "ff" (Device fell off the bus)？
      • 是 → 物理层/电源掉线（检查 AER Fatal/Surprise Down、ASPM L1.2/CLKREQ# 兼容或供电）
      • 否 → 链路仍在 L0，继续排查协议层

Step 2: 看 PCIe 层有没有报错？
   └─ dmesg | grep -iE "AER|pcieport|DPC"
      • Completion Timeout → 设备没按时回 CplD，可能 SSD 固件卡死或被限流
      • Poisoned TLP / Malformed TLP → 内存或 TLP 封装异常

Step 3: 看 NVMe 控制器状态 (CSTS) 与命令返回
   └─ nvme show-regs /dev/nvme0
      • CSTS.CFS = 1 (Controller Fatal Status) → 控制器内部致命错误，必须 Reset 恢复
      • CSTS.RDY = 0 → 控制器未就绪或正在重置中
```

---

## 3. 主机侧常用工具速查（≤ 15 行实操命令）

```bash
# 1. 查看 PCIe 链路速率、宽度、MPS/MRRS 与 AER 错误状态
lspci -vvv -s 01:00.0 | grep -E "LnkCap|LnkSta|DevCtl|CESta|UESta"

# 2. 查看 NVMe 控制器寄存器 (CAP/VS/CC/CSTS) 及队列/命名空间列表
sudo nvme show-regs /dev/nvme0
sudo nvme list-ns /dev/nvme0

# 3. 实时统计块层 I/O 延迟分布（需安装 blktrace / bpftrace / iostat）
iostat -xz 1 nvme0n1

# 4. 临时关闭指定设备的 ASPM（排查低功耗唤醒导致的延迟抖动或掉盘）
echo 0 > /sys/bus/pci/devices/0000:01:00.0/link/l1_aspm
```

---

## 关联与引申概念

- **MPS 与 TLP 效率**：数据包载荷上限如何影响总线有效带宽，详见 [TLP 格式](../pcie/wiki/transaction-layer/tlp-format.md)。
- **AER 与错误恢复**：Linux `aer_isr` 与 `pci_error_handlers` 的恢复状态机，详见 [PCIe 错误恢复](../pcie/wiki/error-handling/recovery.md) 与 [AER 实现](../pcie/wiki/implementation/aer.md)。
- **ASPM 与 L1.2 排错**：`CLKREQ#` 时序异常引发的唤醒超时，详见 [ASPM](../pcie/wiki/power-management/aspm.md) 与 [CLKREQ#](../pcie/wiki/power-management/clkreq.md)。
- **NVMe 复位与致命错误**：`CSTS.CFS` 与 Controller Level Reset / NSSR 的区别，详见 [控制器使能、关闭与重置](../nvme/concepts/controller-enable-shutdown-reset.md)。
