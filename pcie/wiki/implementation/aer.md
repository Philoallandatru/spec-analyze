# AER 实现

## 概述

AER (Advanced Error Reporting) 的实现涉及硬件错误检测、软件错误处理和错误日志记录。本文介绍 AER 在实际系统中的完整实现。

## AER Capability 结构回顾

```
AER Extended Capability (Offset 0x100+):
+0x00: [15:0]  Extended Capability ID = 0x0001
       [19:16] Capability Version = 0x2
       [31:20] Next Capability Offset

+0x04: [31:0]  Uncorrectable Error Status
+0x08: [31:0]  Uncorrectable Error Mask
+0x0C: [31:0]  Uncorrectable Error Severity
+0x10: [31:0]  Correctable Error Status
+0x14: [31:0]  Correctable Error Mask
+0x18: [31:0]  Advanced Error Capabilities and Control
+0x1C: [31:0]  Header Log DW0
+0x20: [31:0]  Header Log DW1
+0x24: [31:0]  Header Log DW2
+0x28: [31:0]  Header Log DW3
+0x2C: [31:0]  Root Error Command (Root Port only)
+0x30: [31:0]  Root Error Status (Root Port only)
+0x34: [31:0]  Error Source ID (Root Port only)
```

## 硬件实现

### 错误检测逻辑

```verilog
// AER 错误检测模块
module aer_error_detector (
    input wire clk,
    input wire rst_n,
    
    // TLP 接口
    input wire        tlp_valid,
    input wire [127:0] tlp_data,
    input wire        tlp_error,
    
    // DLLP 接口
    input wire        dllp_nak_received,
    input wire        dllp_replay_timeout,
    
    // 物理层接口
    input wire        phy_link_down,
    input wire        phy_receiver_error,
    
    // AER 寄存器
    output reg [31:0] uce_status,    // Uncorrectable Error Status
    output reg [31:0] ce_status,     // Correctable Error Status
    output reg [127:0] header_log,   // TLP Header Log
    output wire       error_interrupt
);

    // 可纠正错误位
    parameter CE_RECEIVER_ERROR     = 0;
    parameter CE_BAD_TLP            = 6;
    parameter CE_BAD_DLLP           = 7;
    parameter CE_REPLAY_NUM_ROLLOVER = 8;
    parameter CE_REPLAY_TIMER_TIMEOUT = 12;
    parameter CE_ADVISORY_NONFATAL  = 13;
    
    // 不可纠正错误位
    parameter UCE_DATA_LINK_PROTOCOL = 4;
    parameter UCE_POISONED_TLP      = 12;
    parameter UCE_FLOW_CONTROL      = 13;
    parameter UCE_COMPLETION_TIMEOUT = 14;
    parameter UCE_COMPLETER_ABORT   = 15;
    parameter UCE_UNEXPECTED_COMPLETION = 16;
    parameter UCE_MALFORMED_TLP     = 18;
    parameter UCE_ECRC_ERROR        = 19;
    parameter UCE_UNSUPPORTED_REQUEST = 20;
    
    // 错误检测逻辑
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            uce_status <= 32'h0;
            ce_status <= 32'h0;
            header_log <= 128'h0;
        end else begin
            // TLP 错误检测
            if (tlp_valid && tlp_error) begin
                case (tlp_error_type)
                    ERR_MALFORMED_TLP:
                        uce_status[UCE_MALFORMED_TLP] <= 1'b1;
                    ERR_ECRC:
                        uce_status[UCE_ECRC_ERROR] <= 1'b1;
                    ERR_POISONED:
                        uce_status[UCE_POISONED_TLP] <= 1'b1;
                endcase
                
                // 记录错误 TLP 头
                header_log <= tlp_data;
            end
            
            // DLLP 错误检测
            if (dllp_replay_timeout) begin
                ce_status[CE_REPLAY_TIMER_TIMEOUT] <= 1'b1;
            end
            
            if (dllp_nak_received) begin
                ce_status[CE_REPLAY_NUM_ROLLOVER] <= 1'b1;
            end
            
            // 物理层错误
            if (phy_receiver_error) begin
                ce_status[CE_RECEIVER_ERROR] <= 1'b1;
            end
            
            if (phy_link_down) begin
                uce_status[UCE_DATA_LINK_PROTOCOL] <= 1'b1;
            end
        end
    end
    
    // 中断生成
    assign error_interrupt = (|uce_status) || (|ce_status);

endmodule
```

### AER 寄存器实现

```verilog
// AER 配置寄存器
module aer_config_regs (
    input wire clk,
    input wire rst_n,
    
    // 配置空间访问
    input wire [11:0]  cfg_addr,
    input wire         cfg_wr,
    input wire [31:0]  cfg_wdata,
    output reg [31:0]  cfg_rdata,
    
    // 错误状态输入
    input wire [31:0]  uce_status_in,
    input wire [31:0]  ce_status_in,
    input wire [127:0] header_log_in,
    
    // 错误掩码输出
    output reg [31:0]  uce_mask,
    output reg [31:0]  ce_mask,
    output reg [31:0]  uce_severity
);

    // AER Capability Offset
    parameter AER_CAP_OFFSET = 12'h100;
    
    // 寄存器
    reg [31:0] uce_status;
    reg [31:0] ce_status;
    reg [127:0] header_log;
    reg [31:0] aer_cap_ctrl;
    
    // 初始化
    initial begin
        uce_mask = 32'h0;
        ce_mask = 32'h0;
        uce_severity = 32'h00462030;  // 默认严重性
        uce_status = 32'h0;
        ce_status = 32'h0;
        header_log = 128'h0;
        aer_cap_ctrl = 32'h0;
    end
    
    // 配置读写
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            cfg_rdata <= 32'h0;
        end else begin
            // 读操作
            case (cfg_addr - AER_CAP_OFFSET)
                12'h00: cfg_rdata <= {12'h000, 4'h2, 16'h0001};  // Header
                12'h04: cfg_rdata <= uce_status;
                12'h08: cfg_rdata <= uce_mask;
                12'h0C: cfg_rdata <= uce_severity;
                12'h10: cfg_rdata <= ce_status;
                12'h14: cfg_rdata <= ce_mask;
                12'h18: cfg_rdata <= aer_cap_ctrl;
                12'h1C: cfg_rdata <= header_log[31:0];
                12'h20: cfg_rdata <= header_log[63:32];
                12'h24: cfg_rdata <= header_log[95:64];
                12'h28: cfg_rdata <= header_log[127:96];
                default: cfg_rdata <= 32'h0;
            endcase
            
            // 写操作
            if (cfg_wr) begin
                case (cfg_addr - AER_CAP_OFFSET)
                    12'h04: uce_status <= uce_status & ~cfg_wdata;  // W1C
                    12'h08: uce_mask <= cfg_wdata;
                    12'h0C: uce_severity <= cfg_wdata;
                    12'h10: ce_status <= ce_status & ~cfg_wdata;   // W1C
                    12'h14: ce_mask <= cfg_wdata;
                    12'h18: aer_cap_ctrl <= cfg_wdata;
                endcase
            end
        end
    end
    
    // 更新错误状态
    always @(posedge clk) begin
        uce_status <= uce_status | (uce_status_in & ~uce_mask);
        ce_status <= ce_status | (ce_status_in & ~ce_mask);
        
        if (|uce_status_in) begin
            header_log <= header_log_in;
        end
    end

endmodule
```

## 软件实现

### 内核 AER 驱动

```c
// drivers/pci/pcie/aer.c
static int aer_probe(struct pcie_device *dev)
{
    struct device *device = &dev->device;
    struct pci_dev *port = dev->port;
    struct aer_rpc *rpc;
    int ret;
    
    // 分配 AER 运行时控制结构
    rpc = devm_kzalloc(device, sizeof(*rpc), GFP_KERNEL);
    if (!rpc)
        return -ENOMEM;
    
    rpc->rpd = port;
    INIT_WORK(&rpc->dpc_handler, aer_isr);
    
    // 设置设备驱动数据
    set_service_data(dev, rpc);
    
    // 注册中断处理程序
    ret = request_irq(dev->irq, aer_irq, IRQF_SHARED,
                      "aerdrv", dev);
    if (ret) {
        dev_err(device, "Failed to request IRQ\n");
        return ret;
    }
    
    // 启用 Root Port 的 AER
    pci_enable_pcie_error_reporting(port);
    
    dev_info(device, "AER enabled\n");
    
    return 0;
}

// AER 中断处理程序
static irqreturn_t aer_irq(int irq, void *context)
{
    struct pcie_device *pdev = context;
    struct aer_rpc *rpc = get_service_data(pdev);
    struct pci_dev *port = rpc->rpd;
    int pos;
    u32 status, source;
    
    // 读取 Root Error Status
    pos = port->aer_cap;
    pci_read_config_dword(port, pos + PCI_ERR_ROOT_STATUS, &status);
    
    if (!(status & (PCI_ERR_ROOT_COR_RCV |
                    PCI_ERR_ROOT_NONFATAL_RCV |
                    PCI_ERR_ROOT_FATAL_RCV))) {
        return IRQ_NONE;
    }
    
    // 读取 Error Source ID
    pci_read_config_dword(port, pos + PCI_ERR_ROOT_ERR_SRC, &source);
    
    // 清除状态位
    pci_write_config_dword(port, pos + PCI_ERR_ROOT_STATUS, status);
    
    // 调度工作队列处理错误
    schedule_work(&rpc->dpc_handler);
    
    return IRQ_HANDLED;
}
```

### 错误处理工作队列

```c
// AER 错误处理
static void aer_isr(struct work_struct *work)
{
    struct aer_rpc *rpc = container_of(work, struct aer_rpc, dpc_handler);
    struct pci_dev *pdev;
    struct aer_err_info e_info;
    
    // 查找错误设备
    pdev = find_source_device(rpc->rpd, source_id);
    if (!pdev) {
        pr_err("AER: Can't find device for source ID %04x\n", source_id);
        return;
    }
    
    // 收集错误信息
    if (collect_error_info(pdev, &e_info) == 0) {
        // 处理错误
        handle_error_source(pdev, &e_info);
        
        // 清除错误状态
        pci_aer_clear_status(pdev);
    }
    
    pci_dev_put(pdev);
}

// 收集错误信息
static int collect_error_info(struct pci_dev *dev,
                                struct aer_err_info *info)
{
    int pos = dev->aer_cap;
    u32 status;
    
    memset(info, 0, sizeof(*info));
    
    // 读取 Uncorrectable Error Status
    pci_read_config_dword(dev, pos + PCI_ERR_UNCOR_STATUS,
                          &info->status);
    
    if (info->status == 0)
        return -EINVAL;
    
    // 读取 Uncorrectable Error Mask
    pci_read_config_dword(dev, pos + PCI_ERR_UNCOR_MASK,
                          &info->mask);
    
    // 读取 Uncorrectable Error Severity
    pci_read_config_dword(dev, pos + PCI_ERR_UNCOR_SEVER,
                          &info->severity);
    
    // 读取 TLP Header Log
    if (info->status & AER_LOG_TLP_MASKS) {
        pci_read_config_dword(dev, pos + PCI_ERR_HEADER_LOG,
                              &info->tlp_header_log.dw0);
        pci_read_config_dword(dev, pos + PCI_ERR_HEADER_LOG + 4,
                              &info->tlp_header_log.dw1);
        pci_read_config_dword(dev, pos + PCI_ERR_HEADER_LOG + 8,
                              &info->tlp_header_log.dw2);
        pci_read_config_dword(dev, pos + PCI_ERR_HEADER_LOG + 12,
                              &info->tlp_header_log.dw3);
    }
    
    // 读取 Correctable Error Status
    pci_read_config_dword(dev, pos + PCI_ERR_COR_STATUS,
                          &info->cor_status);
    
    return 0;
}
```

### 错误恢复

```c
// 错误恢复回调
static pci_ers_result_t aer_error_detected(struct pci_dev *dev,
                                            pci_channel_state_t state)
{
    struct device_driver *drv = dev->driver;
    struct pci_driver *pdrv;
    pci_ers_result_t result;
    
    if (!drv || !drv->pm)
        return PCI_ERS_RESULT_DISCONNECT;
    
    pdrv = to_pci_driver(drv);
    if (!pdrv->err_handler || !pdrv->err_handler->error_detected)
        return PCI_ERS_RESULT_DISCONNECT;
    
    // 调用驱动的错误检测回调
    result = pdrv->err_handler->error_detected(dev, state);
    
    pr_info("AER: Device %s: error_detected result: %d\n",
            pci_name(dev), result);
    
    return result;
}

// 槽位复位
static pci_ers_result_t aer_slot_reset(struct pci_dev *dev)
{
    struct pci_driver *pdrv = to_pci_driver(dev->driver);
    pci_ers_result_t result;
    
    // 复位设备
    pci_reset_function(dev);
    
    // 恢复配置空间
    pci_restore_state(dev);
    pci_save_state(dev);
    
    // 调用驱动的槽位复位回调
    if (pdrv->err_handler && pdrv->err_handler->slot_reset)
        result = pdrv->err_handler->slot_reset(dev);
    else
        result = PCI_ERS_RESULT_RECOVERED;
    
    pr_info("AER: Device %s: slot_reset result: %d\n",
            pci_name(dev), result);
    
    return result;
}

// 恢复完成
static void aer_error_resume(struct pci_dev *dev)
{
    struct pci_driver *pdrv = to_pci_driver(dev->driver);
    
    if (pdrv->err_handler && pdrv->err_handler->resume)
        pdrv->err_handler->resume(dev);
    
    pci_aer_clear_status(dev);
    pci_enable_pcie_error_reporting(dev);
    
    pr_info("AER: Device %s: error recovery completed\n",
            pci_name(dev));
}
```

## FEMU 实现

### AER Capability 添加

```c
// hw/block/nvme.c
static int nvme_add_aer_cap(PCIDevice *pci_dev)
{
    uint8_t *config = pci_dev->config;
    uint8_t *wmask = pci_dev->wmask;
    int offset;
    
    // 添加 AER Extended Capability
    offset = pcie_add_capability(pci_dev, PCI_EXT_CAP_ID_ERR,
                                  PCI_ERR_VER, PCI_ERR_SIZEOF);
    if (offset < 0) {
        return offset;
    }
    
    // 设置 Uncorrectable Error Mask
    pci_set_long(config + offset + PCI_ERR_UNCOR_MASK,
                 PCI_ERR_UNC_DLP | PCI_ERR_UNC_POISON_TLP |
                 PCI_ERR_UNC_FCP | PCI_ERR_UNC_COMP_TIME |
                 PCI_ERR_UNC_COMP_ABORT | PCI_ERR_UNC_UNX_COMP |
                 PCI_ERR_UNC_MALF_TLP | PCI_ERR_UNC_ECRC |
                 PCI_ERR_UNC_UNSUP);
    
    // 设置 Uncorrectable Error Severity
    pci_set_long(config + offset + PCI_ERR_UNCOR_SEVER,
                 PCI_ERR_UNC_DLP | PCI_ERR_UNC_FCP |
                 PCI_ERR_UNC_MALF_TLP | PCI_ERR_UNC_ECRC);
    
    // 设置 Correctable Error Mask
    pci_set_long(config + offset + PCI_ERR_COR_MASK,
                 PCI_ERR_COR_RCVR | PCI_ERR_COR_BAD_TLP |
                 PCI_ERR_COR_BAD_DLLP | PCI_ERR_COR_REP_ROLL |
                 PCI_ERR_COR_REP_TIMER);
    
    // 设置写掩码（W1C 位）
    pci_set_long(wmask + offset + PCI_ERR_UNCOR_STATUS, 0xffffffff);
    pci_set_long(wmask + offset + PCI_ERR_COR_STATUS, 0xffffffff);
    
    return offset;
}
```

### 错误注入

```c
// AER 错误注入（测试用）
void nvme_inject_aer_error(NvmeCtrl *n, uint32_t error_type)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    int pos = pci_dev->exp.aer_cap;
    uint32_t status;
    
    if (!pos) {
        return;
    }
    
    // 读取当前状态
    status = pci_get_long(pci_dev->config + pos + PCI_ERR_UNCOR_STATUS);
    
    // 设置错误位
    status |= error_type;
    pci_set_long(pci_dev->config + pos + PCI_ERR_UNCOR_STATUS, status);
    
    // 触发 AER 中断
    if (pci_dev->exp.aer_cap) {
        pcie_aer_inject_error(pci_dev, error_type);
    }
    
    trace_nvme_aer_error_injected(error_type);
}
```

## 用户空间工具

### aer-inject 工具

```bash
# 安装 aer-inject
apt-get install aer-inject

# 注入可纠正错误
echo 1 > /sys/bus/pci/devices/0000:01:00.0/aer_dev_correctable

# 注入不可纠正非致命错误
echo 1 > /sys/bus/pci/devices/0000:01:00.0/aer_dev_nonfatal

# 注入不可纠正致命错误
echo 1 > /sys/bus/pci/devices/0000:01:00.0/aer_dev_fatal
```

### 错误日志查看

```bash
# 查看 AER 日志
dmesg | grep -i aer

# 查看错误统计
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_correctable
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_nonfatal
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_fatal
```

## 总结

AER 实现的关键点：

1. **硬件检测**：实时监测各层的错误条件
2. **寄存器实现**：正确实现 W1C 语义和掩码逻辑
3. **中断通知**：通过 Root Port 报告错误
4. **错误恢复**：实现 error_detected/slot_reset/resume 回调
5. **日志记录**：保存 TLP Header Log 用于调试
6. **测试验证**：使用错误注入测试恢复路径

参见：
- [AER 详解](../error-handling/aer.md)
- [错误处理流程](../error-handling/recovery.md)
- [驱动开发指南](driver-guide.md)
