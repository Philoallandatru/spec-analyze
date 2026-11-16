# Key Per I/O（基于密钥标签的 I/O 加密）

**Key Per I/O** 是 NVMe 2.1 引入的精细化数据加密机制：允许主机在**每一条读写 I/O 命令**中携带一个 16 位的 **密钥标签（Key Tag）**，由控制器用该标签绑定的密钥完成本次传输数据的加解密。密钥本身的注入、轮换与撤销完全由 NVMe 规范之外的外部安全系统（如 KMS/KMIP）管理，主机在 I/O 通路上从不直接接触密钥明文。[PDF pp. 548-549](../_source/pages/page-548.md)

可以把 Key Per I/O 想象成**银行保险箱的"凭标签取件"**：控制器内部是带加密引擎的保险箱，里面每个格子（Key Tag）锁着一把专属密钥；外部金库管理员（KMS）提前把钥匙放进格子并告知主机编号；主机每次发读写快递单时只需写上格子编号（`KEYTAG`），就能实现不同租户、不同敏感度数据的隔离加密与快速密钥轮换。

## 双层解耦架构与工作流程

```text
  阶段 1：密钥准备（NVMe 规范外）            阶段 2：I/O 加密选择（NVMe 命令内）
  +---------------------------+              +--------------------------------+
  | 外部 KMS / 安全代理        |              | 主机提交 Read / Write 命令      |
  +-------------+-------------+              +---------------+----------------+
                |                                            |
                v                                            v
     注入密钥并绑定 KEYTAG                      设置 CETYPE=1h (KPIOTAG), CEV=KEYTAG
                |                                            |
                v                                            v
     [控制器 Key Tag 映射表就绪]              控制器查表取密钥 -> 加解密本次 I/O 数据
```

| 架构层面 | 职责边界 | 定义方 |
|---|---|---|
| **密钥管理层** | 密钥生成、注入、激活、轮换、撤销及与 `KEYTAG` 的绑定 | NVMe 规范之外的安全标准（如 TCG、KMIP） |
| **命令选择层** | 在 I/O 命令的 Dword 12 中通过 `CETYPE` 和 `CEV` 指定密钥标签 | NVMe 2.1 规范 |

## 启用条件、作用域与 `CETYPE` 规则

要使用 Key Per I/O，Identify Controller 或 Identify Namespace 中的 **`KPIOS`（Key Per I/O Supported / Enabled）** 必须使能，且命令参数需符合以下规则：[PDF pp. 548-549](../_source/pages/page-548.md)

| `CETYPE` 值（CDW12） | 含义 | Key Per I/O 启用时的行为 |
|---|---|---|
| `0h` | 无加密 / 保留 | **启用后禁用**：作为保留值，使用会返回 `Invalid Field in Command`（仅在未启用时表示无加密） |
| `1h` | **`KPIOTAG`** | `CEV` 字段携带 16 位命名空间专属的 `KEYTAG` |
| `2h–Eh` | 保留 | 返回 `Invalid Field in Command` |
| `Fh` | 厂商特定 | 行为由控制器厂商自定义 |

- **Key Tag 越界检查**：使用的 `KEYTAG` 必须在该命名空间的 **Maximum Key Tag（`MAXKT`）** 范围内，超出则返回 `Invalid Field in Command`。[PDF p. 549](../_source/pages/page-549.md)
- **作用域（Key Per I/O Scope）**：`KPIOC` 区分子系统级启用（整个 NVM 子系统共享启用策略）与命名空间级独立启用。注意**不同命名空间中相同数值的 `KEYTAG` 不一定指向同一个密钥**。[PDF pp. 339, 548-549](../_source/pages/page-548.md)
- **RPMB 与 Boot Partition 例外**：RPMB（重放保护内存块）使用 Security Send/Receive 命令，Boot Partition 使用专用读写接口，均不使用标准命名空间 I/O 命令，因此不受 Key Per I/O 影响。[PDF p. 549](../_source/pages/page-549.md)

## 典型应用场景

- **多租户隔离加密**：云存储中为每个租户分配独立 `KEYTAG`，租户密钥撤销时无需整盘擦除。
- **数据分级与在线轮换**：同一命名空间内按文件敏感度使用不同密钥，或在不停止业务的前提下逐步将新写入切换到新 `KEYTAG`。

## 规范依据

- [Key Per I/O 能力、标签选择与作用域，PDF p. 548](../_source/pages/page-548.md)
- [CETYPE 字段定义表、支持条件与错误处理，PDF p. 549](../_source/pages/page-549.md)
- [Identify Controller KPIOC 与 Identify Namespace KPIOS/MAXKT，PDF pp. 339, 359-360](../_source/pages/page-339.md)

## 关联与引申概念

- **RPMB（Replay Protected Memory Block）**：通过认证防重放计数器保护的受控存储区域，独立于 Key Per I/O 机制。详见 [重放保护内存块](replay-protected-memory-block.md)。
- **安全协议交换（Security Send/Receive）**：用于在主机与控制器之间传递 TCG/外部安全协议报文（如密钥配置通道）。详见 [安全协议交换](security-protocol-exchange.md)。
- **数据指针布局（Keyed SGL）**：SGL 类型 `4h`（Keyed Data Block）用于传输层/内存密钥校验，与命令级数据加密的 Key Per I/O 属于不同层次。详见 [数据指针布局](data-pointer-layouts.md)。
- **命名空间写保护**：Key Per I/O 与写保护正交，密文写入同样受写保护状态约束。详见 [命名空间写保护](namespace-write-protection.md)。
