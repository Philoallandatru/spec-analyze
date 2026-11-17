# NVMe-oF 自动化发现（CDC / mDNS）与端到端安全（TLS / CHAP）

在拥有上千台计算节点与数百台存储阵列的云数据中心里，靠人工逐台敲 `nvme discover` / `nvme connect` 根本无法运维。NVMe-oF 通过 **TP8009（mDNS/DNS-SD）+ TP8010（CDC 集中发现控制器）** 实现“插上网线即自动发现挂载”，并通过 **TLS 1.3 PSK + DH-HMAC-CHAP** 构筑零信任安全防线 `[NVMe TP8009 / TP8010 / Base §8]`。

---

## 自动化发现架构：mDNS / DNS-SD + CDC

```text
  计算节点 (nvme-stas 守护进程)
        │
        ├── 1. 组播查询 _nvme-disc._tcp (mDNS / DNS-SD, TP8009)
        ▼
  [ 中央发现控制器 (CDC, TP8010) ] ◀── 存储阵列主动 Push 注册 (DIM 命令)
        │
        └── 2. 返回该主机有权限访问的所有 Target NQN、IP/端口与 ANA 路径清单
```

- **CDC（Centralized Discovery Controller）**：充当整个存储网络的“总通信录”，存储阵列上线后主动向 CDC 注册自己，主机只需连上 CDC 即可拉取全网最新目标与多路径变化（通过 AEN 异步事件实时推送）。
- **`nvme-stas`（Storage Appliance Services）**：Linux 生态标准后台服务（`stafd` + `stacd`），自动监听 mDNS 广播并维护 CDC 连接与盘符自动重连。

---

## 传输加密与双向认证：TLS 1.3 PSK + DH-HMAC-CHAP

| 安全层 | 协议机制 | 核心作用 |
| :--- | :--- | :--- |
| **传输通道加密** | **NVMe/TCP over TLS 1.3（PSK）** | 对 TCP 流上的所有 PDU（命令与数据）进行加密防窃听；Linux 内核通过 **`kTLS` + 用户态握手守护进程 `tlshd`** 实现接近零拷贝的线速加密 |
| **带内身份认证** | **`DH-HMAC-CHAP`**（新一代升级为 `KX-HMAC-CHAP`） | 在 NVMe `Connect` 之后立即执行挑战-应答握手：**结合 Diffie-Hellman 前向保密密钥协商与双向身份验证**，防止未授权主机或伪造存储阵列接入 |

```bash
# 使用 nvme-cli 通过 DH-HMAC-CHAP 密钥与 TLS 加密连接远端 NVMe/TCP 子系统
nvme connect -t tcp -n nqn.2026-01.org.example:subsys1 -a 192.168.10.50 -s 4420 \
  --dhchap-secret="DHHC-1:00:..." --tls
```

---

## 关联与引申概念

- **Discover 基础协议**：Discovery Controller 日志页与快照一致性，详见 [Fabrics 发现与认证](../nvme/concepts/fabrics-discovery-and-authentication.md)。
- **NVMe/TCP 帧格式**：TLS 加密包裹之下的 PDU 结构，详见 [NVMe/TCP 传输规范](nvme-tcp-transport.md)。
- **无盘启动认证**：`NBFT` 表中如何携带 TLS PSK 与 DH-HMAC-CHAP 凭据，详见 [NVMe Boot 规范与 ACPI NBFT 表](../nvme-extended-specs/nvme-boot-and-nbft.md)。
