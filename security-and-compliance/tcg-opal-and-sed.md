# 自加密盘（SED）与 TCG Opal 2.0

**SED（Self-Encrypting Drive，自加密盘）** 在 SSD 控制器内部集成硬件 AES-256 引擎，所有写入 NAND 的数据在落盘前自动加密，**主机 CPU 零开销**。在未解锁状态下，即使把 SSD 拆下来插到另一台机器上，读取到的也只是密文（或被拒绝访问）。企业级与消费级 NVMe SSD 最主流的 SED 标准是 **TCG Opal 2.0** `[TCG Opal 2.0 §3]`。

---

## NVMe 承载隧道：`Security Send (81h)` / `Security Receive (82h)`

NVMe 协议本身不定义加密钥匙怎么管理，而是通过两条标准 Admin 命令作为“信封”，把 TCG Opal 的 **IF-MIB（Interface Messaging）** 载荷透传给 SSD 内部的安全处理器 `[Base §5.1.24–25]`：
- **`Security Send`（Opcode `81h`）**：主机下发 TCG 会话建立、密码认证、锁区（Locking Range）配置指令。
- **`Security Receive`（Opcode `82h`）**：主机取回盘内返回的状态、Discovery 列表或认证结果。
- **`SECP`（Security Protocol ID）**：用于标识协议族，其中 `00h` 为支持列表发现，**`01h` / `02h`（TCG 定义）** 用于传输 Opal 指令。

---

## `Admin SP` vs `Locking SP` 与 `PSID Revert`

TCG Opal 将盘内的安全管理划分为两个独立的安全域（Security Provider, SP）：

| 实体 | 职责 | 典型操作 |
| :--- | :--- | :--- |
| **`Admin SP`** | 管理全局设备所有权与生命周期 | 初始化 `SID`（Security ID）密码、激活 `Locking SP`、执行全局重置 |
| **`Locking SP`** | 管理用户数据区域访问权限与密钥（DEK） | 划分 **Locking Ranges**（锁定区间）、配置读/写锁定状态、配置上电预启动认证（MBR Shadow） |
| **`PSID Revert`** | **物理安全 ID 恢复**（出厂标签上的 32 位字符） | 忘记密码时，凭盘面印刷的 `PSID` 发起强行重置——**直接销毁内部 DEK，瞬间密码学擦除全部数据** |

> **排错与运维红线**：
> 1. **休眠（S3/S0ix）唤醒后盘变只读/报错**：Opal 盘断电即自动重新上锁，系统休眠唤醒时若 BIOS/OS 未缓存凭据并自动重新发送 `Unlock` 指令，会导致读写全部失败。
> 2. **区分 `Sanitize Crypto Erase` 与 `PSID Revert`**：两者都能秒级销毁 DEK，但 `PSID Revert` 还会把整个 TCG Opal 状态机恢复到未激活的出厂态。

```bash
# 使用 nvme-cli 发送 Security Receive (SECP=0x00) 探测 SSD 支持的安全协议列表
nvme security-recv /dev/nvme0 -p 0x00 -s 0x0000 -t 256 -x 256
# 生产环境常配合 sedutil-cli 管理 Opal 2.0 锁定与 PSID Revert：
# sedutil-cli --scan
```

---

## 关联与引申概念

- **安全协议隧道**：`Security Send/Receive` 在 NVMe Base 规范中的信封格式，详见 [安全协议交换](../nvme/concepts/security-protocol-exchange.md)。
- **每 I/O 密钥**：与整盘锁定的 Opal 不同，支持按 I/O 粒度和租户切换密钥，详见 [每 I/O 密钥](../nvme/concepts/key-per-io.md)。
- **密码学擦除**：除 PSID 外通过 Admin 命令发起的擦除生命周期，详见 [清理操作生命周期](../nvme/concepts/sanitize-operation-lifecycle.md)。
