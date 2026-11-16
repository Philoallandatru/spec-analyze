# NVMe over PCIe 传输规范（NVMe PCIe Transport）

本模块基于 **NVMe over PCIe Transport Specification**（与 NVMe Base 2.1 协同），补全 Base 规范中未展开的 **PCIe 物理绑定细节**——回答“主机驱动如何通过 PCIe BAR0、MMIO 门铃、PRP/SGL 内存描述符、MSI-X 中断和 PCIe 复位/电源状态与 NVMe SSD 真正对话”。

> **20/80 裁剪原则**：聚焦日常驱动开发、虚拟化直通与硬件联调最常碰到的 4 个核心机制，不机械照搬规范中的保留位与历史兼容条款。

---

## 文档导航

1. **[BAR0 控制器寄存器与门铃步长（BAR0 & Doorbells）](bar0-registers-and-doorbells.md)**
   - 64 位 `BAR0/BAR1`（`MLBAR`/`MUBAR`）布局、`CAP`/`CC`/`CSTS`/`AQA`/`ASQ`/`ACQ` 精确偏移、`CAP.DSTRD` 门铃步长计算与影子门铃（Shadow Doorbell）。
2. **[PRP / SGL 内存描述符与对齐规则](prp-sgl-and-alignment.md)**
   - PCIe 传输下的 PRP1/PRP2/PRP List 4KB 页对齐、SGL 在 PCIe 下的支持条件（`Identify Controller.SGLS`），以及高频 `Invalid Field` 踩坑排查。
3. **[MSI-X 中断向量绑定与屏蔽机制](msix-and-interrupt-binding.md)**
   - MSI-X Table/PBA 与 CQ `IV`（Interrupt Vector）绑定、`INTMS`/`INTMC` 在 MSI-X 下的禁区，以及中断合并（`FID 08h/09h`）调优。
4. **[PCIe 复位层级、FLR 与电源状态交互](pcie-reset-power-and-flr.md)**
   - `CC.EN` 软复位、`NSSR` 子系统复位、PCIe `FLR` 与 `PERST#` 硬复位的边界对比，以及 `D0–D3cold` 与 `L1.2` 低功耗交互。

---

## 关联与引申概念

- **传输抽象层**：内存模型与 Fabrics 消息模型的宏观对比，详见 [传输模型](../nvme/concepts/transport-models.md)。
- **控制器寄存器基础**：通用属性定义与握手流程，详见 [控制器属性](../nvme/concepts/controller-properties.md)。
- **PCIe BAR 与枚举**：主机如何在开机时分配 BAR0 空间，详见 [BARs 详解](../pcie/wiki/configuration/bars.md) 与 [上电、UEFI/ACPI 与 PCIe 枚举](../host-system/boot-and-enumeration.md)。
