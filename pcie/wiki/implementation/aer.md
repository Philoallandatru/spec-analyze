# AER 实现

> **一句话总结**：AER（Advanced Error Reporting，高级错误报告）是 PCIe 的"黑匣子 + 急诊分诊台"——当链路上出现数据包损坏或协议异常时，硬件会自动锁存出错瞬间的 TLP 包头现场，按严重程度分级上报给 Root Port，再由操作系统驱动执行隔离、复位与恢复。

---

## AER Extended Capability 寄存器布局

AER 位于 PCIe **扩展配置空间**（Extended Configuration Space，`0x100–0xFFF`），通过 Capability 链表挂载，Extended Capability ID 为 `0x0001`：

| 偏移 | 寄存器名称 | 位宽 | 读写属性 | 核心作用 |
| :--- | :--- | :--- | :--- | :--- |
| `+0x00` | Extended Capability Header | 32-bit | RO | `[15:0]` ID=`0x0001`，`[19:16]` Ver=`0x2`，`[31:20]` Next Offset |
| `+0x04` | Uncorrectable Error Status | 32-bit | RW1CS | 不可纠正错误状态（DLP、Poisoned TLP、Completion Timeout 等） |
| `+0x08` | Uncorrectable Error Mask | 32-bit | RWS | 掩码寄存器：置 1 则屏蔽对应不可纠正错误的报告与 Header Log |
| `+0x0C` | Uncorrectable Error Severity | 32-bit | RWS | 严重性配置：`0` = Non-Fatal（非致命），`1` = Fatal（致命） |
| `+0x10` | Correctable Error Status | 32-bit | RW1CS | 可纠正错误状态（Receiver Error、Bad TLP/DLLP、Replay Timeout 等） |
| `+0x14` | Correctable Error Mask | 32-bit | RWS | 掩码寄存器：置 1 则屏蔽对应可纠正错误的报告 |
| `+0x18` | Advanced Error Cap & Control | 32-bit | RO/RWS | First Error Pointer（首个错误位索引）、ECRC 生成/检查使能 |
| `+0x1C–0x28` | Header Log (DW0–DW3) | 128-bit | ROS | 锁存触发首个不可纠正错误的 **TLP 4DW 包头**，供软件排错 |
| `+0x2C` | Root Error Command | 32-bit | RW | **仅 Root Port**：控制是否将收到的 COR/NONFATAL/FATAL 消息转为中断 |
| `+0x30` | Root Error Status | 32-bit | RW1CS | **仅 Root Port**：记录是否收到可纠正/非致命/致命错误消息 |
| `+0x34` | Error Source ID | 32-bit | ROS | **仅 Root Port**：记录上报错误的源设备 **BDF**（Bus/Device/Function） |

> **关键术语**：**W1C / RW1CS**（Write-1-to-Clear Sticky）指"写 1 清零"且在热复位后保持状态。软件读出错误位后必须向对应位写 `1` 才能清除，避免覆盖同时发生的新错误。

---

## 硬件层错误检测与上报机制

硬件在物理层（PHY）、数据链路层（DLL）和事务层（TL）实时监测异常，并按如下流水线处理：

```
  [PHY] Receiver Error / Link Down ──┐
  [DLL] Bad DLLP / Replay Timeout  ──┼─► 错误状态置位 (CE / UCE Status)
  [TL]  Malformed / ECRC / Poison  ──┘         │
                                               ▼
                                     Mask 掩码过滤 (未屏蔽?)
                                               │ 是
                                               ▼
                        锁存 128-bit TLP Header Log (仅 UCE)
                        按 Severity 分级发送 Error Message TLP
                                               │
                                               ▼
                              Root Port 置位 Root Error Status
                              记录 Error Source ID (BDF) 并触发 IRQ
```

常见错误分类：
- **可纠正错误（Correctable, CE）**：硬件可通过重传或纠错自动恢复（如 `Bad TLP`、`Bad DLLP`、`Replay Timer Timeout`），仅作统计告警，不影响业务。
- **不可纠正非致命错误（Non-Fatal UCE）**：单次事务失败但链路仍可用（如 `Completion Timeout`、`Unsupported Request`），通常可通过设备级复位或驱动重试恢复。
- **不可纠正致命错误（Fatal UCE）**：链路或协议栈已损坏（如 `Data Link Protocol Error`、`Malformed TLP`），需要触发链路复位或热复位（Slot Reset）。

---

## Linux 内核 AER 驱动处理流程

Linux 内核通过 `drivers/pci/pcie/aer.c` 管理 Root Port 的 AER 服务，采用 **"硬中断快收 + 工作队列慢处理"** 的两阶段架构：

```
1. 中断上半部 (aer_irq)
   ├─ 读取 Root Port 的 Root Error Status 与 Error Source ID (BDF)
   ├─ 向 Root Error Status 写 1 清除中断源
   └─ schedule_work() 调度工作队列 aer_isr

2. 工作队列下半部 (aer_isr)
   ├─ 根据 Source ID 定位出错的 pci_dev
   ├─ 读取设备 CE/UCE Status、Mask、Severity 与 4DW Header Log 并打印 dmesg
   ├─ 清除设备 AER Status (W1C)
   └─ 若为不可纠正错误，按严重性执行 pci_error_handlers 三段式恢复回调
```

### 驱动错误恢复三段式回调（`struct pci_error_handlers`）

| 回调阶段 | 触发时机 | 驱动职责 | 典型返回值 |
| :--- | :--- | :--- | :--- |
| `error_detected(dev, state)` | 刚检测到 UCE | 停止下发新 I/O、冻结队列、评估是否可恢复 | `PCI_ERS_RESULT_NEED_RESET` / `DISCONNECT` |
| `slot_reset(dev)` | 平台完成槽位/链路复位后 | 重新 `pci_enable_device`、恢复配置空间、重建队列 | `PCI_ERS_RESULT_RECOVERED` |
| `resume(dev)` | 恢复完成 | 重新允许上层提交 I/O 请求 | `void` |

---

## QEMU / FEMU 中的 AER 模拟

在 QEMU/FEMU 虚拟设备中，AER 主要用于验证内核驱动的错误处理与恢复路径：
1. **挂载能力结构**：设备初始化时调用 `pcie_add_capability(pci_dev, PCI_EXT_CAP_ID_ERR, ...)` 在扩展配置空间分配 AER 寄存器块，并为 Status 寄存器设置 `w1cmask = 0xffffffff` 以模拟 W1C 语义。
2. **错误注入**：测试代码可调用 `pcie_aer_inject_error(pci_dev, &err)`，设置设备 UCE/CE Status 并向虚拟 Root Port 发送 Error Message，从而在 Guest OS 中完整复现 `aer_irq -> aer_isr -> error_detected` 链路。

---

## 调试与错误注入验证

在 Linux 环境中可通过 `sysfs`、`dmesg` 与 `lspci` 快速定位或模拟 AER 错误：

```bash
# 1. 查看内核打印的 AER 错误详情（含 TLP Header Log 与 Source ID）
dmesg | grep -i -E "aer|pcieport"

# 2. 查看指定设备的 AER 累计错误统计
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_correctable
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_nonfatal

# 3. 使用 lspci 查看 AER 能力与当前状态位
lspci -vvv -s 01:00.0 | grep -A 15 "Advanced Error Reporting"
```

> 若需主动注入故障测试驱动恢复能力，可使用开源工具 `aer-inject`（需内核开启 `CONFIG_PCIEAER_INJECT`）向指定 BDF 注入可纠正或不可纠正错误。

---

## 关联与引申概念

- **DPC（Downstream Port Containment）**：当发生致命错误时，DPC 能在硬件层面立即切断下行端口流量，防止损坏数据污染系统内存，常与 AER 配合使用。
- **FLR（Function Level Reset）**：单个 PCIe Function 的轻量级复位，在 AER 非致命错误恢复时优先于整槽位复位（Slot Reset）。
- **延伸阅读**：
  - [AER 协议详解](../error-handling/aer.md)
  - [PCIe 错误恢复流程](../error-handling/recovery.md)
  - [PCIe 错误注入测试](../error-handling/error-injection.md)
  - [TLP 数据包格式](../transaction-layer/tlp-format.md)
  - [PCIe 驱动开发指南](driver-guide.md)
