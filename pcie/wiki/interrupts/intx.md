# Legacy INTx 中断

## 概述

Legacy INTx 是 PCI/PCIe 的传统中断机制，使用物理中断线（INTA#、INTB#、INTC#、INTD#）。在 PCIe 中，INTx 通过消息（Message）模拟实现，保持向后兼容性。

## INTx 架构

### 传统 PCI 中的 INTx

```
传统 PCI:
设备 --[INTA#物理线]--> 中断控制器 --> CPU

特点：
- 4 根物理中断线
- 低电平有效
- 多设备共享同一中断线
- 需要中断路由和仲裁
```

### PCIe 中的 INTx 模拟

```
PCIe INTx Emulation:
设备 --[Assert_INTx Message]--> Root Complex --> 中断控制器 --> CPU

消息类型：
- Assert_INTA / Assert_INTB / Assert_INTC / Assert_INTD
- Deassert_INTA / Deassert_INTB / Deassert_INTC / Deassert_INTD

优势：
- 无需物理中断线
- 保持 PCI 兼容性
- 简化硬件设计

劣势：
- 性能低于 MSI/MSI-X
- 中断共享问题
- 中断延迟较高
```

## INTx 消息格式

### Message TLP 结构

```
INTx Message TLP (无数据):
+--------+--------+--------+--------+
| Header | Header | Header |  -     |
| DW0    | DW1    | DW2    |        |
+--------+--------+--------+--------+

DW0:
Fmt = 000 (3DW, No Data)
Type = 10100 (Message, routed by ID)

DW1:
Requester ID (Bus:Dev.Func)

DW2:
Message Code:
  0x20 = Assert_INTA
  0x21 = Assert_INTB
  0x22 = Assert_INTC
  0x23 = Assert_INTD
  0x24 = Deassert_INTA
  0x25 = Deassert_INTB
  0x26 = Deassert_INTC
  0x27 = Deassert_INTD
```

### 发送 INTx 消息

```c
// 设备发送 Assert_INTA 消息
void send_assert_inta(struct pcie_device *dev)
{
    struct tlp_header hdr = {
        .fmt = 0x0,              // 3DW, No Data
        .type = 0x14,            // Message
        .length = 0,
        .requester_id = dev->bdf,
        .tag = get_next_tag(dev),
        .message_code = 0x20,    // Assert_INTA
    };
    
    send_message_tlp(dev, &hdr);
    
    dev->intx_asserted = true;
}

// 发送 Deassert_INTA 消息
void send_deassert_inta(struct pcie_device *dev)
{
    struct tlp_header hdr = {
        .fmt = 0x0,
        .type = 0x14,
        .length = 0,
        .requester_id = dev->bdf,
        .tag = get_next_tag(dev),
        .message_code = 0x24,    // Deassert_INTA
    };
    
    send_message_tlp(dev, &hdr);
    
    dev->intx_asserted = false;
}
```

## 中断引脚分配

### Interrupt Pin 寄存器

```
配置空间 +0x3D: Interrupt Pin

值：
0x00 = 不使用中断
0x01 = INTA#
0x02 = INTB#
0x03 = INTC#
0x04 = INTD#

示例：
设备 A: Interrupt Pin = 0x01 (INTA#)
设备 B: Interrupt Pin = 0x02 (INTB#)
设备 C: Interrupt Pin = 0x01 (INTA#) - 与设备 A 共享
```

### 多功能设备的引脚轮换

```
PCI 规范建议的引脚分配：

Single Function Device:
Function 0 --> INTA#

Multi-Function Device:
Function 0 --> INTA#
Function 1 --> INTB#
Function 2 --> INTC#
Function 3 --> INTD#
Function 4 --> INTA# (循环)
Function 5 --> INTB#
Function 6 --> INTC#
Function 7 --> INTD#

目的：分散中断负载，减少共享
```

## 中断共享机制

### 共享中断处理

```c
// 注册共享中断
static int setup_intx(struct pci_dev *pdev, struct my_device *mydev)
{
    int err;
    u8 irq_pin;
    
    // 读取 Interrupt Pin
    pci_read_config_byte(pdev, PCI_INTERRUPT_PIN, &irq_pin);
    
    if (irq_pin == 0) {
        dev_info(&pdev->dev, "Device does not use interrupts\n");
        return 0;
    }
    
    dev_info(&pdev->dev, "Using INT%c# (IRQ %d)\n",
             'A' + irq_pin - 1, pdev->irq);
    
    // 启用 INTx
    pci_intx(pdev, 1);
    
    // 注册中断处理程序（必须指定 IRQF_SHARED）
    err = request_irq(pdev->irq, my_intx_handler,
                      IRQF_SHARED,  // 共享标志
                      "mydriver-intx", mydev);
    if (err) {
        dev_err(&pdev->dev, "Failed to request IRQ: %d\n", err);
        return err;
    }
    
    return 0;
}

// INTx 中断处理程序
static irqreturn_t my_intx_handler(int irq, void *data)
{
    struct my_device *mydev = data;
    u32 status;
    
    // 读取设备中断状态寄存器
    status = ioread32(mydev->bar0 + INT_STATUS_REG);
    
    // 检查是否是我们的中断（关键！）
    if (!(status & INT_PENDING)) {
        // 不是我们的中断，返回 IRQ_NONE
        return IRQ_NONE;
    }
    
    // 清除中断状态（写 1 清除）
    iowrite32(status, mydev->bar0 + INT_STATUS_REG);
    
    // 处理中断
    handle_device_interrupt(mydev, status);
    
    return IRQ_HANDLED;
}

// 禁用 INTx
static void disable_intx(struct pci_dev *pdev, struct my_device *mydev)
{
    free_irq(pdev->irq, mydev);
    pci_intx(pdev, 0);
}
```

### 中断确认的重要性

```
共享中断场景：

IRQ 11 被 3 个设备共享：
- 设备 A (NIC)
- 设备 B (Sound Card)
- 设备 C (USB Controller)

中断触发：
1. 设备 A 产生中断
2. 内核调用所有注册的处理程序
   - 设备 A 的处理程序：检查状态 --> 是我的 --> IRQ_HANDLED
   - 设备 B 的处理程序：检查状态 --> 不是我的 --> IRQ_NONE
   - 设备 C 的处理程序：检查状态 --> 不是我的 --> IRQ_NONE
3. 至少一个返回 IRQ_HANDLED，中断被正确处理

关键点：
- 必须检查设备状态寄存器
- 不能假设中断一定是自己的
- 错误地返回 IRQ_HANDLED 会导致其他设备中断丢失
- 错误地返回 IRQ_NONE 会导致内核报告 "nobody cared"
```

## Interrupt Disable 位

### Command 寄存器

```
配置空间 +0x04: Command Register

Bit 10: Interrupt Disable
  0 = INTx 启用（默认）
  1 = INTx 禁用

用途：
- 软件可以临时禁用设备的 INTx 中断
- 切换到 MSI/MSI-X 时应禁用 INTx
- 调试时可以选择性禁用中断

代码示例：
// 禁用 INTx
u16 command;
pci_read_config_word(pdev, PCI_COMMAND, &command);
command |= PCI_COMMAND_INTX_DISABLE;
pci_write_config_word(pdev, PCI_COMMAND, command);

// 启用 INTx
pci_read_config_word(pdev, PCI_COMMAND, &command);
command &= ~PCI_COMMAND_INTX_DISABLE;
pci_write_config_word(pdev, PCI_COMMAND, command);
```

### pci_intx() 内核函数

```c
// 内核提供的便利函数
void pci_intx(struct pci_dev *pdev, int enable)
{
    u16 pci_command, new;
    
    pci_read_config_word(pdev, PCI_COMMAND, &pci_command);
    
    if (enable)
        new = pci_command & ~PCI_COMMAND_INTX_DISABLE;
    else
        new = pci_command | PCI_COMMAND_INTX_DISABLE;
    
    if (new != pci_command) {
        pci_write_config_word(pdev, PCI_COMMAND, new);
    }
}
```

## INTx 与 MSI 互斥

### 切换到 MSI

```c
// 优先使用 MSI，回退到 INTx
static int setup_interrupts(struct pci_dev *pdev, struct my_device *mydev)
{
    int err;
    
    // 尝试启用 MSI
    err = pci_enable_msi(pdev);
    if (err == 0) {
        dev_info(&pdev->dev, "Using MSI\n");
        
        // 禁用 INTx（重要！）
        pci_intx(pdev, 0);
        
        // 注册 MSI 中断
        err = request_irq(pdev->irq, my_msi_handler, 0,
                          "mydriver-msi", mydev);
        if (err) {
            pci_disable_msi(pdev);
            goto try_intx;
        }
        
        mydev->using_msi = true;
        return 0;
    }
    
try_intx:
    dev_info(&pdev->dev, "MSI not available, using INTx\n");
    
    // 启用 INTx
    pci_intx(pdev, 1);
    
    // 注册 INTx 中断
    err = request_irq(pdev->irq, my_intx_handler,
                      IRQF_SHARED, "mydriver-intx", mydev);
    if (err) {
        dev_err(&pdev->dev, "Failed to request IRQ\n");
        return err;
    }
    
    mydev->using_msi = false;
    return 0;
}
```

## 硬件实现

### INTx 消息生成

```verilog
// INTx 消息生成器
module intx_msg_generator (
    input wire clk,
    input wire rst_n,
    
    // 中断请求
    input wire irq_request,
    input wire irq_clear,
    
    // 配置
    input wire [1:0] int_pin,      // 00=A, 01=B, 10=C, 11=D
    input wire int_disable,        // Command[10]
    input wire [15:0] requester_id,
    
    // TLP 输出
    output reg msg_valid,
    output reg [95:0] msg_data
);

    reg intx_asserted;
    
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            intx_asserted <= 1'b0;
            msg_valid <= 1'b0;
        end else begin
            msg_valid <= 1'b0;
            
            if (!int_disable) begin
                // Assert
                if (irq_request && !intx_asserted) begin
                    intx_asserted <= 1'b1;
                    
                    // 构造 Assert_INTx Message
                    msg_data[31:0] <= {8'h14, 24'h000000};  // Fmt/Type
                    msg_data[63:32] <= {requester_id, 16'h0000};
                    msg_data[95:64] <= {8'h20 + int_pin, 24'h000000};  // Message Code
                    
                    msg_valid <= 1'b1;
                end
                
                // Deassert
                if (irq_clear && intx_asserted) begin
                    intx_asserted <= 1'b0;
                    
                    // 构造 Deassert_INTx Message
                    msg_data[31:0] <= {8'h14, 24'h000000};
                    msg_data[63:32] <= {requester_id, 16'h0000};
                    msg_data[95:64] <= {8'h24 + int_pin, 24'h000000};
                    
                    msg_valid <= 1'b1;
                end
            end
        end
    end

endmodule
```

### Root Complex 处理

```c
// Root Complex 接收 INTx 消息
void rc_handle_intx_message(struct root_complex *rc, struct tlp *tlp)
{
    u8 message_code = tlp->header.message_code;
    u16 requester_id = tlp->header.requester_id;
    int irq_line;
    
    // 根据 Message Code 确定中断引脚
    int pin = (message_code & 0x03);  // 0=A, 1=B, 2=C, 3=D
    bool assert = (message_code < 0x24);
    
    // 查找中断路由表
    irq_line = lookup_irq_routing(rc, requester_id, pin);
    
    if (assert) {
        // Assert 中断
        trigger_interrupt(rc->pic, irq_line);
    } else {
        // Deassert 中断
        clear_interrupt(rc->pic, irq_line);
    }
}
```

## 性能考虑

### INTx 延迟

```
INTx 中断延迟组成：

1. 设备生成中断请求
2. 构造 Assert_INTx Message TLP
3. TLP 通过 PCIe 链路传输
4. Root Complex 接收并解析
5. 触发中断控制器
6. CPU 响应中断
7. 内核调用中断处理程序
8. 所有共享该 IRQ 的处理程序依次执行

典型延迟：5-20 μs

对比：
- MSI: 2-5 μs
- MSI-X: 1-3 μs

原因：
- 消息传输开销
- 中断共享导致的额外处理
- 中断确认机制
```

### 中断风暴

```
中断风暴场景：

设备持续产生中断，但驱动未正确清除中断状态
    ↓
中断处理程序被反复调用
    ↓
CPU 负载 100%，系统响应缓慢

检测：
cat /proc/interrupts
看到某个 IRQ 的计数快速增长

解决方法：
1. 修复驱动，正确清除中断状态
2. 临时禁用该中断：
   echo 0 > /proc/irq/<irq>/smp_affinity
3. 使用 irqbalance 分散中断负载
```

## 调试技巧

### 查看 INTx 配置

```bash
# lspci 显示中断信息
lspci -vv -s 01:00.0 | grep "Interrupt:"

# 示例输出：
# Interrupt: pin A routed to IRQ 16

# 查看中断统计
cat /proc/interrupts
# 16:   1234567   IO-APIC  16-fasteoi  mydriver

# 查看共享该 IRQ 的设备
grep " 16:" /proc/interrupts
```

### 内核日志

```bash
# 查看中断相关日志
dmesg | grep -i "irq\|interrupt"

# 共享中断问题
# irq 16: nobody cared (try booting with the "irqpoll" option)
```

### /proc/interrupts 分析

```bash
# 实时监控中断
watch -n 1 'cat /proc/interrupts'

# 分析中断分布
cat /proc/interrupts | awk '{print $1, $2}'
```

## 最佳实践

### 1. 总是检查中断状态

```c
// 错误做法
static irqreturn_t bad_handler(int irq, void *data)
{
    // 假设中断一定是我们的
    handle_interrupt();
    return IRQ_HANDLED;  // 危险！
}

// 正确做法
static irqreturn_t good_handler(int irq, void *data)
{
    u32 status = read_interrupt_status();
    
    if (!(status & MY_INTERRUPT_BIT))
        return IRQ_NONE;  // 不是我们的中断
    
    clear_interrupt_status(status);
    handle_interrupt();
    return IRQ_HANDLED;
}
```

### 2. 优先使用 MSI/MSI-X

```c
// 优先级: MSI-X > MSI > INTx
if (pci_enable_msix_range(pdev, ...) > 0) {
    use_msix();
} else if (pci_enable_msi(pdev) == 0) {
    use_msi();
} else {
    use_intx();
}
```

### 3. 正确清除中断

```c
// 确保清除中断状态
void handle_interrupt(struct my_device *dev)
{
    u32 status = ioread32(dev->bar0 + INT_STATUS);
    
    // 处理中断
    process_interrupt(dev, status);
    
    // 清除状态（关键！）
    iowrite32(status, dev->bar0 + INT_STATUS);  // Write-1-to-Clear
}
```

### 4. 使用 IRQF_SHARED

```c
// INTx 必须使用 IRQF_SHARED
request_irq(irq, handler, IRQF_SHARED, name, dev_id);
```

## 总结

Legacy INTx 的关键点：

1. **消息模拟**：PCIe 通过 Message 模拟物理中断线
2. **中断共享**：多设备共享 IRQ，必须检查状态
3. **性能较低**：比 MSI/MSI-X 延迟高
4. **向后兼容**：支持传统软件和设备
5. **互斥关系**：与 MSI/MSI-X 互斥，需要正确切换

参见：
- [中断概述](overview.md)
- [MSI 详解](msi.md)
- [MSI-X 详解](msix.md)
- [中断路由](routing.md)
