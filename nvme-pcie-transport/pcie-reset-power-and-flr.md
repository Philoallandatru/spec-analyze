# PCIe 复位层级、FLR 与电源状态交互

当 NVMe SSD 卡死、固件热升级或虚拟机热插拔回收设备时，主机必须选择恰当的复位级别。不同级别的复位在**影响范围、寄存器清零程度以及与 PCIe 链路的关系**上截然不同 `[PCIe Transport §3.3]`。

可以把复位层级想成**重启电脑的四种方式**：
1. **`CC.EN` 软复位** = 注销重登（保留硬件连接，清空队列状态）。
2. **`NSSR` 子系统复位** = 整台服务器断电重启（波及同一子系统下的所有端口/控制器）。
3. **PCIe `FLR`（Function Level Reset）** = 只重启某一张虚拟网卡/功能（PCIe 链路不断，其他 VF 不受影响）。
4. **`PERST#` / Hot Reset** = 拔掉电源/重插总线（连同 PCIe LTSSM 链路重新训练）。

---

## 四级复位行为对比

| 复位方式 | 触发方式 | 是否重置 PCIe 链路 | 是否清零 PCIe 配置空间 | 对其他 VF / 端口的影响 |
| :--- | :--- | :--- | :--- | :--- |
| **Controller Reset** | 写 `CC.EN` 从 `1` → `0` | 否 | 否（保留 BAR/MSI-X） | 无影响（仅重置当前控制器） |
| **PCIe FLR** | 写 PCIe Device Control `Initiate FLR` bit | 否（保持 L0） | 部分重置（Sticky 位保留，BAR 需 OS 恢复） | 无影响（仅重置当前 PF/VF；若 PF FLR 则可能连带 VF） |
| **NVM Subsystem Reset (`NSSR`)** | 向 BAR0 `NSSR`（`20h`）写 `4E564D65h` | 否 | 否（但控制器进入重置，需重新初始化） | **波及整个 NVM 子系统内所有控制器** |
| **Conventional / Hot Reset** | 拉低 `PERST#` 或 Bridge `Secondary Bus Reset` | **是**（LTSSM 回 Detect） | **是**（非 Sticky 寄存器全清零） | 波及该下游链路上所有设备 |

### 关键握手规则

- **等待 `CSTS.RDY`**：将 `CC.EN` 置 `0` 后，主机必须轮询 `CSTS.RDY == 0`（最长等待 `CAP.TO × 500ms`），才能修改 `AQA`/`ASQ`/`ACQ` 并重新置 `CC.EN = 1`。
- **正常关机 vs 异常复位**：正常卸载驱动前应先写 `CC.SHN = 01b`（Normal Shutdown），等待 `CSTS.SHST == 10b`，让 SSD 把易失性缓存刷入 NAND；直接拉 `CC.EN = 0` 或 `PERST#` 相当于断电，可能触发下次上电的元数据恢复延迟。
- **FLR 超时**：PCIe 规范要求 FLR 在 **100ms** 内完成，期间设备不得发起新的 DMA 请求（Bus Master 事务必须静默）。

---

## 电源状态交互：NVMe Power States vs PCIe D-States & ASPM L1.2

主机对 SSD 进行功耗管理时，存在**两套并行的电源状态机**，它们之间有严格约束 `[PCIe Transport §5]`：

1. **NVMe 运行电源状态（`FID 02h`）**：控制 SSD 内部 ASIC 与 NAND 通道的功耗（如 PS0–PS4，分为操作态与非操作态）。
2. **PCIe 设备电源状态（`D0` / `D3hot` / `D3cold`）与链路省电（`ASPM L0s/L1/L1.2`）**：控制 PCIe PHY 收发器与参考时钟。

```text
  进入低功耗 (例如系统休眠 S0ix / 运行时省电)：
  1. Host 先发 Normal Shutdown (CC.SHN) 或切 NVMe 至非操作态 (如 PS3/PS4)
  2. Host 将 PCIe PMCSR 置为 D3hot ──▶ 链路进入 ASPM L1 / L1.2 (CLKREQ# 拉高停时钟)
  3. (可选) 主板移除主供电进入 D3cold ──▶ 仅保留 3.3Vaux 维持侧带唤醒
```

> **常见死锁坑（L1.2 / APST）**：部分消费级与早期企业级 NVMe 盘在开启 **APST（Autonomous Power State Transition）** 与 **PCIe ASPM L1.2** 时，若固件从 `L1.2` 唤醒（`CLKREQ#` 拉低重锁 PLL）耗时超过 `CAP`/平台容忍窗口，会导致 **NVMe I/O Timeout 掉盘**。排错时常用内核参数 `pcie_aspm=off` 或 `nvme_core.default_ps_max_latency_us=0` 验证是否由此引起。

```bash
# 查看当前设备的电源状态 (D0/D3) 与 ASPM L1/L1.2 使能情况
lspci -vvv -s 0000:01:00.0 | grep -E "PM|LnkCtl:|ASPM"
# 对指定 PCIe Function 触发一次硬件级 FLR
echo 1 | sudo tee /sys/bus/pci/devices/0000:01:00.0/reset
```

---

## 关联与引申概念

- **控制器初始化与关机**：`CC`/`CSTS` 的完整状态机，详见 [控制器就绪模式](../nvme/concepts/controller-ready-modes.md) 与 [控制器属性](../nvme/concepts/controller-properties.md)。
- **电源状态描述符**：`Identify Controller` 中的 `PSD0–PSD31` 延迟与功耗定义，详见 [电源状态描述符](../nvme/concepts/power-state-descriptors.md)。
- **PCIe 电源管理**：`D0–D3` 与 `ASPM L0s/L1/L1.2` 细节，详见 [PCIe 电源状态](../pcie/wiki/power-management/power-states.md) 与 [ASPM](../pcie/wiki/power-management/aspm.md)。
- **掉盘排错 SOP**：主机侧 `I/O Timeout` 与链路重置排查流程，详见 [主机侧性能调优与排错 SOP](../host-system/host-debug-and-tuning.md)。
