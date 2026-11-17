# PCIe 热插拔、DPC 错误隔离与 EDR 恢复机制

在企业级全闪存服务器中，前部几十块 NVMe SSD 支持不停机带电热插拔（Surprise Hot-Plug）；同时，如果某块 SSD 硬件故障向外喷射错误 TLP（Poisoned TLP），系统必须**精准切断该插槽、保住整台服务器不宕机（Kernel Panic / MCE）**。这依赖 **PCIe `pciehp`、DPC（Downstream Port Containment）与 EDR（Error Disconnect Recover）** 三者协同 `[PCIe Base §6.2 / ACPI 6.5]`。

---

## DPC 与 EDR 如何把故障关进“隔离笼”？

```text
  1. 某块 NVMe SSD 硬件异常（如发出 Poisoned TLP 或链路突发致命错误）
        │
        ▼
  2. 上游 Root Port / Switch 下行端口触发 DPC（Downstream Port Containment）
     • 硬件在微秒级强制把链路拉回 Disable/Down，阻断坏数据污染主机内存！
        │
        ├── [ 传统无 EDR ]：Linux pciehp 误以为盘被“意外拔出”，粗暴卸载驱动，无法自动恢复
        │
        └── [ 开启 EDR（固件+OS协同）]：
            3. 固件捕获 DPC 中断，通过 ACPI Notify 通知 OS 执行 EDR 恢复流程
            4. Linux 暂挂 pciehp 对链路断开的误判 ──▶ 执行 pcie_do_recovery() 重置插槽
            5. 若盘恢复正常，重新加载驱动恢复业务；若盘彻底坏死，安全下线该槽位
            6. Linux 调用 ACPI `_OST` 方法向固件汇报恢复成功或失败状态
```

---

## 热插拔（`pciehp`）与 DPC 排错实战

- **意外热拔（Surprise Removal）vs 优雅下线（Orderly Removal）**：直接带电拔盘时，高速差分对断开先于 `PRSNT#` 信号变化，必然触发一次 **Link Down** 告警；若未做好 DPC/AER 屏蔽区分，会导致内核打印大量 `AER: Uncorrected (Fatal)` 误报甚至误触发整机 NMI 宕机。
- **固件权限前提**：无论是原生热插拔（`pciehp`）还是原生 `DPC/AER` 恢复，都必须在开机时通过 ACPI **`_OSC`** 成功拿到固件授权。

```bash
# 查看 Linux 内核是否开启了原生 pciehp 与 DPC 服务
dmesg | grep -iE "pciehp|DPC|EDR"
# 优雅下线（热拔前先软件解绑与断电，避免触发 Surprise Removal 错误）
echo 0 | sudo tee /sys/bus/pci/slots/<slot_id>/power
```

---

## 关联与引申概念

- **ACPI 权限谈判**：`_OSC` 与 `_HPP`/`_HPX` 热插拔参数，详见 [ACPI 6.5 与 UEFI 2.10：固件到操作系统的 PCIe/NVMe 交接](acpi-uefi-pcie-deep-dive.md)。
- **错误报告基础**：AER 可纠正与不可纠正错误分层，详见 [PCIe 高级错误报告（AER）](../pcie/wiki/error-handling/aer.md)。
- **物理短针防拉弧**：`PRSNT1#/PRSNT2#` 硬件机制，详见 [PCIe CEM 插槽供电与边带信号时序](../pcie-form-factors/pcie-cem-and-sideband.md)。
