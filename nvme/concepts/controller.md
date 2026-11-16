# 控制器（Controller）

控制器是主机（Host）与 NVM 子系统之间的接口实体：主机把命令写入提交队列（Submission Queue，SQ），控制器取出执行后把结果写入完成队列（Completion Queue，CQ）。无论是读写数据还是管理设备，所有命令与数据都必须经过控制器。[PDF 第 29 页](../_source/pages/page-029.md)

## 心智模型：银行柜台

把控制器想象成**银行柜台**：
- **柜台 = 控制器**：客户（主机）不能直接进金库（存储介质），只能通过柜台办业务。
- **取号单 = 提交队列（SQ）**：主机把要做的事（读、写、管理命令）排队放进去，再“敲门铃”（Doorbell）通知控制器来取。
- **叫号回执 = 完成队列（CQ）**：控制器执行完后写入完成队列条目（CQE），并通过中断或轮询通知主机。
- **多窗口 = 多队列**：不同优先级的 SQ 就像 VIP 通道和普通窗口，控制器按公平与优先级仲裁（Arbitration）调度执行。

## 工作流程

```text
  +-------------------+  1. 写命令到 SQ    +----------------+
  | 主机软件          | -----------------> |  提交队列 (SQ) |
  |  (驱动/用户态)    |  2. 敲 Doorbell    +----------------+
  +-------------------+ ---------------------------|
                                                   v
                                          +----------------+
                                          |  NVMe 控制器   |
                                          |  - 取命令/仲裁 |
                                          |  - 操作介质    |
                                          +----------------+
                                                   |
                                                   v  3. 写 CQE
                                          +----------------+
  +-------------------+  4. 中断通知       |  完成队列 (CQ) |
  | 主机软件          | <----------------- +----------------+
  +-------------------+
```

> PCIe 下“敲 Doorbell”是写 MMIO 寄存器；NVMe over Fabrics（NVMe-oF）下则是通过网络发送 Capsule。

## 两种工作模型：静态 vs 动态

同一个 NVM 子系统内的**所有控制器必须采用同一种模型**，不能混用。[PDF 第 58 页](../_source/pages/page-058.md)

| 模型 | 核心特点 | 适用传输 | 典型场景 |
|---|---|---|---|
| **静态控制器（Static）** | 拥有持久化 Controller ID 和 Feature 设置，跨控制器级复位（CLR）与跨关联保留状态 | PCIe 强制使用；NVMe-oF 可选 | 本地 SSD、需要固定身份与配置的存储 |
| **动态控制器（Dynamic）** | 按需临时分配、无状态，每次关联都是全新一致的初始状态 | 仅 NVMe-oF（Discovery 控制器强制使用） | 云存储、大规模网络池化环境 |

- **动态模型连接约定**：在 NVMe-oF 下连接动态控制器时，主机在 Fabrics Connect 命令中将 `CNTLID` 设为 `FFFFh`（表示由子系统分配任意可用控制器）。同一子系统端口下所有动态控制器初始挂载的命名空间集合相同。

## 三种功能类型

Identify Controller 数据结构中的 `CNTRLTYPE` 字段用于标识控制器类型（规范 3.1.3）：[PDF 第 59-61 页](../_source/pages/page-059.md)

| 控制器类型 | 支持队列 | 可挂载命名空间？ | 核心用途 |
|---|---|---|---|
| **I/O 控制器** | Admin 队列 + 多个 I/O 队列 | ✅ 是 | 处理用户数据读写与设备管理，可同时支持多个 I/O 命令集（日常 SSD 绝大多数为此类） |
| **发现控制器（Discovery）** | 仅 Admin 队列 | ❌ 否 | 专用于 NVMe-oF，只提供 Discovery Log Page 指引主机找到可用的 I/O 控制器 |
| **管理控制器（Administrative）** | 仅 Admin 队列 | ❌ 否 | 专用于企业级带内/带外管理（如健康轮询、命名空间管理、虚拟化管理、NSSR/NSSD），隔离管理与数据面 |

## 核心规则与易混点

- **必有 Admin 队列**：任何控制器至少有一对 Admin SQ/CQ；I/O 队列由主机通过 `Create I/O SQ/CQ` 命令按需创建。
- **PCIe 下控制器 = PCI Function**：内存映射传输中，一个 PCIe Function 对应一个 NVMe 控制器。
- **关联排他与终止**：一个控制器同一时刻只能与一台主机建立关联（Association）。没有专门的“断开关联”命令（`Disconnect` 仅删除 I/O 队列），关联仅在以下 4 种情况终止：控制器 Shutdown、控制器级复位（CLR）、Admin 队列传输断开、或不支持单队列删除时任一 I/O 队列断开。
- **命令与完成无顺序保证**：除融合操作（Fused Operation）外，控制器对同一 SQ 内或跨 SQ 的命令都不保证执行和完成顺序；主机通过 CQE 中的 `SQID + CID` 匹配对应命令。
- **快速排错口诀**：`lspci` 能看到设备但 `nvme list` 看不到盘，先查 `dmesg` 是否卡在使能阶段（`CC.EN=1` 等待 `CSTS.RDY=1` 超时），再查 `nvme id-ctrl` 与 `nvme list-ns` 确认是否尚未创建/挂载命名空间（**控制器在 = 链路通；命名空间在 = 业务通**）。

## 关联与引申概念

- **提交队列 / 完成队列（SQ / CQ）**：主机与控制器之间传递命令和回执的环形缓冲区，见 [队列对（Queue Pair）](queue-pair.md)。
- **命名空间（Namespace）**：可格式化为块设备的逻辑存储分区，只能挂载到 I/O 控制器，见 [命名空间（Namespace）](namespace.md)。
- **NVM 子系统（NVM Subsystem）**：包含一个或多个控制器、端口和存储介质的完整逻辑边界，见 [NVM 子系统](nvm-subsystem.md)。
- **命令集（Command Sets）**：规定控制器能识别哪些 Admin 与 I/O 命令，见 [命令集（Command Sets）](command-sets.md)。
- **主机-控制器关联（Association）**：主机通过 Fabrics Connect 或 PCIe 初始化与控制器建立排他会话的过程，见 [关联与命令生命周期](association-and-command-lifecycle.md)。

## 规范依据

- [控制器定义（1.5.20），PDF 第 29 页](../_source/pages/page-029.md)
- [控制器架构：静态 vs 动态模型，PDF 第 58 页](../_source/pages/page-058.md)
- [三种控制器类型与 Figure 23，PDF 第 59 页](../_source/pages/page-059.md)
- [I/O 控制器与多 I/O 命令集（3.1.3.1 / Figure 24），PDF 第 60 页](../_source/pages/page-060.md)
- [Administrative 控制器（3.1.3.2 / Figure 25），PDF 第 61 页](../_source/pages/page-061.md)
