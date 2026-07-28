# Key Per I/O（基于密钥标签的 I/O 加密）

## 一句话说明

Key Per I/O 是 NVMe 2.1 引入的一项命名空间级加密能力：允许主机在**每条 I/O 命令**上指定一个 16 位的 Key Tag，由控制器用与该 Key Tag 绑定的密钥来完成这次读写的加解密；密钥的注入与生命周期管理完全在 NVMe 规范之外。

## 生活化类比

把它想成**保险箱的"凭标签取件"**：

- **保险箱** = 控制器内的加密引擎
- **保险箱里的一个个格子** = 各个 Key Tag 对应的密钥
- **送件单上的标签号** = I/O 命令里的 `KEYTAG` 字段
- **格子钥匙由外部金库保管** = 密钥注入、轮换、撤销由 NVMe 规范之外的安全系统负责
- **送件员（主机）从不接触钥匙** = 主机只管传标签，密钥始终不出控制器

> 关键：Key Per I/O 不是"主机管密钥"，而是"主机管标签，密钥被控制器与外部密钥管理系统共同托管"。

## 工作流程

```text
   阶段 1: 密钥准备（NVMe 规范外）                阶段 2: I/O 命令（NVMe 规范内）
   +-------------------+                           +--------------------------+
   | KMS / 安全代理    |                           | 主机                     |
   +---------+---------+                           +------------+-------------+
             |                                                  |
             v                                                  v
   注入密钥到控制器                            命令: Read/Write
             |                                                  |
             v                                                  v
   绑定: KEYTAG <-> Key                       参数: CETYPE=1h (KPIOTAG)
             |                                                  |
             v                                                  v
   [Key Tag 表格就绪]                          控制器按 KEYTAG 查表 + 加解密
```

**作用域模式**：

| 模式 | 启用范围 | 适用场景 |
|------|---------|----------|
| 子系统级 | 整个 NVM Subsystem 所有 NS | 数据中心统一加密策略 |
| 命名空间级 | 每个 NS 独立启用 | 多租户、租户级独立密钥 |

## 初学者案例

**场景：云存储平台，100 个租户共用一台 NVMe SSD，如何隔离加密？**

1. KMS（外部密钥管理服务）给每个租户生成独立密钥。
2. KMS 把密钥注入控制器，并把"租户 7"绑定到 Key Tag `0x0007`，"租户 8"绑定到 `0x0008`，依此类推。
3. 主机在为租户 7 写数据时，提交 `Write` 命令并设 `CETYPE=1h`、`CEV=0x0007`。
4. 控制器用租户 7 的密钥加密这块数据。
5. 读取时同样指定 `KEYTAG=0x0007`，控制器用同一密钥解密。
6. 租户 8 的密钥泄露时，只需撤销 `0x0008` 的 Key Tag，无需整机擦除。

> 关键收获：Key Per I/O 把"密钥"和"主机"解耦——主机不存密钥，租户切换或密钥轮换不需要重启主机或重新格式化磁盘。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 控制器能力前提 | 控制器 `KPIOS=1` 才支持 Key Per I/O |
| 命令参数 | `CETYPE=1h` 表示 `CEV` 字段是 Key Tag |
| Key Tag 范围 | Key Tag 必须在该 NS 的 Maximum Key Tag 范围内 |
| CETYPE 取值 | `0h` 保留/无加密；`1h` KPIOTAG；`2h-Eh` 保留；`Fh` 厂商自定义 |
| 错误码 | 保留值或越界 → `Invalid Field in Command` |
| 不影响 RPMB / Boot | RPMB 和 Boot Partition 走安全协议/专用命令，不使用 NS I/O 命令 |
| 标签数值不跨 NS | 不同 NS 同一 Key Tag 不一定指向同一密钥 |
| 密钥管理不在规范内 | 注入、激活、绑定关系由规范之外的安全标准定义 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Key Per I/O vs Key Per Namespace | 前者每 I/O 命令可选密钥；后者整个 NS 固定一个密钥 |
| Key Tag vs Key | Tag 是 16 位数字 ID；Key 是真正的位序列；规范里主机只见 Tag |
| KPIOS vs KPIOS Scope | `KPIOS` 是能力位；`Key Per I/O Scope` 决定启用粒度（子系统/NS） |
| CETYPE=0h vs 未启用 | 启用 Key Per I/O 后 `0h` 是"非法"（保留），未启用时 `0h` 才是"无加密" |
| 厂商定义 vs 保留 | `CETYPE=Fh` 是厂商定义；`2h-Eh` 是规范保留，未来扩展 |

## 进阶细节

- **能力字段**（规范 5.2.13.2.19）：控制器 Identify Controller 数据结构中的 `KPIOS` 位；非零即支持 Key Per I/O。
- **命名空间范围**（规范 5.2.13.2.20）：每个 NS 的 Identify Namespace 数据结构可报告独立的 `KPIOS` 与 Maximum Key Tag。
- **命令参数**（规范 6.3.1）：Read/Write 命令的 Dword 12 中：
  - `CETYPE`（bit 31:28）：加密类型
  - `CEV`（bit 15:0 或 bit 27:16，取决于命令）：加密值
  - 当 `CETYPE=1h` 时，CEV 为 16 位 Key Tag
- **错误处理**（规范 6.3.2）：
  - `CETYPE` 为保留值 → `Invalid Field in Command`
  - Key Tag 超过 NS 的 Maximum Key Tag → `Invalid Field in Command`
  - `CETYPE=Fh` → 厂商行为，可能合法也可能失败
- **RPMB 例外**（规范 6.3.3）：RPMB 操作使用 Security Send/Receive，绕开标准 I/O 命令，因此 Key Per I/O 不会影响 RPMB。
- **Boot Partition 例外**（规范 6.3.4）：Boot Partition 读写通过专用接口，亦不参与 Key Per I/O。
- **作用域配置**（规范 5.2.26）：`Key Per I/O Scope`（bit X）选择子系统级或 NS 级。
- **密钥生命周期**（规范附录）：规范未规定密钥如何注入、轮换、撤销；通常由 SPDK、Keylime、KMIP 等外部组件管理。
- **与命名空间写保护的兼容性**：Key Per I/O 不与 Namespace Write Protection 互斥，但加解密后的密文与写保护独立。

## 规范依据

- [Key Per I/O 能力与作用范围，PDF 第 548 页](../_source/pages/page-548.md)
- [CETYPE 字段定义与错误处理，PDF 第 549 页](../_source/pages/page-549.md)

## 相关阅读

- [数据指针布局](data-pointer-layouts.md) - Keyed Data Block 是 SGL 类型 4h
- [重放保护内存块](replay-protected-memory-block.md) - 同属认证防重放家族
- [安全协议交换](security-protocol-exchange.md) - 也走安全命令路径
- [命名空间写保护](namespace-write-protection.md) - 写保护兼容性参考
- [通用命令格式](common-command-format.md) - CETYPE 字段在 CDW 中的位置
