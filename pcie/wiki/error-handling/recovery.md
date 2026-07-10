# PCIe 错误恢复

## 概述

PCIe 错误恢复机制允许系统在检测到错误后尝试自动恢复，而无需重启整个系统。本文详细介绍 PCIe 的错误恢复流程、策略和实现。

## 错误恢复层次

```
错误严重性                恢复策略
───────────────────────────────────────
可纠正错误                自动透明恢复
(Correctable)            无需软件干预
    ↓
非致命错误                设备级恢复
(Non-Fatal)              驱动参与恢复
    ↓
致命错误                  链路级恢复
(Fatal)                  可能需要热复位
    ↓
无法恢复                  设备隔离
                         系统降级运行
```

## 可纠正错误恢复

### 自动恢复机制

```
1. Receiver Error (接收错误)
   检测 --> 物理层自动重同步
        --> 透明恢复
        --> 记录 CE Status

2. Replay Timeout (重传超时)
   超时 --> 发送 NAK DLLP
        --> 发送端从重试缓冲区重传
        --> 继续正常传输

3. Bad TLP/DLLP
   检测 --> 丢弃数据包
        --> 等待重传
        --> 透明恢复

特点：
- 硬件自动处理
- 软件无感知
- 不中断数据流
- 仅记录统计信息
```

### 链路重训练

```c
// 可纠正错误导致的链路重训练
void handle_correctable_error_recovery(struct pcie_port *port)
{
    struct error_stats *stats = &port->error_stats;
    
    // 增加错误计数
    stats->correctable_errors++;
    
    // 如果错误率过高，触发链路重训练
    if (stats->correctable_errors > CE_ERROR_THRESHOLD) {
        pr_warn("High correctable error rate, retraining link\n");
        
        // 发起链路重训练
        retrain_link(port);
        
        // 复位错误计数
        stats->correctable_errors = 0;
        stats->retrain_count++;
    }
}

// 链路重训练实现
int retrain_link(struct pcie_port *port)
{
    u16 link_ctrl;
    int timeout;
    
    // 读取 Link Control 寄存器
    pcie_capability_read_word(port->pdev, PCI_EXP_LNKCTL, &link_ctrl);
    
    // 设置 Retrain Link 位
    link_ctrl |= PCI_EXP_LNKCTL_RL;
    pcie_capability_write_word(port->pdev, PCI_EXP_LNKCTL, link_ctrl);
    
    // 等待链路重训练完成
    timeout = 1000;  // 1 秒超时
    while (timeout--) {
        u16 link_status;
        
        pcie_capability_read_word(port->pdev, PCI_EXP_LNKSTA, &link_status);
        
        // 检查 Link Training 位
        if (!(link_status & PCI_EXP_LNKSTA_LT)) {
            pr_info("Link retrain completed successfully\n");
            return 0;
        }
        
        msleep(1);
    }
    
    pr_err("Link retrain timeout\n");
    return -ETIMEDOUT;
}
```

## 非致命错误恢复

### 恢复流程

```
1. 错误检测
   设备检测到错误
        ↓
   设置 UCE Status[错误位]
        ↓
   如果未被掩码，触发 AER 中断

2. 错误报告
   Root Port 收到错误信号
        ↓
   Root Error Status 记录
        ↓
   系统中断到 AER 驱动

3. 错误分析
   AER 驱动读取错误状态
        ↓
   识别错误设备
        ↓
   收集错误信息（Header Log）

4. 驱动通知
   调用驱动的 error_detected 回调
        ↓
   驱动决定恢复策略
        ↓
   返回恢复结果

5. 恢复执行
   根据驱动返回值执行恢复
        ↓
   - PCI_ERS_RESULT_CAN_RECOVER: 继续
   - PCI_ERS_RESULT_NEED_RESET: 复位
   - PCI_ERS_RESULT_DISCONNECT: 隔离设备

6. 恢复完成
   调用驱动的 resume 回调
        ↓
   清除错误状态
        ↓
   设备恢复正常运行
```

### Linux 内核实现

```c
// PCI 错误恢复回调接口
struct pci_error_handlers {
    pci_ers_result_t (*error_detected)(struct pci_dev *dev,
                                        pci_channel_state_t error);
    pci_ers_result_t (*mmio_enabled)(struct pci_dev *dev);
    pci_ers_result_t (*slot_reset)(struct pci_dev *dev);
    void (*resume)(struct pci_dev *dev);
};

// 驱动实现错误检测回调
static pci_ers_result_t my_error_detected(struct pci_dev *pdev,
                                           pci_channel_state_t state)
{
    struct my_device *mydev = pci_get_drvdata(pdev);
    
    pr_info("Error detected on device %s, state=%d\n",
            pci_name(pdev), state);
    
    // 根据通道状态决定恢复策略
    switch (state) {
    case pci_channel_io_normal:
        // I/O 正常，可能是软件错误
        return PCI_ERS_RESULT_CAN_RECOVER;
        
    case pci_channel_io_frozen:
        // I/O 冻结，停止所有 I/O 操作
        stop_device_io(mydev);
        
        // 尝试 MMIO 访问以检查设备状态
        if (can_access_device(mydev))
            return PCI_ERS_RESULT_CAN_RECOVER;
        else
            return PCI_ERS_RESULT_NEED_RESET;
        
    case pci_channel_io_perm_failure:
        // 永久失败，无法恢复
        pr_err("Permanent failure on device %s\n", pci_name(pdev));
        return PCI_ERS_RESULT_DISCONNECT;
    }
    
    return PCI_ERS_RESULT_NEED_RESET;
}

// MMIO 启用后的回调
static pci_ers_result_t my_mmio_enabled(struct pci_dev *pdev)
{
    struct my_device *mydev = pci_get_drvdata(pdev);
    
    // 尝试读取设备寄存器
    u32 status = ioread32(mydev->bar0 + STATUS_REG);
    
    if (status == 0xFFFFFFFF) {
        // 设备无响应，需要复位
        pr_warn("Device not responding, need reset\n");
        return PCI_ERS_RESULT_NEED_RESET;
    }
    
    // 清除设备错误状态
    iowrite32(0xFFFFFFFF, mydev->bar0 + ERROR_CLEAR_REG);
    
    // 设备可以恢复
    return PCI_ERS_RESULT_RECOVERED;
}

// 槽位复位回调
static pci_ers_result_t my_slot_reset(struct pci_dev *pdev)
{
    struct my_device *mydev = pci_get_drvdata(pdev);
    int err;
    
    pr_info("Performing slot reset on device %s\n", pci_name(pdev));
    
    // 重新启用设备
    err = pci_enable_device(pdev);
    if (err) {
        pr_err("Failed to re-enable device: %d\n", err);
        return PCI_ERS_RESULT_DISCONNECT;
    }
    
    // 恢复配置空间
    pci_restore_state(pdev);
    
    // 重新设置 Bus Master
    pci_set_master(pdev);
    
    // 重新初始化设备
    err = reinit_device_hw(mydev);
    if (err) {
        pr_err("Failed to reinitialize device: %d\n", err);
        return PCI_ERS_RESULT_DISCONNECT;
    }
    
    pr_info("Slot reset completed successfully\n");
    return PCI_ERS_RESULT_RECOVERED;
}

// 恢复完成回调
static void my_error_resume(struct pci_dev *pdev)
{
    struct my_device *mydev = pci_get_drvdata(pdev);
    
    pr_info("Resuming normal operation on device %s\n", pci_name(pdev));
    
    // 重启 I/O 操作
    restart_device_io(mydev);
    
    // 清除 AER 错误状态
    pci_aer_clear_nonfatal_status(pdev);
    pci_cleanup_aer_uncorrect_error_status(pdev);
    
    // 重新启用错误报告
    pci_enable_pcie_error_reporting(pdev);
}

// 注册错误处理回调
static struct pci_error_handlers my_err_handlers = {
    .error_detected = my_error_detected,
    .mmio_enabled = my_mmio_enabled,
    .slot_reset = my_slot_reset,
    .resume = my_error_resume,
};

static struct pci_driver my_pci_driver = {
    .name = "my_driver",
    .id_table = my_pci_ids,
    .probe = my_probe,
    .remove = my_remove,
    .err_handler = &my_err_handlers,  // 注册错误处理
};
```

### 恢复结果代码

```c
// 错误恢复结果
enum pci_ers_result {
    PCI_ERS_RESULT_NONE = 1,        // 没有结果，继续下一步
    PCI_ERS_RESULT_CAN_RECOVER,     // 设备可以恢复
    PCI_ERS_RESULT_NEED_RESET,      // 需要复位设备
    PCI_ERS_RESULT_DISCONNECT,      // 无法恢复，隔离设备
    PCI_ERS_RESULT_RECOVERED,       // 已恢复
};

// 通道状态
typedef enum {
    pci_channel_io_normal = 0,      // I/O 正常
    pci_channel_io_frozen = 1,      // I/O 冻结
    pci_channel_io_perm_failure = 2 // 永久失败
} pci_channel_state_t;
```

## 致命错误恢复

### 热复位 (Hot Reset)

```
热复位流程：
1. 系统检测到致命错误
2. 冻结所有到该设备的 I/O
3. 执行热复位
   - 发送 TS1 训练序列
   - 链路进入 Recovery 状态
   - 重新训练链路
4. 恢复配置空间
5. 重新初始化设备
6. 恢复 I/O 操作

特点：
- 不影响其他设备
- 保持系统运行
- 可能丢失未完成的事务
```

#### 热复位实现

```c
// 执行热复位
int pcie_do_hot_reset(struct pci_dev *pdev)
{
    struct pci_bus *bus = pdev->bus;
    u16 bridge_ctl;
    int ret;
    
    pr_info("Performing hot reset on device %s\n", pci_name(pdev));
    
    // 保存设备配置空间
    pci_save_state(pdev);
    
    // 如果是 Root Port 或 Bridge，通过 Bridge Control 复位
    if (pci_is_bridge(pdev)) {
        // 读取 Bridge Control 寄存器
        pci_read_config_word(pdev, PCI_BRIDGE_CONTROL, &bridge_ctl);
        
        // 设置 Secondary Bus Reset 位
        bridge_ctl |= PCI_BRIDGE_CTL_BUS_RESET;
        pci_write_config_word(pdev, PCI_BRIDGE_CONTROL, bridge_ctl);
        
        // 保持复位至少 100 ms
        msleep(100);
        
        // 清除复位位
        bridge_ctl &= ~PCI_BRIDGE_CTL_BUS_RESET;
        pci_write_config_word(pdev, PCI_BRIDGE_CONTROL, bridge_ctl);
        
        // 等待链路稳定
        msleep(200);
    } else {
        // Endpoint 设备，通过 FLR 或其他方法复位
        ret = pci_try_reset_function(pdev);
        if (ret) {
            pr_err("Failed to reset device: %d\n", ret);
            return ret;
        }
    }
    
    // 恢复配置空间
    pci_restore_state(pdev);
    
    // 重新启用错误报告
    pci_enable_pcie_error_reporting(pdev);
    
    pr_info("Hot reset completed\n");
    return 0;
}
```

### FLR (Function Level Reset)

```c
// 功能级复位
int perform_flr(struct pci_dev *pdev)
{
    int pos;
    u16 devctl;
    
    // 查找 PCIe Capability
    pos = pci_pcie_cap(pdev);
    if (!pos)
        return -ENOTTY;
    
    // 读取 Device Control 寄存器
    pci_read_config_word(pdev, pos + PCI_EXP_DEVCTL, &devctl);
    
    // 检查是否支持 FLR
    u32 devcap;
    pci_read_config_dword(pdev, pos + PCI_EXP_DEVCAP, &devcap);
    if (!(devcap & PCI_EXP_DEVCAP_FLR)) {
        pr_warn("Device does not support FLR\n");
        return -ENOTTY;
    }
    
    // 保存状态
    pci_save_state(pdev);
    
    // 发起 FLR
    devctl |= PCI_EXP_DEVCTL_BCR_FLR;
    pci_write_config_word(pdev, pos + PCI_EXP_DEVCTL, devctl);
    
    // 等待复位完成（至少 100 ms）
    msleep(100);
    
    // 恢复配置空间
    pci_restore_state(pdev);
    
    pr_info("FLR completed on device %s\n", pci_name(pdev));
    return 0;
}
```

## 恢复策略

### 决策流程

```
错误发生
    ↓
读取错误状态和严重性
    ↓
    ┌─────────────┬─────────────┬─────────────┐
    ↓             ↓             ↓             ↓
可纠正错误    非致命错误      致命错误     永久失败
    ↓             ↓             ↓             ↓
硬件自动恢复  驱动参与恢复   热复位/FLR   隔离设备
    ↓             ↓             ↓             ↓
透明恢复      恢复 I/O       重新初始化   降级运行
```

### 策略选择

```c
// 选择恢复策略
static int select_recovery_strategy(struct pci_dev *pdev,
                                     u32 uce_status,
                                     u32 uce_severity)
{
    // 致命错误
    if (uce_status & uce_severity) {
        pr_warn("Fatal error detected, attempting hot reset\n");
        return RECOVERY_HOT_RESET;
    }
    
    // 非致命但严重的错误
    if (uce_status & (PCI_ERR_UNC_POISON_TLP |
                      PCI_ERR_UNC_COMP_TIME |
                      PCI_ERR_UNC_MALF_TLP)) {
        pr_info("Serious error, attempting FLR\n");
        return RECOVERY_FLR;
    }
    
    // 一般非致命错误
    if (uce_status) {
        pr_info("Non-fatal error, attempting driver recovery\n");
        return RECOVERY_DRIVER;
    }
    
    // 可纠正错误
    return RECOVERY_AUTO;
}
```

## DPC (Downstream Port Containment)

### DPC 机制

```
DPC 是 PCIe 4.0+ 引入的硬件错误隔离机制：

特点：
- 硬件自动触发
- 快速隔离错误设备
- 防止错误传播
- 无需软件干预

触发条件：
- 检测到不可纠正错误
- 满足 DPC 触发条件
- 硬件自动执行隔离

隔离动作：
1. 阻止所有下行 TLP
2. 丢弃所有上行 TLP（除了某些类型）
3. 向上游发送 ERR_FATAL 或 ERR_NONFATAL
4. 等待软件恢复

软件恢复：
1. 读取 DPC 状态
2. 识别错误源
3. 执行热复位
4. 清除 DPC 状态
5. 恢复链路
```

#### DPC 配置和处理

```c
// 配置 DPC
static int configure_dpc(struct pci_dev *pdev)
{
    int pos;
    u16 cap, ctl;
    
    // 查找 DPC Extended Capability
    pos = pci_find_ext_capability(pdev, PCI_EXT_CAP_ID_DPC);
    if (!pos) {
        pr_info("Device does not support DPC\n");
        return -ENOTTY;
    }
    
    // 读取 DPC Capabilities
    pci_read_config_word(pdev, pos + PCI_EXP_DPC_CAP, &cap);
    
    // 配置 DPC Control
    ctl = PCI_EXP_DPC_CTL_EN_FATAL |      // 致命错误触发
          PCI_EXP_DPC_CTL_EN_NONFATAL;     // 非致命错误触发
    
    pci_write_config_word(pdev, pos + PCI_EXP_DPC_CTL, ctl);
    
    pr_info("DPC enabled on device %s\n", pci_name(pdev));
    return 0;
}

// DPC 中断处理
static irqreturn_t dpc_irq_handler(int irq, void *context)
{
    struct pcie_device *dev = context;
    struct pci_dev *pdev = dev->port;
    int pos = pdev->dpc_cap;
    u16 status, source;
    
    // 读取 DPC Status
    pci_read_config_word(pdev, pos + PCI_EXP_DPC_STATUS, &status);
    
    if (!(status & PCI_EXP_DPC_STATUS_TRIGGER)) {
        return IRQ_NONE;
    }
    
    pr_err("DPC triggered on port %s\n", pci_name(pdev));
    
    // 读取错误源
    pci_read_config_word(pdev, pos + PCI_EXP_DPC_SOURCE_ID, &source);
    pr_err("DPC Source: %04x\n", source);
    
    // 调度恢复工作
    schedule_work(&dev->dpc_work);
    
    return IRQ_HANDLED;
}

// DPC 恢复工作
static void dpc_work_func(struct work_struct *work)
{
    struct pcie_device *dev = container_of(work, struct pcie_device, dpc_work);
    struct pci_dev *pdev = dev->port;
    int pos = pdev->dpc_cap;
    u16 ctl;
    
    pr_info("Starting DPC recovery\n");
    
    // 执行热复位
    pcie_do_hot_reset(pdev);
    
    // 清除 DPC Status
    pci_write_config_word(pdev, pos + PCI_EXP_DPC_STATUS,
                          PCI_EXP_DPC_STATUS_TRIGGER);
    
    // 重新启用链路
    pci_read_config_word(pdev, pos + PCI_EXP_DPC_CTL, &ctl);
    ctl &= ~PCI_EXP_DPC_CTL_INT_EN;  // 暂时禁用中断
    pci_write_config_word(pdev, pos + PCI_EXP_DPC_CTL, ctl);
    
    // 恢复下游设备
    pci_walk_bus(pdev->subordinate, pci_dev_restore_state, NULL);
    
    // 重新启用 DPC 中断
    ctl |= PCI_EXP_DPC_CTL_INT_EN;
    pci_write_config_word(pdev, pos + PCI_EXP_DPC_CTL, ctl);
    
    pr_info("DPC recovery completed\n");
}
```

## 恢复测试

### 错误注入测试

```bash
# 使用 aer-inject 工具测试恢复
cat > error.txt << EOF
AER
COR RCVR
EOF

aer-inject -s 01:00.0 error.txt

# 观察恢复过程
dmesg | tail -20
```

### 驱动恢复测试

```c
// 测试驱动的错误恢复能力
static int test_error_recovery(struct pci_dev *pdev)
{
    pci_ers_result_t result;
    
    pr_info("Testing error recovery\n");
    
    // 模拟错误检测
    result = my_error_detected(pdev, pci_channel_io_frozen);
    
    if (result == PCI_ERS_RESULT_NEED_RESET) {
        // 测试槽位复位
        result = my_slot_reset(pdev);
        
        if (result == PCI_ERS_RESULT_RECOVERED) {
            // 测试恢复
            my_error_resume(pdev);
            pr_info("Recovery test passed\n");
            return 0;
        }
    }
    
    pr_err("Recovery test failed\n");
    return -EIO;
}
```

## 最佳实践

### 1. 实现完整的错误处理回调

```c
// 必须实现所有关键回调
static struct pci_error_handlers my_err_handlers = {
    .error_detected = my_error_detected,  // 必需
    .mmio_enabled = my_mmio_enabled,       // 推荐
    .slot_reset = my_slot_reset,           // 必需
    .resume = my_error_resume,             // 必需
};
```

### 2. 保存和恢复状态

```c
// 在关键点保存状态
pci_save_state(pdev);

// 复位后恢复
pci_restore_state(pdev);
```

### 3. 清理待处理的 I/O

```c
// 错误检测时停止 I/O
stop_device_io(mydev);
cancel_pending_requests(mydev);

// 恢复后重启 I/O
restart_device_io(mydev);
```

### 4. 超时保护

```c
// 所有恢复操作都应有超时
int timeout = 1000;  // 1 秒
while (timeout-- && !recovery_complete()) {
    msleep(1);
}
```

## 总结

PCIe 错误恢复的关键点：

1. **分层恢复**：根据错误严重性选择策略
2. **驱动参与**：非致命错误需要驱动配合
3. **热复位**：致命错误的最后手段
4. **DPC 隔离**：硬件自动隔离错误
5. **状态保存**：恢复时正确恢复配置
6. **超时保护**：防止恢复过程hang

参见：
- [错误类型](error-types.md)
- [错误检测](detection.md)
- [AER 详解](aer.md)
- [AER 实现](../implementation/aer.md)
