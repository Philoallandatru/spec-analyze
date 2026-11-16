# Sanitize 操作生命周期（Sanitize Operation Lifecycle）

Sanitize（数据清除）命令用于**启动或恢复**一次覆盖**整个 NVM 子系统（Subsystem）**范围的后台数据擦除。它支持**块擦除（Block Erase）**、**加密擦除（Crypto Erase）**和**覆写（Overwrite）**三种擦除方式，并可选配**介质验证（Media Verification）**与**验证后空间释放（Post-Verification Deallocation）**。**重要限制**：导出型 NVM 子系统（Exported NVM Subsystem）禁止执行 Sanitize。[PDF pp. 387-390](../_source/pages/page-387.md)

可以把 Sanitize 想成**仓库经理下达清仓指令**：
- **Block Erase**：拆掉旧货架彻底重刷（物理块级擦除）
- **Crypto Erase**：直接更换加密密钥（原密文瞬间不可解密，速度最快）
- **Overwrite**：用新油漆反复涂抹覆盖旧标签若干遍（支持指定 Pass 次数和 32 位模式）
- **Sanitize 命令完成**：只是经理发完指令回办公室，**不代表清仓已完成**；实际清理在后台异步运行，需要通过 **Sanitize Status 日志（LID `81h`）**查看进度。

---

## 状态流转与命令边界

```text
                       EMVS=1 + 支持验证 + NDAS=0
   +-------+  Sanitize 命令   +-----------------+  Exit Media Verify
   | Idle  | ---------------> | Media Verify    | ------------------+
   | 空闲  |                  | (介质验证阶段)  |                   |
   +-------+                  +-----------------+                   v
       ^                               |                   +------------------+
       |                               `-----------------> | Post-Verify      |
       |  处理完成 / 失败(Restricted/Unrestricted)         | Dealloc 空间释放 |
       +-------------------------------------------------- +------------------+
```

**命令与后台操作的严格边界**：
1. **命令成功 ≠ 擦除完成**：CQE 成功仅表示后台操作已启动；控制器在返回 CQE **之前**会先更新 Sanitize Status 日志（LID `81h`），确保主机看到成功 CQE 时日志已反映 `Sanitizing` 状态。[PDF pp. 389-390](../_source/pages/page-389.md)
2. **失败零副作用**：任何失败的 Sanitize 命令都不会启动后台操作，既不修改状态日志也不改动用户数据。
3. **禁止静默降级**：不支持的擦除类型（`SANICAP` 中未置位）或非法选项组合必须直接返回 `Invalid Field in Command`，绝不静默降级为其他方式，以满足安全审计要求。[PDF pp. 387-388](../_source/pages/page-387.md)

---

## 参数控制与启动门控

### 核心参数与约束

| 参数 / 动作 | 作用与约束规则 |
|-------------|----------------|
| **`SANACT`**（Sanitize Action） | 选择 Block Erase、Crypto Erase、Overwrite，或 `101b`（Exit Media Verification，仅在当前处于介质验证状态时有效） |
| **`EMVS`**（启用介质验证） | 需 `SANICAP.VERS=1`、`NDAS=0`，且**不能与 Overwrite 组合**，否则报 `Invalid Field` |
| **`NDAS`**（不释放空间） | 请求擦除后保留块分配状态；若控制器在 `SANICAP` 中禁止此行为，则由 **Sanitize Config（FID `17h`）** 决定行为 |
| **`AUSE`**（允许非受限退出） | `0`=Restricted 完成（必须完整擦除才能退出）；`1`=Unrestricted（允许从失败中直接重发退出） |
| **Overwrite 参数** | 指定 Pass 次数（`0` 表示 16 次）、可选模式反转（多 Pass 交替反转比特掩盖残影）及 32 位数据模式 |

### 启动拒绝条件与序列化互斥

满足以下任一条件时，Sanitize 命令会被直接拒绝：[PDF pp. 387-390](../_source/pages/page-387.md)
- **PMR（持久内存区域）已启用**
- **存在写保护的命名空间**
- **有待执行的固件激活**（需重置激活的固件尚未生效）；同时 Sanitize 期间也**禁止新的固件激活**
- **子系统中任一控制器处于挂起（Suspended）状态**
- **多域（Multi-Domain）子系统被分割**（控制器无法访问全部命名空间，返回 `Asymmetric Access Inaccessible` 等错误）
- **使用了不合规的 CMB（控制器内存缓冲区）队列放置方式**

### Sanitize Config（Feature ID `17h`）

仅当 `SANICAP` 报告控制器禁止 `NDAS=1` 时生效（子系统作用域）：[PDF p. 409](../_source/pages/page-409.md)
- **Error Mode（错误模式）**：直接拒绝携带 `NDAS=1` 的 Sanitize 命令。
- **Warning Mode（警告模式）**：接受命令，但实际仍会释放空间，并在完成后在状态日志中记录 `SOS=100b`（Sanitized with Unexpected Deallocation）。

---

## 实践要点与易混淆概念

```bash
# 发起加密擦除（Crypto Erase）并轮询 Sanitize Status 日志
nvme sanitize /dev/nvme0n1 -a 0x02
nvme sanitize-log /dev/nvme0n1
```

| 容易混淆的概念 | 实际区别 |
|----------------|----------|
| **Sanitize vs Format NVM** | **Sanitize** 是**子系统级**后台异步擦除（覆盖所有 NS 及未分配空间）；**Format NVM** 是针对**单个命名空间**的格式化（可选 Secure Erase）[PDF pp. 218-220](../_source/pages/page-218.md) |
| **Restricted vs Unrestricted 失败** | 规范 8.1.24.3 状态机中，`AUSE=0` 失败进入 Restricted Failure（旧数据可能残留，需特定恢复动作）；`AUSE=1` 失败进入 Unrestricted Failure（可直接重发 Sanitize） |
| **Media Verification vs 数据擦除** | 验证阶段只检查介质是否真的被擦除干净，本身不擦除数据，且耗时可能很长 |

---

## 关联与引申概念

- **Sanitize Status Log（LID `81h`）**：Sanitize 操作的持久化只读观察口，记录进度 `SPROG`、状态 `SOS`、时间估计与详细状态信息 `SSI`。
- **PMR（Persistent Memory Region）与 CMB（Controller Memory Buffer）**：均为控制器暴露的 PCIe 内存区域，由于 Sanitize 涉及全子系统数据安全，规范要求 PMR 禁用且 CMB 队列放置合规后才能启动。
- **相关文档**：
  - [Sanitize 操作状态](sanitize-operation-status.md) — 进度、时间估计与结果观察接口（LID `81h`）
  - [Format NVM 生命周期](format-nvm-lifecycle.md) — 命名空间级别的格式化与安全擦除对比
  - [固件更新生命周期](firmware-update-lifecycle.md) — 与 Sanitize 的序列化互斥关系
  - [控制器迁移](controller-migration.md) / [控制器内存窗口](controller-memory-windows.md) — 其他序列化门控依赖
  - [持久事件日志](persistent-event-log.md) — 记录 Sanitize 开始与完成的持久事件流

## 规范依据

- [Sanitize 范围、操作类型、SANICAP 门控与多域限制，PDF p. 387](../_source/pages/page-387.md)
- [介质验证转换、序列化门控与拒绝条件，PDF p. 388](../_source/pages/page-388.md)
- [命令启动原子性、Overwrite 参数与 CQE/日志顺序，PDF pp. 389-390](../_source/pages/page-389.md)
- [Sanitize Config（Feature ID 17h）错误/警告模式，PDF p. 409](../_source/pages/page-409.md)
