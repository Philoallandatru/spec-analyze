# NVMe Boot 规范与 ACPI `NBFT` 表

在超大规模云数据中心与无盘计算节点（Diskless Server）中，服务器机身内部不再插任何本地 SSD，操作系统直接从远端存储池的 **NVMe/TCP（或 NVMe/FC）** 命名空间网络启动。**NVMe Boot Specification** 标准化了“UEFI 固件如何连上网盘加载内核，并把连接参数无缝移交给操作系统”的全过程 `[NVMe Boot §1–4]`。

---

## `NBFT`（NVMe Boot Firmware Table）：无盘启动的交接棒

过去 iSCSI 无盘启动依赖 `iBFT` 表；NVMe Boot 规范定义了同等的 ACPI 表——**`NBFT`（NVMe Boot Firmware Table）**，结构上彻底解决了厂商私有驱动无法互通的痛点：

```text
  1. UEFI 启动阶段：
     BIOS/UEFI 网卡驱动连上交换机 ──▶ UEFI NVMe-oF 驱动连接远端 Discovery / I/O 控制器
     ──▶ 读取远端盘 EFI 分区加载 Linux 内核与 initramfs ──▶ 在内存中构建 ACPI NBFT 表
                                                                      │
  2. ExitBootServices() 移交操作系统：                                ▼
     Linux 内核 / dracut 启动 ──▶ 读取 /sys/firmware/acpi/tables/NBFT
     ──▶ 自动配置同款 IP/VLAN、TLS/DH-HMAC-CHAP 密钥并重新挂载根文件系统 (/)
```

### `NBFT` 内部包含的关键描述符

- **HFI（Host Fabric Interface）**：记录启动所用网卡的 PCIe BDF、MAC、IP 地址、子网掩码、网关与 VLAN ID。
- **SSNS（Subsystem and Namespace）**：记录目标 NVMe-oF 控制器的 NQN、IP/端口、传输类型（TCP/RDMA/FC）、Namespace ID 以及安全认证（如 `DH-HMAC-CHAP` / TLS PSK）参数。
- **Security Descriptor**：安全凭据索引，确保启动盘连接在 OS 接管后依然保持认证与加密。

```bash
# 在支持 NVMe-oF 启动的 Linux 主机中查看 ACPI NBFT 表与解析结果
ls -la /sys/firmware/acpi/tables/NBFT*
nvme nbft show -H
```

---

## 关联与引申概念

- **ACPI 移交机制**：`MCFG`/`_OSC` 与 UEFI 交接基础，详见 [ACPI 6.5 与 UEFI 2.10：固件到操作系统的 PCIe/NVMe 交接](../host-system/acpi-uefi-pcie-deep-dive.md)。
- **NVMe/TCP 协议**：NBFT 最常承载的底层网络传输，详见 [NVMe/TCP 传输规范](../nvme-of-transports/nvme-tcp-transport.md)。
- **Fabrics 认证**：`DH-HMAC-CHAP` 在建连阶段的握手机制，详见 [Fabrics 发现与认证](../nvme/concepts/fabrics-discovery-and-authentication.md)。
