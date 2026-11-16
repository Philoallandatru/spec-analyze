# 传输模型（Transport Models）

NVMe 将主机与 NVM 子系统之间的通信抽象为**基于内存的传输（Memory-Based，如 PCIe）**和**基于消息的传输（Message-Based，如 NVMe over Fabrics）**两种基本模型，并通过**传输绑定层（Transport Binding）**将核心协议与底层物理/网络传输解耦。[PDF pp. 39-46](../_source/pages/page-039.md)

可以把这两种模型想成**两种快递方式**：
- **基于内存（PCIe）** = 同栋写字楼内部传递：主机把命令放进共享内存的"前台桌"（SQ），**敲一下门铃（Doorbell MMIO）**，控制器直接通过 DMA 来取件和送数据。
- **基于消息（Fabrics）** = 跨城网络快递：主机把命令打包成**胶囊（Capsule）**通过 TCP/RDMA/FC 发给远方控制器；小件数据可直接塞进胶囊里（In-Capsule Data），大件数据则通过显式内存事务（如 RDMA）单独运输。

---

## NVMe 协议分层与三种传输模型

```text
  ┌───────────────────────────────────────────────┐
  │  NVMe 核心架构（队列、命令集、控制器属性）     │  ← 与传输无关（基础规范）
  └───────────────────────────────────────────────┘
                        ↕  传输绑定层 (Transport Binding)
  ┌───────────────────────────────────────────────┐
  │  NVMe 传输层（在底层网络上提供可靠交付）        │
  └───────────────────────────────────────────────┘
                        ↕  底层物理/网络（不在 NVMe 规范族范围）
  PCIe  /  RDMA  /  TCP  /  Fibre Channel
```

[PDF pp. 43-44](../_source/pages/page-043.md)

| 传输模型 | 典型代表 | 命令/响应传递 | 数据传输方式 |
|----------|----------|---------------|--------------|
| **基于内存（Memory-Based）** | PCIe | 共享内存（SQ/CQ）+ MMIO 门铃 | 显式内存读写（DMA + PRP/SGL） |
| **纯消息（Message-Only）** | NVMe/TCP | 胶囊（Capsule） | 仅通过胶囊内数据传递 |
| **消息/内存混合（Message/Memory）** | NVMe/RDMA | 胶囊（Capsule） | 胶囊或显式内存操作（如 RDMA 读写） |

---

## 胶囊（Capsule）机制与数据放置（Fabrics）

在基于消息的传输中，信息交换的基本单位是**胶囊（Capsule）**。Admin 队列的胶囊大小由传输绑定固定；I/O 队列的胶囊大小由 `Identify Controller`（`IOCCSZ` / `IORCSZ`）报告。[PDF pp. 29, 110-114](../_source/pages/page-110.md)

```text
  命令胶囊（主机 → 控制器）：
  ┌──────────────────────────┬─────────────────────────────────────┐
  | 64 字节 SQE              | 数据 (从 64 + ICDOFF×16 起)         |
  | (含唯一 CID 与首条 SGL)  | 或 额外的 SGL 描述符段 (不可同时混合) |
  └──────────────────────────┴─────────────────────────────────────┘

  响应胶囊（子系统 → 主机）：
  ┌──────────────────┬─────────────────────────────────────────────┐
  | 16 字节 CQE      | 可选响应数据                                |
  └──────────────────┴─────────────────────────────────────────────┘
```

**关键规则与 SGL 放置策略**：
- **命令胶囊互斥规则**：SQE 之后**不能同时混合**内联数据和额外的 SGL 描述符段；未使用且非保留的字节属于未定义状态，不得解释。
- **SGL（Scatter-Gather List）放置方式**：
  1. **内存地址**：数据通过传输层支持的显式内存事务（如 RDMA）传输，不受胶囊大小限制。
  2. **偏移量（Offset）**：数据直接承载在命令或响应胶囊内部（命令内联数据从 `64 + ICDOFF × 16` 字节偏移开始）。
  3. **额外描述符段**：额外的 SGL 描述符紧接 SQE 之后从字节 64 开始连续排列。
- **错误状态**：违反描述符数量上限返回 `Invalid Number of SGL Descriptors`；内联偏移配置不当（例如数据落入前 64 字节 SQE 区间）返回 `SGL Offset Invalid`。

---

## 传输层与子系统端口（Port）

- **NVM 子系统端口**：通过 **NQN（NVMe Qualified Name）** 和 **16 位 Port ID** 标识的协议接口，可聚合一个或多个物理网络接口，支持多种传输协议；已关联的控制器绑定到特定子系统端口。[PDF pp. 39, 44](../_source/pages/page-039.md)
- **命令投放与完成语义**：内存模型下，"投放"指 SQ Tail Doorbell 写入跨过槽位；消息模型下，指主机将命令胶囊加入 SQ。两种模型下，"完成"均指命令处理完毕、CQE 状态更新并写入关联 CQ。

---

## 关联与引申概念

- **MMIO（Memory-Mapped I/O）与 DMA（Direct Memory Access）**：PCIe 传输模型的核心基石。主机通过 MMIO 读写控制器 BAR0 寄存器（如敲门铃），控制器则通过 DMA 自主读写主机内存中的队列和数据缓冲区。
- **PRP vs SGL**：PCIe 传统上主要使用按物理页对齐的 **PRP（Physical Region Page）**，而 Fabrics（以及可选的 PCIe I/O 命令）使用更灵活、支持任意字节长度与偏移的 **SGL**。
- **NQN（NVMe Qualified Name）**：类似 iSCSI IQN 的全局唯一名称格式，用于在 Fabrics 网络中标识主机和 NVM 子系统。
- **相关文档**：
  - [队列对](queue-pair.md) — 内存模型与 Fabrics 消息模型下 SQ/CQ 的映射与流控差异
  - [关联与命令生命周期](association-and-command-lifecycle.md) — 传输类型如何决定控制器关联建立方式
  - [Fabrics 发现与认证](fabrics-discovery-and-authentication.md) — Fabrics 传输下的端口发现与安全认证
  - [规范族与适用范围](specification-family-and-scope.md) — 传输规范在 NVMe 规范族中的协作定位

## 规范依据

- [传输模型分类与三种子类型，PDF pp. 39-40, 46](../_source/pages/page-039.md)
- [Fabrics 协议栈分层与子系统端口绑定，PDF pp. 43-44](../_source/pages/page-043.md)
- [Capsule 定义与基于消息的队列/SGL 契约，PDF pp. 29, 110](../_source/pages/page-110.md)
- [Command/Response Capsule 布局与数据放置规则，PDF pp. 111-114](../_source/pages/page-111.md)
- [传输规范在规范族中的位置，PDF p. 23](../_source/pages/page-023.md)
