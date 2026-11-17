# NVMe/FC 光纤通道传输规范（FC-NVMe-2）

在金融、电信与大型企业级核心存储阵列（SAN）中，客户机房已铺设大量 **Fibre Channel（光纤通道，32G/64G FC）** 交换机。**NVMe/FC（由 INCITS T11 `FC-NVMe-2` 与 NVMe-oF 协同定义）** 允许在同一条 FC 光纤上同时承载传统的 FCP（SCSI）流量与全新的 NVMe 流量，实现零布线改造的平滑升级 `[FC-NVMe-2 §4]`。

---

## 协议映射核心机制

| FC-NVMe 概念 | 对应协议机制 | 作用 |
| :--- | :--- | :--- |
| **`PRLI`（Process Login）** | FC-4 Link Service | 端口登录时协商 `FC-NVMe` 服务参数（支持 Discovery / I/O 队列、最大突发长度 `First Burst`） |
| **`NVMe_CMND IU`** | 信息单元（Information Unit） | 承载 64B NVMe SQE（及连接标识 Connection ID），等价于 `CapsuleCmd` |
| **`NVMe_XFER_RDY IU`** | 流控帧 | Target 通知 Initiator 可以发送写数据（类似 TCP 的 `R2T`） |
| **`NVMe_DATA IU`** | 数据帧 | 承载读写数据载荷（可开启 **First Burst** 随命令一起发送首段写数据，省去一次 `XFER_RDY` 往返） |
| **`NVMe_ERSP IU`** | 扩展响应帧 | 承载 16B CQE 与完整 SQ 头指针（`SQHD`）流控信息 |

---

## 工程优势与生态定位

- **天然无损与硬件隔离**：FC 协议自带基于 **Buffer-to-Buffer Credit（BB_Credit）** 的硬件级流量控制，永不丢包；配合 **Zoning（分区）** 天然实现多租户物理隔离。
- **原生多路径（ANA）**：FC SAN 标配双交换机冗余（Fabric A / Fabric B），与 NVMe Base 的 **ANA（Asymmetric Namespace Access）** 完美契合，链路故障可在毫秒级无感切换。

```bash
# 查看 Linux 主机识别到的 FC HBA 端口与已连接的 NVMe/FC 子系统
ls /sys/class/fc_host/
nvme list-subsys
```

---

## 关联与引申概念

- **多路径访问模型**：ANA 状态（Optimized / Non-Optimized / Inaccessible）切换机制，详见 [命名空间访问模型](../nvme/concepts/namespace-access-models.md)。
- **通信丢失与重试**：链路闪断后的命令回收与 `KATO` 超时机制，详见 [通信丢失与命令重试](../nvme/concepts/communication-loss-and-command-retry.md)。
