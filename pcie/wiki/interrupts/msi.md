# MSI 中断详解

## 概述

MSI (Message Signaled Interrupts) 是 PCIe 1.0 引入的中断机制，通过发送 Memory Write TLP 来触发中断，取代了传统的物理中断线。MSI 提供了比 INTx 更高的性能，并且是 PCIe 设备的必需特性。

## MSI 架构

### 基本原理

```
传统 INTx:
设备 --[物理中断线]--> 中断控制器 --> CPU

MSI:
设备 --[Memory Write TLP]--> 系统内存地址 --> 中断控制器 --> CPU

优势：
- 无需物理中断线
- 无需中断共享
- 更低延迟
- 支持多个向量（最多32个）
```

### MSI vs MSI-X 对比

| 特性 | MSI | MSI-X |
|------|-----|-------|
| 最大向量数 | 32 | 2048 |
| 向量数限制 | 2的幂次 (1,2,4,8,16,32) | 任意数量 |
| 地址/数据对 | 所有向量共享1对 | 每个向量独立 |
| 掩码支持 | 可选（Per-Vector Masking） | 必需 |
| 表结构 | 在配置空间中 | 在 BAR 空间中 |
| 复杂度 | 简单 | 复杂 |
| PCIe 要求 | 必需 (Gen 1+) | 推荐 (Gen 2+) |

## MSI Capability 结构

### 基本格式

```
MSI Capability (无 Per-Vector Masking, 32-bit地址):
+0x00: [7:0]   Capability ID = 0x05
       [15:8]  Next Capability Pointer
+0x02: [15:0]  Message Control
       [0]     MSI Enable
       [3:1]   Multiple Message Capable (MMC)
       [6:4]   Multiple Message Enable (MME)
       [7]     64-bit Address Capable
       [8]     Per-Vector Masking Capable
       [15:9]  Reserved
+0x04: [31:0]  Message Address (32-bit)
+0x08: [15:0]  Message Data
+0x0A: [15:0]  Reserved

总大小：10 字节
```

### 64-bit 地址格式

```
MSI Capability (64-bit地址):
+0x00: [15:0]  Capability Header
+0x02: [15:0]  Message Control
+0x04: [31:0]  Message Address Lower 32 bits
+0x08: [31:0]  Message Address Upper 32 bits
+0x0C: [15:0]  Message Data
+0x0E: [15:0]  Reserved

总大小：14 字节
```

### Per-Vector Masking

```
MSI Capability (带 Masking, 32-bit地址):
+0x00-0x08: 同上
+0x0C: [31:0]  Mask Bits
       每位对应一个向量，1=masked, 0=unmasked
+0x10: [31:0]  Pending Bits
       每位指示该向量是否有pending中断

MSI Capability (带 Masking, 64-bit地址):
+0x00-0x0C: 同上
+0x10: [31:0]  Mask Bits
+0x14: [31:0]  Pending Bits

总大小：24 字节
```

### Message Control 寄存器

```
Bit [0]: MSI Enable
  0 = 禁用 MSI
  1 = 启用 MSI

Bits [3:1]: Multiple Message Capable (MMC)
  000 = 1 vector
  001 = 2 vectors
  010 = 4 vectors
  011 = 8 vectors
  100 = 16 vectors
  101 = 32 vectors
  (只读，设备能力)

Bits [6:4]: Multiple Message Enable (MME)
  000 = 1 vector
  001 = 2 vectors
  010 = 4 vectors
  011 = 8 vectors
  100 = 16 vectors
  101 = 32 vectors
  (软件配置，≤ MMC)

Bit [7]: 64-bit Address Capable
  0 = 只支持32-bit地址
  1 = 支持64-bit地址
  (只读)

Bit [8]: Per-Vector Masking Capable
  0 = 不支持Per-Vector Masking
  1 = 支持Per-Vector Masking
  (只读)
```

## Message 地址和数据格式

### x86/x64 架构

```
Message Address:
+---+---+---+---+---+---+---+---+---+---+---+---+
|31           20|19           12|11  4|3 |2 |1|0|
+---+---+---+---+---+---+---+---+---+---+---+---+
|    0xFEE      | Destination ID| RH |DM|xx|0|
+---+---+---+---+---+---+---+---+---+---+---+---+

固定前缀：0xFEE (APIC base address)
Bits [19:12]: Destination ID (目标 APIC ID)
Bit [3]: Redirection Hint
  0 = 直接发送到指定的处理器
  1 = 发送到最低优先级的处理器
Bit [2]: Destination Mode
  0 = Physical (物理 APIC ID)
  1 = Logical (逻辑 APIC ID)
Bits [1:0]: 保留，必须为 0

Message Data:
+---+---+---+---+---+---+---+---+---+---+---+---+
|31  16|15|14|13|12|11 10|9  8|7            0|
+---+---+---+---+---+---+---+---+---+---+---+---+
| Rsvd | L| TM|Rsvd| DM  |Rsvd|   Vector     |
+---+---+---+---+---+---+---+---+---+---+---+---+

Bits [7:0]: Interrupt Vector (0-255)
Bits [10:8]: Delivery Mode
  000 = Fixed
  001 = Lowest Priority
  010 = SMI
  011 = Reserved
  100 = NMI
  101 = INIT
  110 = Reserved
  111 = ExtINT
Bit [14]: Trigger Mode
  0 = Edge
  1 = Level (MSI 通常使用 Edge)
Bit [15]: Level
  0 = Deassert
  1 = Assert

对于多向量 MSI，向量计算：
Vector = Base_Vector + Interrupt_Offset

其中 Interrupt_Offset 根据触发的中断源确定
```

## 软件配置

### 查询和启用 MSI

```c
// 查询 MSI 能力
static int query_msi_capability(struct pci_dev *pdev)
{
    int pos;
    u16 control;
    int nvec;
    
    // 查找 MSI Capability
    pos = pci_find_capability(pdev, PCI_CAP_ID_MSI);
    if (!pos) {
        dev_err(&pdev->dev, "MSI not supported\n");
        return -ENODEV;
    }
    
    // 读取 Message Control
    pci_read_config_word(pdev, pos + PCI_MSI_FLAGS, &control);
    
    // 检查是否支持 64-bit 地址
    bool msi_64bit = !!(control & PCI_MSI_FLAGS_64BIT);
    
    // 检查是否支持 Per-Vector Masking
    bool msi_maskable = !!(control & PCI_MSI_FLAGS_MASKBIT);
    
    // 读取设备支持的向量数
    nvec = 1 << ((control & PCI_MSI_FLAGS_QSIZE) >> 1);
    
    dev_info(&pdev->dev, "MSI: %d vectors, %s, %s\n",
             nvec,
             msi_64bit ? "64-bit" : "32-bit",
             msi_maskable ? "maskable" : "non-maskable");
    
    return nvec;
}

// 配置 MSI
static int setup_msi(struct pci_dev *pdev, int nvec)
{
    int pos, ret;
    u16 control;
    u32 addr_lo, addr_hi = 0;
    u16 data;
    
    pos = pci_find_capability(pdev, PCI_CAP_ID_MSI);
    if (!pos)
        return -EINVAL;
    
    // 读取 Message Control
    pci_read_config_word(pdev, pos + PCI_MSI_FLAGS, &control);
    
    // 计算 MME (Multiple Message Enable)
    int mme = ilog2(nvec);  // 0=1vec, 1=2vec, 2=4vec, ...
    
    // 验证请求的向量数不超过设备能力
    int mmc = (control & PCI_MSI_FLAGS_QSIZE) >> 1;
    if (mme > mmc) {
        dev_warn(&pdev->dev, "Requested %d vectors but device supports %d\n",
                 nvec, 1 << mmc);
        mme = mmc;
        nvec = 1 << mme;
    }
    
    // 分配中断向量
    ret = pci_alloc_irq_vectors_affinity(pdev, nvec, nvec,
                                          PCI_IRQ_MSI, NULL);
    if (ret < 0) {
        dev_err(&pdev->dev, "Failed to allocate MSI vectors: %d\n", ret);
        return ret;
    }
    
    // 获取 Message Address 和 Data
    // (内核 API 会自动设置，这里展示底层操作)
    struct msi_desc *desc = first_pci_msi_entry(pdev);
    struct msi_msg msg;
    
    get_cached_msi_msg(desc->irq, &msg);
    addr_lo = msg.address_lo;
    addr_hi = msg.address_hi;
    data = msg.data;
    
    // 写入 Message Address
    bool msi_64bit = !!(control & PCI_MSI_FLAGS_64BIT);
    pci_write_config_dword(pdev, pos + PCI_MSI_ADDRESS_LO, addr_lo);
    
    if (msi_64bit) {
        pci_write_config_dword(pdev, pos + PCI_MSI_ADDRESS_HI, addr_hi);
        pci_write_config_word(pdev, pos + PCI_MSI_DATA_64, data);
    } else {
        pci_write_config_word(pdev, pos + PCI_MSI_DATA_32, data);
    }
    
    // 设置 MME 并启用 MSI
    control &= ~PCI_MSI_FLAGS_QMASK;
    control |= (mme << 4);
    control |= PCI_MSI_FLAGS_ENABLE;
    
    pci_write_config_word(pdev, pos + PCI_MSI_FLAGS, control);
    
    dev_info(&pdev->dev, "MSI enabled with %d vectors\n", nvec);
    return nvec;
}
```

### 使用内核 API

```c
// 现代内核推荐的方式
static int enable_msi_modern(struct pci_dev *pdev, int min_vecs, int max_vecs)
{
    int nvec;
    
    // 请求 MSI 向量（会自动fallback到可用数量）
    nvec = pci_alloc_irq_vectors(pdev, min_vecs, max_vecs, PCI_IRQ_MSI);
    if (nvec < 0) {
        dev_err(&pdev->dev, "Failed to enable MSI: %d\n", nvec);
        return nvec;
    }
    
    dev_info(&pdev->dev, "Allocated %d MSI vectors\n", nvec);
    
    // 注册中断处理程序
    for (int i = 0; i < nvec; i++) {
        int irq = pci_irq_vector(pdev, i);
        int ret = request_irq(irq, my_msi_handler, 0,
                              "mydriver-msi", pdev);
        if (ret) {
            dev_err(&pdev->dev, "Failed to request IRQ %d\n", irq);
            goto free_irqs;
        }
    }
    
    return nvec;

free_irqs:
    while (--i >= 0)
        free_irq(pci_irq_vector(pdev, i), pdev);
    pci_free_irq_vectors(pdev);
    return -EINVAL;
}

// 禁用 MSI
static void disable_msi(struct pci_dev *pdev, int nvec)
{
    // 释放所有中断
    for (int i = 0; i < nvec; i++) {
        free_irq(pci_irq_vector(pdev, i), pdev);
    }
    
    // 释放向量
    pci_free_irq_vectors(pdev);
}
```

## 多向量处理

### 向量分配策略

```c
// 根据队列数量分配向量
static int allocate_msi_vectors(struct pci_dev *pdev, int num_queues)
{
    int nvec;
    
    // 尝试为每个队列分配一个向量
    nvec = pci_alloc_irq_vectors(pdev, 1, num_queues, PCI_IRQ_MSI);
    if (nvec < 0) {
        dev_err(&pdev->dev, "MSI allocation failed\n");
        return nvec;
    }
    
    if (nvec < num_queues) {
        dev_warn(&pdev->dev,
                 "Only allocated %d MSI vectors for %d queues\n",
                 nvec, num_queues);
        // 多个队列共享向量
    }
    
    return nvec;
}

// 向量到队列的映射
static int map_vector_to_queue(int vector, int nvec, int queue, int num_queues)
{
    // 简单轮询映射
    if (nvec >= num_queues) {
        // 一对一映射
        return queue;
    } else {
        // 多个队列共享向量
        return queue % nvec;
    }
}
```

### 中断处理

```c
// MSI 中断处理程序
static irqreturn_t my_msi_handler(int irq, void *data)
{
    struct pci_dev *pdev = data;
    struct my_device *mydev = pci_get_drvdata(pdev);
    int vector;
    
    // 确定是哪个向量
    vector = irq - pci_irq_vector(pdev, 0);
    
    // 读取设备状态寄存器
    u32 status = ioread32(mydev->bar0 + INT_STATUS_REG);
    
    // 检查是否是该向量的中断
    if (!(status & (1 << vector))) {
        return IRQ_NONE;
    }
    
    // 清除中断状态
    iowrite32(1 << vector, mydev->bar0 + INT_CLEAR_REG);
    
    // 处理中断
    handle_interrupt(mydev, vector);
    
    return IRQ_HANDLED;
}
```

## Per-Vector Masking

### 掩码操作

```c
// 掩码指定的向量
static void msi_mask_vector(struct pci_dev *pdev, int vector)
{
    int pos;
    u32 mask;
    u16 control;
    
    pos = pdev->msi_cap;
    if (!pos)
        return;
    
    // 检查是否支持 masking
    pci_read_config_word(pdev, pos + PCI_MSI_FLAGS, &control);
    if (!(control & PCI_MSI_FLAGS_MASKBIT)) {
        dev_warn(&pdev->dev, "MSI Per-Vector Masking not supported\n");
        return;
    }
    
    // 计算 Mask Bits 偏移
    bool msi_64bit = !!(control & PCI_MSI_FLAGS_64BIT);
    int mask_offset = msi_64bit ? PCI_MSI_MASK_64 : PCI_MSI_MASK_32;
    
    // 读取当前 Mask Bits
    pci_read_config_dword(pdev, pos + mask_offset, &mask);
    
    // 设置对应位
    mask |= (1 << vector);
    
    pci_write_config_dword(pdev, pos + mask_offset, mask);
}

// 解除掩码
static void msi_unmask_vector(struct pci_dev *pdev, int vector)
{
    int pos;
    u32 mask;
    u16 control;
    
    pos = pdev->msi_cap;
    if (!pos)
        return;
    
    pci_read_config_word(pdev, pos + PCI_MSI_FLAGS, &control);
    if (!(control & PCI_MSI_FLAGS_MASKBIT))
        return;
    
    bool msi_64bit = !!(control & PCI_MSI_FLAGS_64BIT);
    int mask_offset = msi_64bit ? PCI_MSI_MASK_64 : PCI_MSI_MASK_32;
    
    pci_read_config_dword(pdev, pos + mask_offset, &mask);
    
    // 清除对应位
    mask &= ~(1 << vector);
    
    pci_write_config_dword(pdev, pos + mask_offset, mask);
}

// 检查 Pending 位
static bool msi_is_pending(struct pci_dev *pdev, int vector)
{
    int pos;
    u32 pending;
    u16 control;
    
    pos = pdev->msi_cap;
    if (!pos)
        return false;
    
    pci_read_config_word(pdev, pos + PCI_MSI_FLAGS, &control);
    if (!(control & PCI_MSI_FLAGS_MASKBIT))
        return false;
    
    bool msi_64bit = !!(control & PCI_MSI_FLAGS_64BIT);
    int pending_offset = msi_64bit ? PCI_MSI_PENDING_64 : PCI_MSI_PENDING_32;
    
    pci_read_config_dword(pdev, pos + pending_offset, &pending);
    
    return !!(pending & (1 << vector));
}
```

## 硬件实现

### Message 生成

```verilog
// MSI Message 生成模块
module msi_generator (
    input wire clk,
    input wire rst_n,
    
    // MSI 配置
    input wire [31:0] msi_addr_lo,
    input wire [31:0] msi_addr_hi,
    input wire [15:0] msi_data,
    input wire msi_enabled,
    input wire msi_64bit,
    input wire [2:0] mme,  // Multiple Message Enable
    
    // 中断请求
    input wire [31:0] irq_vector,  // 32个中断源
    
    // Masking
    input wire [31:0] mask_bits,
    output reg [31:0] pending_bits,
    
    // TLP 输出
    output reg tlp_valid,
    output reg [127:0] tlp_data
);

    reg [4:0] vector_to_send;
    reg [15:0] modified_data;
    
    // 检测中断边沿
    reg [31:0] irq_prev;
    wire [31:0] irq_edge = irq_vector & ~irq_prev;
    
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            irq_prev <= 32'h0;
            pending_bits <= 32'h0;
            tlp_valid <= 1'b0;
        end else begin
            irq_prev <= irq_vector;
            
            // 记录 pending 中断
            pending_bits <= (pending_bits | irq_edge) & ~mask_bits;
            
            // 如果 MSI 启用且有未 masked 的中断
            if (msi_enabled && (|pending_bits)) begin
                // 找到最高优先级的pending中断
                vector_to_send = find_first_set(pending_bits);
                
                // 修改 Message Data 的向量字段
                modified_data = msi_data;
                modified_data[4:0] = modified_data[4:0] + vector_to_send;
                
                // 构造 Memory Write TLP
                if (msi_64bit) begin
                    // 4DW Header
                    tlp_data[31:0] = {8'h60, 8'h00, 16'h0001};  // Fmt/Type, Length=1DW
                    tlp_data[63:32] = {16'h0000, 16'h0000};     // Requester ID, Tag
                    tlp_data[95:64] = msi_addr_hi;              // Address Upper
                    tlp_data[127:96] = msi_addr_lo;             // Address Lower
                end else begin
                    // 3DW Header
                    tlp_data[31:0] = {8'h40, 8'h00, 16'h0001};  // Fmt/Type, Length=1DW
                    tlp_data[63:32] = {16'h0000, 16'h0000};     // Requester ID, Tag
                    tlp_data[95:64] = msi_addr_lo;              // Address
                    tlp_data[127:96] = {modified_data, 16'h0}; // Data
                end
                
                tlp_valid <= 1'b1;
                
                // 清除已发送的pending位
                pending_bits[vector_to_send] <= 1'b0;
            end else begin
                tlp_valid <= 1'b0;
            end
        end
    end
    
    // 找到第一个置位的bit
    function [4:0] find_first_set;
        input [31:0] bits;
        integer i;
        begin
            find_first_set = 0;
            for (i = 0; i < 32; i = i + 1) begin
                if (bits[i]) begin
                    find_first_set = i;
                    i = 32;  // 退出循环
                end
            end
        end
    endfunction

endmodule
```

## 调试技巧

### 查看 MSI 配置

```bash
# lspci 显示 MSI 配置
lspci -vvv -s 01:00.0 | grep -A 10 "MSI:"

# 示例输出：
# Capabilities: [50] MSI: Enable+ Count=8/32 Maskable+ 64bit+
#   Address: 00000000fee00000  Data: 4022
#   Masking: 00000000  Pending: 00000000

# 使用 setpci 读取
setpci -s 01:00.0 CAP_MSI+2.W   # Message Control
setpci -s 01:00.0 CAP_MSI+4.L   # Message Address Lo
setpci -s 01:00.0 CAP_MSI+8.L   # Message Address Hi (64-bit)
setpci -s 01:00.0 CAP_MSI+C.W   # Message Data (64-bit)
```

### 内核日志

```bash
# 查看 MSI 相关日志
dmesg | grep -i msi

# 查看中断分配
cat /proc/interrupts | grep mydriver
```

## 最佳实践

1. **优先使用 MSI-X**：如果设备支持，MSI-X 更灵活
2. **请求合适数量**：不要请求超过实际需要的向量
3. **设置亲和性**：将向量绑定到特定 CPU 提高性能
4. **错误处理**：检查所有分配和配置的返回值
5. **清理资源**：卸载时正确释放向量和中断

## 总结

MSI 的关键点：

1. **Message 机制**：通过 Memory Write TLP 触发中断
2. **多向量支持**：最多 32 个向量，必须是 2 的幂次
3. **配置简单**：在配置空间中设置地址和数据
4. **Per-Vector Masking**：可选的向量级掩码支持
5. **性能优势**：比 INTx 更低延迟，无共享

参见：
- [中断概述](overview.md)
- [MSI-X 详解](msix.md)
- [Legacy INTx](intx.md)
- [中断实现](../implementation/interrupts.md)
