# Fabric 分区数据传输（Fabric Zoning Data Transfer）

## 一句话说明

Fabric Zoning 用三组命令（Lookup / Receive / Send）让 DDC 找到 CDC 持有的 Zoning 数据结构并按 dword 片段来回搬运：Lookup 把操作数据映射为 Zoning Data Key，Receive/Send 按 offset+length 传片段，最后片段用 LF 标志表示结束。

## 生活化类比

把 Fabric Zoning 想成**两个办公室之间传递大件**：

- **Lookup（FZL）** = 你（Direct Discovery Controller）给前台（Centralized Discovery Controller）报"我要 Z1234 这件货物"——前台查表，给你一张提货单（**Zoning Data Key**）。
- **Receive（FZR）** = 你用提货单到仓库按页取货：每页 4 字节对齐（dword），告诉仓库"从第几页起、取几页"；仓库发给你，最后一页时仓库在回执上盖"完"章（**LF=1**）。
- **Send（FZS）** = 你把货物退给仓库：你告诉他"从第几页起、送几页"；你自己在送货单上盖"最后一片"章（**command Dword 12 的 LF=1**）。

`ZDKC` 就像"提货单 vs 内部批次号"的区分位——前台和仓库各认各的标识，混用就拒。

## 工作流程

```text
  DDC 操作数据  ──FZL──→  CDC Zoning 数据库
                                |
                                ↓
                       Zoning Data Key (ZDK)
                                |
              ┌─────────────────┼─────────────────┐
              ↓                                   ↓
       FZR (CDC→DDC)                       FZS (DDC→CDC)
       offset + length; CQE.LF            offset + length + cmd LF
              ↓                                   ↓
        [片段循环]                          [片段循环]
              ↓                                   ↓
        LF=1: 结束                          LF=1: 结束
```

简化说明：`ZDKC` 区分 ZDK（Zoning Data Key）vs 交易 ID（Transaction ID）。CDC 接受 ZDK；DDC 接受交易 ID；反过来报 `Invalid Field in Command`。`NUMD` 必须非零。

## 初学者案例

**场景：DDC 拉 Zoning 数据，第一片正常，第二片 `Invalid Field in Command`**

1. DDC 发 `FZL` 命令，操作数据 8 字节；CQE Dword 0 返回 `ZDK=0xABCD`。
2. 发 `FZR`：offset=0, NUMD=16（dword），`ZDKC=1`（用 ZDK）→ OK。
3. 发 `FZR`：offset=16, NUMD=16, `ZDKC=1` → 报 `Invalid Field in Command`。
4. 排查：
   - CDC 拒绝 ZDK 重复使用？→ 不常见，ZDK 应在结构传输完成前有效
   - 偏移超过结构大小？→ `offset` + `NUMD × 4` 是否超总长
   - `NUMD` 为 0？→ 必须非零
5. 解决：根据错误码定位是 offset 超界还是上下文错。

> 错误码速记：ZDK / Transaction ID 上下文错 → `Invalid Field in Command`；结构锁/缺失/未授权 / DDC 不在 ZoneGroup → 命令特定错误。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 三组命令 | FZL（Lookup）/ FZR（Receive）/ FZS（Send） |
| FZL 行为 | 操作数据放入命令缓冲；CQE Dword 0 返回 ZDK |
| FZR 行为 | DDC 用 ZDK；CDC 用当前 Transaction ID；CQE Dword 0 返回 LF |
| FZS 行为 | DDC 用 ZDK；CDC 用当前 Transaction ID；**command Dword 12 设置 LF** |
| ZDKC 位 | 区分 ZDK vs Transaction ID |
| 上下文错 | CDC 接受 ZDK；DDC 接受交易 ID；反之 `Invalid Field in Command` |
| NUMD | 必须**非零** |
| 偏移对齐 | 偏移 dword 对齐 |
| 低位偏移 | 控制器可拒绝非零低位；如不拒绝按零处理 |
| FZR 越界 | offset 超过请求结构大小 → 拒绝 |
| LF 含义 | Last Fragment = 1 表示本片是最后一片 |
| 锁定/缺失 | 命令特定错误（结构锁/缺失/Fabric Zoning 禁用/DDC 不在 ZoneGroup） |
| 完整流程 | Lookup → 多次 Receive/Send 循环 → LF=1 结束 |
| 传输单位 | dword（4 字节） |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| FZL vs FZR/FZS | FZL 是"查表拿钥匙"；FZR/FZS 是"用钥匙搬数据" |
| FZR vs FZS | FZR = CDC→DDC；FZS = DDC→CDC |
| ZDK vs Transaction ID | ZDK 是"业务标识"；Transaction ID 是"传输会话标识" |
| ZDKC 位 | 区分上面两个；CDC 与 DDC 接受不同 |
| LF 在哪 | FZR 的 LF 在 CQE Dword 0；FZS 的 LF 在 command Dword 12 |
| NUMD 为 0 vs 越界 | NUMD=0 必被拒；越界是被拒；错误码不同 |
| 偏移对齐 vs 偏移非零 | 偏移 dword 对齐；偏移值本身可非零 |
| 锁定 vs 缺失 | 都是命令特定错误，但语义不同 |
| Fabric Zoning vs Allowed Host List | Zoning 是 fabric 级访问控制配置；Allowed Host List 是 Exported NVM Subsystem 的允许主机清单 |
| ZDKC 上下文 | CDC 接受 ZDK；DDC 接受 Transaction ID；不能反着用 |

## 进阶细节

- **三组命令一览**（规范 5.2.13 / 4.2.5）：
  - **Fabric Zoning Lookup (FZL)**：DDC 把操作数据放命令缓冲；CQE Dword 0 返回 `ZDK`
  - **Fabric Zoning Receive (FZR)**：DDC 用 `ZDK`、CDC 用当前 Transaction ID；CQE Dword 0 返回 `LF`
  - **Fabric Zoning Send (FZS)**：DDC 用 `ZDK`、CDC 用当前 Transaction ID；**command Dword 12 设置 `LF`**
- **`ZDKC` 位的语义**（规范 4.2.5 / 5.2.13）：
  - `ZDKC=1` 标识使用 Zoning Data Key
  - `ZDKC=0` 标识使用 Transaction ID
  - CDC 接受 ZDK 上下文；DDC 接受 Transaction ID 上下文
  - 反过来 → `Invalid Field in Command`
- **`NUMD`**（规范 4.2.5）：dword 计数值；必须非零；为 0 即拒。
- **偏移与对齐**（规范 4.2.5）：
  - 偏移 dword 对齐
  - 控制器可拒绝非零低位；如不拒绝按零处理
  - FZR 越界（offset ≥ 请求结构大小）→ 拒绝
- **LF（Last Fragment）**（规范 4.2.5 / Figures 471-477）：
  - FZR 的 LF 在 **CQE Dword 0**
  - FZS 的 LF 在 **command Dword 12**
  - `LF=1` 表示本片是最后一片
- **片段循环**（规范 4.2.5）：选定上下文 → offset + 非零 NUMD → 传输 → 循环至 `LF=1`。
- **错误场景**（规范 4.2.5）：
  - 结构被锁
  - 结构缺失
  - Fabric Zoning 禁用
  - DDC 不被允许访问该 ZoneGroup
- **Fabric Zoning 的定位**（规范 1.5.x / 4.2.5）：fabric 范围的访问控制配置机制；与 Exported NVM Subsystem 的 Allowed Host List 是**不同机制**。
- **DDC 与 CDC 的角色**（规范 5.2.13 / 4.2.5）：
  - DDC（Direct Discovery Controller）= 发起方
  - CDC（Centralized Discovery Controller）= Zoning 数据库持有方

## 规范依据

- [Lookup 与返回 Zoning Data Key，PDF 第 457 页](../_source/pages/page-457.md)
- [Receive 字段与完成状态，PDF 第 458 页](../_source/pages/page-458.md)
- [Receive 终止与 Send 字段，PDF 第 459 页](../_source/pages/page-459.md)
- [Send 完成状态，PDF 第 460 页](../_source/pages/page-460.md)
- [Fabric Zoning 与 Allowed Host List 的边界，PDF 第 31 页](../_source/pages/page-031.md)

## 相关阅读

- [fabric-zoning-model.md](fabric-zoning-model.md) - 数据传输依赖分区模型
- [fabrics-discovery-and-authentication.md](fabrics-discovery-and-authentication.md) - 通过发现服务连接 CDC
- [command-sets.md](command-sets.md) - 使用管理命令集实现传输
- [association-and-command-lifecycle.md](association-and-command-lifecycle.md) - 命令提交到完成的完整流程
