# PCIe 访问控制服务（ACS - Access Control Services）

> **一句话总结**：ACS 是 Root Port 和 Switch 端口上的"交通安检门禁"，用来决定设备之间的点对点（P2P）流量能否直接抄近道，还是必须绕道 Root Complex 接受 IOMMU 的权限检查。

如果把 PCIe 拓扑比作一栋写字楼，Switch 就是同一楼层的走廊，Root Complex 是大楼一层的安保总台。没有门禁时，A 房间（设备 A）可以直接穿过走廊去 B 房间（设备 B）拿东西，大楼保安（操作系统和 IOMMU）完全不知道；开启 ACS 后，所有跨房间的请求都必须先乘电梯到一楼总台（Root Complex）验明身份与权限，再决定是否放行。

---

## 为什么需要 ACS？

在现代服务器与虚拟化场景中，多个设备（如多张 GPU、NVMe SSD、网卡）往往挂载在同一个 PCIe Switch 或 Root Port 下。如果没有 ACS 管控 Peer-to-Peer（P2P，点对点）流量，会带来三类核心风险：

1. **绕过 IOMMU 隔离（虚拟机逃逸）**：当不同设备直通给不同虚拟机（VM）时，恶意或被攻陷的设备可以直接发起 P2P 内存读写，访问同一 Switch 下其他设备的 BAR 空间，完全绕过位于 Root Complex 中的 IOMMU（输入输出内存管理单元）地址转换与权限检查。
2. **Requester ID 欺骗**：下游设备可能在 TLP（事务层数据包）中伪造其他设备的 BDF（Bus:Device.Function）作为 Requester ID，导致错误归因或越权访问。
3. **已转换地址误用**：支持 ATS（地址转换服务）的设备如果将已经由 IOMMU 转换好的物理地址（Translated TLP）直接发往 P2P 设备，目标设备可能将其误认为自己的 MMIO 地址，造成数据错乱或越权。

---

## ACS Capability 与 Control 寄存器

ACS 作为一个 **PCIe Extended Capability（扩展能力结构，Capability ID `0x000D`）**，位于 Root Port、Switch Upstream Port 和 Switch Downstream Port 的扩展配置空间（`0x100+`）中。

| 偏移 | 寄存器 | 作用 |
| :--- | :--- | :--- |
| `0x00` | Extended Capability Header | 标识 Capability ID (`0x000D`)、版本及下一项偏移 |
| `0x04` | **ACS Capability Register** (16-bit) | 只读，声明硬件支持哪些 ACS 子功能 |
| `0x06` | **ACS Control Register** (16-bit) | 可读写，软件通过置位来启用对应的 ACS 子功能 |
| `0x08+` | **ACS Egress Control Vector**（可选） | 当支持 P2P Egress Control 时存在，按位指定允许直接 P2P 的目标端口 |

### 7 个核心控制位详解

| Bit | 名称 | 简称 | 核心作用 |
| :--- | :--- | :--- | :--- |
| `0` | **Source Validation** | `V` / `SrcValid` | 源验证：检查下游传入 TLP 的 Requester ID（Bus 号）是否落在该端口的 `[Secondary Bus, Subordinate Bus]` 范围内，防止 ID 欺骗 |
| `1` | **Translation Blocking** | `B` / `TransBlk` | 转换阻止：拦截带有 `AT = Translation Request` 或 `AT = Translated` 标记的 P2P 请求（返回 Unsupported Request），防止已转换地址走 P2P |
| `2` | **P2P Request Redirect** | `R` / `ReqRedir` | P2P 请求重定向：强制将下游发往其他下游端口的 Memory Read/Write/AtomicOp 请求重定向到上游（Root Complex） |
| `3` | **P2P Completion Redirect** | `C` / `CmpltRedir` | P2P 完成重定向：强制将原本可以直接转发给同级下游端口的 Completion（完成包）也先发往上游 Root Complex |
| `4` | **Upstream Forwarding** | `U` / `UpstreamFwd` | 上游转发：控制从 Downstream Port 收到的 TLP 是否允许转发到 Upstream Port（主要用于 Multi-Root 多根拓扑隔离） |
| `5` | **P2P Egress Control** | `E` / `EgressCtrl` | P2P 出口控制：结合 Egress Control Vector 位向量，按端口粒度精确允许或禁止直接 P2P |
| `6` | **Direct Translated P2P** | `T` / `DirectTranslatedP2P` | 直接转换 P2P：例外机制，当确认相关设备处于同一受信任的 IOMMU 域时，允许已转换地址直接走 P2P |

---

## 核心工作机制与流量路径

### 1. 无 ACS vs 启用 ACS 的 P2P 路径

```text
【无 ACS（直接 P2P，高性能但无隔离）】
Device A (Port 1) ──TLP──▶ Switch ──直接转发──▶ Device B (Port 2)
                             ▲
                       Root Complex / IOMMU 完全不可见

【启用 P2P Request Redirect + Completion Redirect（强制安检）】
Device A (Port 1) ──请求──▶ Switch ──上行──▶ Root Complex (IOMMU 检查)
                                                │
Device B (Port 2) ◀──下发── Switch ◀────下行────┘
  (Completion 同样经由 Switch → Root Complex → Switch 返回 Device A)
```

### 2. Source Validation（源验证）拦截逻辑

```text
Switch Downstream Port 收到下游 TLP
  │
  ▼
提取 TLP Header 中的 Requester ID (Bus:Dev:Func)
  │
  ▼
Bus 是否在 [Secondary Bus, Subordinate Bus] 范围内？
  ├── 否 ──▶ 判定为非法/伪造 TLP，丢弃并向 AER 报告 ACS Violation
  └── 是 ──▶ 放行，继续路由判断
```

### 3. P2P Egress Control（混合隔离策略）

当同一台服务器里既有需要高速直连的 GPU 集群，又有需要严格隔离的网卡和存储控制器时，可以通过 **Egress Control Vector** 实现分组隔离：

```text
假设 4 端口 Switch（Port 0/1 为 GPU，Port 2 为网卡，Port 3 为 NVMe）：
- Port 0 (GPU 0) 的 Egress Vector = 0b0010  → 仅允许直接 P2P 到 Port 1 (GPU 1)
- Port 1 (GPU 1) 的 Egress Vector = 0b0001  → 仅允许直接 P2P 到 Port 0 (GPU 0)
- Port 2 (NIC)   的 Egress Vector = 0b0000  → 禁止直接 P2P，全部重定向到 RC
- Port 3 (NVMe)  的 Egress Vector = 0b0000  → 禁止直接 P2P，全部重定向到 RC
```

---

## ACS 与 IOMMU / VFIO 虚拟化隔离

在 Linux 虚拟化（KVM/QEMU）中，**VFIO（Virtual Function I/O）** 框架要求一个设备必须处于独立的 **IOMMU Group（隔离组）** 才能安全直通给单个虚拟机。

- **硬件不支持或未启用 ACS**：同一 Switch（或不支持 ACS 的 Root Port 多功能根端口）下的所有设备无法硬件隔离，会被操作系统划入**同一个 IOMMU Group**，导致它们必须打包分配给同一个 VM，无法拆分直通。
- **硬件启用 ACS**：每个支持 ACS 的 Downstream Port 形成独立隔离边界，设备各自拥有独立的 IOMMU Group，可安全分配给不同租户。

### 常用状态检查命令

```bash
# 查看端口的 ACS 能力 (+) 与当前启用状态 (+/-)
lspci -vvv -s 00:01.0 | grep -A 3 "Access Control Services"

# 查看系统中每个 IOMMU Group 包含的设备
for d in /sys/kernel/iommu_groups/*/devices/*; do
    n=${d#*/iommu_groups/*}; n=${n%%/*}
    printf 'IOMMU Group %s: ' "$n"
    lspci -nns "${d##*/}"
done
```

> **关于 `pcie_acs_override` 内核参数的警示**：部分消费级主板由于未实现完整 ACS，玩家会使用 `pcie_acs_override=downstream,multifunction` 强行在软件层拆分 IOMMU Group。这**破坏了硬件隔离保证**，仅可用于本地可信测试环境，严禁用于多租户生产云环境。

---

## 性能与安全的权衡

| 策略 | 适用场景 | 优势 | 代价 |
| :--- | :--- | :--- | :--- |
| **完全启用 ACS** | 多租户云计算、KVM/VFIO 直通 | 最强硬件隔离，所有 P2P 受 IOMMU 保护与审计 | P2P 需往返 Root Complex，延迟升高，RC 可能成为带宽瓶颈 |
| **禁用 ACS** | 单机 AI 训练/HPC（全可信环境） | 同一 Switch 下 GPU/NVMe 直接 P2P，延迟最低、吞吐最高 | 无设备间隔离，无法安全划分给不同租户 |
| **Egress Control 混合** | 异构算力服务器 | 仅对指定端口放行直接 P2P，其余走 RC 隔离 | 需 Switch 硬件与驱动支持 Egress Vector |

---

## 关联与引申概念

- **SR-IOV & ARI**（[sriov.md](sriov.md)、[ari.md](ari.md)）：SR-IOV 将单张网卡/SSD 拆分为多个 VF，ARI 将单设备 Function 上限扩展到 256 个；而 **ACS**（尤其是针对多功能设备的 ACS）是保证不同 VF 之间不发生未受控 P2P 互访、能够划分到独立 IOMMU Group 的前提。
- **IOMMU / VFIO**：IOMMU（如 Intel VT-d / AMD-Vi）在 Root Complex 处负责 DMA 地址重映射与权限校验，VFIO 是 Linux 用户态安全设备直通框架；ACS 确保 P2P 流量不会绕过 IOMMU。
- **ATS / PRI / PASID**：ATS（Address Translation Services）允许设备缓存地址转换结果，PRI（Page Request Interface）支持设备缺页请求，PASID（Process Address Space ID）标识进程地址空间。ACS 中的 `Translation Blocking` 与 `Direct Translated P2P` 专门用于管控带有 ATS `Translated` 标记的 TLP。
- **AER**（[../error-handling/aer.md](../error-handling/aer.md)）：Advanced Error Reporting，当 Source Validation 检测到非法 Requester ID 时，会触发 **ACS Violation** 错误并由 AER 记录上报。
- **IDE & 机密计算**（[ide.md](ide.md)）：在机密计算（TEE/TDX/SEV-SNP）环境中，ACS 负责拓扑隔离，IDE（Integrity and Data Encryption）负责链路级 TLP 加密，共同防止物理探测与侧信道攻击。
