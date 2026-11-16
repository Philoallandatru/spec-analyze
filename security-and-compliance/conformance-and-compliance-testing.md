# 合规与互操作测试：UNH-IOL NVMe & PCI-SIG Compliance

一款 NVMe SSD 或主机平台在量产前，即使自研驱动能跑通读写，仍必须通过行业两大权威合规体系——**PCI-SIG Compliance（底层总线合规）** 与 **UNH-IOL NVMe Conformance（存储协议合规）**——才能进入服务器厂商 AVL（Approved Vendor List）与 **NVMe Integrator's List** `[UNH-IOL / PCI-SIG]`。

---

## 两层合规测试体系对比

| 测试体系 | 覆盖范围 | 核心工具 / 机构 | 典型必测项目 |
| :--- | :--- | :--- | :--- |
| **PCI-SIG Compliance** | 物理层（PHY）电气信号、数据链路层、事务层与配置空间 | PCI-SIG Plugfest / 示波器（Tx/Rx）与协议分析仪 | 发射端眼图/抖动（RJ/DJ）、**链路均衡（Link Equalization）**、协议边界异常注入、配置空间寄存器合规 |
| **UNH-IOL NVMe Conformance** | NVMe Base + PCIe Transport + NVM/ZNS/MI 协议语义 | **IOL INTERACT**（可配合 Sanblaze / Teledyne LeCroy） | Admin/I/O 命令边界校验、保留位清零、非法门铃/错误状态码、Queue 创建/删除顺序、Reset/Shutdown 时序 |
| **Interop（互操作测试）** | 跨品牌主板、CPU、Switch、BIOS 与 OS 矩阵实测 | 各大 OEM/ODM 实验室 | 冷/热重启循环、带压热插拔、S3/S4/S0ix 休眠唤醒、ASPM L1.2 压力测试 |

---

## 研发最常挂掉的 4 类合规红线

1. **Link Equalization（链路均衡）超时**：PCIe Gen 4/5/6 速率下，PHY 在 LTSSM `Recovery.Equalization` Phase 0–3 期间未在规定时间内（如 24ms）完成 Preset 协商，导致降速（Degraded Link Width/Speed）甚至直接掉线。
2. **保留字段（Reserved）未报错或私自占用**：规范规定向保留字段写非零值或向无效 `LID`/`FID`/`CNS` 发请求时，控制器必须返回明确的 `Invalid Field in Command`（`02h`），而不能静默忽略或返回成功。
3. **Reset / Shutdown 期间遗留未完成事务**：触发 `FLR` 或 `CC.EN = 0` 后，控制器在未等安全静默前仍向主机发出了残余的 DMA Memory Write 或 MSI-X 中断。
4. **SMBus / MCTP 分片与时钟拉伸**：NVMe-MI 合规测试中，对 64B MTU 多包重组与超时恢复（`Control Primitive`）极易因 BMC/SSD 时序差异失败。

```bash
# 快速自检链路是否发生非预期降速/降宽 (对照 LnkCap 与 LnkSta)
lspci -vvv -s 0000:01:00.0 | grep -E "LnkCap:|LnkSta:"
```

---

## 关联与引申概念

- **PCIe PHY 均衡与眼图**：Tx FFE/Preset 与 Rx CTLE/DFE 原理，详见 [PCS、PMA 与 PIPE 接口](../pcie/wiki/physical-layer/pcs-pma-pipe.md) 与 [链路训练（LTSSM）](../pcie/wiki/physical-layer/link-training.md)。
- **错误注入与恢复**：OS 如何利用 ACPI `EINJ` 验证 AER 恢复，详见 [ACPI 6.5 与 UEFI 2.10：固件到操作系统的 PCIe/NVMe 交接](../host-system/acpi-uefi-pcie-deep-dive.md)。
- **主机侧调优与排错**：实际生产环境中定位掉盘与降速的操作步骤，详见 [主机侧性能调优与排错 SOP](../host-system/host-debug-and-tuning.md)。
