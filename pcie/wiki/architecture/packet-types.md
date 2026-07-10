# PCIe 数据包类型

## 概述

PCIe 协议栈使用三种主要的数据包类型来实现完整的通信功能：
- **TLP (Transaction Layer Packet)**：事务层数据包，用于数据传输
- **DLLP (Data Link Layer Packet)**：数据链路层数据包，用于链路管理
- **Ordered Set**：物理层有序集，用于链路训练和同步

## TLP (Transaction Layer Packet)

### TLP 基本结构

```
+----------------+
|  TLP Prefix    |  (可选，PCIe 3.0+)
+----------------+
|  TLP Header    |  (3 或 4 DW)
+----------------+
|  Data Payload  |  (0-1024 DW)
+----------------+
|  Digest (ECRC) |  (可选，1 DW)
+----------------+
```

### TLP 类型

| Fmt[2:0] | Type[4:0] | 描述 | 数据负载 |
|----------|-----------|------|----------|
| 000 | 00000 | Memory Read Request (3DW) | 无 |
| 001 | 00000 | Memory Read Request (4DW) | 无 |
| 010 | 00000 | Memory Write Request (3DW) | 有 |
| 011 | 00000 | Memory Write Request (4DW) | 有 |
| 000 | 00001 | Memory Read Lock Request (3DW) | 无 |
| 001 | 00001 | Memory Read Lock Request (4DW) | 无 |
| 000 | 00010 | I/O Read Request | 无 |
| 010 | 00010 | I/O Write Request | 有 |
| 000 | 00100 | Config Read Type 0 | 无 |
| 010 | 00100 | Config Write Type 0 | 有 |
| 000 | 00101 | Config Read Type 1 | 无 |
| 010 | 00101 | Config Write Type 1 | 有 |
| 000 | 1xxxx | Message (无数据) | 无 |
| 010 | 1xxxx | Message (有数据) | 有 |
| 000 | 01010 | Completion (无数据) | 无 |
| 010 | 01010 | Completion (有数据) | 有 |

### Memory Request TLP

**3DW Header (32-bit 地址)**：
```
DW0: [31:30]Fmt [29:24]Type [23]T9 [22:20]TC [19]T8 [18]Attr[2]
     [17]LN [16]TH [15]TD [14]EP [13:12]Attr[1:0] [11:10]AT [9:0]Length

DW1: [31:16]Requester ID [15:0]Tag

DW2: [31:2]Address[31:2] [1:0]First/Last BE
```

**4DW Header (64-bit 地址)**：
```
DW0: [31:30]Fmt [29:24]Type [23]T9 [22:20]TC [19]T8 [18]Attr[2]
     [17]LN [16]TH [15]TD [14]EP [13:12]Attr[1:0] [11:10]AT [9:0]Length

DW1: [31:16]Requester ID [15:0]Tag

DW2: [31:0]Address[63:32]

DW3: [31:2]Address[31:2] [1:0]First/Last BE
```

### Completion TLP

```
DW0: [31:30]Fmt=000/010 [29:24]Type=01010 [23]T9 [22:20]TC [19]T8
     [18]Attr[2] [17]LN [16]TH [15]TD [14]EP [13:12]Attr[1:0]
     [11:10]AT [9:0]Byte Count

DW1: [31:16]Completer ID [15:13]Completion Status [12]BCM
     [11:0]Byte Count (lower)

DW2: [31:16]Requester ID [15:0]Tag

+--- Data Payload (如果 Fmt[0]=1) ---+
```

**Completion Status**：
- `000`：Successful Completion (SC)
- `001`：Unsupported Request (UR)
- `010`：Configuration Request Retry Status (CRS)
- `100`：Completer Abort (CA)

### Message TLP

```
DW0: [31:30]Fmt [29:27]Type=10 [26:24]r [23]T9 [22:20]TC [19]T8
     [18]Attr[2] [17]LN [16]TH [15]TD [14]EP [13:12]Attr[1:0]
     [11:10]AT [9:0]Length

DW1: [31:16]Requester ID [15:0]Tag

DW2: [31:24]Message Code [23:16]r [15:8]Target/Vendor [7:0]r

DW3: [31:0]Lower Address/Data (Message 类型相关)

+--- Data Payload (可选) ---+
```

**常见 Message 类型**：
- `00000`：INTx 中断消息 (ASSERT_INTx/DEASSERT_INTx)
- `00001`：电源管理消息 (PM_Active_State_Nak)
- `00011`：错误消息 (ERR_COR/ERR_NONFATAL/ERR_FATAL)
- `00100`：Locked Memory Read
- `00101`：Slot Power Limit
- `01xxx`：Vendor Defined Messages

## DLLP (Data Link Layer Packet)

### DLLP 格式

```
+----------------+  ----+
|  DLLP Type (8) |      |
+----------------+      |
|   Reserved (8) |      | 16 bits
+----------------+  ----+
|   Seq# (12)    |      |
| DLLP Data (4)  |      | 16 bits
+----------------+  ----+
| CRC-16 DLLP    |      | 16 bits
+----------------+  ----+
```

**总长度**：6 字节（48 位）

### DLLP 类型

| Type | 名称 | 用途 |
|------|------|------|
| 00h | Ack DLLP | 确认 TLP 接收成功 |
| 10h | Nak DLLP | 请求重传 TLP |
| 20h | PM_Enter_L1 | 进入 L1 电源状态 |
| 21h | PM_Enter_L23 | 进入 L23 电源状态 |
| 22h | PM_Active_State_Request_L1 | 请求退出 L1 |
| 23h | PM_Request_Ack | 电源管理请求确认 |
| 30h | Vendor Specific | 厂商自定义 |
| 40h-4Fh | InitFC1-P/NP/Cpl | Flow Control 初始化 |
| 50h-5Fh | UpdateFC-P/NP/Cpl | Flow Control 更新 |

### Ack/Nak DLLP

```
Ack DLLP:
+----------------+
|  Type = 00h    |  DLLP Type
+----------------+
|  Reserved = 0  |
+----------------+
|  AckSeq[11:0]  |  确认序列号
|  Reserved[3:0] |
+----------------+
|  CRC-16        |
+----------------+

Nak DLLP:
+----------------+
|  Type = 10h    |  DLLP Type
+----------------+
|  Reserved = 0  |
+----------------+
|  NakSeq[11:0]  |  请求重传的序列号
|  Reserved[3:0] |
+----------------+
|  CRC-16        |
+----------------+
```

**工作机制**：
- 接收端接收 TLP 成功 → 发送 Ack DLLP
- 接收端检测 TLP 错误 → 发送 Nak DLLP
- 发送端收到 Nak → 从指定序列号重传

### Flow Control DLLP

**InitFC (初始化流控)**：
```
+----------------+
|  Type = 4xh    |  x: 0=P, 4=NP, 8=Cpl
+----------------+
|  HdrFC[7:0]    |  Header Credits
+----------------+
|  DataFC[11:0]  |  Data Credits
|  VC[3:0]       |  Virtual Channel
+----------------+
|  CRC-16        |
+----------------+
```

**UpdateFC (更新流控)**：
```
+----------------+
|  Type = 5xh    |  x: 0=P, 4=NP, 8=Cpl
+----------------+
|  HdrFC[7:0]    |  Header Credits
+----------------+
|  DataFC[11:0]  |  Data Credits
|  VC[3:0]       |  Virtual Channel
+----------------+
|  CRC-16        |
+----------------+
```

## Ordered Set (有序集)

### 基本结构

Ordered Set 是物理层使用的特殊符号序列，用于链路训练、同步和控制。

**格式**：
```
+--------+--------+--------+--------+
|  COM   |  Data0 |  Data1 |  Data2 |
+--------+--------+--------+--------+
```

- **COM (K28.5)**：起始符号，值为 `10111100b` (0xBC)
- **Data0-2**：数据符号或控制符号

### 常用 Ordered Set

**1. TS1 (Training Sequence 1)**：
```
COM, TS1 ID, Lane#, N_FTS, Rate ID, Reserved, ...
```
- 用于链路训练
- 交换链路能力
- 位锁定和符号锁定

**2. TS2 (Training Sequence 2)**：
```
COM, TS2 ID, Lane#, N_FTS, Rate ID, Reserved, ...
```
- 链路训练后期
- 确认链路参数
- 进入 L0 状态前最后一步

**3. FTS (Fast Training Sequence)**：
```
COM, FTS, FTS, FTS
```
- 从 L0s 快速恢复
- 恢复位锁定和符号锁定
- N_FTS 参数决定 FTS 数量

**4. SKIP Ordered Set**：
```
COM, SKP, SKP, SKP
```
- 时钟容差补偿
- 每 1180-1538 个符号插入一次
- 接收端可以添加/删除 SKP

**5. IDLE Ordered Set**：
```
COM, IDL, IDL, IDL
```
- 链路空闲时发送
- 保持链路活跃
- 维持位锁定

**6. EIOS (Electrical Idle Ordered Set)**：
```
COM, EIOS, EIOS, EIOS
```
- 进入电气空闲状态前发送
- 准备进入低功耗状态

**7. EIEOS (Electrical Idle Exit Ordered Set)**：
```
COM, EIEOS, EIEOS, EIEOS (重复 16 次)
```
- 退出电气空闲状态
- 恢复链路活跃

## 数据包封装和解封装

### 发送路径

```
应用层数据
    |
    v
+------------------+
| Transaction Layer |
+------------------+
    | 构造 TLP Header + Data + ECRC
    v
+------------------+
| Data Link Layer  |
+------------------+
    | 添加 Sequence# + LCRC
    v
+------------------+
| Physical Layer   |
+------------------+
    | 添加 Start/End Token, 编码 (8b/10b 或 128b/130b)
    v
物理链路传输
```

### 接收路径

```
物理链路接收
    |
    v
+------------------+
| Physical Layer   |
+------------------+
    | 解码, 移除 Start/End Token
    v
+------------------+
| Data Link Layer  |
+------------------+
    | 校验 LCRC, 发送 Ack/Nak, 移除 Sequence# + LCRC
    v
+------------------+
| Transaction Layer |
+------------------+
    | 校验 ECRC, 解析 TLP Header
    v
应用层数据
```

## 物理层帧结构

### Gen 1/2 (8b/10b 编码)

```
+-------+------+----------+----------+------+-------+
| STP   | TLP  | Seq#     | LCRC     | END  | SDP   |
| (K27.7)| Data | (2 bytes)| (4 bytes)|(K29.7)|(K28.5)|
+-------+------+----------+----------+------+-------+
```

- **STP (Start of TLP)**：K27.7
- **SDP (Start of DLLP)**：K28.5
- **END (End)**：K29.7
- **Seq#**：数据链路层序列号
- **LCRC**：链路层 CRC

### Gen 3+ (128b/130b 编码)

```
+------+-------+------+----------+----------+------+------+
| Sync | Token | TLP  | Seq#     | LCRC     | Token| Sync |
|Header|       | Data | (2 bytes)| (4 bytes)|      |Header|
+------+-------+------+----------+----------+------+------+
```

- **Sync Header (2-bit)**：`01b` 或 `10b`
- **Token**：STP/SDP/END 等
- 数据块：128 位数据 + 2 位同步头

## 实例分析

### Memory Read Request

**场景**：CPU 读取 PCIe 设备的 BAR0 + 0x1000 地址

```
TLP Header (3DW):
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|1|  DW0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  Fmt=000 (3DW, No Data)
      Type=00000 (Memory Read)
          TC=000, Attr=00, Length=1 (4 bytes)

+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  0  1  :  0  0  :  0  0  .  0  |  0  0  :  0  0  :  0  0  1  0 |  DW1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  Requester ID = 01:00.0 (Bus 1, Dev 0, Func 0)
                                    Tag = 0x02

+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Address[31:2] = 0xF8000400                   |1|1|1|1|  DW2
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                                                  FirstBE=0xF
                                                      LastBE=0xF

LCRC: 0x12345678 (示例)
```

### Completion with Data

**场景**：设备返回读取的数据 0xAABBCCDD

```
TLP Header (3DW):
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|0|1|0|0|0|1|0|1|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|1|  DW0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  Fmt=010 (3DW, With Data)
      Type=01010 (Completion)
          TC=000, Attr=00, Byte Count=4

+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  0  2  :  0  0  :  0  0  .  0  |0|0|0|  0  0  0  0  0  0  0  4 |  DW1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  Completer ID = 02:00.0
                                  Status=000 (SC)
                                            Byte Count=4

+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  0  1  :  0  0  :  0  0  .  0  |  0  0  :  0  0  :  0  0  1  0 |  DW2
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  Requester ID = 01:00.0
                                    Tag = 0x02

Data Payload:
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  0xAABBCCDD                                                    |  DW3
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

LCRC: 0x87654321 (示例)
```

### Ack DLLP

```
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  0x00         |  0x00         |  AckSeq=0x123  |r|r|r|r|       | 6 bytes
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  Type=Ack        Reserved        Sequence #         Reserved
                                                            CRC-16
```

## FEMU 实现

### TLP 发送

```c
// hw/block/nvme.c
static void nvme_send_completion(NvmeCtrl *n, NvmeRequest *req)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    NvmeCqe cqe = req->cqe;
    
    // 构造 TLP (由 QEMU PCIe 层自动处理)
    // 这里只需写入 Guest 内存
    pci_dma_write(pci_dev, 
                  cq->dma_addr + cq->tail * sizeof(cqe),
                  &cqe, sizeof(cqe));
    
    // 触发 MSI-X 中断 (发送 Message TLP)
    msix_notify(pci_dev, cq->vector);
}
```

### DMA Read (生成 Memory Read Request TLP)

```c
static int nvme_dma_read(NvmeCtrl *n, uint8_t *ptr, 
                          uint32_t len, uint64_t addr)
{
    PCIDevice *pci_dev = PCI_DEVICE(n);
    
    // 底层会生成 Memory Read Request TLP
    return pci_dma_read(pci_dev, addr, ptr, len);
}
```

## 调试技巧

### 使用 PCIe 分析仪

```
捕获的 TLP:
[00:00:01.123] MRd64 Addr=0xF8001000 Length=64 Req=01:00.0 Tag=0x15
[00:00:01.124] CplD Cpl=02:00.0 Status=SC Req=01:00.0 Tag=0x15 Data=...
[00:00:01.125] Ack AckSeq=0x125
```

### Linux 下追踪

```bash
# 使用 trace-cmd 追踪 PCIe
trace-cmd record -e pci_read_config -e pci_write_config

# 查看 TLP 统计
cat /sys/kernel/debug/pci/0000:01:00.0/stats
```

## 总结

PCIe 三种数据包类型各司其职：

1. **TLP**：负责实际数据传输，实现内存/IO/配置/消息事务
2. **DLLP**：负责链路可靠性，实现 Ack/Nak 和流控
3. **Ordered Set**：负责物理层控制，实现链路训练和同步

参见：
- [TLP 格式详解](../transaction-layer/tlp-format.md)
- [DLLP 格式详解](../data-link-layer/dllp-format.md)
- [LTSSM 状态机](../physical-layer/ltssm.md)
- [链路训练](../physical-layer/link-training.md)
