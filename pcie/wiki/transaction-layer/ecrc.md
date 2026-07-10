# ECRC (端到端 CRC) 详解

End-to-End CRC (ECRC) 是 PCIe 事务层提供的可选端到端数据完整性保护机制，用于检测 TLP 在传输过程中的错误。

---

## 概述

ECRC 是一个**可选的 32 位 CRC 校验码**，附加在 TLP 的末尾：
- 由**发送方事务层**计算并添加
- 由**接收方事务层**验证
- 覆盖 TLP Header 和 Data Payload
- 独立于数据链路层的 LCRC

**ECRC 与 LCRC 的区别**：

| 特性 | ECRC | LCRC |
|------|------|------|
| **层次** | Transaction Layer | Data Link Layer |
| **范围** | 端到端（发送者到最终接收者）| 链路到链路（相邻设备之间）|
| **是否可选** | 可选（通过配置启用）| 强制（PCIe 规范要求）|
| **保护内容** | TLP Header + Data | TLP Header + Data + Seq# |
| **检测能力** | 整个路径的错误 | 单条链路的错误 |

**为什么需要 ECRC**：
```
设备 A ──────> Switch ──────> 设备 B
         ↓              ↓
      LCRC 保护     LCRC 保护

问题：Switch 内部处理错误（如缓冲区数据损坏）无法被 LCRC 检测
解决：ECRC 提供端到端保护
```

---

## ECRC 在 TLP 中的位置

### TLP 完整结构（带 ECRC）

```
┌────────────────────────────────────────────────────┐
│           TLP Header (3 or 4 DW)                   │
│  Byte 0-11 (3 DW) or 0-15 (4 DW)                   │
│  包含 TD bit = 1（指示存在 ECRC）                  │
├────────────────────────────────────────────────────┤
│           Data Payload (0-1024 DW)                 │  可选
│  最大 4096 字节                                     │
├────────────────────────────────────────────────────┤
│           ECRC (1 DW = 4 bytes)                    │  ← 这里
│  32-bit CRC-32 校验码                               │
└────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  LCRC 由数据链路层添加（覆盖上述所有内容）           │
└─────────────────────────────────────────────────────┘
```

### TD Bit（TLP Digest Present）

ECRC 的存在由 TLP Header 的 **TD bit** 指示：

```
Byte 0 (Header DW 0):
 0 1 2 3 4 5 6 7 8 9 ...
┌───┬─────┬───┬─┬─┬─┬───┬──...
│Fmt│Type │ R │T│T│A│ R │  ...
│   │     │   │C│E│t│   │
│   │     │   │ │ │t│   │
└───┴─────┴───┴─┴─┴─┴───┴──...
              ↑
              TD bit (bit 15)
              
TD = 0: 无 ECRC
TD = 1: TLP 末尾包含 ECRC
```

---

## ECRC 计算

### CRC-32 多项式

ECRC 使用标准的 **CRC-32** 算法（与以太网相同）：

```
多项式: G(x) = x³² + x²⁶ + x²³ + x²² + x¹⁶ + x¹² + x¹¹ 
              + x¹⁰ + x⁸ + x⁷ + x⁵ + x⁴ + x² + x + 1

十六进制表示: 0x04C11DB7
二进制表示: 100000100110000010001110110110111
```

### 计算步骤

1. **初始化**：CRC 寄存器 = `0xFFFFFFFF`

2. **处理 TLP Header**：
   - 按字节顺序处理 Header 的所有字节
   - 3 DW Header: 12 字节
   - 4 DW Header: 16 字节

3. **处理 Data Payload**（如果存在）：
   - 按字节顺序处理所有数据字节
   - 0 到 4096 字节

4. **求反**：CRC 结果按位取反

5. **附加到 TLP**：32-bit ECRC 附加到 TLP 末尾

### 伪代码

```c
uint32_t calculate_ecrc(uint8_t *tlp, size_t length) {
    uint32_t crc = 0xFFFFFFFF;  // 初始值
    
    // 处理 TLP Header + Data Payload
    for (size_t i = 0; i < length; i++) {
        crc = crc32_table[(crc ^ tlp[i]) & 0xFF] ^ (crc >> 8);
    }
    
    return ~crc;  // 按位取反
}

// CRC-32 查表法
uint32_t crc32_table[256] = {
    0x00000000, 0x04C11DB7, 0x09823B6E, 0x0D4326D9,
    // ... (标准 CRC-32 表)
};
```

### 计算示例

```
示例 TLP (Memory Write):
Header (3 DW):
  DW0: 0x40000001  (Fmt=010, Type=00000, TD=1, Length=1)
  DW1: 0x03000500  (Requester ID=0x0300, Tag=0x05)
  DW2: 0x00001000  (Address=0x1000)

Data (1 DW):
  DW3: 0x12345678

计算过程:
1. CRC = 0xFFFFFFFF
2. 处理 12 字节 Header (按字节序)
3. 处理 4 字节 Data
4. CRC = ~CRC
5. 结果 ECRC: 0xABCD1234 (示例值)

最终 TLP:
  Header (12 bytes) + Data (4 bytes) + ECRC (4 bytes) = 20 bytes
```

---

## ECRC 验证

### 接收方处理流程

```
1. 接收 TLP（数据链路层已验证 LCRC）
   ↓
2. 检查 TD bit
   ├─> TD=0: 无 ECRC，跳过验证
   └─> TD=1: 有 ECRC，继续
   ↓
3. 提取 TLP 末尾的 ECRC 值 (E_received)
   ↓
4. 重新计算 ECRC (E_calculated)
   - 使用相同算法
   - 覆盖 Header + Data（不包含 ECRC 本身）
   ↓
5. 比较
   ├─> E_received == E_calculated: ✅ 验证通过
   └─> E_received != E_calculated: ❌ ECRC 错误
   ↓
6. ECRC 错误处理
   ├─> 丢弃 TLP（如果配置为丢弃）
   ├─> 标记为 Poisoned（设置 EP bit）
   └─> 记录错误（AER 寄存器）
```

### 错误处理策略

PCIe 设备可以配置 ECRC 错误的处理方式：

**Device Control Register (Offset 0x08 in PCIe Capability)**：

```
Bit 15: ECRC Generation Enable
  0 = 发送时不生成 ECRC（TD=0）
  1 = 发送时生成 ECRC（TD=1）

Bit 14: ECRC Check Enable
  0 = 接收时不检查 ECRC（忽略）
  1 = 接收时检查 ECRC
```

**AER (Advanced Error Reporting) 寄存器**：

```
Correctable Error Status (Offset 0x10):
  Bit 7: ECRC Error (可纠正错误)

Uncorrectable Error Status (Offset 0x04):
  (ECRC 错误通常不作为不可纠正错误)
```

---

## ECRC 启用和配置

### 设备能力检查

```bash
# 查看设备是否支持 ECRC
lspci -vvv -s 01:00.0 | grep -i ecrc
  Capabilities: [100] Advanced Error Reporting
    AERCap: First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
    
ECRCGenCap+: 支持 ECRC 生成
ECRCGenEn-:  ECRC 生成未启用
ECRCChkCap+: 支持 ECRC 检查
ECRCChkEn-:  ECRC 检查未启用
```

### 启用 ECRC（Linux）

```bash
# 方法 1：通过 setpci 手动启用
# 启用 ECRC 生成 (Device Control bit 15)
setpci -s 01:00.0 CAP_EXP+8.w=8000

# 启用 ECRC 检查 (Device Control bit 14)
setpci -s 01:00.0 CAP_EXP+8.w=4000

# 同时启用生成和检查
setpci -s 01:00.0 CAP_EXP+8.w=C000

# 方法 2：通过 AER 驱动
echo 1 > /sys/bus/pci/devices/0000:01:00.0/aer_dev_correctable
```

### 启用 ECRC（BIOS/UEFI）

许多系统 BIOS 提供 ECRC 选项：

```
Advanced → PCI Configuration → ECRC Support
  Options: Disabled / Enabled
```

---

## 中间设备对 ECRC 的处理

### Switch 和 Bridge 行为

**关键规则**：中间设备（Switch/Bridge）**不能修改带 ECRC 的 TLP**

```
场景 1: TLP 通过 Switch（无修改）
设备 A ────> Switch ────> 设备 B
  (TD=1)      ↓            (TD=1)
            只转发
          不修改 TLP
          ✅ ECRC 保持有效

场景 2: TLP 通过 Switch（有修改）
设备 A ────> Switch ────> 设备 B
  (TD=1)      ↓            (??)
           需要修改 TLP
         (如修改 Traffic Class)
           ↓
         问题：ECRC 会失效！
```

### TLP 修改与 ECRC

如果中间设备需要修改 TLP（如修改 TC、调整地址），有两种选择：

**选项 1：剥离并重新计算 ECRC**
```
1. 验证原始 ECRC
2. 剥离 ECRC（TD=0）
3. 修改 TLP Header
4. 重新计算 ECRC
5. 附加新 ECRC（TD=1）
```

**选项 2：剥离 ECRC 不重新计算**
```
1. 验证原始 ECRC
2. 剥离 ECRC（TD=0）
3. 修改 TLP Header
4. 转发（无 ECRC）

缺点：下游失去端到端保护
```

**大多数 Switch 的实际行为**：
- 简单转发：不修改 TLP → 保持 ECRC
- 需要修改：剥离 ECRC，不重新生成

---

## ECRC 的实际应用

### 何时启用 ECRC

**推荐启用的场景**：

1. ✅ **关键任务系统**
   - 金融交易、医疗设备
   - 数据完整性要求极高

2. ✅ **长距离传输**
   - 外置设备（如 Thunderbolt）
   - 光纤 PCIe（距离 > 1米）

3. ✅ **恶劣环境**
   - 高电磁干扰（EMI）
   - 高温、辐射环境

4. ✅ **调试和验证**
   - 新硬件设计验证
   - 查找信号完整性问题

**不推荐或无需启用的场景**：

1. ❌ **标准桌面/服务器**
   - LCRC 已提供足够保护
   - ECRC 开销不值得

2. ❌ **性能敏感应用**
   - ECRC 计算有微小延迟
   - 增加 TLP 大小（+4 字节）

3. ❌ **复杂拓扑**
   - 多级 Switch 可能剥离 ECRC
   - 端到端保护无法保证

### 性能影响

**ECRC 的开销**：

| 方面 | 影响 |
|------|------|
| **带宽** | +4 字节/TLP（约 1-2% 开销）|
| **延迟** | +数个时钟周期（CRC 计算）|
| **计算** | 硬件 CRC 引擎，影响极小 |

**示例**：
```
无 ECRC 的 TLP:
  Header (12 bytes) + Data (256 bytes) = 268 bytes

有 ECRC 的 TLP:
  Header (12 bytes) + Data (256 bytes) + ECRC (4 bytes) = 272 bytes
  
开销: 4/272 = 1.47%
```

---

## ECRC 错误诊断

### 检测 ECRC 错误

```bash
# 查看 AER 错误状态
lspci -vvv -s 01:00.0 | grep -A 20 "Advanced Error Reporting"
  UESta:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- ...
  CESta:  RxErr- BadTLP- BadDLLP- Rollover- Timeout- NonFatalErr+
          ^^^^^^
          包含 ECRC 错误

# 查看内核日志
dmesg | grep -i "ecrc\|aer"
  [12345.678] pcieport 0000:00:01.0: AER: Corrected error received
  [12345.679] pcieport 0000:00:01.0: AER:   [ 7] ECRC
```

### 常见 ECRC 错误原因

1. **信号完整性问题**
   ```
   原因：电缆质量差、连接松动、走线过长
   症状：间歇性 ECRC 错误
   解决：更换电缆、检查连接、改进 PCB 设计
   ```

2. **中间设备修改 TLP**
   ```
   原因：Switch/Bridge 修改了 TLP 但未更新 ECRC
   症状：所有经过该设备的 TLP 都有 ECRC 错误
   解决：更新固件、禁用 TLP 修改功能
   ```

3. **配置错误**
   ```
   原因：发送方生成 ECRC，但使用了错误的算法
   症状：100% ECRC 错误
   解决：检查设备固件、驱动版本
   ```

4. **硬件故障**
   ```
   原因：CRC 计算电路损坏、内存损坏
   症状：持续的 ECRC 错误，重启后依然存在
   解决：更换硬件
   ```

### 调试工具

```bash
# 1. 强制启用 ECRC 检查
echo 1 > /sys/bus/pci/devices/0000:01:00.0/aer_dev_correctable

# 2. 监控 ECRC 错误
watch -n 1 'lspci -vvv -s 01:00.0 | grep -A 5 "CESta"'

# 3. 使用 PCIe 协议分析仪
# 硬件分析仪可以捕获和解析 ECRC
```

---

## FEMU/QEMU 中的 ECRC 实现

### ECRC 相关代码

```c
// hw/pci/pcie.c
// ECRC 能力设置
void pcie_cap_aer_init(PCIDevice *dev)
{
    // 设置 AER Extended Capability
    uint16_t offset = pcie_add_capability(dev, PCI_EXT_CAP_ID_ERR, ...);
    
    // 设置 ECRC 能力位
    pci_set_long(dev->config + offset + PCI_ERR_CAP,
                 PCI_ERR_CAP_ECRC_GENC | PCI_ERR_CAP_ECRC_CHKC);
}

// hw/pci/pcie_aer.c
// ECRC 错误注入
void pcie_aer_inject_error(PCIDevice *dev, const PCIEAERErr *err)
{
    if (err->flags & PCIE_AER_ERR_ECRC) {
        // 设置 ECRC 错误状态位
        pci_long_test_and_set_mask(
            dev->config + dev->exp.aer_cap + PCI_ERR_COR_STATUS,
            PCI_ERR_COR_ECRC);
    }
}
```

### 模拟 ECRC 错误

```bash
# QEMU 启动时启用 AER
qemu-system-x86_64 \
  -device pcie-root-port,id=rp1,bus=pcie.0,aer=on \
  -device nvme,bus=rp1,serial=deadbeef,aer=on

# 在 QEMU monitor 中注入 ECRC 错误
(qemu) pcie_aer_inject_error nvme0 correctable ecrc
```

---

## 实用技巧

### 1. 逐步启用 ECRC

```bash
# 步骤 1: 仅启用检查（不生成）
# 检查接收到的 TLP 的 ECRC
setpci -s 01:00.0 CAP_EXP+8.w=4000

# 步骤 2: 观察是否有错误
dmesg | grep -i ecrc
# 如果无错误，继续

# 步骤 3: 启用生成
setpci -s 01:00.0 CAP_EXP+8.w=C000

# 步骤 4: 全系统启用
for dev in /sys/bus/pci/devices/*; do
    setpci -s $(basename $dev) CAP_EXP+8.w=C000
done
```

### 2. ECRC 与 TLP Poisoning

ECRC 错误通常导致 TLP 被标记为 **Poisoned**（EP bit = 1）：

```
ECRC 错误 TLP 处理:
1. 检测 ECRC 错误
2. 设置 EP bit = 1（标记为 Poisoned）
3. 继续转发 TLP（不丢弃）
4. 最终接收者决定如何处理 Poisoned TLP
```

### 3. 跨厂商互操作性

不同厂商的 ECRC 实现可能存在差异：

```bash
# 测试 ECRC 互操作性
# 1. 连接不同厂商的设备
# 2. 启用 ECRC
# 3. 运行压力测试
fio --name=test --rw=randrw --size=10G --numjobs=4 --time_based --runtime=3600

# 4. 监控 ECRC 错误
watch -n 1 'grep -i ecrc /sys/bus/pci/devices/*/aer_*'
```

---

## 总结

### 关键要点

1. ✅ **ECRC 是可选的端到端 CRC 保护**，覆盖整个 TLP
2. ✅ **TD bit 指示 ECRC 存在**，ECRC 为 32 位 CRC-32 校验码
3. ✅ **独立于 LCRC**：LCRC 保护链路，ECRC 保护端到端路径
4. ✅ **中间设备不能修改带 ECRC 的 TLP**，否则 ECRC 失效
5. ✅ **适用于关键任务系统**，标准应用通常不需要
6. ✅ **通过 Device Control 和 AER 寄存器配置**

### 决策树：是否启用 ECRC？

```
需要最高级别的数据完整性？
├─> 是 → 启用 ECRC
└─> 否 → 继续

有信号完整性问题（如长电缆、高 EMI）？
├─> 是 → 启用 ECRC
└─> 否 → 继续

拓扑简单（无多级 Switch）？
├─> 是 → 可以启用 ECRC
└─> 否 → 可能无效，慎用

性能是否极端敏感（如高频交易）？
├─> 是 → 不启用 ECRC
└─> 否 → 可以启用

结论：大多数应用不需要 ECRC，LCRC 已足够
```

---

## 下一步学习

- [TLP 格式详解](tlp-format.md) - TLP 的完整结构
- [错误处理机制](../error-handling/error-detection.md) - PCIe 错误检测
- [AER (高级错误报告)](../error-handling/aer.md) - 错误报告和记录
- [DLLP 和 LCRC](../data-link-layer/dllp-format.md) - 数据链路层的 CRC

---

## 参考资料

- **规范**：PCIe Base Spec Chapter 2.7 (End-to-End Data Integrity)
- **算法**：PCIe Base Spec Appendix A (CRC-32 Calculation)
- **寄存器**：PCIe Base Spec Chapter 7.8 (Device Control Register)
- **AER**：PCIe Base Spec Chapter 6.2 (Advanced Error Reporting)

---

**相关页面**：
- [← TLP 格式](tlp-format.md)
- [流控制 →](flow-control.md)
- [返回首页](../README.md)

---

*最后更新：2026-07-06*
