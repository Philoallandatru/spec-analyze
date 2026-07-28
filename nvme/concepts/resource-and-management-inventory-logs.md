# 资源与管理清单日志（Resource and Management Inventory Logs）

## 一句话说明

资源与管理清单日志是 NVMe 的一组**只读、零协调成本**的清单型日志，用于在 NVM 子系统边界报告"目前有什么资源、谁在管它"，包括旋转介质的物理特征（Endurance Group 范围， LID `16h`）和管理端点的 URI 列表（子系统范围， LID `18h`）。

> 已尽量精简：本主题在原 NVMe 规范中跨度小（Figures 269/271/272），文档对应内容也较短；本节在不漏事实的前提下压缩到最小必要结构。

## 生活化类比

把 NVM 子系统想成**一台服务器机箱**：

- **机箱里有几块盘** = 几个 Endurance Group
- **如果某块盘是机械硬盘** = 它有"主轴电机"和"磁头臂"——LID `16h` 告诉你有几根"机械臂"、转速多少、起停失败几次
- **机箱的"管理面板"** = 子系统的管理端点列表，告诉你"用 `https://bmc.example.com` 找 BMC，用 `service:snmp:...` 找 SNMP 代理"——LID `18h` 给你这些 URI
- 这两类日志都是**看一眼**就行（read-only inventory），不需要主机来回握手

## 工作流程

```text
Endurance Group selected by ENDGID  --  LID 16h (Rotational Media Information)
  `-- 旋转介质清单：
        actuators（机械臂数）
        RPM      （当前 Power State 0 下的转速）
        Spinup   count（成功上电次数，饱和 32 位）
        Load     count（成功加载次数，饱和 32 位）

NVM subsystem  --  LID 18h (Management Address List)
  `-- 0~8 个 Management Address Descriptor：
        MAT = 1h  -> 子系统管理代理 URI（如 BMC）
        MAT = 2h  -> Fabric 接口管理代理 URI（如 SNMPv3 agent）
        MAT = 0h  -> 保留（Management Address 字段无效）
        MAT = FFh -> 终止符（后续 descriptor 全部保留）
        Address   = null-terminated UTF-8 URI
```

**端到端流程**：

1. 主机发 `Get Log Page`，选择 LID。
2. **LID `16h`**：在 Command Dword 11 写入 `ENDGID`（Endurance Group ID）作为选择器；控制器返回该 EG 的旋转介质信息。
3. **LID `18h`**：无需选择器，控制器返回子系统级的 Management Address List。
4. 主机按规范解析：旋转介质字段读 32 位无符号整数；管理地址列表从 Descriptor 0 开始扫，到 Descriptor 7 或遇到 `MAT=FFh` 终止。

## 初学者案例

**场景：数据中心管理员想用 `nvme` CLI 看看 SSD 后台有没有 BMC，怎么搞？**

1. 管理员执行 `nvme get-log /dev/nvme0n1 --log-id=0x18 --log-len=4096`。
2. 工具解析返回的 4096 字节（8 个 Descriptor × 512 字节）。
3. 从 Descriptor 0 开始：若 `MAT=1h`，URI 字段是一个以 UTF-8 NUL 结尾的字符串，例如 `https://bmc.example.com:8443/redfish/v1`。
4. 工具在 Descriptor 0 找到 `MAT=FFh`（终止符），后续 Descriptor 忽略。
5. 管理员用 curl 调这个 URL，做带外管理（BMC 重启、看 SMART、升级固件等）。

> 配套场景：管理员想知道"某块机械盘（Endurance Group 2）的主轴电机多少次起停失败"，就用 `nvme get-log /dev/nvme0n1 --log-id=0x16 --log-len=512` 并把 `--endgid=2` 传进去。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 旋转介质日志按 EG 选择 | LID `16h` 是 Endurance Group 范围；通过 Command Dword 11 的 Log Specific Identifier 传入 ENDGID |
| 旋转介质计数器饱和 | Spinup / Load 计数器为 32 位，递增到 `FFFFFFFFh` 后**饱和**（不绕回） |
| 旋转日志跨重置保留 | 旋转介质日志数据在**电源循环和重置**后保留（lifetime counters） |
| 子系统无旋转 EG 不支持 | NVM 子系统若没有任何"使用旋转介质"的 Endurance Group，**不应支持** LID `16h` |
| 上电/加载区分 | Spinup = 控制器 power state 从非 operational 进 operational；Load = 某个 actuator 的状态转换 |
| 成功/失败分别计数 | 计数器区分"成功"与"失败"两类转换；不是单一总数 |
| Management List 固定 8 项 | LID `18h` 总是 0~8 个 Descriptor；扫描到 Descriptor 7 或 `MAT=FFh` 截止 |
| MAT 编码 | `0h` 保留 Address；`1h` = 子系统管理代理；`2h` = Fabric 接口管理代理；`FFh` 终止符；其他保留 |
| URI 必须是 UTF-8 NUL 结尾 | Management Address 是 RFC 3986 URI，用 UTF-8 编码，结尾是 0 字节 |
| 这类日志只读不写 | 资源清单是只读快照；没有 Set Log Page 入口 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| LID `16h` vs Endurance Group Information | `16h` 是**旋转介质物理特征**；EG Information 是**寿命与容量**（DUR/DUW/TEGCAP 等） |
| Spinup vs Load | Spinup = 控制器级 power state 转换；Load = 单个 actuator 的状态转换（不同物理对象） |
| 饱和 32 位 vs 绕回 | 这两个计数器到 `FFFFFFFFh` **饱和**停留，不是绕回 0；正确判定"满了" |
| MAT `0h` vs MAT `FFh` | `0h` = 保留 Address（MAT 自身有效但 URI 字段无意义）；`FFh` = 终止符（后续 Descriptor 全部 reserved） |
| Management Address vs Controller | 管理地址是带外通道，**不**是数据路径；与 NVMe 队列、Association 无关 |
| `1h` vs `2h` 代理类型 | `1h` = SSD 子系统自身管理代理（如板载 BMC）；`2h` = Fabric 侧管理代理（如交换机 SNMP） |
| NVM 子系统级 vs EG 级 | 管理地址列表是**子系统**级（一次 Get Log 覆盖整个子系统）；旋转介质是**EG** 级（要传选择器） |
| 旋转日志 vs Device Self-Test | 旋转日志是被动观察；Device Self-Test 是**主动**发起测试 |

## 进阶细节

- **Rotational Media Information Log Page 字段布局**（Figure 269, 规范 5.1.12.1.22）：
  - Bytes `01:00` = ENDGID（被 Get Log Page 命令指定的 Endurance Group ID）
  - Bytes `03:02` = NUMA（Number of Actuators）
  - Bytes `05:04` = NRS（Nominal Rotational Speed，Power State 0 下；`0000h` = Not reported；`0001h` 保留；`FFFFh` Reserved）
  - Bytes `11:08` = SPINC（Spinup Count，成功上电次数，饱和 32 位）
  - 后续字段包括成功的 Load 计数、失败的 Spinup/Load 计数等
- **NRS 特殊值**：规范明确禁止 `0001h` 以"保持与其他标准的向后兼容"；这是初看容易忽略的细节。
- **SPINC 增量触发**（规范 5.1.12.1.22）：每发生一次"控制器从非 operational power state 进入 operational power state"时，控制器将 SPINC 加 1（前提是小于 `FFFFFFFFh`）。
- **Management Address List 字段布局**（Figure 271）：
  - `MAD0` 在 `511:00`
  - `MAD1` 在 `1023:512`
  - ...
  - `MAD7` 在 `4095:3584`
  - 每个 Descriptor 512 字节
- **Management Address Descriptor 字段布局**（Figure 272）：
  - Byte `00` = MAT（Management Address Type）
  - Bytes `03:01` = Reserved
  - Bytes `511:04` = MADRS（Management Address，null-terminated UTF-8 URI）
- **扫描终止条件**：从 `MAD0` 起逐个检查 MAT；遇到 `MAT=FFh` 或扫到 `MAD7` 停止；不假定 8 个都有效。
- **与 Dispersed Namespace 的耦合**：LID `19h`（Dispersed Namespace Participating NVM Subsystems Log）是另一种子系统级 inventory，但它列的是 NQN，不是管理 URI；不要混。
- **保留 vs 不支持**：EG 没有旋转介质 → "should not be supported"（不应支持）；`MAT=0h` → "reserved"（保留但仍占位置）；这两种"空"语义不同。

## 规范依据

- [Rotational Media Information Log 选择器与首字段（Figure 269），PDF 第 287 页](../_source/pages/page-287.md)
- [Rotational Media 生命周期计数器与 Endurance Group 边界，PDF 第 288 页](../_source/pages/page-288.md)
- [Management Address List 与 Descriptor（Figures 271/272），PDF 第 289 页](../_source/pages/page-289.md)
- [Dispersed Namespace Participating NVM Subsystems Log，PDF 第 289 页](../_source/pages/page-289.md)
- [Log Page Retrieval 通用机制（LID 选择、RAE、offset），PDF 第 223 页](../_source/pages/page-223.md)

## 相关阅读

- [log-page-retrieval.md](log-page-retrieval.md) - 通用 Get Log Page 机制
- [nvm-sets-and-endurance-groups.md](nvm-sets-and-endurance-groups.md) - Endurance Group 资源模型
- [dispersed-namespace-lifecycle.md](dispersed-namespace-lifecycle.md) - LID 19h 姊妹清单
- [controller-virtualization-resources.md](controller-virtualization-resources.md) - 资源管理视角
