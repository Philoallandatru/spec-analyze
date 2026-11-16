# 安全协议交换（Security Protocol Exchange）

NVMe 本身不重新发明安全协议，而是通过 **`Security Send`**（主机→控制器）和 **`Security Receive`**（控制器→主机）这一对通用管理命令作为"标准信封"，把具体的安全协议载荷（如 **TCG Opal** 全盘加密或 **RPMB** 认证存储）透传给控制器处理。[PDF pp. 390-392](../_source/pages/page-390.md)

可以把安全协议交换想成**带编号的密封信箱**：
- **Security Send**：把请求信投进信箱（指明协议编号与子功能）
- **Security Receive**：从信箱取回回信（状态与返回数据）
- **协议 `00h`**：信箱的"说明书"，只需用 `Security Receive` 读取即可发现控制器支持哪些安全协议，无需先发 `Send`
- **协议 `EAh`**：专门留给 NVMe 接口的信箱格，其中 `SPSP=0001h` 对应 RPMB，`NSSF` 指定具体 RPMB 目标

---

## 命令交互模型与字段结构

```text
  主机                                      控制器 / 安全协议模块
   |  Security Send (主机 → 控制器)                |
   |  CDW10: SECP[31:24] | SPSP[23:08] | NSSF[7:0] |
   |  CDW11: Transfer Length (TL)                  |
   |---------------------------------------------->|
   |                                        [协议执行]
   |  Security Receive (控制器 → 主机)             |
   |  CDW10: SECP | SPSP | NSSF                    |
   |  CDW11: Allocation Length (AL)                |
   |<----------------------------------------------|
```

| 命令字段 | 位域 / 位置 | 说明与规则 |
|----------|-------------|------------|
| **`SECP`**（Security Protocol） | `CDW10[31:24]` | 选择安全协议类型（`00h`–`FFh`），由 SPC-5 / ACS-4 等外部规范定义。不支持或保留的值返回 `Invalid Field in Command` |
| **`SPSP`**（Protocol Specific） | `CDW10[23:08]` | 16 位协议内部子功能选择器（高字节 `SPSP1` 在 `23:16`，低字节 `SPSP0` 在 `15:08`） |
| **`NSSF`**（NVMe Specific） | `CDW10[07:00]` | **仅在 `SECP=EAh` 时有定义**（指定 RPMB Target）；在其他协议下为保留字段（写非零会报错） |
| **长度字段（`TL` / `AL`）** | `CDW11` | `Send` 为传输长度（TL），`Receive` 为接收缓冲区分配长度（AL），语义与 SPC-5 `INC_512=0h` 一致 |
| **`DPTR`** | SQE 数据指针 | 指向命令或响应的数据缓冲区（其余命令特定字段保留） |

**关键机制约束**：
- **协议主导配对**：Send 与 Receive 的配对关系、状态保留时长完全由所选安全协议定义，NVMe 仅提供传输通道。
- **数据易失性**：等待 `Security Receive` 取回的数据在发生通信中断或控制器级别重置（Controller Level Reset）后可能丢失。[PDF pp. 390-392](../_source/pages/page-390.md)

---

- **CDW10 位定义**（规范 Figures 376/380）：
  - `31:24` SECP（按 SPC-5 定义）
  - `23:16` SPSP1（SPSP bit 15:08）
  - `15:08` SPSP0（SPSP bit 07:00）
  - `07:00` NSSF（仅 SECP=EAh 有定义；其余协议保留）
- **CDW11**：
  - Receive：`AL` Allocation Length
  - Send：`TL` Transfer Length
  - 均与 SPC-5 的 `INC_512=0h` 情形一致
- **协议 `00h` 行为**（规范 5.1.23.2）：返回控制器支持的协议列表；用作发现过程，不需要对应 Send。
- **协议 `EAh` 子协议**（Figure 378）：
  - `SPSP=0001h` → Replay Protected Memory Block；`NSSF` = RPMB Target
  - `SPSP=0002h..FFFFh` → 保留
- **命令字段使用**：仅 DPTR + CDW10 + CDW11，其他命令特定字段保留。
- **错误码**：
  - `Invalid Field in Command`（保留/不支持 SECP、SPSP 非法、NSSF 在非 EAh 协议下非零等）
- **强制支持触发条件**：控制器在 Identify Controller 中报告非零 RPMB Target 数时，必须实现 Security Send 和 Security Receive。
- **协议规范引用**：
  - SPC-5（SCSI Primary Commands - 5）：定义大多数协议的请求/响应格式
  - ACS-4（ATA Command Set - 4）：定义 `EAh` 协议的 NVMe 用途
  - NVMe Base 仅作"信封"，不重定义协议细节
- **典型使用场景**：
  1. 启动时 `SECP=00h` Receive 做协议发现
  2. RPMB 访问 `SECP=EAh` + `SPSP=0001h`
  3. TCG Opal / 其它加密协议使用其他 SECP 值

1. **安全协议发现（`SECP = 00h`）**：主机通过发送 `SECP=00h` 的 `Security Receive` 命令获取控制器支持的安全协议列表。这是一次独立的查询，**不需要先发送 `Security Send`**。[PDF p. 391](../_source/pages/page-391.md)
2. **NVMe 专用协议（`SECP = EAh`）**：由 ACS-4 分配给 NVMe 接口使用：
   - `SPSP = 0001h`：访问 **RPMB（重放保护内存块）**，此时 `NSSF` 选择目标 RPMB Target（索引从 0 开始，必须小于 `Identify Controller` 报告的 RPMB 目标数）。
   - `SPSP = 0002h–FFFFh`：保留。
3. **强制支持要求**：只要控制器在 `Identify Controller` 中声明了非零的 RPMB 目标数量，就**必须实现** `Security Send` 和 `Security Receive` 命令。[PDF pp. 333-334](../_source/pages/page-333.md)

**排错提示**：如果访问 RPMB 时返回 `Invalid Field in Command`，应检查 `nvme id-ctrl` 中的 `RPMB` 字段确认目标索引 `NSSF` 未越界，并确认非 `EAh` 协议（如 TCG Opal）下 `NSSF` 已清零。

---

## 关联与引申概念

- **TCG Opal（Trusted Computing Group Opal）**：业界主流的硬件自加密驱动器（SED）规范，在 NVMe 上正是通过 `Security Send/Receive` 携带特定 `SECP` 协议号来配置加密锁和密钥的。
- **SPC-5 / ACS-4**：分别来自 SCSI 和 ATA 的标准规范，NVMe 直接引用它们定义的安全协议号与载荷格式，避免重复造轮子。
- **相关文档**：
  - [重放保护内存块](replay-protected-memory-block.md) — `SECP=EAh, SPSP=0001h` 承载的 RPMB 认证存储协议
  - [识别命令模型](identify-command-model.md) — 声明 RPMB 目标数与支持能力
  - [通用命令格式](common-command-format.md) — Security 命令的 64 字节 SQE 布局
  - [数据指针布局](data-pointer-layouts.md) — `DPTR` 指向协议数据缓冲区的规则
  - [Key Per I/O](key-per-io.md) — 同属 NVMe 数据安全加密体系
  - [自加密盘（SED）与 TCG Opal 2.0](../../security-and-compliance/tcg-opal-and-sed.md) — Admin SP / Locking SP 与 PSID Revert 详解

## 规范依据

- [Security Receive 用途、协议发现（00h）与 NVMe 协议（EAh），PDF pp. 390-391](../_source/pages/page-390.md)
- [Security Send 字段、配对关系与响应规则，PDF p. 392](../_source/pages/page-392.md)
- [Identify Controller 中 RPMB 能力声明，PDF pp. 333-334](../_source/pages/page-333.md)
