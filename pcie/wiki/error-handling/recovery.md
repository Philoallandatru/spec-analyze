# PCIe 错误恢复机制

**PCIe 错误恢复**是系统在检测到总线或设备异常后，在不重启整台服务器的前提下，通过硬件自愈、驱动协调或局部复位让设备恢复正常工作的容错流程。就像大楼的消防应急预案：**小火由自动喷淋当场扑灭（可纠正错误），局部故障由楼层管理员断电排查后重启（非致命错误/FLR），重大火情则立即关闭防火门隔离楼层再抢修（DPC 硬件隔离 + 热复位）**。

---

## 分层恢复决策架构

系统收到 AER 错误报告后，会根据错误严重程度和设备状态选择对应的恢复路径：

```
                        检测到 PCIe 错误
                               │
         ┌─────────────────────┼─────────────────────┐
         ▼                     ▼                     ▼
   Correctable           Uncorrectable         Uncorrectable
  （可纠正错误）         Non-Fatal（非致命）      Fatal（致命错误）
         │                     │                     │
         ▼                     ▼                     ▼
  硬件自动透明恢复        OS 调度驱动回调         立即冻结 I/O / DPC 隔离
  • 重传 / 物理层重同步    • 尝试在线恢复或 FLR    • 执行 Hot Reset（热复位）
  • 频繁出错触发链路重训练  • 失败则升级为复位      • 仍失败则永久隔离（Disconnect）
```

---

## 一、可纠正错误恢复（硬件自愈）

可纠正错误由物理层和数据链路层在微秒~毫秒级自动解决，**对操作系统和业务软件完全透明**：

1. **Receiver Error**：物理层通过时钟数据恢复（CDR）重新锁定信号。
2. **Bad TLP / Replay Timeout**：接收端丢弃坏包并回复 **NAK DLLP**，发送端从 **Retry Buffer** 重传未确认的 TLP。
3. **链路重训练（Link Retrain）**：若短时间内可纠正错误计数过高（如 **REPLAY_NUM Rollover**），硬件或端口驱动会置位 Link Control 寄存器的 **Retrain Link（`RL`）** 位，令 **LTSSM（链路训练与状态机）** 重新进入 `Recovery` 状态校准均衡参数与信号质量。

---

## 二、非致命错误恢复（驱动协同四步曲）

当发生 **Non-Fatal** 错误（如 Completion Timeout、Poisoned TLP）时，链路本身通常仍完好，Linux 内核 AER 驱动会通过设备驱动注册的 `pci_error_handlers` 回调接口，按以下状态机协调恢复：

```
1. error_detected(dev, channel_state)
   告知驱动错误已发生及当前通道状态：
   • io_normal       → 通道正常，驱动返回 CAN_RECOVER
   • io_frozen       → I/O 已冻结，驱动停止下发新请求，返回 CAN_RECOVER 或 NEED_RESET
   • io_perm_failure → 永久损坏，返回 DISCONNECT（卸载/隔离设备）
          │
          ├──[返回 CAN_RECOVER]──▶ 2. mmio_enabled(dev)
          │                           允许驱动读取 BAR 寄存器清除设备内部错误；
          │                           若读取返回 0xFFFFFFFF（无响应），改判 NEED_RESET
          │
          └──[返回 NEED_RESET]───▶ 3. slot_reset(dev)
                                      内核执行 FLR 或热复位后调用，
                                      驱动恢复 PCIe 配置空间、重新使能 Bus Master 并重初始化硬件
                                             │
                                             ▼
                                   4. resume(dev)
                                      清除 AER 状态寄存器（RW1C），恢复正常 I/O 队列
```

| 回调返回值（`pci_ers_result_t`） | 含义与后续动作 |
|-------------------------------|--------------|
| `PCI_ERS_RESULT_CAN_RECOVER` | 驱动认为无需复位即可尝试恢复，进入 `mmio_enabled` |
| `PCI_ERS_RESULT_NEED_RESET` | 必须复位硬件才能清状态，内核执行复位后调用 `slot_reset` |
| `PCI_ERS_RESULT_RECOVERED` | 硬件状态已清理完毕，准备进入 `resume` 恢复业务 |
| `PCI_ERS_RESULT_DISCONNECT` | 设备无法恢复，内核将其从系统隔离，防止拖垮主机 |

---

## 三、致命错误恢复：FLR vs Hot Reset

当发生 **Fatal** 错误（如 Data Link Protocol Error、Malformed TLP）或非致命错误恢复失败时，必须对设备甚至整条链路执行复位。两种核心复位手段的对比如下：

| 特性 | FLR（Function Level Reset，功能级复位） | Hot Reset（热复位 / Secondary Bus Reset） |
|-----|---------------------------------------|----------------------------------------|
| **影响范围** | **仅复位单个 Function**（如 SR-IOV 中的单个 VF，不影响同卡其他功能） | **复位下游整条总线/链路**，所有下游设备都会重新训练 |
| **触发方式** | 向设备 PCIe Cap 的 Device Control 寄存器写入 `Initiate FLR` 位 | 向 Root Port / Switch 下行端口的 **Bridge Control** 寄存器置位 `Secondary Bus Reset` |
| **典型耗时** | 至少等待 **100 ms** | 保持复位脉冲至少 **100 ms**，释放后再等链路训练稳定（约 200 ms） |
| **使用前提** | 设备 Device Capabilities 必须声明支持 FLR | 任何 PCIe 链路均支持（带内发送 TS1 热复位序列） |

> **关键注意事项**：无论是 FLR 还是 Hot Reset，复位都会清空设备 BAR、中断（MSI-X）和 Bus Master 等配置。因此内核在复位前必须先保存配置空间（`pci_save_state`），复位完成后再恢复（`pci_restore_state`）并重新启用 AER 错误报告。

---

## 四、DPC（Downstream Port Containment，下行端口隔离）

**DPC** 是 PCIe 3.1/4.0+ 广泛部署的**硬件级快速隔离机制**。在没有 DPC 之前，如果下游设备发生致命错误并继续向上游狂发损坏的 TLP，往往在操作系统 AER 中断还没来得及响应时，系统内存或 CPU 就已经被污染导致宕机（Kernel Panic / MCE）。

DPC 就像安装在 Root Port 或 Switch 下行端口的**自动空气开关**：

```
下游设备发生不可纠正错误
         │
         ▼
下行端口硬件立即触发 DPC（无需等软件介入）：
  1. 强制将链路拉入隔离态，阻断所有下行 TLP
  2. 丢弃所有上行 TLP，彻底防止“毒数据”污染主机内存
  3. 向上游发送错误信号并触发 DPC 中断
         │
         ▼
OS DPC 驱动异步介入恢复：
  1. 读取 DPC Status 与 DPC Error Source ID 定位祸首
  2. 对下游执行 Hot Reset 清理残局
  3. 清除 DPC Status 释放链路，恢复下游设备配置空间与业务
```

### 恢复状态检查命令

在 Linux 中可通过 `dmesg` 和 `lspci` 观察 AER/DPC 的隔离与恢复过程：

```bash
# 查看内核 AER 与 DPC 恢复日志
dmesg | grep -iE "aer|dpc|slot_reset|frozen"

# 检查 Root Port 是否支持并启用了 DPC（Extended Cap ID 0x001D）
lspci -vvv -s 00:01.0 | grep -A 5 "Downstream Port Containment"
```

---

## 关联与引申概念

- **[AER 高级错误报告](aer.md)**：错误恢复的触发源头，提供错误分级与 Source ID 定位。
- **[PCIe 错误类型](error-types.md)**：区分哪些错误会触发自愈、驱动回调或致命复位。
- **[PCIe 错误注入](error-injection.md)**：利用 `aer-inject` 在测试机上主动验证上述恢复流程是否生效。
- **[LTSSM 状态机](../physical-layer/ltssm.md)** 与 **[链路训练](../physical-layer/link-training.md)**：理解 Hot Reset 和 Link Retrain 在物理层如何通过 `Recovery` 状态重新建链。
- **[热插拔（Hot-Plug）](../advanced-features/hot-plug.md)** 与 **[SR-IOV](../advanced-features/sriov.md)**：了解 Surprise Down 与热拔出处理的区别，以及 FLR 在虚拟化多 VF 隔离中的关键作用。
