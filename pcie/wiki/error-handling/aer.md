# AER (Advanced Error Reporting) 高级错误报告

AER (Advanced Error Reporting) 是 PCIe 的高级错误处理机制，提供详细的错误检测、报告和恢复功能，是构建可靠 PCIe 系统的关键技术。

---

## 概述

**AER 的目的**：
- **错误检测**：识别各种 PCIe 传输和协议错误
- **错误分类**：区分可纠正、不可纠正和致命错误
- **错误报告**：通过 Message 向 Root Complex 报告
- **错误记录**：保存错误详情供诊断
- **错误恢复**：支持自动或手动错误恢复

**错误严重程度分类**：

| 类型 | 严重性 | 影响 | 处理方式 |
|------|--------|------|---------|
| Correctable Error | 低 | 已自动纠正，无数据丢失 | 记录，继续运行 |
| Uncorrectable Non-Fatal | 中 | 可能数据损坏，但设备可用 | 记录，尝试恢复 |
| Uncorrectable Fatal | 高 | 严重错误，设备不可用 | 记录，重置设备/系统 |

**AER 在 PCIe 架构中的位置**：
```
┌─────────────────────────────────────────┐
│  设备检测到错误                         │
│  (CRC 错误, 超时, 协议违规等)           │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  设备 AER Capability                    │
│  ┌──────────────────────────────────┐   │
│  │ 记录错误到 AER 寄存器            │   │
│  │ - Uncorrectable Error Status     │   │
│  │ - Correctable Error Status       │   │
│  │ - Header Log (保存错误 TLP)      │   │
│  └──────────────────────────────────┘   │
└──────────────────┬──────────────────────┘
                   │
                   ▼ 发送 Error Message
┌─────────────────────────────────────────┐
│  Root Complex                           │
│  ┌──────────────────────────────────┐   │
│  │ Root Error Status                │   │
│  │ Error Source Identification      │   │
│  │ 触发系统中断 (AER IRQ)           │   │
│  └──────────────────────────────────┘   │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  OS AER 驱动                            │
│  - 读取错误信息                         │
│  - 记录日志 (dmesg)                     │
│  - 执行恢复策略                         │
└─────────────────────────────────────────┘
```

---

## 错误分类详解

### Correctable Errors (可纠正错误)

**特点**：
- 硬件自动纠正（如 ECC 纠正、重传成功）
- 对软件透明，无需干预
- 记录用于监控和预警

**常见可纠正错误**：

| Bit | 错误名称 | 描述 |
|-----|---------|------|
| 0 | Receiver Error | 接收器检测到错误但已恢复 |
| 6 | Bad TLP | 接收到格式错误的 TLP，但已丢弃 |
| 7 | Bad DLLP | 接收到格式错误的 DLLP |
| 8 | REPLAY_NUM Rollover | 重传计数器溢出（链路质量差） |
| 12 | Replay Timer Timeout | 重传定时器超时（但最终成功） |
| 13 | Advisory Non-Fatal | 非致命建议性错误 |

**处理流程**：
```
1. 检测到错误 (如 LCRC 错误)
2. 数据链路层自动重传
3. 重传成功 → 纠正
4. 设置 Correctable Error Status 寄存器
5. 如果启用，发送 ERR_COR Message 到 RC
6. 继续正常操作
```

### Uncorrectable Non-Fatal Errors (不可纠正非致命错误)

**特点**：
- 无法自动纠正
- 可能导致数据损坏或操作失败
- 设备仍可用，可尝试恢复

**常见非致命错误**：

| Bit | 错误名称 | 描述 |
|-----|---------|------|
| 4 | Data Link Protocol Error | 数据链路协议错误 |
| 12 | Poisoned TLP Received | 接收到毒化 TLP（数据不可靠） |
| 13 | Flow Control Protocol Error | 流控制协议错误 |
| 14 | Completion Timeout | Completion 超时（未收到响应） |
| 15 | Completer Abort | 完成者中止事务 |
| 16 | Unexpected Completion | 收到未预期的 Completion |
| 17 | Receiver Overflow | 接收缓冲区溢出 |
| 18 | Malformed TLP | TLP 格式错误 |
| 19 | ECRC Error | 端到端 CRC 错误 |
| 20 | Unsupported Request | 不支持的请求 |

### Uncorrectable Fatal Errors (不可纠正致命错误)

**特点**：
- 严重错误，设备不可靠
- 需要重置设备或系统
- 可能导致系统崩溃

**触发条件**：
- 某些错误类型天然是 Fatal（如内部错误）
- Non-Fatal 错误未处理升级为 Fatal
- Severity 寄存器配置

**示例**：
```
内部错误 (Internal Error):
  设备内部逻辑错误，无法继续工作
  
多次 Completion Timeout:
  重复超时表明设备或链路严重故障
  
Uncorrectable Internal Error:
  ECC 无法纠正的内存错误
```

---

## AER Capability 结构

### Capability 寄存器布局

AER 是一个 **Extended Capability**，位于配置空间的扩展区域（0x100+）。

```
AER Extended Capability Header:
Offset 0x00: Cap ID (0x0001) + Version (0x2) + Next
┌─────────────┬─────────┬──────────────────┐
│ Next (12b)  │ Ver (4b)│ Cap ID = 0x0001  │
└─────────────┴─────────┴──────────────────┘

Offset 0x04: Uncorrectable Error Status
31                                        0
┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
│  │  │  │  │  │  │  │  │  │  │  │  │  │  │
└──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
RW1C: 写 1 清除错误状态

Offset 0x08: Uncorrectable Error Mask
31                                        0
┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
│  │  │  │  │  │  │  │  │  │  │  │  │  │  │
└──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
1 = 屏蔽该错误（不报告）
0 = 启用该错误

Offset 0x0C: Uncorrectable Error Severity
31                                        0
┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
│  │  │  │  │  │  │  │  │  │  │  │  │  │  │
└──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
1 = Fatal, 0 = Non-Fatal

Offset 0x10: Correctable Error Status
31                                        0
┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
│  │  │  │  │  │  │  │  │  │  │  │  │  │  │
└──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
RW1C

Offset 0x14: Correctable Error Mask
31                                        0
┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
│  │  │  │  │  │  │  │  │  │  │  │  │  │  │
└──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘

Offset 0x18: Advanced Error Capabilities and Control
┌────────────┬────────────┬──────────────┐
│ ECRC Gen   │ ECRC Check │ First Error  │
│ Enable (1) │ Enable (1) │ Ptr (5)      │
└────────────┴────────────┴──────────────┘

Offset 0x1C-0x2B: Header Log (4 DW)
┌────────────────────────────────────────┐
│  DW 0: 发生错误的 TLP Header 第 1 DW  │
│  DW 1: 第 2 DW                         │
│  DW 2: 第 3 DW                         │
│  DW 3: 第 4 DW (如果是 4 DW Header)    │
└────────────────────────────────────────┘
保存导致错误的 TLP，用于诊断

Offset 0x2C: Root Error Command (仅 Root Port)
┌────────────────────────────────────────┐
│ Fatal Error Reporting Enable          │
│ Non-Fatal Error Reporting Enable      │
│ Correctable Error Reporting Enable    │
└────────────────────────────────────────┘

Offset 0x30: Root Error Status (仅 Root Port)
┌────────────────────────────────────────┐
│ ERR_COR Received                       │
│ Multiple ERR_COR Received              │
│ ERR_FATAL/NONFATAL Received            │
│ Multiple ERR_FATAL/NONFATAL Received   │
│ First Uncorrectable Fatal              │
│ ...                                    │
└────────────────────────────────────────┘

Offset 0x34: Error Source Identification (仅 Root Port)
┌──────────────────┬──────────────────────┐
│ ERR_COR Source   │ ERR_FATAL/NONFATAL   │
│ ID (16b)         │ Source ID (16b)      │
└──────────────────┴──────────────────────┘
记录错误来源的 Requester ID
```

---

## 错误报告流程

### Error Message 发送

**Endpoint 检测到错误后**：
```
1. 设置对应的 Error Status 位
   Uncorrectable Error Status[14] = 1  (Completion Timeout)

2. 保存 TLP Header 到 Header Log (如果适用)
   Header Log[0-3] = 导致错误的 TLP 的前 4 DW

3. 检查 Error Mask
   if (Uncorrectable Error Mask[14] == 1)
       return;  // 屏蔽，不报告

4. 检查 Severity
   if (Uncorrectable Error Severity[14] == 1)
       error_type = FATAL;
   else
       error_type = NON_FATAL;

5. 发送 Error Message
   构造 Message TLP:
   ┌────────────────────────────────┐
   │ Type: Message (ERR_FATAL/      │
   │       ERR_NONFATAL/ERR_COR)    │
   │ Requester ID: 本设备 BDF       │
   │ Message Code:                  │
   │   0x30 = ERR_COR               │
   │   0x31 = ERR_NONFATAL          │
   │   0x32 = ERR_FATAL             │
   │ Routing: Implicit (到 RC)      │
   └────────────────────────────────┘

6. 如果是 Fatal Error
   停止发送新 TLP，等待恢复
```

### Root Complex 处理

**Root Port 接收 Error Message**：
```
1. 接收 ERR_* Message

2. 设置 Root Error Status
   ERR_FATAL/NONFATAL Received = 1

3. 记录 Error Source ID
   Error Source Identification[15:0] = 
       Message 中的 Requester ID

4. 检查 Root Error Command
   if (Fatal Error Reporting Enable)
       触发系统中断 (AER IRQ)

5. OS 中断处理程序被调用
   aer_irq_handler()
```

### OS 错误处理

**Linux AER 驱动处理**：
```c
// drivers/pci/pcie/aer.c
static irqreturn_t aer_irq_handler(int irq, void *context)
{
    struct pcie_device *dev = context;
    struct aer_rpc *rpc = get_service_data(dev);
    
    // 1. 读取 Root Error Status
    u32 status = read_root_error_status(dev);
    
    // 2. 读取 Error Source ID
    u16 source_id = read_error_source_id(dev);
    
    // 3. 定位出错设备
    struct pci_dev *err_dev = pci_get_domain_bus_and_slot(
        pci_domain_nr(dev->port->bus),
        (source_id >> 8) & 0xff,
        source_id & 0xff);
    
    // 4. 读取设备的 AER 寄存器
    u32 uncor_status = read_aer_uncor_status(err_dev);
    u32 cor_status = read_aer_cor_status(err_dev);
    u32 header_log[4];
    read_aer_header_log(err_dev, header_log);
    
    // 5. 记录错误日志
    aer_print_error(err_dev, uncor_status, cor_status, header_log);
    
    // 6. 执行错误恢复
    if (is_fatal_error(uncor_status)) {
        do_fatal_recovery(err_dev);  // 重置设备
    } else if (is_nonfatal_error(uncor_status)) {
        do_nonfatal_recovery(err_dev);  // 尝试恢复
    }
    
    // 7. 清除错误状态
    write_aer_uncor_status(err_dev, uncor_status);  // W1C
    write_root_error_status(dev, status);
    
    return IRQ_HANDLED;
}
```

---

## FEMU 实现

### AER Capability 初始化

```c
// hw/pci/pcie_aer.c
int pcie_aer_init(PCIDevice *dev, uint16_t offset,
                  uint16_t size, Error **errp)
{
    PCIExpressDevice *exp = &dev->exp;
    uint8_t *cfg = dev->config + offset;
    uint8_t *wmask = dev->wmask + offset;
    
    // 检查是否已有 AER
    if (exp->aer_cap) {
        return -EINVAL;
    }
    
    // 分配 AER 结构
    exp->aer_cap = offset;
    exp->aer_log = g_malloc0(sizeof(PCIEAERLog));
    
    // 初始化 Extended Capability Header
    pcie_add_capability(dev, PCI_EXT_CAP_ID_ERR, 0x2,
                       offset, PCI_ERR_SIZEOF);
    
    // 设置默认值
    // Uncorrectable Error Mask: 默认不屏蔽
    pci_set_long(cfg + PCI_ERR_UNCOR_MASK, 0);
    pci_set_long(wmask + PCI_ERR_UNCOR_MASK, PCI_ERR_UNC_SUPPORTED);
    
    // Uncorrectable Error Severity: 默认 Severity
    pci_set_long(cfg + PCI_ERR_UNCOR_SEVER, PCI_ERR_UNC_SEVERITY_DEFAULT);
    pci_set_long(wmask + PCI_ERR_UNCOR_SEVER, PCI_ERR_UNC_SUPPORTED);
    
    // Correctable Error Mask: 默认不屏蔽
    pci_set_long(cfg + PCI_ERR_COR_MASK, 0);
    pci_set_long(wmask + PCI_ERR_COR_MASK, PCI_ERR_COR_SUPPORTED);
    
    // Advanced Error Capabilities and Control
    pci_set_long(cfg + PCI_ERR_CAP, 0);
    pci_set_long(wmask + PCI_ERR_CAP,
                 PCI_ERR_CAP_ECRC_GENC | PCI_ERR_CAP_ECRC_CHKC);
    
    return 0;
}
```

### 错误注入（测试用）

```c
// hw/pci/pcie_aer.c
void pcie_aer_inject_error(PCIDevice *dev, const PCIEAERErr *err)
{
    uint8_t *aer_cap;
    uint32_t uncor_status, cor_status;
    
    if (!pcie_aer_is_available(dev)) {
        return;
    }
    
    aer_cap = dev->config + dev->exp.aer_cap;
    
    if (err->flags & PCIE_AER_ERR_IS_CORRECTABLE) {
        // 可纠正错误
        cor_status = pci_get_long(aer_cap + PCI_ERR_COR_STATUS);
        cor_status |= err->status;
        pci_set_long(aer_cap + PCI_ERR_COR_STATUS, cor_status);
        
        // 发送 ERR_COR Message
        pcie_aer_msg(dev, PCI_ERR_ROOT_COR_RCV);
    } else {
        // 不可纠正错误
        uncor_status = pci_get_long(aer_cap + PCI_ERR_UNCOR_STATUS);
        uncor_status |= err->status;
        pci_set_long(aer_cap + PCI_ERR_UNCOR_STATUS, uncor_status);
        
        // 保存 Header Log
        if (err->flags & PCIE_AER_ERR_HEADER_VALID) {
            memcpy(aer_cap + PCI_ERR_HEADER_LOG,
                   err->header, sizeof(err->header));
        }
        
        // 判断 Fatal/Non-Fatal
        uint32_t severity = pci_get_long(aer_cap + PCI_ERR_UNCOR_SEVER);
        bool is_fatal = !!(uncor_status & severity);
        
        // 发送 ERR_FATAL 或 ERR_NONFATAL Message
        pcie_aer_msg(dev, is_fatal ? 
                     PCI_ERR_ROOT_FATAL_RCV : PCI_ERR_ROOT_NONFATAL_RCV);
        
        // 如果是 Fatal，链路进入 DL_Down 状态
        if (is_fatal) {
            pcie_cap_lnksta_dllla_write(dev, 0);
        }
    }
}
```

---

## 实用技巧

### 查看 AER 状态

**读取 AER 寄存器**：
```bash
# 查看设备 AER Capability
lspci -vvv -s 01:00.0 | grep -A20 "Advanced Error Reporting"
  Capabilities: [100 v2] Advanced Error Reporting
    UESta:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    UEMsk:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    UESvrt: DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
    CESta:  RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
    CEMsk:  RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr+

# UESta: Uncorrectable Error Status (全为 - 表示无错误)
# CESta: Correctable Error Status
# UEMsk: Uncorrectable Error Mask
# UESvrt: Uncorrectable Error Severity (+ = Fatal, - = Non-Fatal)
```

**使用 setpci 读取**：
```bash
# 假设 AER 在 0x100
# Uncorrectable Error Status
setpci -s 01:00.0 104.l
  00000000  # 无错误

# Correctable Error Status
setpci -s 01:00.0 110.l
  00000001  # Bit 0 (Receiver Error) = 1
```

### 监控 AER 错误

**查看 dmesg**：
```bash
dmesg | grep -i "aer\\|pcie.*error"
  pcieport 0000:00:01.0: AER: Corrected error received: 0000:01:00.0
  nvme 0000:01:00.0: AER: PCIe Bus Error: severity=Corrected, type=Physical Layer, (Receiver ID)
  nvme 0000:01:00.0: AER:   device [144d:a808] error status/mask=00000001/00000000
  nvme 0000:01:00.0: AER:    [ 0] RxErr

# 解读:
# - Corrected error: 可纠正错误
# - Physical Layer: 物理层错误
# - RxErr: 接收错误
# - 已自动纠正，无需干预
```

---

## 总结

### 关键要点

1. ✅ **AER 提供三级错误分类**：Correctable, Uncorrectable Non-Fatal, Uncorrectable Fatal
2. ✅ **错误通过 Message TLP 报告**到 Root Complex，触发系统中断
3. ✅ **Header Log 保存错误 TLP**，用于详细诊断
4. ✅ **Mask 和 Severity 寄存器**允许灵活配置错误处理策略
5. ✅ **OS AER 驱动**自动处理错误，记录日志并尝试恢复
6. ✅ **FEMU 实现完整 AER**，支持错误注入和模拟

**最佳实践**：
- 生产环境启用 AER，监控 Correctable Errors 预警链路问题
- Correctable Errors 频繁出现时，检查链路质量和信号完整性
- Fatal Errors 后需要重置设备或系统
- 定期检查 dmesg 中的 AER 日志

---

## 下一步学习

- [错误恢复](recovery.md) - AER 错误后的恢复流程
- [链路训练](../physical-layer/link-training.md) - 链路质量与错误的关系

---

## 参考资料

- **规范**：PCIe Base Spec Chapter 6.2 (Error Signaling and Logging)
- **实现**：`hw/pci/pcie_aer.c` (FEMU)
- **Linux**：`drivers/pci/pcie/aer.c` (内核 AER 驱动)

---

**相关页面**：
- [← 错误处理概述](overview.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
