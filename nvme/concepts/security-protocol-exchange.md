# 安全协议交换（Security Protocol Exchange）

## 一句话说明

NVMe 不自己定义安全协议细节，只用 **Security Send**（主机→控制器）和 **Security Receive**（控制器→主机）这对命令当"信封"，具体协议内容由 SPC-5 / ACS-4 等规范定义，主机通过协议发现机制查支持列表。

## 生活化类比

把安全协议交换想成**"密封信箱"**：

- **Security Send** = 你把"请求信"投进信箱（带协议类型 + 协议特定字段 + 信件正文）。
- **Security Receive** = 你打开信箱取"回信"。
- **协议 `00h`** = 信箱的"说明书"，直接告诉你"这个信箱支持哪几种信件格式"（不需要先投信才能读）。
- **协议 `EAh`（NVMe 专用）** = 信箱里专门为 NVMe 用户留的格子，其中 `SPSP=0001h` 用来访问 RPMB。
- **NSSF** = 当格子太大，再细分"几个具体仓位"（比如哪个 RPMB Target）。

## 工作流程

```text
  主机                                       控制器 / 安全协议模块
    |                                            |
    |  Security Send (主机→控制器)              |
    |  CDW10: SECP(31:24) | SPSP1(23:16)        |
    |         | SPSP0(15:08) | NSSF(7:0)        |
    |  CDW11: Transfer Length (TL)              |
    |  DPTR : 数据缓冲                          |
    |-----------------------------------------> |
    |                  [协议执行]               |
    |                                            |
    |  Security Receive (控制器→主机)           |
    |  CDW10: SECP | SPSP1 | SPSP0 | NSSF       |
    |  CDW11: Allocation Length (AL)            |
    |  DPTR : 接收缓冲                          |
    | <---------------------------------------- |
    |                                            |
    ※ Send/Receive 配对关系由协议决定（SPC-5/ACS-4）
    ※ SECP=00h 的 Receive 是"协议发现"，不需要先 Send
```

## 初学者案例

**场景：RPMB 写不进去，怀疑是 Target 选错了。**

1. 你要做 RPMB 编程操作，先用 `nvme security-receive -p 0xEA -s 0x0001 -l 512` 读 RPMB 配置。
2. 命令中：`SECP=EAh`（NVMe 专用协议），`SPSP=0001h`（RPMB 子协议），`NSSF=<target_idx>`（第几个 RPMB 目标）。
3. 控制器回 `Invalid Field in Command`。
4. 排错：用 `nvme id-ctrl` 看 `RPMB` 字段（`RNUM`=`RPMB Unit Number`）；若目标索引 ≥ RNUM 就是越界。
5. 还要确认 `Identify` 的 RPMB 能力位声明了非零 RPMB 目标数，否则控制器可能根本不支持 RPMB（不实现 Security Send/Receive 时连命令都不收）。
6. 改 `NSSF=0`（默认第一个 RPMB）重试。

> 排错提示：`NSSF` 字段仅在 `SECP=EAh` 时有定义；其他协议下它是保留位，写了非零值就报错。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 命令对 | Security Send（Out）+ Security Receive（In） |
| SECP 范围 | `00h`-`FFh`，由 SPC-5 / ACS-4 等外部规范定义 |
| 协议 `00h` | 专用：协议发现；只需 Security Receive，不与任何 Send 关联 |
| 不支持的 SECP | 命令中止，状态 `Invalid Field in Command` |
| CDW10 位域 | `31:24` SECP；`23:16` SPSP1（bit 15:08 of SPSP）；`15:08` SPSP0（bit 07:00 of SPSP）；`07:00` NSSF |
| NSSF 含义 | 仅当 `SECP=EAh` 时有定义；其他协议此字段保留 |
| 协议 `EAh` 用途 | 分配给 NVMe 接口使用（参见 ACS-4） |
| SPSP=0001h (EAh) | 选 RPMB；`NSSF`=RPMB Target |
| SPSP=0002h..FFFFh (EAh) | 保留 |
| CDW11 含义 | Send=Transfer Length (TL)；Receive=Allocation Length (AL) |
| 长度语义 | 与 SPC-5 的 `INC_512=0h` 情形相同 |
| DPTR | 数据缓冲起始地址，按 Common Command Format Figure 92 解释 |
| 其他命令字段 | 除 DPTR/CDW10/CDW11 外，命令特定字段保留 |
| 配对关系 | 由协议规范定义；NVMe Base 不规定状态保留时长 |
| 数据易失性 | Receive 取得的数据可能在通信丢失或控制器级重置后丢失 |
| RPMB 强制支持 | 控制器声明非零 RPMB Target 数时，必须实现 Send/Receive 命令 |
| RPMB 能力 | 在 Identify Controller 数据结构中声明目标数与共享能力 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Security Send vs Security Receive | Send 是主机→控制器；Receive 是控制器→主机 |
| SECP vs SPSP | SECP=协议族；SPSP=协议内子功能 |
| SPSP1 vs SPSP0 | SPSP1 = SPSP 字段的 bit 15:08；SPSP0 = bit 07:00 |
| NSSF 含义 | 仅在 `SECP=EAh` 下作为子选择；其他协议保留 |
| 协议 `00h` vs 其他协议 | `00h` 是发现入口；其他协议是实际工作 |
| Transfer Length vs Allocation Length | TL=要发送的字节数；AL=接收缓冲预留大小 |
| SECP=EAh vs SCSI SPC 协议 | EAh 由 ACS-4 定义给 NVMe；其余多由 SPC-5 定义 |
| Security 命令 vs Format NVM | Security 走信令通道；Format NVM 是 NVMe 内置命令 |

## 进阶细节

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

## 规范依据

- [Security Receive 命令与保留边界，PDF 第 390 页](../_source/pages/page-390.md)
- [Security Receive 字段 Figures 375-377，PDF 第 391 页](../_source/pages/page-391.md)
- [协议 00h 发现 + 协议 EAh + RPMB 定义，PDF 第 391 页](../_source/pages/page-391.md)
- [Security Send 命令 Figures 379-381，PDF 第 392 页](../_source/pages/page-392.md)
- [Identify Controller 中 RPMB 能力声明，PDF 第 333-334 页](../_source/pages/page-333.md)

## 相关阅读

- [重放保护内存块](replay-protected-memory-block.md) - SPSP=0001h 访问 RPMB
- [通用命令格式](common-command-format.md) - Security 命令的 SQE 布局
- [数据指针布局](data-pointer-layouts.md) - DPTR 指向协议数据缓冲
- [Key Per I/O](key-per-io.md) - 同属 NVMe 安全协议栈
