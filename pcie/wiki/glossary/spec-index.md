# PCIe 规范索引

本页面汇总 **PCI Express Base Specification** 的代际演进、标准章节与本 Wiki 文档的对应关系，以及配套的机电规范、上层设备协议与常用调试工具。

---

## PCI Express Base Specification 演进

| 版本 | 发布年份 | 单 Lane 速率 | 编码/调制 | 关键特性 |
|------|----------|--------------|-----------|----------|
| **1.0 / 1.1** | 2002 / 2005 | 2.5 GT/s | 8b/10b (NRZ) | 初代串行点对点架构、三层协议栈 |
| **2.0 / 2.1** | 2007 / 2009 | 5.0 GT/s | 8b/10b (NRZ) | 速率翻倍，引入 ARI、ACS、LTR 等系统扩展 |
| **3.0 / 3.1** | 2010 / 2014 | 8.0 GT/s | 128b/130b (NRZ) | 取消 20% 编码开销，引入 Tx/Rx 链路均衡、L1 Substates |
| **4.0** | 2017 | 16.0 GT/s | 128b/130b (NRZ) | 速率翻倍，引入 Retimer 规范、DPC 增强 |
| **5.0** | 2019 | 32.0 GT/s | 128b/130b (NRZ) | 32 GT/s 物理层规范，支持 CXL 交替协议与 IDE/DOE 安全扩展 |
| **6.0** | 2022 | 64.0 GT/s | PAM4 + 242B/256B Flit | 改用 4 电平 PAM4 调制、固定大小 Flit 帧与轻量级 FEC |

---

## 规范核心章节与 Wiki 映射

| 规范章节 | 规范主题 | 核心内容 | 对应 Wiki 文档 |
|----------|----------|----------|----------------|
| **Ch 1** | Introduction | 架构概述、拓扑（RC/Switch/EP）、分层模型 | [概述](../introduction/overview.md) · [拓扑](../introduction/topology.md) · [三层模型](../architecture/layering.md) |
| **Ch 2** | Transaction Layer | TLP 格式、事务类型、路由、基于 Credit 的流控、排序、VC、ECRC | [TLP 格式](../transaction-layer/tlp-format.md) · [流控](../transaction-layer/flow-control.md) · [排序](../transaction-layer/ordering.md) |
| **Ch 3** | Data Link Layer | DLLP 格式、Ack/Nak 协议、Replay Buffer、LCRC、流控初始化 | [DLLP 格式](../data-link-layer/dllp-format.md) · [Ack/Nak](../data-link-layer/ack-nak.md) · [LCRC](../data-link-layer/lcrc.md) |
| **Ch 4** | Physical Layer | Lane/速率、8b/10b 与 128b/130b 编码、LTSSM、均衡、电气与 Retimer | [物理层概述](../physical-layer/overview.md) · [LTSSM](../physical-layer/ltssm.md) · [均衡](../physical-layer/link-equalization.md) |
| **Ch 5** | Power Management | 设备电源状态 (D0–D3)、链路状态 (L0–L3)、ASPM、L1 Substates、CLKREQ# | [电源状态](../power-management/power-states.md) · [ASPM](../power-management/aspm.md) · [L1 子状态](../power-management/l1-substates.md) |
| **Ch 6** | System Architecture | 中断 (INTx/MSI/MSI-X)、错误与 AER、热插拔、ACS/ARI/Atomic/TPH/PTM/DOE/IDE | [中断概述](../interrupts/overview.md) · [AER](../error-handling/aer.md) · [热插拔](../advanced-features/hot-plug.md) |
| **Ch 7** | Configuration Registers | Type 0/1 配置头、BAR、PCI/PCIe Capability 与 Extended Capability | [配置空间](../configuration/config-space.md) · [BAR](../configuration/bars.md) · [Capabilities](../configuration/capabilities.md) |
| **Ch 9** | SR-IOV | 单根 I/O 虚拟化（PF/VF 模型与地址映射） | [SR-IOV](../advanced-features/sriov.md) |

> [!TIP]
> 需要按主题/角色快速定位全站 70+ 篇文档，请查阅仓库根目录的 [完整规范索引 (SPEC_INDEX)](../../SPEC_INDEX.md)。

---

## 配套规范与相关标准

- **PCI-SIG 机械与接口规范**：
  - **CEM (Card Electromechanical)**：标准台式机/服务器 AIC 插卡机械与金手指定义
  - **M.2 / Mini Card / U.2 / E1.S / E3.S**：面向紧凑型 SSD 与服务器背板的外形与连接器规范
  - **OCuLink**：高速内部/外部线缆直连规范
- **上层设备与互连协议**：
  - **NVMe (NVM Express)**：基于 PCIe 的高性能非易失性存储协议（参见 [NVMe over PCIe](../implementation/nvme.md)）
  - **CXL (Compute Express Link)**：基于 PCIe 5.0/6.0 物理层的缓存一致性与内存池化互连协议
  - **AHCI (SATA)** / **xHCI (USB)**：传统存储与通用外设控制器接口

---

## 开源参考、工具与书籍

| 类别 | 资源 | 用途说明 |
|------|------|----------|
| **开源实现** | [Linux Kernel (`drivers/pci`)](https://www.kernel.org/) · [QEMU / FEMU](https://github.com/vtess/FEMU) | 真实内核枚举/驱动与虚拟化设备模拟参考 |
| **调试工具** | `lspci` / `setpci` (`pciutils`) · PCIe 协议分析仪 | 枚举检查、配置空间读写与物理/链路层抓包 |
| **硬件工具** | Vivado PCIe IP / Quartus PCIe Hard IP | FPGA 端 PCIe Endpoint / Root Port 开发 |
| **经典书籍** | *PCI Express System Architecture* (Budruk) · *PCI Express Technology* (Jackson) | 系统性精读 PCIe 协议原理与硬件实现 |
| **官方站点** | [PCI-SIG](https://pcisig.com/) | 官方规范发布与合规测试标准 |

---

## 关联与引申概念

- **全站完整目录**：[PCIe 规范与文档总索引](../../SPEC_INDEX.md) · [Wiki 主页](../README.md)
- **速查与入门**：[寄存器速查表](register-quick-ref.md) · [术语表](index.md) · [常见问题 (FAQ)](faq.md) · [学习路径](../introduction/learning-path.md)
