# NVMe/RDMA 传输规范（RoCEv2 / R_Key / 零拷贝）

**NVMe/RDMA Transport Specification** 是超低延迟分布式存储与 AI 训推集群的首选方案，支持运行在 **RoCEv2（UDP Port `4791`）**、**InfiniBand** 或 **iWARP** 之上。它属于**消息/内存混合模型**——控制面用消息传胶囊，数据面直接让远端网卡通过 **RDMA Read / RDMA Write** 读写主机物理内存，全程旁路 CPU 与内核协议栈 `[NVMe/RDMA §1–3]`。

---

## 建连与控制/数据双平面分工

1. **建连阶段（`RDMA CM`）**：主机发起 **`CM REQ`** 请求与目标端口建立一对一的 **RC（Reliable Connected）Queue Pair**，并在私有数据（Private Data）中协商队列深度（`SQSIZE`）与 `HRQSIZE`/`HSQSIZE`。
2. **控制面（`IBV_WR_SEND` / `RECV`）**：主机将 64B SQE 封装为 `CapsuleCmd` 通过 RDMA `Send` 发往目标；目标通过 RDMA `Send` 回传 `CapsuleResp`（16B CQE）。
3. **数据面（`RDMA Read` / `RDMA Write` + `R_Key`）**：
   - 主机发起大块 I/O 前，先用 `ibv_reg_mr()` 向本机 RDMA 网卡注册内存，拿到内存地址与 **`R_Key`（Remote Memory Key）**。
   - 主机把 `(VA, Length, R_Key)` 填入 SQE 的 **Keyed SGL** 描述符中发给 Target。
   - **Write 操作**：Target 主动发起 **`RDMA Read`** 直接从主机内存拉取数据。
   - **Read 操作**：Target 主动发起 **`RDMA Write`** 直接把数据推入主机内存，随后紧跟一条 `CapsuleResp` 通知完成。

```text
  Host (Initiator)                                Target (Controller)
        │── 1. RDMA Send (CapsuleCmd + Keyed SGL[R_Key]) ─▶│
        │◀─ 2. RDMA Read/Write (NIC 零拷贝直读/直写主机内存) ─│
        │◀─ 3. RDMA Send (CapsuleResp / CQE) ─────────────│
```

---

## RoCEv2 无损网络排错关键点

- **PFC（Priority Flow Control, 802.1Qbb）与 ECN（Explicit Congestion Notification）**：RoCEv2 跑在以太网上，一旦交换机丢包触发重传，性能会断崖式下跌。生产环境必须在交换机与网卡上对 RoCEv2 流量（通常 DSCP 26 / Priority 3）配置 **ECN 拥塞标记 + PFC 暂停帧**。
- **`R_Key` 失效与内存注册开销**：频繁 `ibv_reg_mr` 会拖垮性能，Linux `nvme-rdma` 与 SPDK 均采用**预注册内存池（MR Pool）**或 **FRWR（Fast Registration Work Request）** 复用 `R_Key`。

```bash
# 查看本机 RDMA 设备、RoCEv2 端口状态与挂载 NVMe/RDMA 目标
rdma link show
nvme connect -t rdma -n nqn.2026-01.org.example:subsys1 -a 192.168.20.50 -s 4420
```

---

## 关联与引申概念

- **Keyed SGL 格式**：携带 `R_Key` 的 SGL 描述符结构，详见 [数据指针布局](../nvme/concepts/data-pointer-layouts.md)。
- **TCP 对比**：无需无损以太网即可部署的纯消息方案，详见 [NVMe/TCP 传输规范](nvme-tcp-transport.md)。
