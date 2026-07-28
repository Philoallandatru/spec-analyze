# 控制器（Controller）

## 一句话说明

控制器是主机与 NVM 子系统之间的接口：主机把命令写到提交队列（Submission Queue），控制器取出执行后把结果写到完成队列（Completion Queue），命令和数据永远经过控制器这一关。

## 生活化类比

把控制器想成**银行柜台**：

- **柜台 = 控制器**：客户（主机）只能到柜台办业务，不能直接进金库（存储介质）。
- **取号单 = 提交队列**：客户把要办的业务写在单子上排队。
- **叫号回执 = 完成队列**：柜员办完把回执交回给客户。
- **共享柜面 = 多队列**：VIP 通道、对公窗口是不同优先级的提交队列，柜员按公平与优先级调度。

NVMe 的所有设计都围绕"让柜台办得快、排队不堵、结果可追溯"展开：每张单据有 16 字节命令头、每次办完都有一份完成回执。

## 工作流程

```text
  +-------------------+  1. 写命令到 SQ    +----------------+
  | 主机软件          | -----------------> |  提交队列 (SQ) |
  |  (用户态/内核)    |  2. 敲 Doorbell   +----------------+
  +-------------------+ -------------------------- |
                                                   v
                                          +----------------+
                                          |  NVMe 控制器   |
                                          |  - 取命令      |
                                          |  - 调度/仲裁   |
                                          |  - 操作介质    |
                                          +----------------+
                                                   |
                                                   v  3. 写 CQE
                                          +----------------+
  +-------------------+  4. 中断通知       |  完成队列 (CQ) |
  | 主机软件          | <----------------- +----------------+
  +-------------------+
```

简化说明：上图为单 SQ/CQ 模型；实际 NVMe 支持多对 I/O 队列，按 3.4.4 节的仲裁方案调度。PCIe 下"敲 Doorbell"是写 MMIO 寄存器；Fabrics 下"提交"是发 capsule 到 SQ。

## 初学者案例

**场景：为什么 `nvme list` 看不到我的盘，但 `lspci` 能看到 NVMe 设备？**

1. `lspci` 看到的是 **PCIe 设备（PCI Function）**，NVMe 控制器在它上面。
2. 但 NVMe 控制器可能还没被"使能"——`CC.EN = 0`（规范 3.7.2）。这时主机侧驱动加载了也读不到命名空间。
3. 排查步骤：
   - `dmesg | grep nvme`：看驱动是否识别、是否报告 `CC.EN timeout` 之类错误。
   - 一些 BIOS 启动慢的服务器，控制器需要几秒才能从 `CC.EN=0` 切到 `CSTS.RDY=1`，**先等再排错**。
   - 如果一直 `CC.EN timeout`：可能是硬件链路问题、或 NVM 子系统还没完成上电自检。
4. 另一个常见原因：盘是"裸"出厂的（`nvm-subsystem.md` 里的案例），NVM 子系统在但没有命名空间——`nvme id-ctrl` 能看到控制器，但 `nvme list-ns` 为空。

> 排错口诀：**控制器在 = PCIe 链路通；命名空间在 = 业务通**。两者分清楚。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 控制器定义（1.5.20） | 主机与 NVM 子系统之间的接口；含三种类型：I/O / Discovery / Administrative |
| 必有 Admin 队列 | 任何控制器都至少有一个 Admin SQ 和一个 Admin CQ |
| 可有 I/O 队列 | I/O 控制器可创建多个 I/O SQ/I/O CQ；Administrative 控制器**不支持** I/O 队列 |
| 命令执行方式 | 控制器从 SQ 取命令执行，完成时把完成队列条目（CQE）写到 CQ |
| PCIe 下控制器 = PCI Function | 内存映射传输下，一个 PCIe Function 就是一个 NVMe 控制器 |
| 模型一致性 | 子系统内所有控制器必须采用同一控制器模型（全部静态或全部动态） |
| 静态模型保留状态 | 控制器 ID、Feature 设置跨控制器级复位和跨关联保留 |
| 动态模型无状态 | 每次关联都重新分配，初始状态对所有动态控制器一致 |
| PCIe 强制静态 | 内存映射传输下，控制器**只能**是静态模型 |
| Discovery 强制动态 | 发现控制器**只能**是动态模型 |
| I/O 控制器可挂载命名空间 | 命名空间只能挂载到 I/O 控制器 |
| Administrative 不可挂命名空间 | Administrative 控制器用于管理，不支持访问用户数据 |
| 关联排他 | 一个控制器同一时刻只能与一个主机建立关联 |
| 关联终止 | 控制器 Shutdown、控制器级复位、Admin Queue 传输断开、或不支持单队列删除时任一 I/O Queue 断开 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 控制器 vs 命名空间 | 控制器是"执行命令的实体"；命名空间是"被读写的逻辑卷" |
| 静态 vs 动态控制器 | 静态保留 ID 和 Feature；动态每次关联是"全新控制台" |
| I/O 控制器 vs Administrative 控制器 | I/O 控制器可读写用户数据；Administrative 只能做管理 |
| Discovery 控制器 vs I/O 控制器 | Discovery 控制器只提供发现日志页面，不挂载命名空间 |
| 控制器 vs 队列 | 一个控制器可管理多对 SQ/CQ；队列只是命令/完成的中转通道 |
| 控制器 vs NVM 子系统 | 控制器是子系统内的执行单元；子系统是边界（控制器 + 端口 + 命名空间 + 资源） |
| 主机驱动 vs 控制器 | 主机驱动是软件，跑在 CPU 上；控制器是硬件或固件实体 |

## 进阶细节

- **三种控制器类型**（规范 3.1.3）：
  - **I/O 控制器**（3.1.3.1）：支持 I/O 命令集访问用户数据，**可同时支持多个 I/O 命令集**。
  - **Administrative 控制器**（3.1.3.2）：用于管理 NVM 子系统，**不支持 I/O 队列**、**不能挂载命名空间**；通过 NVMe-MI 轮询健康、命名空间管理、虚拟化管理、NSSR/NSSD 等。
  - **Discovery 控制器**（3.1.3.3）：用在 NVMe over Fabrics，**只提供发现日志页面**（Discovery Log Page）。
- **CNTRLTYPE 字段**：Identify Controller 数据结构中的 Controller Type 字段标识控制器类型（规范 3.1.3）。
- **静态 vs 动态控制器模型**（规范 3.1）：
  - 静态模型保留：控制器 ID、Feature 设置，跨 Controller Level Reset（CLR）和跨关联都保留。
  - 动态模型：每次关联重新分配控制器，初始状态对所有动态控制器一致；同一 NVM 子系统端口访问的动态控制器初始挂载的命名空间集合相同。
  - 子系统内**所有控制器必须**支持同一模型。
  - 内存映射（PCIe）传输下**只能**是静态模型。
  - Discovery 控制器**只能**是动态模型。
  - Fabrics 下使用动态模型时，主机在 Fabrics Connect 命令里把 CNTLID 设为 `FFFFh`（任意可用）。
- **关联的建立**（规范 3.1.2）：通过 Fabrics Connect 命令建立，传入 Host NQN、NVM Subsystem NQN、Host Identifier、可指定 Controller ID。
- **关联的终止**（规范 3.1.3）：四种情况——控制器 Shutdown、CLR、Admin Queue 传输断开、或不支持单队列删除时任一 I/O Queue 断开。
- **没有"显式断开"命令**：Disconnect 命令只删除 I/O Queue；要让控制器重新空闲只能触发以上四种条件。
- **命令处理顺序**：除融合操作外，控制器对同一 SQ 或跨 SQ 的命令**无顺序保证**；主机不应把不能重排的命令放在同一 SQ。
- **多 SQ 优先级**：高优先级命令放在对应 SQ 中；仲裁按 3.4.4 节的方案（公平 + 优先级）。
- **完成顺序**：完成顺序也**无顺序保证**，主机用 CQE 中的 SQID + CID 与命令关联。
- **I/O 队列由主机创建**（规范 3.1.3.1）：通过 Create I/O Submission Queue 和 Create I/O Completion Queue 命令。
- **Figure 24** 典型多 I/O 控制器拓扑：3 个 I/O 控制器、共享命名空间 B、私有命名空间 A 和 C；Figure 25 展示 1 个 Administrative + 2 个 I/O 控制器的典型场景。

## 规范依据

- [控制器定义（1.5.20），PDF 第 29 页](../_source/pages/page-029.md)
- [控制器架构：静态 vs 动态模型，PDF 第 58 页](../_source/pages/page-058.md)
- [三种控制器类型与 Figure 23，PDF 第 59 页](../_source/pages/page-059.md)
- [I/O 控制器与多 I/O 命令集（3.1.3.1 / Figure 24），PDF 第 60 页](../_source/pages/page-060.md)
- [Administrative 控制器（3.1.3.2 / Figure 25），PDF 第 61 页](../_source/pages/page-061.md)

## 相关阅读

- [queue-pair.md](queue-pair.md) - 命令通过队列对提交与完成
- [association-and-command-lifecycle.md](association-and-command-lifecycle.md) - 主机与控制器的关联建立
- [nvm-subsystem.md](nvm-subsystem.md) - 控制器所属的子系统
- [command-sets.md](command-sets.md) - 控制器可启用的命令集
