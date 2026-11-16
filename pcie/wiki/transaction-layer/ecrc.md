# ECRC（端到端 CRC）详解

**ECRC（End-to-End CRC）** 是 PCIe 事务层提供的可选 **32 位端到端数据完整性校验码**。它由最初的发送方（Requester/Completer）事务层生成并附在 TLP 末尾，穿越所有中间 Switch 后，由最终接收方的事务层进行校验，专门用来捕捉中间交换节点内部缓存或路由逻辑可能引入的数据损坏。

> **直观类比**：如果说数据链路层的 **LCRC** 是每段快递车队交接时贴在车厢外的"分段铅封"，那么 **ECRC** 就是发货方贴在最内层货箱上的"原厂防伪封条"——哪怕货箱在某个中转站（Switch）里换车时被内部磕碰损坏，最终收货方拆箱时也能立刻发现。

---

## 1. ECRC 在 TLP 中的位置与标识

当 ECRC 启用时，TLP Header 中的 **TD（TLP Digest）位** 置为 `1`，并在 TLP 数据负载之后附加 1 个 DW（4 字节）的 ECRC：

```
┌──────────────────────────────────────────────────────────┐
│  TLP Header (3 或 4 DW)  —— 包含 TD = 1                  │
├──────────────────────────────────────────────────────────┤
│  Data Payload (0 ~ 1024 DW，可选)                         │
├──────────────────────────────────────────────────────────┤
│  ECRC / TLP Digest (1 DW = 4 字节)  ◀── 事务层端到端校验  │
└──────────────────────────────────────────────────────────┘
  ▲                                                      ▲
  └────────────── ECRC 计算覆盖范围 ──────────────────────┘
  (注：再往外由数据链路层加上 Seq# 和 LCRC，构成完整链路帧)
```

---

## 2. ECRC 与 LCRC 的核心区别

| 对比维度 | ECRC (End-to-End CRC) | LCRC (Link CRC) |
|----------|-----------------------|-----------------|
| **所属层** | 事务层（Transaction Layer） | 数据链路层（Data Link Layer） |
| **是否强制** | **可选**（需两端协商支持并启用） | **强制**（所有 TLP 必须包含） |
| **保护范围** | **端到端**（穿越 Switch 直到最终接收者） | **逐跳**（仅覆盖相邻两个端口之间的单条链路） |
| **Switch 处理** | 原则上透传不重算 | 每跳校验并剥离，出向端口重新生成 |
| **计算掩码** | 对路由途中可能变化的字段（如 `EP`、Type 0/1 转换相关位）按规范置 1 掩码处理 | 对实际发送的链路字节直接计算 |
| **出错后果** | **无法靠链路重传修复**，上报 AER 不可纠正错误（`ECRC Error`） | 触发 `Nak` 硬件自动重传，对软件透明 |

---

## 3. ECRC 校验与错误处理流程

1. **生成侧**：当设备配置空间启用 **ECRC Generation Enable** 时，事务层在构造 TLP 时置位 `TD=1`，使用标准 IEEE 802.3 CRC-32 多项式（`0x04C11DB7`）对 Header + Payload 计算 32 位校验值并附在末尾。
2. **Switch 中间转发**：Switch 仅校验并重算逐跳的 LCRC，不得随意改动带 ECRC 的 TLP；若某些特殊桥接必须修改 TLP 字段，则需按规范剥离 ECRC（清零 `TD`）或重新生成。
3. **接收侧**：当开启 **ECRC Check Enable** 的接收方收到 `TD=1` 的 TLP 时，重新计算 ECRC 并与末尾值比对：
   - **一致**：数据端到端完整，正常交付上层逻辑。
   - **不一致**：丢弃该 TLP，并在 **AER 不可纠正错误状态寄存器**（Uncorrectable Error Status）中置位 **ECRC Error**（同时可根据策略向上游 Root Complex 发送 `ERR_NONFATAL` 或 `ERR_FATAL` 错误消息）。

---

## 4. 查看与启用 ECRC（Linux 实战）

通过 `lspci` 可以查看设备 **AER（Advanced Error Reporting）** 能力寄存器中对 ECRC 的支持与当前状态：

```bash
lspci -vvv -s 01:00.0 | grep -i ecrc
# 输出示例：
# AERCap: First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
```

- `ECRCGenCap+` / `ECRCChkCap+`：硬件支持生成 / 校验 ECRC。
- `ECRCGenEn-` / `ECRCChkEn-`：当前默认未开启（常见于普通桌面/服务器环境以节省 4 字节带宽与延迟；在金融、高可靠存储或长距多级 Switch 拓扑中可通过 BIOS 或内核参数 `pci=ecrc=on` 统一开启）。

---

## 5. 关联与引申概念

- **LCRC（Link CRC）**：数据链路层强制的逐跳 CRC-32 校验，负责物理链路误码的自动重传，详见 [LCRC 详解](../data-link-layer/lcrc.md)。
- **TLP Digest（TD Bit）**：TLP Header DW0 中的 1-bit 标志位，用于指示 TLP 尾部是否携带了 1 DW 的 ECRC，详见 [TLP 格式详解](tlp-format.md)。
- **AER（Advanced Error Reporting）**：PCIe 扩展配置空间中的标准化错误报告框架，负责 ECRC 的能力通告、开关控制以及 `ECRC Error` 状态记录与消息上报。
- **Poisoned TLP（EP Bit）**：当设备发现数据本身已损坏但仍需按协议转发时，会置位 Header 中的 `EP`（Error/Poisoned）位通知下游；ECRC 计算规则对 `EP` 位有特殊的掩码处理以兼容错误传播。
