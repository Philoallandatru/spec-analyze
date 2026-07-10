# 数据对象交换（Data Object Exchange, DOE）

## 概述

数据对象交换（DOE）是 PCIe 6.0 规范中引入的一项重要特性，提供了一种通过 PCIe 配置空间安全交换数据对象的标准化机制。DOE 使得 PCIe 设备能够支持各种安全协议和管理功能，而无需占用宝贵的内存映射 I/O（MMIO）资源。

### DOE 的核心价值

- **安全协议支持**：为 SPDM、CMA 等安全协议提供传输通道
- **标准化接口**：统一的数据交换机制，减少设备特定实现
- **配置空间访问**：通过配置空间进行通信，无需 BAR 资源
- **协议扩展性**：支持多种协议和供应商自定义扩展

## DOE 架构

### Mailbox 机制

DOE 使用 Mailbox（邮箱）模型进行数据交换：

```
┌─────────────────────────────────────┐
│         Host Software               │
│   (SPDM, CMA, IDE 管理层)           │
└──────────────┬──────────────────────┘
               │
    ┌──────────▼──────────┐
    │  DOE Mailbox 接口   │
    │  (配置空间寄存器)    │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │   PCIe 设备逻辑     │
    │  (协议处理器)        │
    └─────────────────────┘
```

**Mailbox 组件：**

1. **Write Mailbox**：主机写入请求数据
2. **Read Mailbox**：主机读取响应数据
3. **Status Register**：状态和控制信息
4. **Control Register**：操作控制

### DOE Extended Capability 结构

```c
// DOE Extended Capability Header
struct doe_cap_header {
    uint16_t cap_id;           // 0x002E (DOE)
    uint16_t cap_version:4;    // Capability 版本
    uint16_t next_offset:12;   // 下一个 Capability 偏移
};

// DOE Capability Register (偏移 0x04)
struct doe_capability {
    uint32_t interrupt_support:1;      // 中断支持
    uint32_t interrupt_message_num:11; // 中断消息号
    uint32_t reserved:20;
};

// DOE Control Register (偏移 0x08)
struct doe_control {
    uint32_t doe_abort:1;        // 中止当前操作
    uint32_t doe_interrupt_en:1; // 中断使能
    uint32_t doe_go:1;           // 启动传输
    uint32_t reserved:29;
};

// DOE Status Register (偏移 0x0C)
struct doe_status {
    uint32_t doe_busy:1;         // Mailbox 忙
    uint32_t doe_interrupt:1;    // 中断状态
    uint32_t doe_error:1;        // 错误状态
    uint32_t data_object_ready:1; // 数据对象就绪
    uint32_t reserved:28;
};

// DOE Write Data Mailbox (偏移 0x10)
// DOE Read Data Mailbox (偏移 0x14)
```

## DOE 协议

### 协议发现

DOE 支持多种协议，设备通过 Discovery 协议报告支持的协议列表：

```c
// DOE Discovery 协议
#define DOE_PROTOCOL_DISCOVERY  0x0000

struct doe_discovery_request {
    uint8_t vendor_id;      // 厂商 ID (0x0001 = PCI-SIG)
    uint8_t protocol;       // 协议类型
    uint8_t index;          // 索引
};

struct doe_discovery_response {
    uint8_t vendor_id;      // 厂商 ID
    uint8_t protocol;       // 支持的协议
    uint8_t next_index;     // 下一个索引
};

// 发现设备支持的所有协议
int doe_discover_protocols(struct pci_dev *pdev, int doe_offset) {
    struct doe_discovery_request req;
    struct doe_discovery_response resp;
    uint8_t index = 0;
    
    dev_info(&pdev->dev, "Discovering DOE protocols:\n");
    
    do {
        req.vendor_id = 0x01;  // PCI-SIG
        req.protocol = 0x00;   // Discovery
        req.index = index;
        
        // 发送 Discovery 请求
        if (doe_send_request(pdev, doe_offset, &req, sizeof(req)) < 0)
            break;
        
        // 接收响应
        if (doe_receive_response(pdev, doe_offset, &resp, sizeof(resp)) < 0)
            break;
        
        if (resp.vendor_id == 0x00)
            break;  // 没有更多协议
        
        dev_info(&pdev->dev, "  Protocol: Vendor=0x%02x, Type=0x%02x\n",
                 resp.vendor_id, resp.protocol);
        
        index = resp.next_index;
    } while (resp.next_index != 0);
    
    return 0;
}
```

### 支持的协议类型

| Vendor ID | Protocol ID | 协议名称 | 描述 |
|-----------|-------------|----------|------|
| 0x0001 | 0x00 | DOE Discovery | 协议发现 |
| 0x0001 | 0x01 | CMA | Component Measurement and Authentication |
| 0x0001 | 0x02 | SPDM | Security Protocol and Data Model |
| 0x0001 | 0x03 | Secured SPDM | 安全 SPDM 消息 |
| 0x0001 | 0x05 | IDE KM | IDE Key Management |
| 0xFFFF | Vendor | Vendor Specific | 厂商自定义协议 |

## DOE 操作流程

### 基本传输流程

```c
// DOE 传输状态机
enum doe_state {
    DOE_IDLE,
    DOE_WRITING,
    DOE_WAITING,
    DOE_READING,
    DOE_ERROR
};

// 发送 DOE 请求
int doe_send_request(struct pci_dev *pdev, int doe_offset,
                     void *request, size_t len) {
    uint32_t status, control;
    uint32_t *data = (uint32_t *)request;
    size_t dwords = (len + 3) / 4;
    int i;
    
    // 1. 检查 Mailbox 是否空闲
    pci_read_config_dword(pdev, doe_offset + DOE_STATUS, &status);
    if (status & DOE_STATUS_BUSY) {
        dev_err(&pdev->dev, "DOE Mailbox busy\n");
        return -EBUSY;
    }
    
    // 2. 写入请求数据到 Write Mailbox
    for (i = 0; i < dwords; i++) {
        pci_write_config_dword(pdev, doe_offset + DOE_WRITE_MAILBOX,
                              data[i]);
    }
    
    // 3. 设置 DOE Go 位启动传输
    pci_read_config_dword(pdev, doe_offset + DOE_CONTROL, &control);
    control |= DOE_CONTROL_GO;
    pci_write_config_dword(pdev, doe_offset + DOE_CONTROL, control);
    
    dev_dbg(&pdev->dev, "DOE request sent (%zu dwords)\n", dwords);
    return 0;
}

// 接收 DOE 响应
int doe_receive_response(struct pci_dev *pdev, int doe_offset,
                        void *response, size_t max_len) {
    uint32_t status;
    uint32_t *data = (uint32_t *)response;
    size_t dwords = 0;
    int timeout = 1000;  // 1 秒超时
    
    // 1. 等待 Data Object Ready
    do {
        pci_read_config_dword(pdev, doe_offset + DOE_STATUS, &status);
        
        if (status & DOE_STATUS_ERROR) {
            dev_err(&pdev->dev, "DOE error detected\n");
            return -EIO;
        }
        
        if (status & DOE_STATUS_DATA_OBJECT_READY)
            break;
        
        usleep_range(1000, 2000);
    } while (--timeout > 0);
    
    if (timeout == 0) {
        dev_err(&pdev->dev, "DOE response timeout\n");
        return -ETIMEDOUT;
    }
    
    // 2. 从 Read Mailbox 读取响应
    // 第一个 DWORD 包含对象长度
    pci_read_config_dword(pdev, doe_offset + DOE_READ_MAILBOX, &data[0]);
    dwords = 1;
    
    // 继续读取剩余数据
    while (dwords * 4 < max_len) {
        pci_read_config_dword(pdev, doe_offset + DOE_STATUS, &status);
        if (!(status & DOE_STATUS_DATA_OBJECT_READY))
            break;
        
        pci_read_config_dword(pdev, doe_offset + DOE_READ_MAILBOX,
                             &data[dwords]);
        dwords++;
    }
    
    dev_dbg(&pdev->dev, "DOE response received (%zu dwords)\n", dwords);
    return dwords * 4;
}
```

### 中断模式

DOE 支持中断驱动的操作模式：

```c
// DOE 中断处理
struct doe_mailbox {
    struct pci_dev *pdev;
    int doe_offset;
    wait_queue_head_t wait_queue;
    bool data_ready;
    spinlock_t lock;
};

irqreturn_t doe_interrupt_handler(int irq, void *data) {
    struct doe_mailbox *mbox = data;
    uint32_t status;
    unsigned long flags;
    
    spin_lock_irqsave(&mbox->lock, flags);
    
    // 读取状态
    pci_read_config_dword(mbox->pdev, mbox->doe_offset + DOE_STATUS,
                         &status);
    
    if (status & DOE_STATUS_INTERRUPT) {
        // 清除中断标志
        pci_write_config_dword(mbox->pdev, mbox->doe_offset + DOE_STATUS,
                              DOE_STATUS_INTERRUPT);
        
        if (status & DOE_STATUS_DATA_OBJECT_READY) {
            mbox->data_ready = true;
            wake_up(&mbox->wait_queue);
        }
        
        spin_unlock_irqrestore(&mbox->lock, flags);
        return IRQ_HANDLED;
    }
    
    spin_unlock_irqrestore(&mbox->lock, flags);
    return IRQ_NONE;
}

// 使能 DOE 中断
int doe_enable_interrupt(struct doe_mailbox *mbox) {
    uint32_t control;
    
    pci_read_config_dword(mbox->pdev, mbox->doe_offset + DOE_CONTROL,
                         &control);
    control |= DOE_CONTROL_INTERRUPT_EN;
    pci_write_config_dword(mbox->pdev, mbox->doe_offset + DOE_CONTROL,
                          control);
    
    return 0;
}
```

## CMA（Component Measurement and Authentication）

CMA 协议允许系统测量和认证 PCIe 组件：

```c
// CMA 协议定义
#define DOE_PROTOCOL_CMA  0x01

// CMA 请求类型
enum cma_request_type {
    CMA_REQ_GET_CERTIFICATE_CHAIN = 0x01,
    CMA_REQ_GET_MEASUREMENT = 0x02,
    CMA_REQ_CHALLENGE = 0x03
};

// CMA 证书链请求
struct cma_certificate_request {
    uint8_t vendor_id;
    uint8_t protocol;  // 0x01 for CMA
    uint16_t length;
    uint8_t slot_id;
    uint16_t offset;
    uint16_t length_requested;
};

struct cma_certificate_response {
    uint8_t vendor_id;
    uint8_t protocol;
    uint16_t length;
    uint16_t certificate_chain_length;
    uint16_t offset;
    uint8_t certificate_data[];  // 可变长度
};

// 获取设备证书链
int cma_get_certificate_chain(struct pci_dev *pdev, int doe_offset,
                              uint8_t slot_id, uint8_t *cert_buffer,
                              size_t buffer_size) {
    struct cma_certificate_request req;
    struct cma_certificate_response *resp;
    uint8_t resp_buffer[4096];
    uint16_t offset = 0;
    uint16_t total_length = 0;
    int ret;
    
    do {
        // 构造请求
        req.vendor_id = 0x01;
        req.protocol = DOE_PROTOCOL_CMA;
        req.length = sizeof(req) / 4 - 2;  // 长度以 DWORD 计
        req.slot_id = slot_id;
        req.offset = offset;
        req.length_requested = min(1024, (int)(buffer_size - offset));
        
        // 发送请求
        ret = doe_send_request(pdev, doe_offset, &req, sizeof(req));
        if (ret < 0)
            return ret;
        
        // 接收响应
        ret = doe_receive_response(pdev, doe_offset, resp_buffer,
                                  sizeof(resp_buffer));
        if (ret < 0)
            return ret;
        
        resp = (struct cma_certificate_response *)resp_buffer;
        
        if (total_length == 0)
            total_length = resp->certificate_chain_length;
        
        // 复制证书数据
        size_t copy_len = min((size_t)(total_length - offset),
                             (size_t)req.length_requested);
        memcpy(cert_buffer + offset, resp->certificate_data, copy_len);
        
        offset += copy_len;
        
    } while (offset < total_length && offset < buffer_size);
    
    dev_info(&pdev->dev, "Retrieved %u bytes of certificate chain\n", offset);
    return offset;
}
```

## SPDM（Security Protocol and Data Model）

SPDM 是 DMTF 定义的安全协议，通过 DOE 传输：

```c
// SPDM 协议定义
#define DOE_PROTOCOL_SPDM  0x02

// SPDM 消息类型
enum spdm_message_type {
    SPDM_GET_VERSION = 0x84,
    SPDM_GET_CAPABILITIES = 0xE1,
    SPDM_NEGOTIATE_ALGORITHMS = 0xE3,
    SPDM_GET_DIGESTS = 0x81,
    SPDM_GET_CERTIFICATE = 0x82,
    SPDM_CHALLENGE = 0x83,
    SPDM_GET_MEASUREMENTS = 0xE0
};

// SPDM 消息头
struct spdm_message_header {
    uint8_t version;        // SPDM 版本
    uint8_t request_code;   // 请求代码
    uint8_t param1;
    uint8_t param2;
};

// SPDM 版本请求
struct spdm_get_version_request {
    struct spdm_message_header header;
};

struct spdm_version_entry {
    uint16_t major:8;
    uint16_t minor:8;
};

struct spdm_get_version_response {
    struct spdm_message_header header;
    uint8_t reserved;
    uint8_t version_count;
    struct spdm_version_entry versions[];
};

// 获取 SPDM 版本
int spdm_get_version(struct pci_dev *pdev, int doe_offset) {
    struct spdm_get_version_request req;
    struct spdm_get_version_response *resp;
    uint8_t resp_buffer[256];
    int ret, i;
    
    // 构造请求
    memset(&req, 0, sizeof(req));
    req.header.version = 0x10;  // SPDM 1.0
    req.header.request_code = SPDM_GET_VERSION;
    
    // 通过 DOE 发送
    ret = doe_send_request(pdev, doe_offset, &req, sizeof(req));
    if (ret < 0)
        return ret;
    
    ret = doe_receive_response(pdev, doe_offset, resp_buffer,
                              sizeof(resp_buffer));
    if (ret < 0)
        return ret;
    
    resp = (struct spdm_get_version_response *)resp_buffer;
    
    dev_info(&pdev->dev, "SPDM supported versions:\n");
    for (i = 0; i < resp->version_count; i++) {
        dev_info(&pdev->dev, "  Version %d.%d\n",
                 resp->versions[i].major, resp->versions[i].minor);
    }
    
    return 0;
}
```

## IDE Key Management (IDE KM)

IDE KM 协议用于管理 IDE（Integrity and Data Encryption）密钥：

```c
// IDE KM 协议
#define DOE_PROTOCOL_IDE_KM  0x05

// IDE KM 消息类型
enum ide_km_message_type {
    IDE_KM_QUERY = 0x00,
    IDE_KM_KEY_PROG = 0x01,
    IDE_KM_KEY_SET_GO = 0x02,
    IDE_KM_KEY_SET_STOP = 0x03
};

// IDE KM 密钥编程
struct ide_km_key_prog {
    uint8_t vendor_id;
    uint8_t protocol;
    uint16_t length;
    uint8_t stream_id;
    uint8_t reserved;
    uint8_t key_sub_stream;
    uint8_t port_index;
    uint8_t key[32];  // AES-256 密钥
    uint8_t iv[12];   // GCM IV
};

// 编程 IDE 密钥
int ide_km_program_key(struct pci_dev *pdev, int doe_offset,
                      uint8_t stream_id, const uint8_t *key,
                      const uint8_t *iv) {
    struct ide_km_key_prog req;
    uint8_t resp_buffer[64];
    int ret;
    
    // 构造密钥编程请求
    memset(&req, 0, sizeof(req));
    req.vendor_id = 0x01;
    req.protocol = DOE_PROTOCOL_IDE_KM;
    req.length = (sizeof(req) / 4) - 2;
    req.stream_id = stream_id;
    req.key_sub_stream = 0;
    req.port_index = 0;
    
    memcpy(req.key, key, 32);
    memcpy(req.iv, iv, 12);
    
    // 发送请求
    ret = doe_send_request(pdev, doe_offset, &req, sizeof(req));
    if (ret < 0) {
        dev_err(&pdev->dev, "Failed to send IDE KM key programming\n");
        return ret;
    }
    
    // 接收响应
    ret = doe_receive_response(pdev, doe_offset, resp_buffer,
                              sizeof(resp_buffer));
    if (ret < 0) {
        dev_err(&pdev->dev, "Failed to receive IDE KM response\n");
        return ret;
    }
    
    dev_info(&pdev->dev, "IDE key programmed for stream %u\n", stream_id);
    return 0;
}
```

## 最佳实践

### 1. DOE 初始化

```c
// DOE Mailbox 初始化
int doe_mailbox_init(struct pci_dev *pdev, struct doe_mailbox *mbox) {
    int pos;
    uint32_t cap;
    
    // 查找 DOE Extended Capability
    pos = pci_find_ext_capability(pdev, PCI_EXT_CAP_ID_DOE);
    if (!pos) {
        dev_info(&pdev->dev, "DOE capability not found\n");
        return -ENODEV;
    }
    
    mbox->pdev = pdev;
    mbox->doe_offset = pos;
    mbox->data_ready = false;
    spin_lock_init(&mbox->lock);
    init_waitqueue_head(&mbox->wait_queue);
    
    // 读取 Capability
    pci_read_config_dword(pdev, pos + DOE_CAPABILITY, &cap);
    
    if (cap & DOE_CAP_INTERRUPT_SUPPORT) {
        dev_info(&pdev->dev, "DOE interrupt supported\n");
        // 设置中断处理程序
        // doe_enable_interrupt(mbox);
    }
    
    // 发现支持的协议
    doe_discover_protocols(pdev, pos);
    
    return 0;
}
```

### 2. 错误处理

```c
// DOE 错误恢复
int doe_abort_and_reset(struct pci_dev *pdev, int doe_offset) {
    uint32_t control, status;
    int timeout = 100;
    
    // 设置 Abort 位
    pci_read_config_dword(pdev, doe_offset + DOE_CONTROL, &control);
    control |= DOE_CONTROL_ABORT;
    pci_write_config_dword(pdev, doe_offset + DOE_CONTROL, control);
    
    // 等待 Busy 位清除
    do {
        pci_read_config_dword(pdev, doe_offset + DOE_STATUS, &status);
        if (!(status & DOE_STATUS_BUSY))
            break;
        msleep(10);
    } while (--timeout > 0);
    
    if (timeout == 0) {
        dev_err(&pdev->dev, "DOE abort timeout\n");
        return -ETIMEDOUT;
    }
    
    // 清除 Error 位
    if (status & DOE_STATUS_ERROR) {
        pci_write_config_dword(pdev, doe_offset + DOE_STATUS,
                              DOE_STATUS_ERROR);
    }
    
    dev_info(&pdev->dev, "DOE mailbox reset successfully\n");
    return 0;
}
```

### 3. 性能优化

```c
// DOE 批量操作
int doe_batch_operations(struct pci_dev *pdev, int doe_offset,
                        struct doe_operation *ops, int count) {
    int i, ret;
    
    for (i = 0; i < count; i++) {
        ret = doe_send_request(pdev, doe_offset,
                              ops[i].request, ops[i].req_len);
        if (ret < 0) {
            dev_err(&pdev->dev, "Batch operation %d failed\n", i);
            return ret;
        }
        
        ret = doe_receive_response(pdev, doe_offset,
                                  ops[i].response, ops[i].resp_len);
        if (ret < 0) {
            return ret;
        }
        
        ops[i].completed = true;
    }
    
    return 0;
}
```

### 4. 安全考虑

```c
// DOE 安全验证
int doe_verify_response(struct doe_mailbox *mbox,
                       void *request, size_t req_len,
                       void *response, size_t resp_len) {
    // 验证响应长度
    if (resp_len < sizeof(struct doe_header)) {
        dev_err(&mbox->pdev->dev, "Response too short\n");
        return -EINVAL;
    }
    
    // 验证协议匹配
    struct doe_header *req_hdr = request;
    struct doe_header *resp_hdr = response;
    
    if (req_hdr->vendor_id != resp_hdr->vendor_id ||
        req_hdr->protocol != resp_hdr->protocol) {
        dev_err(&mbox->pdev->dev, "Protocol mismatch in response\n");
        return -EPROTO;
    }
    
    // 可以添加更多验证，如 CRC、签名等
    
    return 0;
}
```

## 应用场景

### 1. 设备认证

使用 CMA/SPDM 在系统启动时认证 PCIe 设备：

```c
int authenticate_pcie_device(struct pci_dev *pdev) {
    struct doe_mailbox mbox;
    uint8_t cert_buffer[8192];
    int ret;
    
    // 初始化 DOE
    ret = doe_mailbox_init(pdev, &mbox);
    if (ret < 0)
        return ret;
    
    // 获取 SPDM 版本
    ret = spdm_get_version(pdev, mbox.doe_offset);
    if (ret < 0)
        return ret;
    
    // 获取设备证书
    ret = cma_get_certificate_chain(pdev, mbox.doe_offset, 0,
                                   cert_buffer, sizeof(cert_buffer));
    if (ret < 0)
        return ret;
    
    // 验证证书链（需要加密库支持）
    // ret = verify_certificate_chain(cert_buffer, ret);
    
    dev_info(&pdev->dev, "Device authentication completed\n");
    return 0;
}
```

### 2. 运行时密钥管理

动态更新 IDE 加密密钥：

```c
int rotate_ide_keys(struct pci_dev *pdev, uint8_t stream_id) {
    struct doe_mailbox mbox;
    uint8_t new_key[32], new_iv[12];
    int ret;
    
    doe_mailbox_init(pdev, &mbox);
    
    // 生成新密钥（使用硬件 RNG）
    get_random_bytes(new_key, sizeof(new_key));
    get_random_bytes(new_iv, sizeof(new_iv));
    
    // 编程新密钥
    ret = ide_km_program_key(pdev, mbox.doe_offset, stream_id,
                            new_key, new_iv);
    if (ret < 0)
        return ret;
    
    // 激活新密钥
    // ide_km_key_set_go(pdev, mbox.doe_offset, stream_id);
    
    dev_info(&pdev->dev, "IDE keys rotated for stream %u\n", stream_id);
    return 0;
}
```

## 总结

DOE 为 PCIe 6.0 及更高版本提供了强大的数据交换机制，特别是在安全和设备管理方面。通过支持 CMA、SPDM、IDE KM 等协议，DOE 使得 PCIe 设备能够实现：

- **设备认证**：在启动时验证设备身份
- **安全通信**：建立加密的通信通道
- **密钥管理**：动态配置和轮换加密密钥
- **度量和证明**：获取设备固件度量值

随着数据中心安全要求的提高，DOE 将成为现代 PCIe 设备的标准配置，为构建可信计算环境提供基础支持。
