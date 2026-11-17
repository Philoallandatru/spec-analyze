# PCIe TDISP：机密计算下的设备直通安全协议

过去我们用 **VFIO + SR-IOV** 把一块 NVMe SSD 或 GPU 的虚拟功能（VF）直通给虚拟机时，默认前提是“**Hypervisor（宿主机）是绝对可信的**”。但在公有云**机密计算（Confidential Computing，如 Intel TDX、AMD SEV-SNP、ARM CCA）** 场景下，租户甚至不信任云厂商的宿主机 Hypervisor！

如果 Hypervisor 被攻破，它依然可以偷窥直通设备发往 PCIe 总线的明文数据或篡改设备配置。**PCI-SIG TDISP（TEE Device Interface Security Protocol）** 正是为解决“**把一个 PCIe 设备接口（TDI）安全地直接绑定到机密虚拟机（TVM）**”而生的协议 `[PCI-SIG TDISP §1–3]`。

---

## TDISP 三位一体安全底座

```text
  机密虚拟机 (TVM, 内存受 TDX/SEV-SNP 加密隔离)
        │  ▲  (受 TDISP 保护的加密 MMIO 与 DMA 通道)
        ▼  │
  [ CPU TSM (TEE Security Manager) ] ◀──(SPDM/CMA 认证 + IDE AES-GCM 密钥)──▶ [ PCIe TDI (PF / SR-IOV VF) ]
        ▲
        │  (宿主机 Hypervisor / VFIO 仅负责转发管理请求，无法解密或篡改数据！)
  [ Hypervisor (Untrusted) ]
```

1. **认证底座（`SPDM` / `CMA`）**：CPU 内的可信安全管理器（TSM）先通过 DOE/SPDM 验明设备硬件身份与固件度量，确保硬件本身未运行恶意固件。
2. **总线加密（`IDE`）**：开启 PCIe 链路或选择性流（Selective Stream）的 **AES-GCM** 硬件加密，防止物理探针或恶意中介窃听。
3. **接口锁定（`TDISP` 状态机）**：
   - **`UNLOCKED` → `CONFIG_LOCKED` → `RUN`**：一旦 TDI（如某个 SR-IOV VF）进入锁定态，任何未经机密虚拟机（TVM）确认的 BAR/MSI-X/配置空间修改都会立刻被硬件拒绝或触发清零复位，彻底切断 Hypervisor 的旁路攻击面。

---

## 关联与引申概念

- **前置认证与加密**：TDISP 依赖的 DOE / SPDM / IDE 基础协议，详见 [设备认证与链路加密：SPDM / CMA / DOE / IDE](spdm-cma-doe-ide.md)。
- **传统设备直通**：非机密计算下的 VFIO 与 IOMMU 隔离机制，详见 [中断重映射、VFIO 与 SR-IOV 虚拟化](../host-system/interrupt-remapping-and-vfio.md)。
