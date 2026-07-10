# 完整性和数据加密（Integrity and Data Encryption, IDE）

## 概述

完整性和数据加密（IDE）是 PCIe 6.0 规范中引入的关键安全特性，旨在保护 PCIe 链路上传输的数据免受窃听、篡改和重放攻击。IDE 提供了硬件级别的加密和完整性保护，确保数据在 PCIe 拓扑结构中的传输安全。

### IDE 的核心价值

- **数据机密性**：使用 AES-GCM 加密 TLP 负载，防止数据泄露
- **数据完整性**：通过 MAC（Message Authentication Code）验证数据未被篡改
- **重放保护**：使用序列号防止重放攻击
- **灵活配置**：支持选择性加密和端到端保护

## IDE 架构

### 加密范围

IDE 定义了三种不同的加密范围：

```
┌────────────────────────────────────────────────────────┐
│                    PCIe 拓扑结构                        │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌──────────┐  Link IDE  ┌────────┐  Link IDE  ┌────┐│
│  │   CPU    │◄──────────►│ Switch │◄──────────►│ EP ││
│  │   RC     │            │        │            │    ││
│  └──────────┘            └────────┘            └────┘│
│       ▲                                          ▲    │
│       │                                          │    │
│       └──────── Selective IDE ──────────────────┘    │
│       └──────── End-to-End IDE ──────────────────┘   │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### 1. Link IDE

Link IDE 保护单个 PCIe 链路上的数据：

- **范围**：两个直接相连的端口之间
- **密钥**：每个链路独立的密钥
- **用途**：防止物理探测和链路窃听

```c
// Link IDE 配置
struct link_ide_config {
    uint8_t stream_id;           // 流 ID
    bool tx_enable;              // 发送加密使能
    bool rx_enable;              // 接收解密使能
    uint8_t key[32];            // AES-256 密钥
    uint8_t iv[12];             // GCM IV
    uint32_t tx_pn;             // 发送包号
    uint32_t rx_pn;             // 接收包号
};
```

### 2. Selective IDE

Selective IDE 允许选择性地加密特定的地址范围或 TLP 类型：

- **范围**：从 Root Complex 到 Endpoint
- **灵活性**：可以只加密敏感数据
- **性能**：减少不必要的加密开销

```c
// Selective IDE 地址范围
struct selective_ide_range {
    uint64_t base_address;       // 起始地址
    uint64_t limit_address;      // 结束地址
    bool memory_read_enable;     // 内存读加密
    bool memory_write_enable;    // 内存写加密
    uint8_t stream_id;          // 关联的流 ID
};
```

### 3. End-to-End IDE（未来扩展）

端到端 IDE 提供跨多个链路的持续保护：

- **范围**：从源到目的地的整个路径
- **一致性**：保持相同的加密上下文
- **复杂性**：需要 Switch 的透明传递支持

## IDE 协议规范

### TLP 加密格式

IDE 使用 AES-256-GCM 算法加密 TLP：

```
原始 TLP:
┌─────────┬──────────┬─────────┬──────┐
│ Header  │ Payload  │ Digest  │ LCRC │
└─────────┴──────────┴─────────┴──────┘

加密后的 TLP (IDE):
┌─────────┬────────────────┬─────────┬──────┐
│ Header  │ Encrypted      │ MAC Tag │ LCRC │
│         │ Payload+Digest │ (16B)   │      │
└─────────┴────────────────┴─────────┴──────┘

AAD (Additional Authenticated Data):
- TLP Header (未加密，但包含在 MAC 计算中)
- Packet Number
- Stream ID
```

### 加密算法实现

```c
#include <linux/crypto.h>
#include <crypto/aead.h>

// IDE 加密上下文
struct ide_crypto_context {
    struct crypto_aead *tfm;     // AES-GCM 转换
    uint8_t key[32];            // 密钥
    uint8_t iv[12];             // IV
    uint32_t pn;                // 包号（Packet Number）
    spinlock_t lock;            // 保护包号
};

// 初始化 IDE 加密上下文
int ide_crypto_init(struct ide_crypto_context *ctx, const uint8_t *key) {
    int ret;
    
    // 分配 AES-GCM 转换
    ctx->tfm = crypto_alloc_aead("gcm(aes)", 0, 0);
    if (IS_ERR(ctx->tfm)) {
        ret = PTR_ERR(ctx->tfm);
        pr_err("Failed to allocate AES-GCM: %d\n", ret);
        return ret;
    }
    
    // 设置密钥
    memcpy(ctx->key, key, 32);
    ret = crypto_aead_setkey(ctx->tfm, ctx->key, 32);
    if (ret < 0) {
        pr_err("Failed to set key: %d\n", ret);
        crypto_free_aead(ctx->tfm);
        return ret;
    }
    
    // 设置认证标签长度（16 字节）
    ret = crypto_aead_setauthsize(ctx->tfm, 16);
    if (ret < 0) {
        pr_err("Failed to set auth size: %d\n", ret);
        crypto_free_aead(ctx->tfm);
        return ret;
    }
    
    // 初始化包号
    ctx->pn = 0;
    spin_lock_init(&ctx->lock);
    
    return 0;
}

// IDE TLP 加密
int ide_encrypt_tlp(struct ide_crypto_context *ctx,
                   const uint8_t *tlp_header, size_t header_len,
                   const uint8_t *payload, size_t payload_len,
                   uint8_t *encrypted_payload, uint8_t *mac_tag) {
    struct aead_request *req;
    struct scatterlist sg[3];
    uint8_t iv[12];
    uint32_t pn;
    int ret;
    
    // 获取并递增包号
    spin_lock(&ctx->lock);
    pn = ctx->pn++;
    spin_unlock(&ctx->lock);
    
    // 构造 IV (Nonce)
    memcpy(iv, ctx->iv, 8);
    *(uint32_t*)(iv + 8) = cpu_to_be32(pn);
    
    // 分配请求
    req = aead_request_alloc(ctx->tfm, GFP_KERNEL);
    if (!req)
        return -ENOMEM;
    
    // 设置 scatter-gather 列表
    // sg[0]: AAD (TLP Header)
    // sg[1]: Payload (输入/输出)
    // sg[2]: MAC Tag (输出)
    sg_init_table(sg, 3);
    sg_set_buf(&sg[0], tlp_header, header_len);
    sg_set_buf(&sg[1], payload, payload_len);
    sg_set_buf(&sg[2], mac_tag, 16);
    
    // 配置请求
    aead_request_set_ad(req, header_len);
    aead_request_set_crypt(req, sg, sg, payload_len, iv);
    
    // 执行加密
    ret = crypto_aead_encrypt(req);
    
    aead_request_free(req);
    return ret;
}

// IDE TLP 解密
int ide_decrypt_tlp(struct ide_crypto_context *ctx,
                   const uint8_t *tlp_header, size_t header_len,
                   const uint8_t *encrypted_payload, size_t payload_len,
                   const uint8_t *mac_tag,
                   uint8_t *decrypted_payload) {
    struct aead_request *req;
    struct scatterlist sg[3];
    uint8_t iv[12];
    uint32_t pn;
    int ret;
    
    // 从 TLP 中提取包号（假设在特定位置）
    // 实际实现需要根据 IDE 规范确定包号位置
    pn = *(uint32_t*)(tlp_header + header_len - 4);
    
    // 构造 IV
    memcpy(iv, ctx->iv, 8);
    *(uint32_t*)(iv + 8) = cpu_to_be32(pn);
    
    // 分配请求
    req = aead_request_alloc(ctx->tfm, GFP_KERNEL);
    if (!req)
        return -ENOMEM;
    
    // 设置 scatter-gather 列表
    sg_init_table(sg, 3);
    sg_set_buf(&sg[0], tlp_header, header_len);
    sg_set_buf(&sg[1], encrypted_payload, payload_len);
    sg_set_buf(&sg[2], mac_tag, 16);
    
    // 配置请求
    aead_request_set_ad(req, header_len);
    aead_request_set_crypt(req, sg, sg, payload_len + 16, iv);
    
    // 执行解密（自动验证 MAC）
    ret = crypto_aead_decrypt(req);
    if (ret < 0) {
        pr_err("IDE decryption failed: %d (possible tampering)\n", ret);
    }
    
    aead_request_free(req);
    return ret;
}
```

## IDE 配置寄存器

### IDE Extended Capability 结构

```c
// IDE Extended Capability Header
#define PCI_EXT_CAP_ID_IDE  0x0030

// IDE Capability Register (偏移 0x04)
struct ide_capability {
    uint32_t link_ide_supported:1;      // Link IDE 支持
    uint32_t selective_ide_supported:1; // Selective IDE 支持
    uint32_t flow_through_ide:1;        // Flow-through IDE
    uint32_t partial_header_encrypt:1;  // 部分头加密
    uint32_t aggregation_supported:1;   // 聚合支持
    uint32_t pcrc_supported:1;          // PCRC 支持
    uint32_t ide_km_supported:1;        // IDE KM 协议支持
    uint32_t ide_algo_supported:8;      // 支持的算法
    uint32_t num_tcs:4;                 // TC 数量
    uint32_t num_streams:8;             // 流数量
    uint32_t reserved:5;
};

// IDE Control Register (偏移 0x08)
struct ide_control {
    uint32_t flow_through_ide_enable:1;  // Flow-through 使能
    uint32_t reserved:31;
};

// Link IDE Stream Control Register
struct link_ide_stream_control {
    uint32_t enable:1;                   // 流使能
    uint32_t tx_aggregation_mode:2;      // 发送聚合模式
    uint32_t pr_enable:1;                // 重放保护使能
    uint32_t algorithm:8;                // 算法选择
    uint32_t tc_bitmap:8;                // TC 位图
    uint32_t stream_id:8;                // 流 ID
    uint32_t reserved:4;
};

// Selective IDE Address Association Register
struct selective_ide_address {
    uint64_t base_address:52;            // 基地址 (4K 对齐)
    uint64_t limit_address_low:12;
    uint64_t limit_address_high:40;
    uint32_t valid:1;                    // 有效位
    uint32_t stream_id:8;                // 关联的流 ID
    uint32_t reserved:3;
};
```

### 驱动配置示例

```c
// 查找 IDE Capability
int find_ide_capability(struct pci_dev *pdev) {
    int pos = pci_find_ext_capability(pdev, PCI_EXT_CAP_ID_IDE);
    if (!pos) {
        dev_info(&pdev->dev, "IDE capability not found\n");
        return -ENODEV;
    }
    return pos;
}

// 配置 Link IDE 流
int configure_link_ide_stream(struct pci_dev *pdev, uint8_t stream_id,
                              const uint8_t *key) {
    int pos, ret;
    uint32_t cap, ctrl;
    struct doe_mailbox mbox;
    
    pos = find_ide_capability(pdev);
    if (pos < 0)
        return pos;
    
    // 检查 Link IDE 支持
    pci_read_config_dword(pdev, pos + IDE_CAPABILITY, &cap);
    if (!(cap & IDE_CAP_LINK_IDE_SUPPORTED)) {
        dev_err(&pdev->dev, "Link IDE not supported\n");
        return -EOPNOTSUPP;
    }
    
    // 通过 DOE 编程密钥
    ret = doe_mailbox_init(pdev, &mbox);
    if (ret < 0)
        return ret;
    
    uint8_t iv[12];
    get_random_bytes(iv, sizeof(iv));
    
    ret = ide_km_program_key(pdev, mbox.doe_offset, stream_id, key, iv);
    if (ret < 0) {
        dev_err(&pdev->dev, "Failed to program IDE key\n");
        return ret;
    }
    
    // 配置流控制寄存器
    ctrl = 0;
    ctrl |= (1 << 0);                    // Enable
    ctrl |= (1 << 3);                    // PR Enable (重放保护)
    ctrl |= (0 << 4);                    // 算法：AES-GCM-256
    ctrl |= (0xFF << 12);                // 所有 TC
    ctrl |= (stream_id << 20);           // 流 ID
    
    pci_write_config_dword(pdev, pos + LINK_IDE_STREAM_CTRL(stream_id),
                          ctrl);
    
    dev_info(&pdev->dev, "Link IDE stream %u configured\n", stream_id);
    return 0;
}

// 配置 Selective IDE 地址范围
int configure_selective_ide_range(struct pci_dev *pdev,
                                 uint64_t base, uint64_t limit,
                                 uint8_t stream_id) {
    int pos;
    uint32_t cap;
    
    pos = find_ide_capability(pdev);
    if (pos < 0)
        return pos;
    
    // 检查 Selective IDE 支持
    pci_read_config_dword(pdev, pos + IDE_CAPABILITY, &cap);
    if (!(cap & IDE_CAP_SELECTIVE_IDE_SUPPORTED)) {
        dev_err(&pdev->dev, "Selective IDE not supported\n");
        return -EOPNOTSUPP;
    }
    
    // 地址必须 4K 对齐
    if ((base & 0xFFF) || (limit & 0xFFF)) {
        dev_err(&pdev->dev, "Addresses must be 4K aligned\n");
        return -EINVAL;
    }
    
    // 写入地址范围寄存器
    uint64_t base_reg = (base >> 12) & 0xFFFFFFFFFFFFF;
    uint64_t limit_low = (limit >> 12) & 0xFFF;
    uint64_t limit_high = (limit >> 24) & 0xFFFFFFFFFF;
    
    pci_write_config_dword(pdev, pos + SELECTIVE_IDE_ADDR_BASE_LOW(0),
                          (uint32_t)base_reg);
    pci_write_config_dword(pdev, pos + SELECTIVE_IDE_ADDR_BASE_HIGH(0),
                          (uint32_t)(base_reg >> 32));
    pci_write_config_dword(pdev, pos + SELECTIVE_IDE_ADDR_LIMIT(0),
                          (limit_low | (stream_id << 12) | (1 << 31)));
    
    dev_info(&pdev->dev, "Selective IDE range configured: 0x%llx-0x%llx\n",
             base, limit);
    return 0;
}
```

## 密钥管理

### 密钥生成

```c
// 生成 IDE 密钥
int generate_ide_key(uint8_t *key, uint8_t *iv) {
    // 使用硬件 RNG 生成密钥
    get_random_bytes(key, 32);  // AES-256
    get_random_bytes(iv, 12);   // GCM IV (96 位)
    
    // 在生产环境中，应使用 KDF (Key Derivation Function)
    // 从根密钥派生，例如：
    // derive_key(root_key, context, key, 32);
    
    return 0;
}
```

### 密钥轮换

```c
// IDE 密钥轮换
struct ide_key_rotation {
    struct pci_dev *pdev;
    uint8_t stream_id;
    uint32_t rotation_interval_sec;  // 轮换间隔（秒）
    struct timer_list timer;
};

void ide_key_rotation_timer(struct timer_list *t) {
    struct ide_key_rotation *rotation = from_timer(rotation, t, timer);
    uint8_t new_key[32], new_iv[12];
    int ret;
    
    // 生成新密钥
    generate_ide_key(new_key, new_iv);
    
    // 编程新密钥
    ret = configure_link_ide_stream(rotation->pdev, rotation->stream_id,
                                   new_key);
    if (ret < 0) {
        dev_err(&rotation->pdev->dev, "Key rotation failed: %d\n", ret);
    } else {
        dev_info(&rotation->pdev->dev, "IDE key rotated for stream %u\n",
                 rotation->stream_id);
    }
    
    // 安全清除旧密钥
    memzero_explicit(new_key, sizeof(new_key));
    
    // 重新设置定时器
    mod_timer(&rotation->timer,
              jiffies + rotation->rotation_interval_sec * HZ);
}

// 启动密钥轮换
int start_ide_key_rotation(struct pci_dev *pdev, uint8_t stream_id,
                          uint32_t interval_sec) {
    struct ide_key_rotation *rotation;
    
    rotation = kzalloc(sizeof(*rotation), GFP_KERNEL);
    if (!rotation)
        return -ENOMEM;
    
    rotation->pdev = pdev;
    rotation->stream_id = stream_id;
    rotation->rotation_interval_sec = interval_sec;
    
    timer_setup(&rotation->timer, ide_key_rotation_timer, 0);
    mod_timer(&rotation->timer, jiffies + interval_sec * HZ);
    
    dev_info(&pdev->dev, "IDE key rotation started (interval: %u sec)\n",
             interval_sec);
    return 0;
}
```

## 性能影响

### 延迟开销

IDE 加密/解密会增加一定的延迟：

```c
// IDE 性能基准测试
struct ide_perf_stats {
    uint64_t tlps_encrypted;
    uint64_t tlps_decrypted;
    uint64_t total_encrypt_ns;
    uint64_t total_decrypt_ns;
    uint64_t errors;
};

void measure_ide_performance(struct pci_dev *pdev) {
    struct ide_perf_stats stats = {0};
    ktime_t start, end;
    uint8_t test_data[4096];
    uint8_t encrypted[4096];
    uint8_t mac[16];
    int i;
    
    for (i = 0; i < 10000; i++) {
        start = ktime_get();
        ide_encrypt_tlp(ctx, header, 16, test_data, sizeof(test_data),
                       encrypted, mac);
        end = ktime_get();
        
        stats.tlps_encrypted++;
        stats.total_encrypt_ns += ktime_to_ns(ktime_sub(end, start));
    }
    
    dev_info(&pdev->dev, "IDE Performance:\n");
    dev_info(&pdev->dev, "  Avg encryption time: %llu ns\n",
             stats.total_encrypt_ns / stats.tlps_encrypted);
    dev_info(&pdev->dev, "  Throughput: ~%llu Gbps\n",
             (uint64_t)(stats.tlps_encrypted * 4096 * 8) /
             (stats.total_encrypt_ns));
}
```

**典型性能影响：**
- 加密延迟：50-100 ns per TLP
- 吞吐量降低：< 5% (硬件加速器)
- CPU 开销：最小（硬件卸载）

## 应用场景

### 1. 机密计算

保护加速器处理的敏感数据：

```c
// 配置 GPU 的 IDE 保护
int secure_gpu_memory(struct pci_dev *gpu, uint64_t base, size_t size) {
    uint8_t key[32];
    int ret;
    
    // 生成专用密钥
    generate_ide_key(key, NULL);
    
    // 配置 Link IDE
    ret = configure_link_ide_stream(gpu, 0, key);
    if (ret < 0)
        return ret;
    
    // 配置 Selective IDE 保护 GPU 内存范围
    ret = configure_selective_ide_range(gpu, base, base + size - 1, 0);
    if (ret < 0)
        return ret;
    
    dev_info(&gpu->dev, "GPU memory secured with IDE\n");
    return 0;
}
```

### 2. 数据中心安全

防止物理访问攻击：

```c
// 系统级 IDE 部署
int deploy_datacenter_ide(void) {
    struct pci_dev *pdev = NULL;
    
    // 为所有支持 IDE 的设备启用加密
    for_each_pci_dev(pdev) {
        if (find_ide_capability(pdev) >= 0) {
            uint8_t key[32];
            generate_ide_key(key, NULL);
            configure_link_ide_stream(pdev, 0, key);
            
            // 启动密钥轮换（每小时）
            start_ide_key_rotation(pdev, 0, 3600);
        }
    }
    
    pr_info("IDE deployed across datacenter\n");
    return 0;
}
```

### 3. 金融和医疗设备

满足合规要求：

```c
// FIPS 140-3 兼容的 IDE 配置
int configure_fips_ide(struct pci_dev *pdev) {
    uint8_t key[32];
    int ret;
    
    // 使用 FIPS 认证的 RNG
    ret = fips_generate_key(key, 32);
    if (ret < 0)
        return ret;
    
    // 配置 IDE
    ret = configure_link_ide_stream(pdev, 0, key);
    if (ret < 0)
        return ret;
    
    // 启用重放保护
    // 启用完整性检查
    // 配置审计日志
    
    dev_info(&pdev->dev, "FIPS-compliant IDE configured\n");
    return 0;
}
```

## 最佳实践

### 1. 安全配置检查清单

```
□ 验证 IDE Capability 支持
□ 使用硬件 RNG 生成密钥
□ 启用重放保护（PR Enable）
□ 配置适当的密钥轮换策略
□ 记录 IDE 配置事件
□ 定期审计 IDE 状态
□ 测试故障恢复流程
□ 监控性能影响
```

### 2. 错误处理

```c
// IDE 错误检测
int monitor_ide_errors(struct pci_dev *pdev) {
    int pos;
    uint32_t status;
    
    pos = find_ide_capability(pdev);
    if (pos < 0)
        return pos;
    
    // 读取 IDE 状态寄存器
    pci_read_config_dword(pdev, pos + IDE_STATUS, &status);
    
    if (status & IDE_STATUS_DECRYPT_ERROR) {
        dev_err(&pdev->dev, "IDE decryption error detected!\n");
        // 可能的原因：密钥不匹配、数据损坏、攻击
        // 触发安全响应
        return -EIO;
    }
    
    if (status & IDE_STATUS_REPLAY_ERROR) {
        dev_err(&pdev->dev, "IDE replay attack detected!\n");
        // 重放攻击检测
        return -EINVAL;
    }
    
    return 0;
}
```

### 3. 调试支持

```c
// IDE 调试信息
void dump_ide_config(struct pci_dev *pdev) {
    int pos;
    uint32_t cap, ctrl, status;
    
    pos = find_ide_capability(pdev);
    if (pos < 0)
        return;
    
    pci_read_config_dword(pdev, pos + IDE_CAPABILITY, &cap);
    pci_read_config_dword(pdev, pos + IDE_CONTROL, &ctrl);
    pci_read_config_dword(pdev, pos + IDE_STATUS, &status);
    
    dev_info(&pdev->dev, "IDE Configuration:\n");
    dev_info(&pdev->dev, "  Link IDE: %s\n",
             cap & IDE_CAP_LINK_IDE_SUPPORTED ? "Supported" : "Not supported");
    dev_info(&pdev->dev, "  Selective IDE: %s\n",
             cap & IDE_CAP_SELECTIVE_IDE_SUPPORTED ? "Supported" : "Not supported");
    dev_info(&pdev->dev, "  Num Streams: %u\n",
             (cap >> 16) & 0xFF);
    dev_info(&pdev->dev, "  Status: 0x%08x\n", status);
}
```

## 与其他安全特性的集成

IDE 通常与其他 PCIe 安全特性配合使用：

```c
// 综合安全配置
int configure_comprehensive_security(struct pci_dev *pdev) {
    int ret;
    
    // 1. 设备认证 (SPDM via DOE)
    ret = authenticate_pcie_device(pdev);
    if (ret < 0) {
        dev_err(&pdev->dev, "Device authentication failed\n");
        return ret;
    }
    
    // 2. 配置 IDE 加密
    uint8_t key[32];
    generate_ide_key(key, NULL);
    ret = configure_link_ide_stream(pdev, 0, key);
    if (ret < 0)
        return ret;
    
    // 3. 启用 ACS (Access Control Services)
    pci_enable_acs(pdev, PCI_ACS_SV | PCI_ACS_RR | PCI_ACS_CR | PCI_ACS_UF);
    
    // 4. 配置 ATS (Address Translation Services) with PASID
    // 5. 启用 PRI (Page Request Interface)
    
    dev_info(&pdev->dev, "Comprehensive security configured\n");
    return 0;
}
```

## 总结

IDE 是 PCIe 6.0 引入的革命性安全特性，为 PCIe 数据传输提供硬件级的机密性和完整性保护。通过灵活的配置选项（Link IDE、Selective IDE），系统可以根据安全需求和性能目标选择合适的保护级别。

随着数据中心对安全性要求的不断提高，IDE 将成为高安全性应用的标准配置，与 SPDM、DOE 等特性共同构建可信的 PCIe 生态系统。

**关键要点：**
- IDE 使用 AES-GCM-256 提供强加密
- 支持灵活的保护范围配置
- 硬件实现确保最小性能影响
- 需要与密钥管理协议（IDE KM）配合使用
- 是构建机密计算和零信任架构的基础
