# PCIe 错误注入

## 概述

错误注入（Error Injection）是一种测试技术，通过人为引入各种错误场景来验证系统的错误处理和恢复机制。在 PCIe 系统中，错误注入对于验证 AER（Advanced Error Reporting）、错误恢复流程和系统鲁棒性至关重要。

## 为什么需要错误注入

```
┌─────────────────────────────────────┐
│     错误注入的价值                   │
├─────────────────────────────────────┤
│                                     │
│  验证错误处理                        │
│  ├─ 确认 AER 正确工作               │
│  ├─ 测试驱动程序错误恢复            │
│  └─ 验证系统级恢复机制              │
│                                     │
│  提高系统可靠性                      │
│  ├─ 发现隐藏的错误处理缺陷          │
│  ├─ 验证边界条件                    │
│  └─ 测试极端场景                    │
│                                     │
│  降低现场故障风险                    │
│  ├─ 在实验室重现实际错误            │
│  ├─ 验证错误恢复时间                │
│  └─ 测试故障隔离能力                │
│                                     │
└─────────────────────────────────────┘
```

### 应用场景

- **驱动开发**：验证驱动程序的错误处理代码路径
- **系统集成测试**：确保多个组件协同处理错误
- **可靠性验证**：测试系统在各种故障场景下的表现
- **合规性测试**：验证是否符合 PCIe 规范要求

## 错误注入方法

### 1. 硬件错误注入

#### a. 通过 AER 能力寄存器

```
配置空间 AER Extended Capability
├─ Uncorrectable Error Status Register (0x04)
├─ Uncorrectable Error Mask Register (0x08)
├─ Uncorrectable Error Severity Register (0x0C)
├─ Correctable Error Status Register (0x10)
├─ Correctable Error Mask Register (0x14)
└─ Advanced Error Capabilities and Control Register (0x18)
    └─ ECRC Generation Enable (位 6)
```

**示例：注入 ECRC 错误**

```c
// 1. 启用 ECRC 生成
uint32_t aer_cap_ctrl;
pci_read_config_dword(dev, aer_cap + 0x18, &aer_cap_ctrl);
aer_cap_ctrl |= (1 << 6);  // ECRC Generation Enable
pci_write_config_dword(dev, aer_cap + 0x18, aer_cap_ctrl);

// 2. 发送带有错误 ECRC 的 TLP
// (需要硬件支持故意生成错误的 ECRC)
```

#### b. 物理层错误注入

- **符号错误**：注入 8b/10b 编码错误
- **链路训练失败**：强制链路进入错误状态
- **时钟误差**：引入时钟偏差导致接收错误

### 2. 软件错误注入

#### a. Linux Kernel AER 注入

Linux 内核提供了 AER 错误注入接口：

```bash
# 加载 aer-inject 模块
modprobe aer-inject

# 创建错误注入配置文件
cat > aer_error.txt << EOF
AER
PCI_ID 0000:01:00.0
UNCOR_STATUS SDES  # Surprise Down Error Status
EOF

# 注入错误
aer-inject aer_error.txt
```

**常用错误代码**：

| 缩写 | 错误类型 | 描述 |
|------|---------|------|
| DLP | Data Link Protocol Error | 数据链路层协议错误 |
| SDES | Surprise Down Error Status | 意外链路断开 |
| TLP | TLP Prefix Blocked Error | TLP 前缀被阻止 |
| FCP | Flow Control Protocol Error | 流控协议错误 |
| CTO | Completion Timeout | 完成超时 |
| CA | Completer Abort | 完成器终止 |
| UC | Unexpected Completion | 意外完成 |
| RO | Receiver Overflow | 接收器溢出 |
| MLFP | Malformed TLP | 格式错误的 TLP |
| ECRC | ECRC Error | ECRC 校验错误 |
| UE | Unsupported Request | 不支持的请求 |
| ACS | ACS Violation | ACS 违规 |

#### b. PCIe 错误注入工具

```bash
# 使用 setpci 直接修改 AER 寄存器
# 查找 AER 能力偏移
lspci -vvv -s 01:00.0 | grep "Advanced Error Reporting"

# 注入 Uncorrectable Error
# (假设 AER 能力在偏移 0x100)
setpci -s 01:00.0 0x104.l=0x00001000  # 设置 Completion Timeout 位
```

### 3. 仿真器/协议分析仪注入

```
┌──────────────────────────────────────┐
│   PCIe Protocol Analyzer             │
│   (Teledyne LeCroy, Keysight, etc.)  │
├──────────────────────────────────────┤
│                                      │
│  功能：                               │
│  • 拦截 TLP/DLLP                     │
│  • 修改数据包内容                     │
│  • 插入错误的数据包                   │
│  • 注入时序错误                       │
│  • 监控错误响应                       │
│                                      │
└──────────────────────────────────────┘
```

## 可注入的错误类型

### 可纠正错误 (Correctable Errors)

| 错误类型 | 注入方法 | 预期行为 |
|---------|---------|---------|
| Receiver Error | 物理层符号损坏 | 自动重训练 |
| Bad TLP | 发送格式错误的 TLP | 丢弃 TLP，记录错误 |
| Bad DLLP | 发送格式错误的 DLLP | 丢弃 DLLP，记录错误 |
| REPLAY_NUM Rollover | 强制重传计数溢出 | 链路重训练 |
| Replay Timer Timeout | 延迟 ACK/NAK | 请求重传 |

### 非致命错误 (Non-Fatal Errors)

| 错误类型 | 注入方法 | 预期行为 |
|---------|---------|---------|
| Poisoned TLP | 设置 TLP EP 位 | 传播错误，最终返回错误完成 |
| ECRC Error | 损坏 ECRC 字段 | 丢弃 TLP，记录错误 |
| Malformed TLP | 发送非法格式 TLP | 设备返回 UR，触发错误报告 |
| Unexpected Completion | 发送未请求的完成 | 记录错误，可能丢弃完成 |
| Completer Abort | 完成器返回 CA 状态 | 请求者收到错误完成 |
| Unsupported Request | 访问未实现的功能 | 返回 UR 完成 |

### 致命错误 (Fatal Errors)

| 错误类型 | 注入方法 | 预期行为 |
|---------|---------|---------|
| Malformed TLP (Fatal) | 严重格式错误 | 链路可能被禁用 |
| Flow Control Protocol Error | 违反流控规则 | 链路重训练或禁用 |
| Completion Timeout | 长时间无完成 | 超时处理，可能触发复位 |
| Surprise Down | 突然拔出设备 | 系统检测热拔出 |

## 错误注入实践示例

### 示例 1：注入完成超时错误

```c
#include <linux/pci.h>
#include <linux/aer.h>

void inject_completion_timeout(struct pci_dev *pdev)
{
    int aer_pos;
    u32 status;
    
    // 查找 AER 能力
    aer_pos = pci_find_ext_capability(pdev, PCI_EXT_CAP_ID_ERR);
    if (!aer_pos) {
        pr_err("AER capability not found\n");
        return;
    }
    
    // 注入 Completion Timeout (位 14)
    pci_read_config_dword(pdev, aer_pos + PCI_ERR_UNCOR_STATUS, &status);
    status |= PCI_ERR_UNC_COMP_TIME;
    pci_write_config_dword(pdev, aer_pos + PCI_ERR_UNCOR_STATUS, status);
    
    pr_info("Completion Timeout injected\n");
}
```

### 示例 2：批量错误注入测试脚本

```bash
#!/bin/bash
# PCIe 错误注入测试脚本

DEVICE="0000:01:00.0"
AER_INJECT="/sys/kernel/debug/aer_inject"

# 错误类型列表
ERRORS=(
    "UNCOR_STATUS DLP"
    "UNCOR_STATUS SDES"
    "UNCOR_STATUS CTO"
    "UNCOR_STATUS MLFP"
    "UNCOR_STATUS ECRC"
    "COR_STATUS RXERR"
    "COR_STATUS BAD_TLP"
)

echo "Starting PCIe error injection test for $DEVICE"

for error in "${ERRORS[@]}"; do
    echo "========================"
    echo "Injecting: $error"
    
    # 创建错误配置
    cat > /tmp/aer_test.txt << EOF
AER
PCI_ID $DEVICE
$error
EOF
    
    # 注入错误
    aer-inject /tmp/aer_test.txt
    
    # 等待系统处理
    sleep 2
    
    # 检查错误状态
    echo "Checking error status..."
    lspci -vvv -s $DEVICE | grep -A 10 "Advanced Error Reporting"
    
    # 检查系统日志
    dmesg | tail -n 20
    
    echo "Press Enter to continue..."
    read
done

echo "Error injection test completed"
```

### 示例 3：Python 错误注入工具

```python
#!/usr/bin/env python3
import os
import subprocess
import time

class PCIeErrorInjector:
    def __init__(self, bdf):
        """
        初始化错误注入器
        :param bdf: PCI 设备 BDF (Bus:Device.Function) 如 "01:00.0"
        """
        self.bdf = f"0000:{bdf}"
        self.aer_cap = self._find_aer_capability()
    
    def _find_aer_capability(self):
        """查找 AER 能力偏移"""
        cmd = f"lspci -vvv -s {self.bdf} | grep 'Advanced Error Reporting'"
        result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
        if result.returncode == 0:
            # 解析偏移量（通常在输出中）
            return 0x100  # 简化示例，实际应从输出解析
        return None
    
    def inject_error(self, error_type, is_correctable=False):
        """
        注入指定类型的错误
        :param error_type: 错误类型标识
        :param is_correctable: 是否为可纠正错误
        """
        if not self.aer_cap:
            print("AER capability not found")
            return False
        
        # 创建错误注入配置
        severity = "COR_STATUS" if is_correctable else "UNCOR_STATUS"
        config = f"""AER
PCI_ID {self.bdf}
{severity} {error_type}
"""
        
        # 写入临时文件
        with open("/tmp/aer_inject.txt", "w") as f:
            f.write(config)
        
        # 执行注入
        try:
            subprocess.run(["aer-inject", "/tmp/aer_inject.txt"], check=True)
            print(f"Successfully injected {error_type}")
            return True
        except subprocess.CalledProcessError as e:
            print(f"Failed to inject error: {e}")
            return False
    
    def monitor_errors(self, duration=10):
        """监控错误状态"""
        print(f"Monitoring errors for {duration} seconds...")
        start_time = time.time()
        
        while time.time() - start_time < duration:
            cmd = f"lspci -vvv -s {self.bdf} | grep -A 5 'UESta\\|CESta'"
            subprocess.run(cmd, shell=True)
            time.sleep(2)

# 使用示例
if __name__ == "__main__":
    injector = PCIeErrorInjector("01:00.0")
    
    # 注入完成超时错误
    injector.inject_error("CTO", is_correctable=False)
    
    # 监控错误状态
    injector.monitor_errors(duration=10)
```

## 错误注入最佳实践

### 测试策略

```
┌─────────────────────────────────────┐
│     错误注入测试流程                 │
├─────────────────────────────────────┤
│                                     │
│  1. 准备阶段                         │
│     ├─ 确认设备支持 AER             │
│     ├─ 备份原始配置                 │
│     └─ 设置监控工具                 │
│                                     │
│  2. 注入阶段                         │
│     ├─ 从简单到复杂                 │
│     ├─ 单一错误隔离测试             │
│     └─ 组合错误场景测试             │
│                                     │
│  3. 验证阶段                         │
│     ├─ 检查错误报告                 │
│     ├─ 验证恢复机制                 │
│     └─ 确认系统稳定性               │
│                                     │
│  4. 清理阶段                         │
│     ├─ 清除错误状态                 │
│     ├─ 恢复原始配置                 │
│     └─ 记录测试结果                 │
│                                     │
└─────────────────────────────────────┘
```

### 安全注意事项

1. **测试环境隔离**
   - 在专用测试系统上进行
   - 避免在生产环境注入错误
   - 使用虚拟化环境进行初步测试

2. **数据保护**
   - 备份重要数据
   - 使用只读文件系统测试
   - 避免对存储设备注入致命错误

3. **系统监控**
   - 实时监控系统日志
   - 记录所有错误事件
   - 准备快速恢复方案

4. **逐步推进**
   ```
   可纠正错误 → 非致命错误 → 致命错误
   单一错误   → 组合错误   → 压力测试
   ```

### 常见陷阱

| 陷阱 | 后果 | 避免方法 |
|------|------|---------|
| 在生产系统测试 | 系统崩溃、数据丢失 | 使用专用测试环境 |
| 未清除错误状态 | 影响后续测试 | 每次测试后清理 |
| 注入过多致命错误 | 设备被禁用 | 控制致命错误频率 |
| 忽略错误掩码 | 错误未被报告 | 检查掩码配置 |
| 未验证错误处理 | 无法发现问题 | 完整的验证流程 |

## 验证错误注入结果

### 检查 AER 状态寄存器

```bash
# 查看可纠正错误状态
lspci -vvv -s 01:00.0 | grep -A 2 "CESta:"

# 查看不可纠正错误状态
lspci -vvv -s 01:00.0 | grep -A 2 "UESta:"

# 查看错误详细信息
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_correctable
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_fatal
cat /sys/bus/pci/devices/0000:01:00.0/aer_dev_nonfatal
```

### 监控系统日志

```bash
# 实时监控内核消息
dmesg -w | grep -i "aer\|pcie"

# 查看 PCIe 错误日志
journalctl -k | grep -i "pcie error"

# 检查 AER 驱动报告
cat /sys/kernel/debug/pci/*/aer_stats
```

### 性能影响评估

```bash
# 测试错误注入前后的性能
# 1. 基准测试
fio --name=baseline --rw=read --size=1G --filename=/dev/nvme0n1

# 2. 注入错误
aer-inject error_config.txt

# 3. 性能测试
fio --name=with_errors --rw=read --size=1G --filename=/dev/nvme0n1

# 4. 比较结果
```

## 工具和资源

### 开源工具

- **aer-inject**：Linux 内核自带的 AER 错误注入工具
- **pci-utils**：包含 lspci 和 setpci 工具
- **PCIe Error Injection Framework**：自动化测试框架

### 商业工具

- **Teledyne LeCroy PCIe Protocol Analyzer**：硬件级错误注入
- **Keysight PCIe Exerciser**：协议级测试和错误注入
- **Synopsys HAPS**：FPGA 原型验证平台

### 调试接口

```
/sys/kernel/debug/pci/
├── <BDF>/
│   ├── aer_stats          # AER 统计信息
│   └── config             # 配置空间原始访问
└── aer_inject             # 错误注入接口

/sys/bus/pci/devices/<BDF>/
├── aer_dev_correctable    # 可纠正错误计数
├── aer_dev_fatal          # 致命错误计数
└── aer_dev_nonfatal       # 非致命错误计数
```

## 参考资料

- PCI Express Base Specification, Chapter 6 (Error Signaling and Logging)
- Linux Kernel Documentation: PCI/pcieaer-howto.rst
- [PCIe AER 错误类型](error-types.md)
- [AER 高级错误报告](aer.md)
- [错误检测机制](detection.md)
- [错误恢复流程](recovery.md)

## 相关主题

- [AER (Advanced Error Reporting)](aer.md)
- [错误检测](detection.md)
- [错误恢复](recovery.md)
- [RAS 特性](../advanced-features/ras.md)
