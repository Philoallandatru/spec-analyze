# NVMe-MI 架构、带内/带外通道与消息类型

**NVMe-MI（NVMe Management Interface）** 定义了一套独立于 I/O 队列之外的管理命令集，允许管理控制器（通常是服务器主板上的 **BMC**）发现、监控和配置整个 NVM 子系统 `[NVMe-MI §1–2]`。

---

## 带外（Out-of-Band）vs 带内（In-Band）双通道

NVMe-MI 并非只能走 BMC 线缆，主机驱动也可以通过 Admin 命令直接转发 NVMe-MI 消息：

```text
  [ BMC 管理控制器 ] ──(SMBus/I3C 或 PCIe VDM over MCTP)──▶ [ SSD 管理端点 (MI Endpoint) ]
                                                                     ▲
  [ 主机 OS / nvme-cli ] ──(Admin 1Dh Send / 1Eh Receive)────────────┘
```

- **带外通道（Out-of-Band）**：走 MCTP over SMBus/I3C 或 PCIe VDM。**完全绕过主机 CPU、OS 与 PCIe BAR0**，即使系统未装驱动、OS 蓝屏死机甚至主机尚未开机（只要 `3.3Vaux` 供电正常），BMC 仍可采集盘温与健康状态。
- **带内隧道（In-Band Tunneling）**：主机通过 Admin 命令 **`NVMe-MI Send`（Opcode `1Dh`）** 与 **`NVMe-MI Receive`（Opcode `1Eh`）**，把 NVMe-MI 消息直接发给控制器执行（常用于在没有 BMC 的环境下调试 MI 功能或读写 VPD） `[Base §5.1.18–19]`。

---

## NVMe-MI 消息头与 `NMIMT` 五大消息族

紧跟在 MCTP `Message Type = 04h` 之后的是 **4 字节 NVMe-MI Message Header（`NMP`）**，其中 **`NMIMT`（NVMe-MI Message Type，Bits `[6:3]`）** 决定了这条消息的业务类型 `[NVMe-MI §3.1]`：

| `NMIMT` 值 | 消息类型 | 典型用途 |
| :--- | :--- | :--- |
| **`0h`** | **Control Primitive** | 轻量级链路/端点控制原语（如 `Pause`、`Resume`、`Abort`、`Get State`、`Replay`），不走常规命令状态机 |
| **`1h`** | **NVMe-MI Command** | 核心带外管理命令：健康状态轮询、配置读写、VPD 读写、控制器列表查询 |
| **`2h`** | **NVMe Admin Command** | **Admin 隧道命令**：在带外直接封装一条标准 NVMe Admin 命令（如 `Identify`、`Get Log Page`、`Format`） |
| **`4h`** | **PCIe Command** | 读取/配置端点上的 PCIe 配置空间或性能计数器 |
| **`5h`** | **Asynchronous Event Notification** | SSD 主动向 BMC 推送严重告警（如温度越限、备用块耗尽） |

> **安全边界提示**：通过带外 `NMIMT = 2h`（Admin 隧道）执行破坏性操作（如 `Format NVM`、`Sanitize`）时，现代企业级 SSD 常默认受限于 **Command and Feature Lockdown** 策略，避免 BMC 被攻破后直接擦除业务数据。

---

## VPD（Vital Product Data）与 FRU 读取

服务器资产盘点与背板点亮（Locate LED）依赖 `VPD Read` / `VPD Write` 命令，用于读取 SSD 出厂序列号、型号、功率档位与自定义 FRU（Field Replaceable Unit）信息，即使盘尚未被 OS 枚举也能完成资产登记。

---

## 关联与引申概念

- **命令与特性锁定**：主机如何限制带外 Admin 隧道的权限，详见 [命令与特性锁定](../nvme/concepts/command-and-feature-lockdown.md)。
- **资源与管理清单**：管理 URI 与资产日志，详见 [资源与管理清单日志](../nvme/concepts/resource-and-management-inventory-logs.md)。
- **MCTP 封装格式**：`NMIMT` 外层的 MCTP 包头与分片规则，详见 [MCTP 传输层与 PCIe VDM / SMBus 封装](mctp-transport-and-vdm.md)。
