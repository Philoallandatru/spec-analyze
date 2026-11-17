# NVMe over Fabrics（NVMe-oF）传输规范族

当存储从单机本地盘（PCIe）走向跨机架池化（Disaggregated Storage）时，**NVMe over Fabrics（NVMe-oF）** 通过 **TCP、RDMA（RoCEv2 / InfiniBand / iWARP）与 Fibre Channel（FC-NVMe）** 三大传输绑定规范，将 NVMe 队列模型无损延伸到数据中心网络 `[NVMe-oF Transport Specs]`。

> **20/80 裁剪原则**：聚焦三种网络传输的建连握手、Capsule 与数据搬运差异、流控/校验机制以及常见生产排错（CRC32c、PFC/ECN、多路径）。

---

## 文档导航

1. **[NVMe/TCP 传输规范（PDU / 流控 / CRC32c 校验）](nvme-tcp-transport.md)**
   - `ICReq`/`ICResp` 建连、`CapsuleCmd`/`CapsuleResp`、`R2T`/`H2CData`/`C2HData` 流控与 `HDGST`/`DDGST` 校验。
2. **[NVMe/RDMA 传输规范（RoCEv2 / R_Key / 零拷贝）](nvme-rdma-transport.md)**
   - `CM REQ` 建立 RC QP、`IBV_WR_SEND` 控制面与 `RDMA Read`/`Write` 数据面、`R_Key` 授权与 RoCEv2 PFC/ECN 无损网络。
3. **[NVMe/FC 光纤通道传输规范（FC-NVMe-2）](nvme-fc-transport.md)**
   - 传统 SAN 架构平滑演进：`PRLI` 协商、`NVMe_IU` 帧映射与原生多路径（ANA）。
4. **[NVMe-oF 自动化发现（CDC / mDNS）与端到端安全（TLS / CHAP）](nvme-of-discovery-and-security.md)**
   - `mDNS/DNS-SD` 零配置发现、中央发现控制器（CDC）、`TLS 1.3 PSK (kTLS)` 与 `DH-HMAC-CHAP` 双向认证。

---

## 三种 Fabrics 传输核心对比

| 维度 | NVMe/TCP | NVMe/RDMA（RoCEv2） | NVMe/FC（FC-NVMe-2） |
| :--- | :--- | :--- | :--- |
| **NVMe 传输模型** | 纯消息模型（Message-Only） | 消息/内存混合（Message/Memory） | 消息/内存混合（Message/Memory） |
| **网络基础设施** | 普通以太网（零专用硬件门槛） | 需 RDMA 网卡 + 无损以太网（PFC/ECN） | 专用 FC HBA + 光纤交换机（SAN） |
| **大块数据搬运** | `H2CData` / `C2HData` PDU（可启用 `R2T`） | Target 主动 **`RDMA Read` / `RDMA Write`**（带 `R_Key`） | FCP 交换序列（`NVMe_XFER_RDY`） |
| **典型端到端延迟** | 100–300 µs（可配合 TLS 1.3 加密） | 10–30 µs（CPU 旁路零拷贝） | 20–50 µs（企业级高可靠 SAN） |

---

## 关联与引申概念

- **传输抽象模型**：Capsule 布局与 `SQHD` 流控基础，详见 [传输模型](../nvme/concepts/transport-models.md)。
- **发现与安全**：Fabrics Discovery Controller 与带内认证，详见 [Fabrics 发现与认证](../nvme/concepts/fabrics-discovery-and-authentication.md)。
- **无盘网络启动**：如何通过 UEFI + `NBFT` 直接从 NVMe/TCP 启动操作系统，详见 [NVMe Boot 规范与 NBFT 表](../nvme-extended-specs/nvme-boot-and-nbft.md)。
