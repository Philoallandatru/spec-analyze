# PCIe 错误类型

**PCIe 错误类型**是 PCIe 协议在物理层、数据链路层和事务层定义的各类异常事件集合。如果把 PCIe 传输系统比作医院的急诊分诊台，所有异常都会按“伤情”分为三类：**可纠正错误（轻伤，硬件当场自愈）**、**不可纠正非致命错误（中伤，单次操作失败但设备仍可抢救）** 和 **不可纠正致命错误（重伤，链路或设备失控必须复位）**。

---

## 两条维度看懂错误分类

### 1. 按严重性分类（软件视角）

```
┌──────────────────────────────────────────────────────────┐
│  致命错误 (Uncorrectable Fatal)                           │
│  • 链路或核心状态已损坏，设备不可信；必须隔离或复位设备      │
├──────────────────────────────────────────────────────────┤
│  非致命错误 (Uncorrectable Non-Fatal)                    │
│  • 无法硬件自动修复，单次读写可能失败；但链路仍完好，可恢复 │
├──────────────────────────────────────────────────────────┤
│  可纠正错误 (Correctable)                                 │
│  • 物理层/链路层硬件自动纠正（如重传成功），对软件完全透明   │
└──────────────────────────────────────────────────────────┘
```

### 2. 按协议层次分类（硬件视角）

| 协议层 | 职责范围 | 典型错误 |
|-------|---------|---------|
| **Physical Layer（物理层）** | 串并转换、8b/10b 或 128b/130b 编解码、差分信号收发 | Receiver Error（符号解码/极性错误） |
| **Data Link Layer（数据链路层）** | 逐跳可靠性保证：LCRC 校验、ACK/NAK 重传、流控信用更新 | Bad TLP、Bad DLLP、Replay Timer Timeout、Flow Control Protocol Error、Data Link Protocol Error |
| **Transaction Layer（事务层）** | 端到端数据包（TLP）组装、地址路由、ECRC 校验、Completion 匹配 | Poisoned TLP、Completion Timeout、Unsupported Request、Completer Abort、Malformed TLP、ECRC Error |

---

## 一、可纠正错误（Correctable Errors）

可纠正错误由硬件自动处理，不会导致数据丢失，软件只需将其计入 AER 的 **Correctable Error Status（`+0x10`）** 寄存器用于监控链路健康度。

| Bit | 错误名称 | 常见诱因 | 硬件自动恢复机制 |
|-----|---------|---------|----------------|
| `0` | **Receiver Error** | 物理层 8b/10b 解码错误、极性（Disparity）错误、瞬时信号噪声 | 物理层重同步；若持续出现则触发链路重训练 |
| `6` | **Bad TLP** | 收到 **LCRC（链路层 CRC）** 校验失败或序列号异常的 TLP | 丢弃该 TLP，不发 ACK（或发 NAK），触发对端从 **Retry Buffer** 重传 |
| `7` | **Bad DLLP** | 收到 **DLLP（数据链路层数据包）** 16-bit CRC 校验失败 | 直接丢弃该 DLLP（DLLP 本身不重传，由后续周期性 DLLP 覆盖） |
| `8` | **REPLAY_NUM Rollover** | 同一 TLP 连续重传 4 次仍失败，重传计数器溢出 | 自动触发物理层链路重训练（Link Retrain） |
| `12` | **Replay Timer Timeout** | 发送 TLP 后在重传定时器超时前未收到 ACK | 发送端主动从 Retry Buffer 重传所有未确认的 TLP |
| `13` | **Advisory Non-Fatal** | 某些无需打断上层的建议性非致命错误（如部分可安全忽略的 UR） | 降级记入可纠正状态并记录日志 |
| `14` | **Corrected Internal Error** | 设备内部逻辑检测到并由 ECC 等机制自动纠正的错误 | 内部自愈，仅统计计数 |
| `15` | **Header Log Overflow** | AER Header Log 寄存器已满时又来了新的不可纠正错误 | 保留首个错误或按策略记录溢出标志 |

---

## 二、不可纠正非致命错误（Uncorrectable Non-Fatal）

不可纠正错误记录在 AER 的 **Uncorrectable Error Status（`+0x04`）** 寄存器中。默认情况下，以下错误属于 **Non-Fatal**（可在 **UE Severity `+0x0C`** 中调整）：

| Bit | 错误名称 | 核心成因与机制说明 |
|-----|---------|------------------|
| `12` | **Poisoned TLP Received** | **毒化 TLP**：发送端在读取内存或缓存时发现无法纠正的 ECC 错误，于是将 TLP 头部的 `EP`（Error Poisoned）位置 `1` 后继续发出，避免把“假数据”静默交给接收端。CPU 收到后可能触发 **MCE（Machine Check Exception）** 终止受损进程 |
| `14` | **Completion Timeout** | **完成超时**：CPU 或设备发出 **Non-Posted 请求**（如 Memory Read，必须等待对端返回 Completion 包），但超过预设时间（通常 50 μs ~ 50 s，常见 1~10 ms，可在 Device Control 2 配置）仍未收到响应 |
| `15` | **Completer Abort (CA)** | **完成端中止**：目标设备收到了合法请求，但因自身内部状态异常无法完成，主动返回带有 `CA` 状态码的 Completion |
| `16` | **Unexpected Completion** | **意外完成包**：收到了一个没有匹配未决请求（Requester ID + Tag 对不上）的 Completion TLP |
| `19` | **ECRC Error** | **端到端 CRC 错误**：TLP 在经过中间 Switch 后到达终点，但 **ECRC（End-to-End CRC）** 校验失败，说明报文在端到端路径上发生篡改（默认 Non-Fatal，部分高可靠系统会将其设为 Fatal） |
| `20` | **Unsupported Request (UR)** | **不支持的请求**：访问了不存在的 **BAR（Base Address Register，基地址寄存器）** 空间、非法配置空间地址，或向 Endpoint 发送了 Type 1 配置请求 |
| `21` | **ACS Violation** | **访问控制违规**：违反了 **ACS（Access Control Services）** 设定的 P2P 隔离策略，被 Switch 拦截并上报 |

---

## 三、不可纠正致命错误（Uncorrectable Fatal）

致命错误意味着链路协议或设备状态已经崩溃，继续运行极易导致系统挂死或数据大面积损坏。PCIe 规范默认将以下错误设为 **Fatal**：

| Bit | 错误名称 | 为什么是致命错误 |
|-----|---------|----------------|
| `4` | **Data Link Protocol Error** | 数据链路层的 Ack/Nak 协议或序列号机制崩溃，无法再保证可靠传输 |
| `5` | **Surprise Down Error** | 链路在未经过正常电源管理或热拔出流程的情况下突然断开（例如热插拔直接拔卡或设备掉电） |
| `13` | **Flow Control Protocol Error** | **流控信用（Flow Control Credits）** 计算违规（如发送量超过接收端通告的缓冲区信用），意味着接收缓冲区可能已溢出破坏 |
| `17` | **Receiver Overflow** | 接收端缓冲区彻底溢出，已有 TLP 丢失 |
| `18` | **Malformed TLP** | **格式严重错误的 TLP**（例如 Length 字段与实际 Payload 不匹配、Type 编码保留未定义、地址越界）。注意：*Bad TLP* 是链路层 CRC 错可重传恢复，而 *Malformed TLP* 是构造者就造出了非法包，必须丢弃并报错 |
| `22` | **Uncorrectable Internal Error** | 设备内部发生无法用 ECC 纠正的严重硬件故障 |

---

## 严重性配置与掩码机制

AER 允许系统软件根据业务需求微调错误策略：

- **Uncorrectable Error Severity（`+0x0C`）**：按 bit 对应 UE Status。设为 `1` 表示将该错误升级为 **Fatal**（触发设备/链路复位），设为 `0` 表示 **Non-Fatal**（交由驱动尝试恢复）。
- **Uncorrectable / Correctable Error Mask（`+0x08` / `+0x14`）**：设为 `1` 表示**屏蔽上报**。注意：屏蔽并不代表错误没有发生，只是硬件不再向 Root Complex 发送 `ERR_*` 报文，常用于屏蔽已知的硬件误报或防止错误风暴。

---

## 影响矩阵与监控

| 错误类型 | 数据完整性 | 系统/设备可用性 | 性能影响 | 典型恢复耗时 |
|---------|-----------|---------------|---------|------------|
| Receiver Error | 无影响 | 无影响 | 极轻微 | < 1 μs（物理层自愈） |
| Replay Timeout | 无影响 | 无影响 | 轻微~中等 | < 10 ms（链路层重传） |
| Poisoned TLP | 受损（已标记隔离） | 单次任务失败 | 中等 | 取决于软件隔离策略 |
| Completion Timeout | 请求失败 | 取决于驱动容错 | 较大 | 50 μs ~ 数十秒 |
| Malformed TLP / DLP | 协议崩溃 | 设备暂不可用 | 严重 | 需 FLR / Hot Reset |

### Linux sysfs 与调试查看

```bash
# 查看设备累计的各类 AER 错误统计（内核自动维护）
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_correctable
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_nonfatal
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_fatal

# 使用 lspci 查看当前 AER 状态与严重性掩码
lspci -vvv -s 01:00.0 | grep -A 6 "Advanced Error Reporting"
```

---

## 关联与引申概念

- **[AER 高级错误报告](aer.md)**：了解上述错误状态位对应的寄存器布局，以及如何通过 `ERR_COR` / `ERR_NONFATAL` / `ERR_FATAL` 报文上报给操作系统。
- **[PCIe 错误检测](detection.md)**：深入物理层、数据链路层和事务层是如何捕获这些错误的。
- **[PCIe 错误恢复](recovery.md)**：了解不同严重等级的错误触发后，系统如何执行驱动回调、FLR、Hot Reset 或 DPC 隔离。
- **[PCIe 错误注入](error-injection.md)**：使用 `aer-inject` 模拟上述错误代码（如 `CTO`、`MLFP`、`SDES`）。
- **[ACS 访问控制服务](../advanced-features/acs.md)**：了解 Switch 间 P2P 路由隔离与 `ACS Violation` 错误的由来。
- **[流控初始化（Flow Control）](../data-link-layer/fc-init.md)** 与 **[ECRC 详解](../transaction-layer/ecrc.md)**：理解流控信用和端到端校验的底层原理。
