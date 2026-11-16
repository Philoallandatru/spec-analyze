# PCIe 寄存器速查表

每个 PCIe Function 拥有 4 KB 配置空间，分为标准 PCI 兼容区（`0x00–0xFF`）和 PCIe 扩展配置区（`0x100–0xFFF`，通过 ECAM 内存映射访问）。本页汇总最常用的配置头寄存器、Capability 结构、关键位域及 `setpci` 调试命令。

```
0x000 ┌───────────────────────────────────────┐
      │ 标准配置头 (Type 0 Endpoint / Type 1)  │ 64 B
0x040 ├───────────────────────────────────────┤
      │ PCI Capabilities 链表 (PM, MSI, PCIe) │ 192 B (由 0x34 指针串联)
0x100 ├───────────────────────────────────────┤
      │ PCIe Extended Capabilities 链表       │ 3840 B (AER, SR-IOV, DOE, IDE...)
0xFFF └───────────────────────────────────────┘
```

---

## 标准配置空间 (0x00–0xFF)

### Type 0 Header (Endpoint，0x00–0x3F)

| 偏移 | 名称 | 大小/位域 | 访问 | 描述 |
|------|------|-----------|------|------|
| `0x00` | Vendor ID | 16-bit | RO | PCI-SIG 分配的厂商 ID（`0xFFFF` 表示无设备） |
| `0x02` | Device ID | 16-bit | RO | 厂商自定义的设备型号 ID |
| `0x04` | Command | 16-bit | RW | 控制设备功能（MMIO、Bus Master DMA、INTx 开关等） |
| `0x06` | Status | 16-bit | RO/RW1C | 设备状态与错误标志（错误位写 1 清零） |
| `0x08` | Revision ID | 8-bit | RO | 硬件修订版本号 |
| `0x09` | Class Code | 24-bit | RO | 设备类别（`0x09` Prog IF、`0x0A` Subclass、`0x0B` Base Class） |
| `0x0C` | Cache Line Size | 8-bit | RW | 缓存行大小（传统 PCI 遗留，PCIe 仍保留） |
| `0x0D` | Latency Timer | 8-bit | RO | 延迟定时器（PCIe 固定为 0，无效） |
| `0x0E` | Header Type | 8-bit | RO | Bit 7=1 表示多功能设备；Bits 6:0 为 `0x00`=Type 0、`0x01`=Type 1 |
| `0x0F` | BIST | 8-bit | RW | 内建自检控制与状态 |
| `0x10–0x24` | BAR 0–5 | 6×32-bit | RW | 基地址寄存器（映射 MMIO 或 I/O 空间，64 位 BAR 占用相邻两个） |
| `0x28` | Cardbus CIS Pointer | 32-bit | RO | CardBus 指针（现代 PCIe 设备通常为 0） |
| `0x2C` | Subsystem Vendor ID | 16-bit | RO | 子系统（板卡 OEM）厂商 ID |
| `0x2E` | Subsystem ID | 16-bit | RO | 子系统型号 ID |
| `0x30` | Expansion ROM Base | 32-bit | RW | 扩展 Option ROM 基地址 |
| `0x34` | Capabilities Pointer | 8-bit | RO | 指向首个 PCI Capability 的偏移地址（DWORD 对齐） |
| `0x3C` | Interrupt Line | 8-bit | RW | 传统中断路由线号（由固件/OS 填写） |
| `0x3D` | Interrupt Pin | 8-bit | RO | 传统中断引脚（`0`=无，`1`=INTA#, `2`=INTB#, `3`=INTC#, `4`=INTD#） |

### Command Register (0x04)

| 位 | 名称 | 描述 |
|----|------|------|
| `0` | I/O Space Enable | 允许设备响应 I/O 空间访问 |
| `1` | Memory Space Enable (MSE) | 允许设备响应 **MMIO** 空间访问（读写 BAR 前必须置 1） |
| `2` | Bus Master Enable (BME) | 允许设备主动发起 **DMA** 与 **MSI/MSI-X** 写事务 |
| `6` | Parity Error Response | 使能奇偶/中毒 TLP 错误响应 |
| `10` | Interrupt Disable | 仅禁用传统 **INTx** 中断（不影响 MSI/MSI-X） |

> [!NOTE]
> 位 3 (Special Cycles)、位 4 (MWI)、位 5 (VGA Palette Snoop)、位 8 (SERR#)、位 9 (Fast B2B) 在 PCIe 中多为兼容保留或忽略。

### Status Register (0x06)

| 位 | 名称 | 访问 | 描述 |
|----|------|------|------|
| `3` | Interrupt Status | RO | 传统 INTx 中断挂起状态 |
| `4` | Capabilities List | RO | `1` 表示存在 `0x34` 指向的 Capability 链表（PCIe 设备必为 1） |
| `8` | Master Data Parity Error | RW1C | 主控模式下检测到数据校验/中毒错误 |
| `11` | Signaled Target Abort | RW1C | 设备发送了 Completer Abort (CA) |
| `12` | Received Target Abort | RW1C | 收到 Completer Abort (CA) |
| `13` | Received Master Abort | RW1C | 收到 Unsupported Request (UR) |
| `14` | Signaled System Error | RW1C | 发送了 Fatal/Non-Fatal 错误消息 |
| `15` | Detected Parity Error | RW1C | 检测到奇偶校验或中毒 TLP 错误 |

---

## 常用 PCI Capabilities (0x40–0xFF 链表)

每个标准 Capability 的前 2 字节均为：`[7:0] Capability ID`、`[15:8] Next Capability Pointer`（`0x00` 表示链表结束）。

### 常见 Capability 偏移布局

| Capability (ID) | 关键偏移 | 名称 | 说明 |
|-----------------|----------|------|------|
| **Power Management** (`0x01`) | `+0x02` | PM Capabilities | 支持的 D-State 与 PME# 唤醒能力 |
| | `+0x04` | PM Control/Status | 读取/切换当前 D0–D3Hot 状态 |
| **MSI** (`0x05`) | `+0x02` | Message Control | 使能位、64 位地址支持、申请/分配向量数 |
| | `+0x04 / +0x08` | Message Address Lo/Hi | 目标 APIC 内存写地址 |
| | `+0x0C` | Message Data | 写入的中断向量数据 |
| **MSI-X** (`0x11`) | `+0x02` | Message Control | 表大小（N-1，最多 2048）与全局屏蔽位 |
| | `+0x04` | Table Offset / BIR | 向量表所在 BAR 编号（低 3 位）与偏移 |
| | `+0x08` | PBA Offset / BIR | Pending Bit Array 所在 BAR 编号与偏移 |

### PCIe Capability (ID = `0x10`)

| 偏移 | 名称 | 大小 | 描述 |
|------|------|------|------|
| `+0x00` | Cap ID / Next | 16-bit | ID=`0x10` + 下一个 Capability 偏移 |
| `+0x02` | PCIe Capabilities | 16-bit | 版本号与 Device/Port Type（EP、RP、Switch 等） |
| `+0x04` | Device Capabilities | 32-bit | 支持的最大 MPS、FLR、扩展 Tag 等硬件能力 |
| `+0x08` | Device Control | 16-bit | 配置 MPS、MRRS、RO、No Snoop 及触发 FLR |
| `+0x0A` | Device Status | 16-bit | 4 类错误检测标志与事务挂起状态 |
| `+0x0C` | Link Capabilities | 32-bit | 硬件支持的最大链路速率、宽度与 ASPM 能力 |
| `+0x10` | Link Control | 16-bit | ASPM 开关、重训练链路（Retrain）、Disable Link 等 |
| `+0x12` | Link Status | 16-bit | 当前实际协商的链路速率、宽度与训练状态 |
| `+0x14–0x1A` | Slot Cap / Ctrl / Sta | 32/16/16-bit | 插槽供电、热插拔与在位状态（仅 Port/Slot） |
| `+0x1C–0x20` | Root Ctrl / Cap / Sta | 16/16/32-bit | Root Port 专属控制与 PME 状态 |

#### Device Control (`+0x08`) 关键位域

| 位 | 名称 | 描述 |
|----|------|------|
| `3:0` | Error Reporting Enable | 分别使能 Correctable / Non-Fatal / Fatal / UR 错误报告 |
| `4` | Enable Relaxed Ordering | 允许宽松排序（RO）提升性能 |
| `7:5` | Max Payload Size (MPS) | 当前生效的最大负载大小（见文末编码表） |
| `8` | Extended Tag Field Enable | 允许使用 8 位（或更多）Tag 提升并发未完成读数量 |
| `11` | Enable No Snoop | 允许 DMA 跳过 CPU Cache 一致性嗅探 |
| `14:12` | Max Read Request Size (MRRS) | 当前生效的最大读请求大小 |
| `15` | Initiate Function Level Reset | 写 1 触发该 Function 的 **FLR** 软复位 |

#### Link Control (`+0x10`) 与 Link Status (`+0x12`) 关键位域

| 寄存器 | 位 | 名称 | 描述 |
|--------|----|------|------|
| **Link Control** (`+0x10`) | `1:0` | ASPM Control | `00`=禁用，`01`=L0s，`10`=L1，`11`=L0s+L1 |
| | `4` | Link Disable | 写 1 强制关闭链路 |
| | `5` | Retrain Link | 写 1 触发 LTSSM 重新训练链路 |
| | `6` | Common Clock Config | `1` 表示两端共用同一参考时钟（Refclk） |
| **Link Status** (`+0x12`) | `3:0` | Current Link Speed | 当前速率（`0x1`=Gen1 .. `0x6`=Gen6，见文末表） |
| | `9:4` | Negotiated Link Width | 当前协商宽度（`0x01`=x1, `0x04`=x4, `0x10`=x16） |
| | `11` | Link Training | `1` 表示链路正在训练中（LTSSM 处于 Recovery/Config） |
| | `13` | Data Link Layer Link Active | `1` 表示数据链路层流控已就绪，可正常收发 TLP |

---

## 扩展配置空间 (0x100–0xFFF)

所有扩展 Capability 共用 32 位头部：`[15:0] Cap ID` · `[19:16] Version` · `[31:20] Next Offset`（`0x000` 表示结束）。

### 常用 Extended Capability ID

| ID | 名称 | 描述 |
|----|------|------|
| `0x0001` | **AER** | 高级错误报告（可纠正/不可纠正错误日志与掩码） |
| `0x0002` | **VC** | 虚拟通道（多通道 QoS 仲裁与缓冲区分配） |
| `0x0003` | **Device Serial Number** | 64 位设备唯一序列号（EUI-64） |
| `0x000D` | **ACS** | 访问控制服务（P2P 流量隔离与源验证） |
| `0x000E` | **ARI** | 备选路由 ID（支持单设备最多 256 个 Function） |
| `0x000F` | **ATS** | 地址翻译服务（设备侧 IOMMU 翻译缓存） |
| `0x0010` | **SR-IOV** | 单根 I/O 虚拟化（PF/VF 资源配置） |
| `0x0015` | **Resizable BAR** | 可调整大小的 BAR（如 GPU 大显存映射） |
| `0x0017` | **TPH** | TLP 处理提示（Cache Steering Tag） |
| `0x0018` | **LTR** | 延迟容忍报告（辅助 L1.2 深度节能决策） |
| `0x0019` | **Secondary PCIe** | Gen 3+ 链路均衡控制与 Lane 错误状态 |
| `0x001B` | **PASID** | 进程地址空间 ID（支持共享虚拟内存 SVM） |
| `0x001D` | **DPC** | 下游端口隔离（故障链路快速熔断） |
| `0x001E` | **L1 PM Substates** | L1.1 / L1.2 深度低功耗子状态配置 |
| `0x001F` | **PTM** | 精确时间测量（纳秒级硬件时钟同步） |
| `0x0023` | **DVSEC** | 厂商指定扩展能力（CXL 等常基于此扩展） |
| `0x0025` | **Data Link Feature** | 数据链路特性协商（如 Scaled Flow Control） |
| `0x0026` | **Physical Layer 16.0 GT/s** | Gen 4 物理层均衡参数与状态 |
| `0x002E` | **DOE** | 数据对象交换（SPDM/CMA 安全认证邮箱） |
| `0x0030` | **IDE** | 完整性和数据加密（链路层硬件 AES-GCM 加密） |

### AER (`0x0001`) 与 SR-IOV (`0x0010`) 核心偏移

| 结构 | 偏移 | 名称 | 大小 | 描述 |
|------|------|------|------|------|
| **AER** (`0x0001`) | `+0x04` | Uncorrectable Error Status | 32-bit | 不可纠正错误状态（RW1C） |
| | `+0x08 / +0x0C` | Uncorrectable Mask / Severity | 32-bit | 屏蔽位与致命性（Fatal/Non-Fatal）配置 |
| | `+0x10 / +0x14` | Correctable Status / Mask | 32-bit | 可纠正错误状态与屏蔽位 |
| | `+0x1C` | Header Log | 128-bit | 触发错误的 TLP 头部快照（4×32-bit） |
| **SR-IOV** (`0x0010`) | `+0x08` | SR-IOV Control | 16-bit | VF Enable、ARI Capable Hierarchy 等开关 |
| | `+0x0C / +0x0E` | Initial VFs / Total VFs | 16-bit | 硬件支持的最大 VF 数量 |
| | `+0x10` | Num VFs | 16-bit | 软件当前启用的 VF 数量 |
| | `+0x14 / +0x16` | First VF Offset / VF Stride | 16-bit | 计算各 VF Routing ID 的偏移与步长 |
| | `+0x24` | VF BAR 0–5 | 6×32-bit | 所有 VF 共享的基地址模板 |

---

## 常用位域编码与 `setpci` 速查

### MPS / MRRS 编码、链路速率与宽度

| 编码值 | MPS / MRRS 大小 | Link Speed (`Link Status[3:0]`) | Link Width (`Link Status[9:4]`) |
|--------|-----------------|---------------------------------|---------------------------------|
| `0x0` | 128 bytes | — | — |
| `0x1` | 256 bytes | 2.5 GT/s (Gen 1) | x1 |
| `0x2` | 512 bytes | 5.0 GT/s (Gen 2) | x2 |
| `0x3` | 1024 bytes | 8.0 GT/s (Gen 3) | — |
| `0x4` | 2048 bytes | 16.0 GT/s (Gen 4) | x4 |
| `0x5` | 4096 bytes | 32.0 GT/s (Gen 5) | — |
| `0x6` | — | 64.0 GT/s (Gen 6) | — |
| `0x08 / 0x10 / 0x20` | — | — | x8 / x16 / x32 |

### `setpci` 实战示例

```bash
# 读取 Vendor/Device ID (0x00, 32-bit) 与 Command 寄存器 (0x04, 16-bit)
setpci -s 01:00.0 0.L
setpci -s 01:00.0 4.W

# 使用符号名直接读取 PCIe Device Control (Cap+0x08)、Link Status (Cap+0x12) 与 AER 状态
setpci -s 01:00.0 CAP_EXP+8.W
setpci -s 01:00.0 CAP_EXP+12.W
setpci -s 01:00.0 ECAP_AER+4.L
```

---

## 关联与引申概念

- **配置空间机制**：[配置空间详解](../configuration/config-space.md) · [Header 类型](../configuration/header-types.md) · [BAR 寄存器](../configuration/bars.md)
- **Capability 链表**：[Capabilities 结构](../configuration/capabilities.md) · [扩展 Capabilities](../configuration/extended-capabilities.md)
- **实战与术语**：[驱动开发指南](../implementation/driver-guide.md) · [术语表](index.md) · [常见问题 (FAQ)](faq.md)
