# NVMe/TCP 传输规范（PDU / 流控 / CRC32c 校验）

**NVMe/TCP Transport Specification** 将 NVMe-oF 绑定在标准 TCP/IP 协议栈之上（默认端口 **`4420`** 用于 I/O 控制器，**`8009`** 用于 Discovery 服务）。它无需昂贵的 RDMA 网卡或专用光纤交换机，直接在现有以太网上跑出接近本地盘的吞吐 `[NVMe/TCP §1–3]`。

可以把 NVMe/TCP 想成**把 NVMe 胶囊装进标准集装箱卡车（TCP 字节流）**：由于 TCP 本身只是一条没有边界的字节流，NVMe/TCP 必须在应用层定义 **PDU（Protocol Data Unit）** 来切分命令、数据与流控信号。

---

## 连接初始化与 PDU 类型

TCP 三次握手成功后，双方**必须先交换初始化 PDU** 才能发送任何 NVMe `Connect` 命令 `[NVMe/TCP §3.3]`：

1. **`ICReq`（Initialize Connection Request，Host → Ctrl）**：协商是否开启 Header/Data Digest（CRC32c）、最大 `H2CData` 长度（`MAXH2CDATA`）与 PDU 对齐参数。
2. **`ICResp`（Initialize Connection Response，Ctrl → Host）**：确认最终生效参数。

随后业务流转依赖以下 5 类核心 PDU：

| PDU 类型 | 方向 | 作用 |
| :--- | :--- | :--- |
| **`CapsuleCmd`** | Host → Ctrl | 携带 64B SQE（及可选 **In-Capsule Data** 小数据写） |
| **`CapsuleResp`** | Ctrl → Host | 携带 16B CQE（含 `SQHD` 队列流控指针与状态码） |
| **`R2T`（Ready To Transfer）** | Ctrl → Host | **写流控信号**：通知主机“我已准备好接收某条写命令的第 X 段数据” |
| **`H2CData`** | Host → Ctrl | 响应 `R2T`，将写数据流式发往控制器 |
| **`C2HData`** | Ctrl → Host | 控制器将读出的数据分片发回主机（末片可置 `SUCCESS` 位直接省掉 `CapsuleResp`） |

```text
大块写操作（超出 In-Capsule Data 限制）时序：
  Host ── CapsuleCmd (Write SQE) ──▶ Controller
  Host ◀── R2T (Offset, Length) ──── Controller  (防 Target 接收缓冲溢出)
  Host ── H2CData (Payload) ───────▶ Controller
  Host ◀── CapsuleResp (CQE) ─────── Controller
```

---

## 数据完整性：`HDGST` 与 `DDGST`（CRC32c）

虽然 TCP 自带 16-bit 校验和，但极易漏检网卡/内存总线上的多比特翻转。NVMe/TCP 支持在 `ICReq`/`ICResp` 中协商开启可选的 **32-bit CRC32c 校验** `[NVMe/TCP §3.2]`：
- **`HDGST`（Header Digest）**：紧跟在 PDU 头部之后的 4 字节 CRC32c。
- **`DDGST`（Data Digest）**：紧跟在 Data Payload 之后的 4 字节 CRC32c（硬件网卡卸载如 Intel/Mellanox TCP Offload 常支持零开销计算）。

```bash
# 使用 nvme-cli 发现并挂载一个 NVMe/TCP 远端存储目标
nvme discover -t tcp -a 192.168.10.50 -s 8009
nvme connect -t tcp -n nqn.2026-01.org.example:subsys1 -a 192.168.10.50 -s 4420
```

---

## 关联与引申概念

- **纯消息模型**：胶囊内数据（In-Capsule Data）与 `ICDOFF` 规则，详见 [传输模型](../nvme/concepts/transport-models.md)。
- **无盘网络启动**：UEFI 阶段如何连接 NVMe/TCP 并将参数通过 `NBFT` 交给内核，详见 [NVMe Boot 规范与 NBFT 表](../nvme-extended-specs/nvme-boot-and-nbft.md)。
- **性能对比**：与 RDMA 零拷贝及 FC 的架构差异，详见 [NVMe/RDMA 传输规范](nvme-rdma-transport.md)。
