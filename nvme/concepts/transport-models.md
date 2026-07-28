# 传输模型（Transport Models）

## 一句话说明

NVMe 把"主机到 NVM 子系统"拆成两种传输模型：基于内存的传输（典型 PCIe，主机与控制器共享内存）和基于消息的传输（典型 Fabrics，命令/响应/数据封装成胶囊在网络上传递）；基础规范与具体传输通过"传输绑定层"解耦。

## 生活化类比

把 NVMe 传输想成**两种快递方式**：

- **基于内存（PCIe）** = 同一栋写字楼里内部传递：主机和控制器共处一个内存空间，写到 SQ 就是快递放进"前台桌"，控制器直接走过去取，**写个 Doorbell 敲门**就通知取件。
- **基于消息（Fabrics）** = 跨城快递：主机把命令打包成"胶囊"（Capsule），走网络（TCP / RDMA / 光纤）发到远方控制器；控制器把响应再打包发回来。胶囊可以夹带小件（数据），大件走单独运单（SGL 指向的内存操作）。
- **混合模式（Message/Memory）** = 快递小件用胶囊，大件单独派车（显式内存事务）。

## 工作流程

```text
  NVMe 协议栈分层
  ┌────────────────────────────────────────┐
  │  NVMe 核心（队列 / 命令 / 控制器属性）  │   ← 与传输无关
  └────────────────────────────────────────┘
                ↕  传输绑定层（Binding）
  ┌────────────────────────────────────────┐
  │  NVMe 传输层（NVMe Transport）          │
  └────────────────────────────────────────┘
                ↕  底层网络（不在 NVMe 规范范围）
  PCIe / RDMA / TCP / FC ...
```

简化说明：基础规范只管"协议怎么做"，传输规范管"在 PCIe / RDMA / TCP / FC 上怎么落地"。基于消息的传输下，信息交换单位是"胶囊"（Capsule），可包含命令/响应 + 可选数据 + 可选 SGL。

## 初学者案例

**场景：为什么有些命令在本地 PCIe SSD 上工作，放到 NVMe-oF TCP 上却失败？**

1. 工程师在本地发了一条自定义 Vendor Specific 命令，PCIe 下走 MMIO 直接生效。
2. 部署到 NVMe-oF TCP 集群后，部分命令报 `SGL Offset Invalid`。
3. 排查：
   - 读 Identify Controller 的 `IOCCSZ` / `IORCSZ`（I/O 命令/响应胶囊大小）。
   - 检查命令 SGL 是否落在胶囊内偏移 `64 + ICDOFF × 16` 字节之后。
   - 报"Offset Invalid"通常意味着 SGL 把数据放在了命令头 64 字节内，PCIe 下没问题，**消息模式下违反数据放置规则**。
4. 解决：把数据从"in-capsule"改成"显式内存地址"SGL；或在 PCIe 下保留旧路径。

> 速记：**PCIe 下的数据布局规则在 Fabrics 下不一定适用**——尤其 SGL 的"偏移/地址/额外描述符段"语义不同。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 两种基本模型 | 内存映射（Memory-Based）+ 消息（Message-Based） |
| PCIe 典型 | 内存映射 = 共享内存 + MMIO + DMA |
| Fabrics 典型 | 消息 = 胶囊（capsule）在网络上传递 |
| 消息子类型 | 纯消息（Message-Only）+ 消息/内存混合（Message/Memory） |
| 分层 | 核心协议 ↔ 传输绑定层 ↔ 传输层 ↔ 底层网络 |
| 网络层不在规范 | 底层 PCIe / RDMA / TCP / FC 本身**不在 NVMe 规范族范围** |
| Capsule 定义 | 信息交换单位 = 命令或响应 + 可选数据 + 可选 SGL |
| Admin Queue capsule 大小 | 由传输绑定（Binding）固定 |
| I/O Queue capsule 大小 | 由 Identify Controller 报告（IOCCSZ、IORCSZ） |
| SGL 支持 | Admin 与 I/O 队列均支持；具体 SGL 类型由传输绑定选 |
| SGL 未用字节 | 胶囊中未用且非保留的字节是**未定义**的，不得解释 |
| SQE 之后不可混合 | 命令胶囊在 SQE 之后**不会同时混合**数据和额外 SGL |
| 唯一 CID | 队列内未完成命令的 Command Identifier 必须唯一 |
| 共享内存含义 | 内存映射下，主机 SQ/CQ/数据在主机内存；控制器可 DMA 访问 |
| 端口定义 | NVM 子系统端口用 NQN + 16 位 Port ID 标识；可聚合多物理接口 |
| 单端口多协议 | 一个子系统端口可支持多种传输协议 |
| 绑定控制器 | 关联的控制器绑定到特定子系统端口 |
| 命令投放（内存） | 内存映射下"提交"= SQ Tail Doorbell 写跨过命令槽位 |
| 命令投放（消息） | 消息模式下"提交"= 主机把命令胶囊加入 SQ |
| 完成定义 | 命令完成 = 处理完毕 + CQE 状态已更新 + CQE 已写入关联 CQ |
| 显式内存事务 | 消息/内存混合下数据可走显式内存操作（不受胶囊大小限制） |
| 描述符数量违规 | 中止命令，返回 `Invalid Number of SGL Descriptors` |
| SGL 偏移无效 | 返回 `SGL Offset Invalid` |
| 传输无关 | NVMe 接口位于绑定层之上、原生 Fabric 之下 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 内存映射 vs 消息 | 内存映射共享内存；消息是网络胶囊 |
| 纯消息 vs 消息/内存混合 | 纯消息全走胶囊；混合可走显式内存事务 |
| Capsule vs SQE/CQE | SQE/CQE 是 64/16 字节的命令/完成结构；Capsule 是 Fabrics 下的传输封装单元 |
| 内存映射传输 vs 内存窗口 | 传输是物理层；内存窗口（CMB/PMA）是设备提供的可被主机读写的设备内存 |
| NVMe Transport vs 传输规范 | Transport 是抽象服务层；传输规范（如 NVMe-oF TCP）是绑定文档 |
| 端口 vs 控制器 | 端口是传输层接口；控制器是命令处理单元；一个控制器绑定到一个端口 |
| NVMe Transport vs PCIe | NVMe Transport 是规范抽象；PCIe 是底层互联 |
| 内存地址 SGL vs 偏移 SGL | 内存地址走显式内存事务；偏移走胶囊内承载 |
| Doorbell vs Capsule 提交 | Doorbell 是 MMIO 寄存器；Capsule 提交是网络消息 |
| 基础规范 vs 传输规范 | 基础规范是协议；传输规范是绑定（具体传输实现） |

## 进阶细节

- **两种基本模型**（规范 1.5.20 / 3.1.1 / 3.1.2 / 4.1）：
  - 内存映射（Memory-Based）= PCIe 之类，控制器可直接 DMA 主机内存
  - 消息（Message-Based）= Fabrics，命令/响应/数据封装成 capsule
- **三种传输分类**（规范 4.1）：
  - 内存映射：命令/响应 + 数据**全部**走显式内存读/写
  - 纯消息：命令/响应 + 数据**全部**走 capsule
  - 消息/内存混合：命令/响应走 capsule，数据**可**走显式内存事务
- **NVMe 协议栈**（规范 4.1）：
  - 核心架构 = 队列、命令集、控制器属性
  - 传输绑定层 = 适配具体传输
  - NVMe Transport = 抽象传输服务
  - 底层网络（PCIe / RDMA / TCP / FC）**不在 NVMe 规范族范围**
- **Capsule 定义**（规范 1.5.15）：NVMe over Fabrics 的信息交换单位；含命令或响应 + 可选数据 + 可选 SGL。
- **命令投放语义**（规范 1.5.19）：
  - 内存映射：SQ Tail Doorbell 写跨过该命令槽位
  - 消息：主机把 capsule 加入 SQ
- **完成定义**（规范 1.5.18）：命令完成 = 处理完毕 + CQE 状态更新 + CQE 已写入关联 CQ。
- **Fabrics 队列 capsule 尺寸**（规范 4.1.4）：
  - Admin Queue capsule 大小 = 传输绑定固定
  - I/O Queue capsule 大小 = Identify Controller 报告
- **Capsule 布局**（规范 4.1.5 / Figures 75-76）：
  - 命令 capsule = 64 字节 SQE + 数据 或 额外 SGL（**不同时混合**）
  - 响应 capsule = 16 字节 CQE + 可选数据
  - 额外 SGL 从字节 64 开始
  - Capsule 内命令数据从字节偏移 `64 + ICDOFF × 16` 开始
  - 中间未定义字节**不得解释**
- **SGL 数据放置**（规范 4.1.5.3）：
  - 内存地址：数据走显式内存事务（如 RDMA）
  - 偏移：数据直接承载在命令/响应 capsule 中
  - 额外描述符段：描述符紧接 SQE 之后连续排列
- **错误码**（规范 4.1.5）：
  - 描述符数量违规 → `Invalid Number of SGL Descriptors`
  - SGL 偏移无效 → `SGL Offset Invalid`
- **子系统端口**（规范 3.1.4）：
  - 协议接口，可聚合一个或多个物理网络接口
  - 由 NQN + 16 位 Port ID 标识
  - 单端口可支持多种传输协议
  - 关联的控制器绑定到特定子系统端口
- **底层网络不在规范**（规范 1.4.1.7 / 4.1）：PCI、PCIe、PCI-X 本身的实现**不在 NVMe 规范族范围**。

## 规范依据

- [传输分类与三种模型，PDF 第 39-40 页](../_source/pages/page-039.md)
- [Fabrics 分层与 port 绑定，PDF 第 43-44 页](../_source/pages/page-043.md)
- [Capsule 定义（1.5.15），PDF 第 29 页](../_source/pages/page-029.md)
- [基于消息的队列交付、capsule 尺寸与 SGL 契约，PDF 第 110 页](../_source/pages/page-110.md)
- [Command/response capsule 布局与数据放置规则，PDF 第 111-114 页](../_source/pages/page-111.md)

## 相关阅读

- [queue-pair.md](queue-pair.md) - 不同传输下队列对的实现
- [association-and-command-lifecycle.md](association-and-command-lifecycle.md) - 传输类型决定关联建立方式
- [fabrics-discovery-and-authentication.md](fabrics-discovery-and-authentication.md) - Fabrics 传输的发现与认证
- [specification-family-and-scope.md](specification-family-and-scope.md) - 传输规范在规范族中的位置
