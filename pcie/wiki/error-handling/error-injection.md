# PCIe 错误注入（Error Injection）

**PCIe 错误注入**是一种主动容错测试技术，通过在受控环境中人为制造或模拟总线异常，来验证系统的 **AER 上报、DPC 隔离及驱动恢复路径** 是否按预期工作。就像大楼定期举行的“消防演习”——绝不能等到生产环境真正着火才发现喷淋头生锈、疏散通道锁死；通过错误注入，工程师可以在实验室里提前暴露驱动死锁、复位失败或数据未隔离等隐藏缺陷。

---

## 三类常见注入方式对比

| 注入层级 | 代表工具 / 方式 | 真实度 | 适用场景 |
|---------|---------------|--------|---------|
| **软件寄存器注入** | Linux `aer-inject` 模块、`setpci` 直接写 AER Status | 模拟 AER 上报与软件恢复流程（不产生真实链路误码） | 验证 OS AER 驱动、设备驱动的 `pci_error_handlers` 回调及日志记录 |
| **固件/平台级注入** | ACPI **EINJ（Error Injection）** 接口（`/sys/kernel/debug/apei/einj`） | 贴近真实硬件中断与固件处理路径 | 服务器级 RAS（可靠性/可用性/可维护性）全链路验收 |
| **硬件/协议分析仪注入** | Teledyne LeCroy、Keysight PCIe Exerciser / 协议分析仪 | **100% 真实物理/链路/事务层故障**（篡改 LCRC、ECRC、丢包、拉低信号） | 芯片流片后硅验证（Post-Silicon）、合规性测试与硬件 DPC 触发测试 |

---

## 软件错误注入实践（Linux `aer-inject`）

Linux 内核提供 `aer_inject` 模块（需内核开启 `CONFIG_PCIEAER_INJECT`），允许用户态通过规则文件向指定 **BDF** 设备注入虚拟 AER 错误，触发完整的内核 AER 中断与恢复流程。

### 1. 使用 `aer-inject` 注入错误

```bash
# 加载内核模块
modprobe aer-inject

# 编写错误描述文件（例如注入 Completion Timeout）
cat > /tmp/aer_cto.txt << 'EOF'
AER
PCI_ID 0000:01:00.0
UNCOR_STATUS CTO
EOF

# 执行注入并观察内核日志
aer-inject /tmp/aer_cto.txt
dmesg | tail -n 15
```

### 2. 常用 `aer-inject` 错误关键字速查

| 类别 | 关键字 | 对应错误类型 | 常见预期行为 |
|-----|-------|-------------|-------------|
| **Correctable** | `RXERR` | Receiver Error | 记录 CE 计数，设备无感继续运行 |
| **Correctable** | `BAD_TLP` / `BAD_DLLP` | Bad TLP / Bad DLLP | 记录 CE 计数，链路层模拟已自愈 |
| **Non-Fatal** | `CTO` | Completion Timeout | 触发驱动 `error_detected`，视驱动策略恢复或 FLR |
| **Non-Fatal** | `POISON_TLP` | Poisoned TLP Received | 记录 UE 状态并锁存 Header Log |
| **Non-Fatal** | `ECRC` / `UR` | ECRC Error / Unsupported Request | 上报不可纠正非致命错误 |
| **Fatal** | `DLP` / `SDES` | Data Link Protocol / Surprise Down | 触发致命恢复（Hot Reset 或 DPC 隔离） |
| **Fatal** | `FCP` / `MLFP` | Flow Control / Malformed TLP | 冻结通道并执行链路复位 |

> **提示**：也可以通过 `setpci` 向支持写注入的寄存器或使用 `lspci -vvv` 配合检查状态变化，但注意真实硬件的 AER Status 通常是 `RW1C`（写 1 清除），只有开启了 `aer_inject` 拦截或特定硬件 Debug 模式时写入才会置位报警。

---

## 验证注入结果与排查陷阱

### 1. 验证观测命令

```bash
# 查看设备 AER 状态位与掩码（确认是否被 Mask 屏蔽）
lspci -vvv -s 01:00.0 | grep -A 5 "Advanced Error Reporting"

# 查看 sysfs 累计计数（验证内核是否捕获并分类统计）
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_correctable
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_nonfatal
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_fatal
```

### 2. 测试流程与常见陷阱

推荐按 **“可纠正 → 非致命 → 致命”** 的顺序渐进测试，并在**专用测试机或虚拟机**上进行：

| 常见陷阱 | 后果 | 规避方法 |
|---------|------|---------|
| 在挂载了生产数据的 NVMe/RAID 卡上注入 Fatal 错误 | 文件系统损坏或系统崩溃 | 使用闲置测试盘、只读挂载或虚拟化环境 |
| 错误被 `UEMsk` / `CEMsk` 屏蔽 | 注入后 `dmesg` 无任何反应 | 注入前先用 `lspci -vvv` 确认对应错误位未置 `+` 屏蔽 |
| 忘记清理残留状态 | 干扰下一轮测试的 First Error 判断 | 每轮测试后确认内核已清除状态或手动复位 |

---

## 关联与引申概念

- **[AER 高级错误报告](aer.md)**：错误注入所模拟的核心寄存器与报文上报机制。
- **[PCIe 错误类型](error-types.md)**：了解每种注入关键字背后的协议层成因。
- **[PCIe 错误恢复](recovery.md)**：错误注入的最终验收目标——验证驱动回调、FLR、Hot Reset 与 DPC 是否正常工作。
- **[PCIe 错误检测](detection.md)**：了解硬件协议分析仪在物理层、链路层注入真实误码时，接收端是如何捕获的。
