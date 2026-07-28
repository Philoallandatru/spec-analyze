# 重放保护内存块（Replay Protected Memory Block, RPMB）

## 一句话说明

RPMB 是 NVMe 控制器中**不属于任何命名空间**的一块抗篡改存储区域：通过对每个目标配置独立密钥、用 HMAC 验证请求、Nonce 防重放、单调计数器防回滚，主机可以在不暴露密钥的前提下完成已认证的读写；目标 0 还附带一个 512 字节的设备配置块（DCB）控制 Boot Partition 保护与命名空间写保护。

## 生活化类比

把 RPMB 想成**银行金库的"凭证保险箱"**：

- **金库** = RPMB 目标（target），一个控制器可有多个
- **保险箱钥匙** = 一次性写入、不可读的认证密钥（Key）
- **存款单** = 请求包：账号（目标）+ 金额（数据）+ 时间戳（计数器）+ 防伪签名（HMAC）
- **取款回执** = 响应包：回填随机数（Nonce）+ 银行签章（MAC）+ 收据
- **银行验签** = 控制器用密钥重算 HMAC 并比对
- **保险箱外的"指示牌"** = DCB，控制谁能进 Boot 分区、谁可以写

> 关键：主机永远不持密钥，只持"凭证 + 签名"；控制器只认"凭证 + 签名"，不认人。

## 工作流程

```text
   [密钥预置: 安全环境]   ->   注入 Key 到目标 (一次性, 不可读)
                                       |
                                       v
   +-------------------- 已认证请求 --------------------+
   | 主机: 生成请求帧                                    |
   |   目标 | Nonce | Counter | Addr | Sector Count |    |
   |   加上 HMAC(目标..类型, Key)                        |
   |       |                                            |
   |       v                                            |
   |   Security Send (NSID=0, NSSF 匹配目标)             |
   |       |                                            |
   |       v                                            |
   |   控制器/RPMB: 校验 HMAC + 校验 Counter + 检查 Addr |
   |       |                                            |
   |       v                                            |
   |   Security Receive: 响应 (回填 Nonce / 新 Counter) |
   +-----------------------------------------------------+
```

**RPMB 帧布局（256 字节前缀）**：

| 字段 | 用途 |
|------|------|
| 填充 + MAC | HMAC-SHA256 输出 256 位 |
| 目标 ID | 指定哪个 RPMB target |
| Nonce | 防重放的随机数（读响应回填）|
| 写计数器 | 防回滚（成功写后 +1）|
| 地址 | 起始扇区 |
| 扇区数 | 操作的扇区数 |
| 结果 | 错误分类（认证失败 / 计数器错误 / 地址错误 等）|
| 类型 | 消息类型（0x0001 编程密钥 / 0x0002 读计数 / 0x0003 写 / 0x0004 读 / 0x0006 写 DCB / 0x0007 读 DCB）|

## 初学者案例

**场景：手机/嵌入式设备安全启动时校验 Boot 镜像完整性。**

1. 产线烧录时，安全预置环境为 RPMB 目标 0 编程一个 256 位 HMAC 密钥。
2. 同时通过 DCB 把"Boot Partition 保护"永久启用。
3. 设备每次启动，Bootloader 试图写 Boot Partition → 控制器检查 DCB → 因保护已启用而拒绝。
4. 需要更新 Boot 镜像时：
   - 发送 Security Send `0x0003` 携带：写计数器、目标地址、新 Boot 数据、HMAC。
   - 控制器校验 HMAC、计数器、地址。
   - 校验通过 → 写 Boot Partition，计数器 +1。
   - 控制器返回 Security Receive `0x0300` 携带新计数器和 MAC 签名。
5. 攻击者即使拿到总线数据，没有 Key 就伪造不了 HMAC；如果把旧帧重放，控制器会发现计数器不匹配而拒绝。

> 关键收获：RPMB = 认证 + 重放保护 + 回滚保护三件套，缺一不可。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 不属命名空间 | RPMB 独立于 Namespace；`NSID=0` |
| 多目标可共享 | 一个子系统可有多个 RPMB 目标，每个目标独立 Key |
| 密钥一次性 | Key 编程后不可读、不可擦除、不可修改 |
| HMAC-SHA256 | 默认认证方法，256 位 Key |
| 计数器单调 | 32 位写计数器，饱和值 `FFFFFFFFh`，不可重置 |
| DCB 独立计数 | DCB 写计数器不与数据区共享，永久启用项不可关闭 |
| NSID=0 + NSSF 匹配 | Security 命令必须 `NSID=0` 且 `NSSF` 与目标一致 |
| Security Send 成功 ≠ 操作成功 | 必须再发 `0x0005`（结果请求）+ `Security Receive` 校验签名结果 |
| MAC 验证是主机责任 | 控制器不会替主机验返回的 MAC |
| 跨目标可交错 | 对不同目标的请求可交错，但需保序的主机应等待完成 |
| 不可关闭永久保护 | DCB 一旦永久启用引导分区保护，无法再关闭 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| RPMB vs Namespace | RPMB 独立于 NS；NS 是用户数据卷 |
| 目标 0 vs 其他目标 | 目标 0 含 DCB；其他目标无 |
| Key vs Counter | Key 防伪造；Counter 防回滚 |
| Security Send vs Receive | Send 携带请求；Receive 取回响应/结果 |
| `0x0005` vs `0x0300` | `0x0005` 是结果请求；`0x0300` 是已签名结果 |
| DCB 保护 vs NS 写保护 | 都由 DCB 控制，但前者锁 Boot 分区，后者锁 NS 写入 |
| HMAC 输入范围 | 从"目标..类型"字段开始，不含填充与 MAC 字段 |
| 写计数 vs 读计数 | 读不消耗计数器；只有"成功写"才递增 |

## 进阶细节

- **目标持久内容**（规范 5.2.13.2.32）：
  - 认证 Key（HMAC-SHA256 为 256 位，一次性写入、不可读）
  - 数据区（128 KiB 到 32 MiB，由 Identify 报告）
- **威胁模型**（规范 8.1.1）：抵御"总线嗅探 + 重放 + 回滚"；不抵御"物理攻破控制器"。
- **帧格式**（规范 8.1.2）：固定 256 字节前缀 + 可变数据；MAC 计算覆盖"目标..类型"+"数据"。
- **结果错误分类**（规范 8.1.3）：认证失败、计数器错误、地址错误、写入失败、读取失败、Key 缺失、配置无效。
- **密钥编程三步**（规范 8.1.4）：
  1. Security Send `0x0001` 携带 Key
  2. Security Send `0x0005` 请求结果
  3. Security Receive `0x0100` 取编程结果
- **读计数器**（规范 8.1.5）：
  1. Security Send `0x0002`（携带 Nonce）
  2. Security Receive `0x0200`（回填 Nonce + Counter + MAC）
- **已认证写**（规范 8.1.6）：
  1. Security Send `0x0003`（Counter + Addr + Data + MAC）
  2. Security Send `0x0005`（结果请求）
  3. Security Receive `0x0300`（结果 + 新 Counter + MAC）
- **已认证读**（规范 8.1.7）：
  1. Security Send `0x0004`（Nonce + Addr + Sector Count）
  2. Security Receive `0x0400`（Nonce + Data + Result + MAC）
- **DCB 写**（规范 8.1.8）：仅目标 0，512 字节，独立计数器；永久启用项不可关闭。
- **DCB 读**（规范 8.1.9）：仅目标 0，固定 512 字节块。
- **计数器饱和**（规范 8.1.10）：达到 `FFFFFFFFh` 后禁止再写；状态位报告"过期"。
- **NSID=0 语义**（规范 8.1.11）：Security Send/Receive 对 RPMB 必须 `NSID=0`，与命名空间读写不同。
- **多目标匹配**（规范 8.1.12）：请求的 NSSF 必须与目标匹配；不匹配会返回协议级错误。

## 规范依据

- [RPMB 威胁模型与目标独立性，PDF 第 569 页](../_source/pages/page-569.md)
- [设备配置块与消息类型表，PDF 第 570 页](../_source/pages/page-570.md)
- [帧布局、结果分类与持久内容，PDF 第 572 页](../_source/pages/page-572.md)
- [HMAC-SHA256 与密钥编程流程，PDF 第 574 页](../_source/pages/page-574.md)
- [读计数器、已认证写与已认证读，PDF 第 575 页](../_source/pages/page-575.md)
- [DCB 独立计数器与读写流程，PDF 第 578 页](../_source/pages/page-578.md)

## 相关阅读

- [安全协议交换](security-protocol-exchange.md) - Send/Receive 承载 RPMB 协议
- [Key Per I/O](key-per-io.md) - 同属 NVMe 认证家族
- [启动分区](boot-partitions.md) - DCB 保护引导分区机制
- [命名空间写保护](namespace-write-protection.md) - DCB 控制命名空间写保护
- [通用控制器特性](common-controller-features.md) - RPMB 能力位声明位置
