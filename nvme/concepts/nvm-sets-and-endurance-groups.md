# NVM 集与耐久度组（NVM Sets and Endurance Groups）

## 一句话说明

NVM 集（NVM Set）是把子系统容量切成多个"逻辑独立的池"，耐久度组（Endurance Group）则划出"一起算寿命"的边界——一个耐久度组里可以装一个或多个 NVM 集，从而决定寿命管理是"按集算"还是"按组算"。

## 生活化类比

把一块企业级 NVMe SSD 想成一座**带分温区的冷库**：

- **冷库** = NVM 子系统
- **一个温区** = 耐久度组（同温区一起做温度/寿命管理）
- **温区里的货架排** = NVM Set（每个货架排可独立放货、贴标签）
- **货架上的箱子** = 命名空间

两种运维方式：
- **大温区模式**：两个货架排（Set A、Set B）都挂在"温区 Y"下，整温区一起做除霜/温度平衡——EG 把寿命管理做大了。
- **小温区模式**：一个货架排（Set C）独占"温区 Z"，除霜/平衡都只看它自己——EG 把寿命管理做小了。

如果产品本身不划分货架排（不支持 NVM Set），寿命管理就退化到"按温区（EG）"或"按楼层（Domain）"做。

## 工作流程

```text
域 (Domain)
 ├─ 耐久度组 Y (EG Y)            ← 跨 NVM Set 一起做寿命管理
 │    ├─ NVM Set A
 │    │    ├─ 命名空间 A1         继承 Set A 的属性（如最优写入大小）
 │    │    ├─ 命名空间 A2
 │    │    └─ 未分配 NVM
 │    └─ NVM Set B
 │         └─ 命名空间 B1
 └─ 耐久度组 Z (EG Z)            ← 寿命管理只在 Set C 内
      └─ NVM Set C
           └─ 命名空间 C1
```

简化说明：NVM Set 是"装格式化的容器"；EG 是"算寿命的边界"。一个 EG 里只有一个 NVM Set 时，寿命管理退化为"按集算"。

## 初学者案例

**场景：主机看到 `nvme list` 里两个命名空间寿命预警同时触发，但设备只暴露了一个 EG。**

1. 主机下发 Get Log Page（Feature ID `18h`，Endurance Group Event Configuration），选 EG = 1，启用"备用空间低于阈值"位。
2. 控制器监测 Set A 和 Set B——它们共享 EG 1——任一集合的备用空间跌破阈值都会触发事件。
3. 异步事件（Asynchronous Event）通知主机，主机读 Event Aggregate 日志拿到有事件的 EG 列表。
4. 主机按列表读该 EG 的 Endurance Group Information log（512 字节），确认是 Set A 还是 Set B 出问题。
5. 故障速查：若只能看到一个 EG 的事件，多半是设备把多个 NVM Set 合并到同一 EG 管理寿命，这是设计选择，不是 bug。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 命名空间包含关系 | 含格式化数据的命名空间**恰好属于一个** NVM Set，且不跨 Set |
| 属性继承 | 创建命名空间时继承所属 NVM Set 的属性（如最优写入大小） |
| NVM Set 归属 | 每个 NVM Set 恰好属于一个耐久度组（EG） |
| EG 归属 | 每个 EG 恰好属于一个域（Domain） |
| 标识符宽度 | NVM Set ID 与 EG ID 均为 16 位，子系统内唯一 |
| 保留值 | 标识符 `0h` 为保留值；在需要时填 `0h` 触发 `Invalid Field in Command` |
| 寿命管理范围 | 同一 EG 下的多个 NVM Set 一并管理寿命；只含一个 NVM Set 的 EG 退化为"按集管理" |
| 不支持 NVM Set 时 | 寿命退化为"按 EG"或"按 Domain"管理；命令中 NVM Set ID 字段全部清零 |
| 支持 NVM Set 时 | 控制器必须在 Identify Controller 中报告 NVM Sets 支持位、暴露 NVM Set List、把每个 NS 的所属 Set ID 写入 Identify Namespace |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| NVM Set vs Endurance Group | NVM Set 是"容量分池"；EG 是"寿命管理边界" |
| EG 多 NVM Set vs EG 单 NVM Set | 前者跨 Set 一起算寿命；后者寿命管理等同于按 NVM Set |
| 不支持 NVM Set vs 不支持 EG | 前者寿命退到 EG 或 Domain；后者退到 Domain |
| NVM Set 标识符 vs Namespace 标识符 | 前者选容量池（16 位），后者选命名空间（32 位 NSID） |
| EG Information log vs SMART/Health | EG 日志按 EG 给出寿命与健康；SMART 是子系统级聚合 |

## 进阶细节

- **规范 3.2.2 NVM 集**：NVM Set 是与其他 NVM 逻辑/物理分离的 NVM 集合；含格式化数据的命名空间继承其属性（规范 3.2.2）。
- **NVM Set 标识符规则**（规范 3.2.2）：16 位、子系统内唯一、`0h` 保留；命令中填 `0h` 触发 `Invalid Field in Command`。
- **支持 NVM Set 时的强制要求**（规范 3.2.2）：
  - Identify Controller 中 `CTRATT` 的 NVM Sets 位置 1；
  - 在所有相关命令中支持 NVM Set ID 字段；
  - 支持 Identify NVM Set List；
  - Identify Namespace 数据结构中报告所属 NVM Set ID；
  - 必须同时支持 EG。
- **规范 3.2.3 耐久度组**：
  - 寿命可在单个 NVM Set 内管理，也可跨多个 NVM Set 管理；
  - EG 标识符 16 位，子系统内唯一，`0h` 保留；
  - EG 恰好属于一个 Domain（规范 3.2.5）。
- **寿命管理的退化路径**：不支持 NVM Set → 按 EG 管；EG 也不支持 → 按 Domain 管。
- **Endurance Group Event Configuration**（Feature ID `18h`）：按 EG 选择哪些关键告警位（critical-warning bits）会触发事件聚合；`ENDGID=0` 关闭该次操作的告警掩码；启用一个"已经为真"的通知会立即发送事件（规范 5.1.25.1.18 / PDF 第 410 页）。
- **Endurance Group Information log**（规范 5.1.12.1.10 / PDF 第 246 页）：512 字节、寿命视图、按 EG ID 选择；覆盖"已分配给 NVM Set 的容量 + 该 EG 仍持有的未分配容量"；包含 4 类信息：
  - 当前状态：剩余备用空间 / 阈值 / 已用寿命百分比 / 告警位；
  - 寿命累计：主机读 + 主机写 / 介质写 / 完整性错误；
  - 容量：总容量 / 未分配容量 / 寿命估算；
  - 旋转介质识别（若为旋转介质组）以及所属 Domain。
- **写入放大估算**：Host Data Units Written 不含控制器内部流量；Media Units Written 包含 GC 等内部写；两者比值即写放大（规范 5.1.12.1.10 / PDF 第 247-248 页）。
- **Endurance Group Event Aggregate 日志**（规范 5.2.1.2 / PDF 第 275 页）：按升序列出有挂起事件的 EG ID；新增挂起项产生异步事件；用 `RAE=0` 成功读取该 EG 的 Information log 即"确认"并从聚合中删除对应项；列表之外的字节读出为 0。
- **关键告警位覆盖**：组级只读、可靠性降级、备用空间低于阈值；若全 EG 都触发同一告警，SMART/Health 也会反映对应子系统告警（规范 5.1.12.1.10 / PDF 第 247 页）。
- **零值的语义**（规范 5.1.12.1.10）：日志中零值表示"该可选估算/计数器未上报"，不是真为 0。

## 规范依据

- [NVM Set 包含关系与命令集绑定，PDF 第 99 页](../_source/pages/page-099.md)
- [NVM Set 属性 / 要求与耐久度组模型，PDF 第 101 页](../_source/pages/page-101.md)
- [耐久度组要求与事件配置，PDF 第 102 页](../_source/pages/page-102.md)
- [Endurance Group Information log 选择与寿命范围，PDF 第 246 页](../_source/pages/page-246.md)
- [Endurance Group 告警/寿命/活动/错误/容量字段，PDF 第 247 页](../_source/pages/page-247.md)
- [Endurance Group 事件聚合与确认，PDF 第 275 页](../_source/pages/page-275.md)

## 相关阅读

- [存储资源层次结构](storage-resource-hierarchy.md) - 上层资源层级
- [NVM 容量模型](nvm-capacity-model.md) - 容量在 EG/Set 间分配
- [灵活数据放置配置](flexible-data-placement-configurations.md) - FDP 替代 NVM Set 组织
- [识别命令模型](identify-command-model.md) - 列表靠 Identify 读取
- [通用控制器特性](common-controller-features.md) - Read Recovery 按 NVM Set 作用
