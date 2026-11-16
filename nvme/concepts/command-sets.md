# 命令集（Command Sets）

NVMe 将所有命令划分为三大类：**管理命令集（Admin Command Set）**、一个或多个 **I/O 命令集（I/O Command Sets）** 以及 **Fabrics 命令集（Fabrics Command Set）**。一条命令必须进哪类队列、在控制器的什么状态下才会被处理，完全取决于它所属的命令集。[PDF p. 41](../_source/pages/page-041.md)

> **类比理解**：把命令集想成银行的业务窗口分工——对公窗口（Admin SQ）只办开/销户等管理业务；普通存取款窗口（I/O SQ）按业务线分块存储（NVM）、分区存储（ZNS）或键值（KV）；远程视频柜台（Fabrics）哪怕网点还没正式开门（`CC.EN = 0`）也能先接通身份验证和建联。

---

## 命令集分类与队列路由（Figure 5）

```text
NVMe 命令集分类
│
├── Admin Command Set ────> Admin Submission Queue (ID = 0)
│   └─ Identify、Get/Set Features、Get Log Page、Create/Delete I/O Queue、Abort…
│
├── I/O Command Sets ─────> I/O Submission Queues (ID = 1..65535)
│   ├── NVM Command Set             （逻辑块 LBA 模型）
│   ├── Zoned Namespace (ZNS) CS    （顺序写分区模型）
│   ├── Key Value (KV) CS           （键值对模型，无 LBA）
│   └── Computational Programs / Simple Management (SLM) CS…
│
└── Fabrics Command Set ──> Admin SQ（部分 Property 命令也可走 I/O SQ）
    └─ Connect、Disconnect、Authentication、Property Get/Set
    ※ 特殊性：无论控制器是否使能（CC.EN），控制器都会处理 Fabrics 命令
```

---

> 排错提示：`Invalid Field in Command` 多半是 `CC.CSS` 和 profile 索引冲突；先 `id-ctrl` 看 `CAP.CSS` 位图。

命名空间（Namespace）在**创建时**就必须绑定到唯一一个 I/O 命令集，且该绑定关系在命名空间的**整个生命周期内不可更改**（只有删除重建才能更换）：[PDF p. 52](../_source/pages/page-052.md)

- **数据模型由命令集决定**：例如 NVM 命令集将数据抽象为固定大小的逻辑块（LBA），KV 命令集则暴露键值对接口。
- **挂载前提**：一个控制器可以同时支持多个 I/O 命令集，但某个命名空间只能挂载到**既支持又已启用**其对应 I/O 命令集的控制器上。[PDF p. 52](../_source/pages/page-052.md)

---

## I/O 命令集配置（Profile）与 `CC.CSS`

控制器通过配置寄存器 **`CC.CSS`**（I/O Command Set Selected）决定当前激活的 I/O 命令集：[PDF pp. 410-412](../_source/pages/page-410.md)

| `CC.CSS` 值 | 含义 | Feature ID `19h`（I/O Command Set Profile）行为 |
|---|---|---|
| `001b` | 仅 NVM 命令集 | Set Features 返回成功，但**无实际效果** |
| `010b` | 单一非 NVM 命令集 | Set Features 返回成功，但**无实际效果** |
| `110b` | **所有支持的 I/O 命令集**（多命令集模式） | 由 **Feature ID `19h`** 选择 Identify I/O Command Set 数据结构中的某个已声明组合（Profile） |

- **Feature ID `19h` 拒绝规则**：当 `CAP.CSS.IOCSS = 1` 时控制器必须实现该特性；传入索引 `0`（保留）或传入排除了"已挂载命名空间正在使用的命令集"的 Profile 均会被拒绝。切换 `CC.CSS` 或 `Feature 19h` 通常需要控制器复位/重新使能才能生效。[PDF pp. 305, 410-411](../_source/pages/page-410.md)

> **常见排错（新盘 `nvme create-ns` 报 `Invalid Namespace Format`）**：若控制器处于 `CC.CSS = 110b` 模式，默认激活的 Profile 索引 1 可能只包含 ZNS/KV 而不含 NVM。用 `nvme get-feature -f 0x19` 检查当前 Profile 并切换到包含目标命令集的组合即可。

---

## 易混概念对比

| 易混概念 | 实际区别 |
|---|---|
| **Admin 命令 vs Fabrics 命令** | 都在 Admin SQ 提交；但 Fabrics 命令在 `CC.EN = 0` 时仍可处理，Admin 命令通常需控制器使能 |
| **命令集（CS） vs Profile（`19h`）** | 命令集是具体的协议/数据模型（如 NVM、ZNS）；Profile 是 `CC.CSS = 110b` 模式下允许同时启用的命令集组合 |
| **NVM CS vs ZNS CS vs KV CS** | NVM 按 LBA 读写；ZNS 将 LBA 划分为顺序写 Zone；KV 直接按 Key-Value 存取 |

---

## 关联与引申概念

- **Admin Queue / I/O Queue**：控制通道（ID=0）与数据通道（ID=1..65535），队列类型决定了可提交的命令集。[queue-pair.md](queue-pair.md)
- **`CC.CSS` 与 `CAP.CSS`**：控制器属性与配置寄存器中用于查询和选择 I/O 命令集的位域。
- **Feature ID `19h`**：I/O Command Set Profile 特性，仅在 `CC.CSS = 110b` 时真正控制激活的命令集组合。
- **延伸阅读**：
  - [controller.md](controller.md) — 控制器使能与命令集支持能力
  - [namespace.md](namespace.md) — 命名空间创建与命令集绑定
  - [specification-family-and-scope.md](specification-family-and-scope.md) — Base 规范与各 I/O 命令集规范的分工

---

## 规范依据

- [命令集分类与队列路由（Figure 5），PDF p. 41](../_source/pages/page-041.md)
- [命名空间与 I/O 命令集的绑定，PDF p. 52](../_source/pages/page-052.md)
- [Identify I/O Command Set 数据结构，PDF p. 305](../_source/pages/page-305.md)
- [I/O Command Set Profile 特性 `19h` 与 `CC.CSS`，PDF pp. 410-412](../_source/pages/page-410.md)
