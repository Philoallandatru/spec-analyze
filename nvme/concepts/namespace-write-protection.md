# 命名空间写保护（Namespace Write Protection）

## 一句话说明

命名空间写保护（Feature ID `84h`）让主机把单个命名空间设为只读，分"无保护 / 写保护 / 直到掉电前保护 / 永久保护"四级；前两级可解除，后两级不可解除；进入后两级时控制器先把缓存与元数据刷到介质，再生效保护。

## 生活化类比

把写保护想成**四种不同严格度的文件封存**：

- **无保护** = 桌面文件：可读可写可改可删。
- **写保护** = 锁起来的文件夹：只读，但你可以解锁（恢复可写）。
- **直到掉电前保护** = 保险柜临时模式：设上后必须**整栋楼断电**才能解锁。
- **永久保护** = 烧成石板：永远不能改，只能读。

进入"保险柜"模式时，档案员会先把所有抽屉里的临时稿子归档归档（**缓存数据刷到介质**），再把柜门焊死——这样不会因为锁上了而丢稿。

## 工作流程

```text
  [无写保护] ─── Set Feature (写保护) ──→ [写保护]
       │                                        │
       │                                        │ Set Feature (无)
       │  Set Feature (直到掉电前)               ↓
       ↓                                   [无写保护]
  [直到掉电前保护] ── 仅可由 Power Cycle 解除 ──→ [无写保护]

  [无写保护] ── Set Feature (永久) ──→ [永久保护]  (不可逆)

  进入"直到掉电前"或"永久"前的屏障:
    1. 刷所有易失性写入数据到非易失介质
    2. 刷所有元数据到非易失介质
    3. 完成后才允许状态生效
```

简化说明：从"无保护"可进入任何状态；普通"写保护"可双向；"直到掉电前"只能由掉电复位回"无保护"；"永久"不可逆。

## 初学者案例

**场景：保护重要固件不被误改**

1. 主机用 `nvme set-feature /dev/nvme0n1 -f 0x84 -v 2` 把命名空间设为"写保护"（普通模式，可解除）。
2. 测试应用仍能读，写操作返回 `Namespace Is Write Protected` 错误。
3. 想升级固件时 `nvme set-feature ... -v 0` 解除保护；升级完再次启用。

**反例**：用 `nvme set-feature -f 0x84 -v 1`（"直到掉电前"）保护后想解除——失败，**只能 power cycle**（断电重启）。在多域子系统中**禁止**使用这个状态（会被拒）。

> 错误码速记：
> - 退出"直到掉电前"或"永久" → `Feature Not Changeable`
> - 写操作被拒 → `Namespace Is Write Protected`

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| Feature ID | `84h`（Namespace Write Protection） |
| 四个级别 | 无保护 / 写保护 / 直到掉电前 / 永久 |
| 写保护可逆 | 写保护 ↔ 无保护 双向可设 |
| 直到掉电前不可逆 | 只能由 Power Cycle 解除；不能 Set Feature 退出 |
| 永久不可逆 | 一旦设置永远生效；不能 Set Feature 退出 |
| 退出错误 | 尝试退出"直到掉电前"或"永久" → `Feature Not Changeable` |
| 缓存屏障 | 进入"直到掉电前"或"永久"前，**所有易失数据 + 元数据**必须刷到非易失介质 |
| 多域禁用 | 多域子系统中**禁止**使用"直到掉电前" |
| RPMB 联动 | 进入"直到掉电前"或"永久"受 RPMB 写保护控制限制 |
| Namespace-Specific | 不同命名空间可有不同保护状态 |
| 不可保存 | 写保护 Feature 本身不可 Save；但状态本身持久 |
| 默认值 | 没有独立默认值；Reset/Power Cycle 后状态由之前决定（"直到掉电前"除外） |
| 初始状态 | 命名空间创建时为"无保护" |
| 强制执行 | 任一控制器支持该能力，则连接到该命名空间的**每个控制器**都强制执行 |
| SMART 不触发 | 写保护不触发 SMART "All Media Read-Only" 警告 |
| 写命令被拒 | Write/Write Zeroes/Format NVM/Sanitize → `Namespace Is Write Protected` |
| 读命令允许 | Read/Identify/Get Log Page/Compare/Verify 照常 |
| Flush 行为 | 可成功执行但无实际效果（屏障已做完） |
| 间接修改 | 命令间接影响受保护命名空间也被拒（如 Format NVM） |
| 无 NSID 但影响 | Sanitize 作用于整个控制器，若会修改受保护命名空间也被拒 |
| 条件允许 | Directive Receive 不能分配 Streams 资源（即便命令本身允许） |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 写保护 vs 永久保护 | 写保护可解除；永久保护不可逆 |
| 写保护 vs 直到掉电前 | 写保护可 Set Feature 退出；"直到掉电前"必须 Power Cycle |
| 写保护 vs 预留 | 写保护是命名空间级只读；预留是主机级的访问仲裁 |
| 不可保存 vs 不可逆 | 不可保存指不能用 Save Feature 机制；不可逆指状态本身改不回来 |
| 多域 vs 单域 | 多域子系统中"直到掉电前"被禁止；单域可用 |
| Namespace-Specific vs Subsystem-wide | 写保护是 Namespace 粒度；但"任一控制器支持"会子系统范围强制执行 |
| RPMB 写保护 vs Namespace 写保护 | RPMB 是介质级安全机制；Namespace 写保护是逻辑层只读 |
| 屏障完成 vs 状态生效 | 屏障是把数据持久化；状态生效是 Set Feature 成功 |
| Flush vs 屏障 | Flush 是命令；屏障是进入保护状态时的隐式动作 |
| SMART 警告 | 写保护**不触发** "All Media Read-Only"；介质级只读才会触发 |

## 进阶细节

- **Feature 标识**（规范 5.2.13 / 8.1.18 / Figure 322）：
  - Feature ID = `84h`
  - Namespace-Specific
  - Non-Saveable
- **状态定义**（规范 8.1.18）：
  - 0 = No Write Protect
  - 1 = Write Protect
  - 2 = Until Power Cycle
  - 3 = Permanent Write Protect
- **状态转换**（规范 8.1.18 / Figure 622）：
  - 0 → 1（普通写保护）
  - 0 → 2（直到掉电前）
  - 0 → 3（永久）
  - 1 → 0（解除）
  - 2 → 0（仅 Power Cycle）
  - 3 → 不可转移（终态）
- **状态持久性**：
  - 无保护、永久写保护：跨 Power Cycle 保持 + 跨控制器重置保持
  - 直到掉电前：跨 Power Cycle **不**保持（恢复无保护）
- **缓存屏障**（规范 8.1.18 / 5.2.13）：进入"直到掉电前"或"永久"前，控制器必须把所有易失性写入数据和元数据提交到非易失介质。
- **多域限制**（规范 8.1.18）：多域子系统中**禁止**使用"直到掉电前"模式。
- **RPMB 写保护联动**（规范 8.1.18）：进入"直到掉电前"或"永久"受 RPMB 写保护控制限制。
- **强制执行**（规范 8.1.18）：如果子系统中**任一**控制器支持该能力，则连接到该命名空间的**每个**控制器都必须强制执行其保护状态。
- **SMART 兼容性**（规范 8.1.18）：通过该特性设置的写保护**不触发** SMART "All Media Read-Only" 警告。
- **允许执行的命令**（规范 5.2.13 / 8.1.18）：
  - 只读：Read、Compare、Verify
  - 信息查询：Identify、Get Log Page
  - 预留管理：Reservation Acquire/Release/Report
  - 命名空间管理：Attach/Detach
  - 自检：Device Self-test
  - Flush：可成功但无效果
  - **条件性**：Directive Receive 不能分配 Streams 资源
- **被阻止的命令**（返回 `Namespace Is Write Protected`）：
  - 直接指定受保护命名空间的写命令
  - 间接修改受保护命名空间的命令（如 Format NVM）
  - 无 NSID 但会修改受保护命名空间的命令（如 Sanitize）
- **错误码分类**：
  - 退出"直到掉电前"或"永久" → `Feature Not Changeable`
  - 写操作被拒 → `Namespace Is Write Protected`
- **初始状态**：命名空间创建时为"无保护"（规范 8.1.18）。

## 规范依据

- [Feature 状态定义与变更限制，PDF 第 425 页](../_source/pages/page-425.md)
- [Multi-Domain 限制与缓存提交屏障，PDF 第 426 页](../_source/pages/page-426.md)
- [状态定义与 Figure 622 转换图，PDF 第 553-554 页](../_source/pages/page-553.md)
- [子系统范围强制执行与命令交互，PDF 第 555-556 页](../_source/pages/page-555.md)

## 相关阅读

- [namespace.md](namespace.md) - 作用在命名空间上的特性
- [namespace-management-lifecycle.md](namespace-management-lifecycle.md) - 与其他命名空间状态机配合
- [feature-values-and-scope.md](feature-values-and-scope.md) - Feature 84h 配置接口
- [sanitize-operation-lifecycle.md](sanitize-operation-lifecycle.md) - 数据擦除安全配套机制
