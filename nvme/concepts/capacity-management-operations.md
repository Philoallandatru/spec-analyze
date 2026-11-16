# 容量管理操作（Capacity Management Operations）

**容量管理命令（Capacity Management）** 用于在 NVM 子系统中组织和分配存储容量层级（`Domain → Endurance Group → NVM Set → Namespace`）。它提供**固定配置**（选择预定义方案）和**动态分配**（按字节数创建/删除）两种模式。[PDF pp. 203-206](../_source/pages/page-203.md)

> **类比理解**：把容量管理想成餐厅的包间划分——整栋楼是 NVM 子系统，楼层是 Domain（故障隔离边界），包间区是 Endurance Group（寿命管理边界），包间是 NVM Set（容量池），餐位是 Namespace。固定配置就像直接选"二人间/四人间"标准套餐；动态分配则是现场按面积砌墙隔出包间。

---

## 两种管理模式与层级模型

```text
层级架构：Domain → Endurance Group → NVM Set → Namespace

                  Capacity Management 命令 (CDW10: ELID[31:16] + OPER[3:0])
                          |
            +-------------+-------------+
            |                           |
   OPER=0h (Select)              OPER=1h..4h (Create/Delete)
   [固定配置模式]                [动态分配模式]
            |                           |
   ELID = 配置描述符 ID          ELID = 父级 ID 或目标 ID
   (0h = 清除当前全部配置)        CDW11(CAPL) + CDW12(CAPU) = 64 位字节容量
            |                           |
   自动创建 EG 和 NVM Set        按需创建/删除 EG 或 NVM Set
   切换前必须先 ELID=0h 清除      删除父级会级联删除其下所有子对象与 NS
```

- **固定配置模式（Fixed Configuration）**：通过 `OPER=0h` 选择预定义的容量配置描述符（Supported Capacity Configuration Descriptor），控制器自动创建对应的 Endurance Group（EG）和 NVM Set，并更新 Media Unit 状态中的"已选配置"字段。重复选择同一非零 ID 成功但无变化；**已有配置生效时不能直接切换到另一个非零 ID**，必须先选 `ELID=0h` 清除全部配置（依次删除 Namespace → NVM Set → EG → 清空状态）。[PDF p. 204](../_source/pages/page-204.md)
- **动态分配模式（Variable Allocation）**：按 64 位字节容量动态创建或删除 EG 与 NVM Set；创建时可指定 Media Unit，否则从指定的父级资源中分配。[PDF p. 205](../_source/pages/page-205.md)
- **级联删除语义**：删除 EG 会同时删除其下所有 NVM Set 和 Namespace；删除 NVM Set 会同时删除其下所有 Namespace。[PDF pp. 203-205](../_source/pages/page-203.md)

---

## 操作码、参数与返回值

| `OPER`（`CDW10[3:0]`） | 操作名称 | `ELID`（`CDW10[31:16]`）含义 | 容量字段（`CDW11..12`） |
|---|---|---|---|
| `0h` | Select Capacity Configuration | 容量配置描述符 ID（`0h` = 清除当前配置） | 保留 |
| `1h` | Create Endurance Group | Domain ID（`0h` = 由控制器自动选择） | 请求的 64 位字节容量 |
| `2h` | Delete Endurance Group | 要删除的 Endurance Group ID | 保留 |
| `3h` | Create NVM Set | 父级 Endurance Group ID（`0h` = 控制器自选） | 请求的 64 位字节容量 |
| `4h` | Delete NVM Set | 要删除的 NVM Set ID | 保留 |

- **创建返回值**：成功创建 EG 或 NVM Set 时，**CQE DW0[15:0]** 返回新分配实体的标识符；其他 `OPER` 下该字段保留。[PDF pp. 205-206](../_source/pages/page-205.md)
- **常见错误码**：
  - `Identifier Unavailable`：可用的非零 EG 或 NVM Set ID 耗尽。
  - `Insufficient Capacity`：请求容量超出可用上限；**Error Information Log** 可报告当前可创建的最大容量。
  - `Invalid Field in Command`：删除时传入零值或不存在的 ID，或控制器不支持 NVM Set 时执行 `OPER=3h/4h`；选择未知配置 ID 也会被拒且不改变任何 Media Unit 状态。

---

## 易混概念对比

| 易混概念 | 实际区别 |
|---|---|
| **Domain vs Endurance Group** | `Domain` 是故障与上电隔离边界；`Endurance Group` 是闪存磨损均衡与寿命管理边界 |
| **Endurance Group vs NVM Set** | `EG` 管理寿命和物理介质池；`NVM Set` 是逻辑容量分配单元，可包含多个 Namespace |
| **Create NVM Set vs Create Namespace** | Capacity Management 创建底层 EG / NVM Set；Namespace Management 在 NVM Set 之上创建 Namespace |
| **CAP 字段 vs 实际大小** | `CDW11..12` 是"请求容量"，控制器可能因对齐要求内部向上或向下取整 |

---

## 关联与引申概念

- **NVM 容量模型（NVM Capacity Model）**：由 `Domain → Endurance Group → NVM Set → Namespace` 构成的四层存储资源组织架构。[PDF p. 141](../_source/pages/page-141.md)
- **Media Unit Status Log**：记录物理介质单元分配情况及当前"已选容量配置"的日志页。
- **Error Information Log**：记录最近失败命令的详细信息；容量不足时可从中查询最大可分配容量。
- **延伸阅读**：
  - [domains-and-divisions.md](domains-and-divisions.md) — Domain 到 Endurance Group 的容量层级详解
  - [format-nvm-lifecycle.md](format-nvm-lifecycle.md) — Namespace 所属 NVM Set 的格式化规则
  - [exported-and-underlying-resources.md](exported-and-underlying-resources.md) — 底层物理资源与导出资源的分层
  - [admin-command-model.md](admin-command-model.md) — Capacity Management 所属的 Admin 命令模型

---

## 规范依据

- [Capacity Management 命令与 OPER/容量字段（Figures 156-157），PDF p. 203](../_source/pages/page-203.md)
- [固定配置模式 Select 流程（Figure 158），PDF p. 204](../_source/pages/page-204.md)
- [动态分配模式 Create/Delete 规则，PDF p. 205](../_source/pages/page-205.md)
- [完成状态码与 CQE 返回值（Figures 159-160），PDF p. 206](../_source/pages/page-206.md)
- [NVM Capacity Model 层级定义，PDF p. 141](../_source/pages/page-141.md)
