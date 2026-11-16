# 安全认证、自加密与合规测试（Security & Compliance）

本模块基于 **DMTF SPDM（DSP0274）**、**PCI-SIG CMA / DOE / IDE**、**TCG Opal 2.0** 以及 **UNH-IOL NVMe / PCI-SIG 合规测试规范**，拆解企业级 NVMe SSD 与 PCIe 设备在“零信任设备认证、总线链路加密、静态数据自加密（SED）以及量产准入测试”中的核心机制。

> **20/80 裁剪原则**：聚焦设备认证与密钥注入流程（`DOE`/`SPDM`/`IDE`）、`Security Send/Receive` 承载的 Opal 锁盘/PSID 擦除，以及量产前最易卡关的链路均衡与协议合规测试项。

---

## 文档导航

1. **[设备认证与链路加密：SPDM / CMA / DOE / IDE](spdm-cma-doe-ide.md)**
   - PCIe `DOE` 邮箱机制、`SPDM (DSP0274)` 证书链与固件度量（`CMA`），以及 `IDE` 链路级 AES-GCM 加密。
2. **[自加密盘（SED）与 TCG Opal 2.0](tcg-opal-and-sed.md)**
   - NVMe `Security Send (81h)` / `Receive (82h)` 隧道、`Admin SP` / `Locking SP` 分区锁定与 `PSID Revert` 密码学擦除。
3. **[合规与互操作测试：UNH-IOL NVMe & PCI-SIG Compliance](conformance-and-compliance-testing.md)**
   - PCI-SIG 电气/链路均衡（Link Eq）合规测试、UNH-IOL NVMe Conformance（Integrator's List）与跨平台 Interop 排错。

---

## 关联与引申概念

- **带内安全隧道**：NVMe `Security Send/Receive` 的协议定位，详见 [安全协议交换](../nvme/concepts/security-protocol-exchange.md)。
- **MCTP 承载**：SPDM 在带外管理总线上的传输（`Message Type 05h`），详见 [MCTP 传输层与 PCIe VDM / SMBus 封装](../nvme-mi/mctp-transport-and-vdm.md)。
- **物理层均衡**：PCIe Gen 3–Gen 6 发射端 Preset 与接收端均衡，详见 [链路训练（LTSSM）](../pcie/wiki/physical-layer/link-training.md)。
