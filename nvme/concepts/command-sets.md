# 命令集（Command Sets）

## 一句话说明

NVMe 把所有命令按用途分成三大类——管理、I/O、Fabrics——并规定每类命令只能进哪类队列、要求控制器处于什么状态，保证控制通道、数据通道、传输通道互不串扰。

## 生活化类比

把命令集想成**银行柜面的窗口分工**：

- **管理命令集** = 对公业务窗口（开/销户、变更限额）：1 个固定窗口（管理队列），1 位柜员（管理 SQ↔CQ 1:1）。
- **I/O 命令集** = 普通存取款窗口：可开多个窗口（多个 SQ），并能"多窗口共用一个显示屏"（多个 SQ 共享一个 CQ）。
- **Fabrics 命令集** = 远程视频柜员机：哪怕网点关门（`CC.EN=0`）它也工作，专门负责"先把连接接通"。
- **命名空间与 I/O 命令集绑定** = 某张银行卡只能在 ATM 机上用，不能拿到对公窗口办理——卡在发卡时就决定了"我属于哪条业务线"。

## 工作流程

```text
  NVMe 命令集（Figure 5）

  ├── Admin Command Set     ──>  Admin Submission Queue (ID=0)
  │   └─ 创建/删除 I/O 队列、Abort、Set Features、Get Log、Identify…
  │
  ├── I/O Command Sets      ──>  I/O Submission Queues (ID=1~65535)
  │   ├── NVM Command Set           （逻辑块）
  │   ├── Zoned Namespace CS        （分区）
  │   ├── Key Value CS              （键值对）
  │   ├── Computational Programs CS
  │   └── Simple Management (SLM)…
  │
  └── Fabrics Command Set   ──>  Admin SQ（部分也可走 I/O SQ）
      └─ Connect / Disconnect / Authentication / Property Get/Set
      ※ 与 Admin / I/O 不同：无论 CC.EN 取值，控制器都会处理
```

## 初学者案例

**场景：`nvme create-ns` 提示 `Invalid Namespace Format`，盘是新的。**

1. 你的盘在出厂时只有 NVM 命令集，但 BIOS 把它配置成 `CC.CSS=110b`（所有支持的 I/O 命令集）。
2. 默认激活的"命令集组合"是 Identify I/O Command Set Data Structure 中索引 1 那一项，**不是** NVM。
3. 用 `nvme get-feature -f 0x19`（I/O Command Set Profile，Feature ID `19h`）查当前 profile 索引，发现是 Zoned 或 KV。
4. `nvme set-feature -f 0x19 -v <NVM_Index>` 切到包含 NVM 的组合，重置后再 `create-ns` 就 OK。
5. 若你的盘根本没启用 `CAP.CSS.IOCSS`（无 profile 能力），需要回 `CC.CSS=001b`（NVM）重新使能控制器。

> 排错提示：`Invalid Field in Command` 多半是 `CC.CSS` 和 profile 索引冲突；先 `id-ctrl` 看 `CAP.CSS` 位图。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 三类命令集 | Admin / I/O / Fabrics；各走各的队列 |
| Admin 命令只能进 Admin SQ | 标识符固定 0；不可用 I/O SQ 提交 |
| I/O 命令集与命名空间绑定 | 创建 NS 时选定，**全生命周期不变**；只能挂载到已支持并启用该 CS 的控制器 |
| Fabrics 命令特殊性 | 与 `CC.EN` 状态无关，控制器无论是否启用都会处理 |
| 控制器可同时启用多 I/O CS | 一个控制器可启用 NVM + KV + Zoned …，但一个 NS 只能绑一种 |
| CC.CSS 含义 | 控制"控制器活跃的 I/O 命令集选择"；值含 000b/001b/010b/110b 等 |
| CC.CSS=110b | "所有支持的 I/O 命令集"模式，由 Feature `19h` 进一步选择具体组合 |
| 其他 CC.CSS 值 | Feature `19h` Set 成功但**无实际效果** |
| Profile 索引拒绝条件 | 索引 = 0 拒绝；profile 排除了已挂载 NS 正在使用的 CS 也拒绝 |
| CAP.CSS.IOCSS | 控制器支持 profile 能力才必须实现 Feature `19h` |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Admin 命令 vs Fabrics 命令 | 都在 Admin SQ 提交；但 Fabrics 在 `CC.EN=0` 仍工作，Admin 不一定 |
| 命令集 vs 配置文件 Profile | 命令集是"我能解释哪些命令"；Profile 是"在 110b 模式下我选哪一组" |
| CC.CSS=001b vs CC.CSS=110b | 前者直接锁 NVM；后者打开多 CS 选单，由 `19h` 决定 |
| 命令集 vs 队列类型 | 命令集是命令的分类；队列类型（Admin SQ / I/O SQ）由用途决定 |
| KV CS vs NVM CS | 都是 I/O CS；KV 暴露键值接口（无 LBA），NVM 暴露块接口（LBA） |
| Zoned NS CS vs NVM CS | Zoned 把 LBA 空间切 zone，命令格式与 NVM 略有差异 |

## 进阶细节

- **命名空间与 CS 绑定**（规范 2.3.2）：NS 创建时绑定的 I/O 命令集是单选且不可改，NS 删除后才能换。
- **CC.CSS 编码**（规范 3.1.5）：000b=保留、001b=NVM、010b=单一非 NVM、110b=所有支持的 I/O CS（具体由 Feature `19h` 选）。
- **Feature ID `19h` 行为**（规范 5.1.25.1.17）：当 `CAP.CSS.IOCSS=1` 时必须实现；`CC.CSS` 非 110b 时 Set Features 成功但无作用，返回成功状态。
- **Profile 数据结构**：来自 Identify I/O Command Set Data Structure（规范 5.1.13.2.19），索引 0 保留为"未指定"。
- **CC.EN 与命令集激活**：`CC.EN` 由 0→1 时控制器根据当前 `CC.CSS`/`Feature 19h` 初始化对应 I/O CS 状态；切换 `CC.CSS` 或 `Feature 19h` 通常需要控制器重置才能生效。
- **Compute Programs / Simple Management CS**：分别由 TP 4050 / TP 4140 规范定义；NVMe Base 只描述它们在命令集分类中的位置。
- **Fabrics 命令的可用性**：Connect、Disconnect、Authentication、Property Get/Set 全部可经 Admin SQ 提交；某些 Property 命令也允许 I/O SQ 提交。

## 规范依据

- [命令集的类型与路由 Figure 5，PDF 第 41 页](../_source/pages/page-041.md)
- [I/O 命令集与命名空间绑定，PDF 第 52 页](../_source/pages/page-052.md)
- [I/O Command Set Profile 特性 19h 定义，PDF 第 410-411 页](../_source/pages/page-410.md)
- [CC.CSS 字段定义，PDF 第 412 页](../_source/pages/page-412.md)
- [Identify I/O Command Set 数据结构，PDF 第 305 页](../_source/pages/page-305.md)

## 相关阅读

- [queue-pair.md](queue-pair.md) - 命令按队列类型分类规则
- [controller.md](controller.md) - 控制器启用与所支持命令集
- [namespace.md](namespace.md) - 命名空间创建时绑定命令集
- [specification-family-and-scope.md](specification-family-and-scope.md) - I/O 命令集是规范族成员之一
