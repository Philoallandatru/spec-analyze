# 命令与功能锁定（Command and Feature Lockdown）

## 一句话说明

**Command and Feature Lockdown** 是 NVM 子系统级可选能力，允许系统按 **Scope（操作码空间）** + **Interface（带内/带外路径）** 精准禁止或允许某些命令（Admin 命令、Set Features、Management Interface 命令、PCIe 命令）的执行，目的是把"谁不能干什么"从控制器外部可控。

## 生活化类比

把 Lockdown 想成**办公楼门禁系统**：

- **整栋楼** = NVM 子系统
- **门禁卡读卡器** = Admin Queue（带内，In-band）和 Management Endpoint（带外，Out-of-band）两个入口
- **可禁名单** = Lockable（哪些命令支持被禁）
- **当前黑名单** = Prohibited（哪些命令此刻被禁）
- **名单张贴栏** = Log Page 14h（可查询"可禁"与"当前禁止"两份名单）
- **保卫处审批台** = Lockdown 命令（写入端，只在 Admin Queue 上发起）

保安（子系统）收到黑名单后，对每条入口都同步执行——"一楼大门不让进的人，三楼偏门同样不让进"，除非名单上明确说"只在某入口禁"。

## 工作流程

```text
              写入端(Admin Queue)            查询端(Admin Queue)
              +-------------------+          +----------------------+
              | Lockdown 命令     |          | Get Log Page         |
              |   Scope           |          |   LID = 14h          |
              |   OFI(操作码)     |          |   Scope (0/2/3/4)    |
              |   IFC(接口)       |          |   Contents(00/01/10) |
              |   PRHBT(禁/允)    |          |   UUID Index(可选)   |
              +---------+---------+          +----------+-----------+
                        |                               |
                        v                               v
              +---------+---------+          +----------+-----------+
              | 子系统范围强制执行|          | 返回 512B 日志页面:  |
              |  所有 Controller  |          |   CFILA(回显选择器)  |
              |  所有 ME 同步生效 |          |   LNGTH(列表字节数)  |
              +------------------+          |   CFIL(操作码列表)   |
                                            +----------------------+

执行后,被禁命令到达时被识别:
  - Admin Queue 路径 -> CQE 状态 "Command Prohibited by Command and Feature Lockdown"
  - ME 路径         -> 管理接口响应 "Access Denied"
```

简化说明：上图把"写入"和"读取"两条路径并列展示；写入端是 **Lockdown 命令**（opcode `24h`），读取端是 **Log Page 14h**。两者组合才能形成完整能力。

## 初学者案例

**场景：交付到生产环境的 SSD，如何防止现场误触发 Format NVM？**

1. 部署前用 `nvme get-log /dev/nvme0n1 -i 14 --scope=0 --contents=0` 查询**可锁定**的 Admin opcode 列表，确认 `Format NVM (80h)` 在列。
2. 执行 Lockdown 命令：`scope=0(Admin)`、`OFI=80h(Format NVM)`、`IFC=00b(Admin Queue)`、`PRHBT=1(禁止)`。
3. 命令成功返回；子系统内所有 Controller 立即生效。
4. 现场再有人执行 `nvme format /dev/nvme0n1`，CQE 状态为 `Command Prohibited by Command and Feature Lockdown`，**命令不会执行**。
5. 但通过 Management Endpoint 仍能执行 Format（如果 `IFC=01b` 同时禁了 ME 路径，则两端都拦下）。
6. 解除时，可重新发 `PRHBT=0` 的 Allow 命令，或对子系统做电源循环。

> 故障速查："我禁了 Format NVM，但现场还是格式化成功了"——检查 `IFC` 是否只禁了 Admin Queue 而 ME 路径未禁；检查是否走了带外管理。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 能力标识 | `OACS.CFLS = 1`（Identify Controller）即同时支持 Lockdown 命令与 Log Page 14h |
| 四个 Scope | `0h` Admin 命令、`2h` Set Features Feature ID、`3h` Management Interface opcode、`4h` PCIe opcode（`1h` 与 `5h..Fh` 保留） |
| 三个 Contents | `00b` 可锁定名单、`01b` 当前在 Admin 路径上禁止的名单、`10b` 当前在 ME 路径上禁止的名单（`11b` 保留） |
| 三种 Interface | `00b` Admin SQ、`01b` Admin SQ + ME、`10b` 仅 ME（`11b` 保留） |
| 子系统级生效 | 一次 Prohibit 在子系统内**所有 Controller** 与**所有 ME** 同时强制执行 |
| 路径独立评估 | 同一操作在 Admin Queue 和 ME 上可有不同的禁止状态（可分别禁/允） |
| PCIe 范围 + Admin 接口 | 非法组合：scope=4h 时若 IFC=00b 或 01b，返回 `Invalid Field in Command` |
| ME 不存在 | IFC 含 ME（`01b`/`10b`）但子系统无 ME，返回 `Invalid Field in Command` |
| 不可锁定 | 命令/Feature ID 不支持 Lockdown 时返回 `Prohibition of Command Execution Not Supported (28h)` |
| 幂等性 | 重复 Prohibit 或重复 Allow 都不算错，正常返回成功 |
| 持久性 | 禁止状态持续到子系统电源循环或被显式 Allow |
| 厂商 UUID | 仅当 scope=2h 且双方（控制器 + Set Features）都支持 UUID 时使用 UUID Index（CDW14） |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Lockdown vs Effects Catalogs | Lockdown 关注"是否被禁"（动态权限）；Effects Catalogs 关注"是否支持、影响什么"（静态能力）。一个命令可"被支持"但"被禁" |
| 可锁定 vs 已禁止 | Contents=`00b` 是"能否被禁"的能力名单；`01b`/`10b` 是"是否已被禁"的状态名单 |
| In-band vs Out-of-band | In-band = Admin SQ（主机通过 PCIe 提交）；Out-of-band = ME（带外管理端口，二者评估独立） |
| Log Page 14h vs Lockdown 命令 | Log Page 是"读"，命令是"写"；查询不会修改任何状态 |
| Scope=0h vs Scope=2h | Scope=0h 操作码是"完整字节"；Scope=2h 是 Feature Identifier，再叠加 UUID Index 区分厂商定义 Feature |
| `PRHBT=0` vs 未执行 | `PRHBT=0` 是显式 Allow；未执行 Lockdown 等于"无任何禁止规则"（行为等价，但来源不同） |
| 解除 vs 电源循环 | 单条 Allow 仅解除对应条目；电源循环清空**所有**禁止状态 |

## 进阶细节

- **Log Page 14h 布局（Figure 266）**：512B 数据结构 = CFILA（1B，回显所选 Contents/Scope） + 保留（2B） + LNGTH（1B，CFIL 字节数） + CFIL（nB，按数值升序排列的操作码/Feature ID） + 保留。
- **Log Page 14h 请求参数（Figure 265）**：
  - CNTTS[13:12] = 00b 可锁定名单 / 01b Admin 路径禁止名单 / 10b ME 路径禁止名单
  - SCP[11:8] = 0h Admin / 2h Feature / 3h MI / 4h PCIe
  - UUID Index：仅当 SCP=2h 时生效
- **Lockdown CDW10 布局（Figure 337）**：

  ```text
  31         16 15  8 7  6   5  4    3   0
  +-------------+------+--+-----+----+-----+
  | Reserved    | OFI  | R| IFC |PRHBT| SCP|
  +-------------+------+--+-----+----+-----+
  ```

  - OFI[15:8] 操作码/Feature ID
  - IFC[6:5] 作用接口（00b Admin / 01b Admin+ME / 10b ME）
  - PRHBT[4] 1=Prohibit，0=Allow
  - SCP[3:0] 同 Log Page 的 Scope

- **CDW14（Figure 338）**：UUID Index[6:0]，仅在双方都支持 UUID 且 SCP=2h 时携带。
- **错误返回码**：
  - `Prohibition of Command Execution Not Supported (28h)` — 目标不支持 Lockdown
  - `Invalid Field in Command` — IFC 含 ME 但子系统无 ME；或 SCP=4h + IFC=00b/01b
- **被禁命令执行时的反馈**：
  - Admin Queue 上被禁的 Admin/Set Features：返回 `Command Prohibited by Command and Feature Lockdown`
  - ME 上被禁的 MI/PCIe：返回 `Access Denied` 错误响应（参见 NVMe-MI 规范）
- **能力发现**：读 Identify Controller 的 `OACS` 字段；bit 25 (`CFLS`) 为 1 表示能力可用。

## 规范依据

- [Log Page 14h 目的与 Figure 265 Log Specific Parameter，PDF 第 285 页](../_source/pages/page-285.md)
- [Figure 266 Log Page 14h 返回结构，PDF 第 286 页](../_source/pages/page-286.md)
- [Lockdown 命令定义与 CDW10，PDF 第 370 页](../_source/pages/page-370.md)
- [CDW14 UUID Index 与状态码、错误条件，PDF 第 371 页](../_source/pages/page-371.md)
- [Lockdown 命令专属状态码 Figure 339，PDF 第 372 页](../_source/pages/page-372.md)
- [Section 8.1.5 子系统级强制与持久性、状态语义，PDF 第 516 页](../_source/pages/page-516.md)

## 相关阅读

- [command-effects-and-support.md](command-effects-and-support.md) - 命令支持的静态能力层
- [admin-command-model.md](admin-command-model.md) - 被锁定的 Admin opcode 范围
- [format-nvm-lifecycle.md](format-nvm-lifecycle.md) - 防止 Format NVM 误触发
- [command-abort-semantics.md](command-abort-semantics.md) - Abort 命令可被锁定
