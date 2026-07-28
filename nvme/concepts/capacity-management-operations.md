# 容量管理操作（Capacity Management Operations）

## 一句话说明

容量管理命令（Capacity Management）通过"选配置描述符"或"按字节数创建/删除"两种方式，组织 NVM 子系统中的 Endurance Group（耐久性组）与 NVM Set（NVM 集合）层级——前者走预定义方案，后者走动态分配。

## 生活化类比

把容量管理想成**餐厅的包间划分**：

- **整栋楼** = NVM Subsystem
- **楼层** = Domain（域，故障/上电隔离）
- **包间区** = Endurance Group（寿命管理边界）
- **包间** = NVM Set（容量池）
- **餐位** = Namespace（命名空间）
- **固定方案** = 选"二人间/四人间/八人间"的标准菜单
- **动态分配** = 老板现场砌墙，按客人数划出房间

两种方式各有所长：固定方案快（预制件直接拼装），动态分配灵活（可大可小、可改可拆）。

## 工作流程

```text
                  Capacity Management 命令
                          |
            +-------------+-------------+
            |                           |
   OPER=0h (Select)              OPER=1h..4h (Create/Delete)
            |                           |
   读 ELID → 容量配置描述符       读 ELID/CAP → 父级 ID + 64 位容量
            |                           |
   控制器按描述符自动创建:         创建/删除 EG 或 NVM Set
   - 创建 EG                       |
   - 创建 NVM Set                 删除父级时级联:
            |                     - 删 EG → 删其下所有 NVM Set + NS
   OPER=0h 且 ELID=0h:             - 删 NVM Set → 删其下所有 NS
   清除所有 EG/NVM Set
   (1) 删 NS  (2) 删 NVM Set
   (3) 删 EG  (4) 清空"已选配置"
```

简化说明：固定模式下，切换配置必须先 `ELID=0h` 清除现有，再选新 ID；不能直接覆盖。

## 初学者案例

**场景：从头初始化一个空子系统**

1. 子系统出厂，Domain 0 存在但无 Endurance Group、无 NVM Set。
2. 管理员用 `OPER=1h`（Create Endurance Group）创建 EG，ELID=0h（控制器自选 Domain），CAP=1TiB。
3. CQE DW0 bit 15:0 返回新 EG ID，例如 `0x0001`。
4. 管理员用 `OPER=3h`（Create NVM Set）在 EG 0x0001 下创建 NVM Set，CAP=512GiB。
5. CQE 返回新 NVM Set ID，例如 `0x0001`。
6. 后续用 Namespace Management 命令在该 NVM Set 上创建 Namespace。
7. 想要扩容时直接再 Create NVM Set；想要回收时 Delete NVM Set（会同时清掉其下 Namespace）。

> 速查：想要"全部推倒重来"，可直接 `OPER=0h, ELID=0h` 走固定模式的"清除配置"路径。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 命令操作码范围 | OPER=0h..4h 有效；5h..Fh 保留 |
| ELID 含义随 OPER 变 | Select 时是配置 ID；Create EG 时是 Domain ID；Create NVM Set 时是父 EG ID；Delete 时是目标 ID |
| 0h = 控制器自选 | Create 类操作的 ELID=0h 表示"由控制器自动选择父级" |
| 容量字段 64 位 | CDW11 = CAPL（低 32 位），CDW12 = CAPU（高 32 位） |
| 删除级联 | 删 EG → 同时删其下所有 NVM Set + Namespace；删 NVM Set → 同时删其下所有 Namespace |
| 固定模式不能直接切换 | 有生效配置时不可直接选另一个非零配置 ID；必须先 ELID=0h 清除 |
| 未知配置被拒 | 选未知配置 ID 时，控制器不改变任何 Media Unit 状态 |
| 返回值 | 成功 Create 的 CQE DW0 bit 15:0 返回新实体 ID；其他 OPER 该字段保留 |
| NVM Set 可选 | 控制器若不支持 NVM Set，OPER=3h/4h 命令无效，返回 Invalid Field in Command |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Domain vs Endurance Group | Domain 是故障/上电隔离边界；EG 是寿命管理边界 |
| Endurance Group vs NVM Set | EG 管磨损均衡；NVM Set 是容量分配单元 |
| 固定模式 vs 动态模式 | 固定模式选预定义描述符；动态模式按字节数自建 |
| OPER=0h ELID=0h vs 选 0 号配置 | ELID=0h 表示"清除所有配置"；选 0 号配置是另一个语义（看厂商定义） |
| Create NVM Set vs Create Namespace | 容量管理创建底层 EG/NVM Set；Namespace Management 创建 Namespace |
| CAP 字段 vs NVM Set 大小 | CAP 是"请求的容量"；控制器可能内部四舍五入或对齐 |

## 进阶细节

- **CDW10 字段布局**（Figure 156）：
  - `ELID`（bit 31:16）：Element Identifier，含义依 OPER 而定。
  - bit 15:04 保留。
  - `OPER`（bit 03:00）：操作码。
- **OPER 详解**：
  - `0h` Select Capacity Configuration（ELID = 容量配置标识符；0h = 清除当前配置）。
  - `1h` Create Endurance Group（ELID = Domain ID；CAP = 请求容量）。
  - `2h` Delete Endurance Group（ELID = 目标 EG ID）。
  - `3h` Create NVM Set（ELID = 父 EG ID；CAP = 请求容量）。
  - `4h` Delete NVM Set（ELID = 目标 NVM Set ID）。
- **错误码**：
  - `Identifier Unavailable`：可用非零 ID 耗尽。
  - `Insufficient Capacity`：请求容量超限；错误信息日志可报告最大可建容量。
  - `Invalid Field in Command`：ELID 为零、ID 不存在或不支持 NVM Set 时执行 OPER=3h/4h。
- **NVM Capacity Model 关联**：容量管理直接修改 Domain → EG → NVM Set → Namespace 的分配层级（规范 5.1.3 与 8.1.4）。
- **Media Unit 状态**：固定模式下 Media Unit Status Log 的"已选配置"字段会被更新为新选择的 ID。

## 规范依据

- [Capacity Management 命令与 OPER 字段（Figure 156），PDF 第 203 页](../_source/pages/page-203.md)
- [固定模式 Select 流程（Figure 158），PDF 第 204 页](../_source/pages/page-204.md)
- [动态模式 Create/Delete 分配规则，PDF 第 205 页](../_source/pages/page-205.md)
- [完成状态码与 CQE 返回值（Figures 159-160），PDF 第 206 页](../_source/pages/page-206.md)
- [NVM Capacity Model 层级定义，PDF 第 141 页](../_source/pages/page-141.md)

## 相关阅读

- [format-nvm-lifecycle.md](format-nvm-lifecycle.md) - NS 所属 NVM Set 的格式化
- [domains-and-divisions.md](domains-and-divisions.md) - Domain 到 EG 的容量层级
- [exported-and-underlying-resources.md](exported-and-underlying-resources.md) - 底层与导出资源分层
- [admin-command-model.md](admin-command-model.md) - Capacity Management opcode
