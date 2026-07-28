# 指令交换（Directive Exchange）

## 一句话说明

指令交换是一对配套的 Admin 命令——`Directive Send`（主机→控制器）与 `Directive Receive`（控制器→主机）——它只负责**搬运 Directive-Type 专属数据**，具体含义交给所选 Directive Type 自己的扩展能力定义。

## 生活化类比

把指令交换想成**快递收发室**：

- **收发室 = 通用命令（Directive Send/Receive）**：只管"按地址送/取包裹"，不拆包裹。
- **包裹里的内容 = Directive-Type 专属数据**：DSPEC、CDW12-13、payload 都是各家厂商/功能自定。
- **包裹上的三联标签 = DTYPE + DOPER + DSPEC**：
  - `DTYPE` 决定走哪个货架（功能大类）。
  - `DOPER` 决定做什么（读/写/启用/禁用等）。
  - `DSPEC` 是 Directive-Type 内部的二级定位参数。

收发室**不验货**，因此要问"包裹里有什么"必须查对应 `DTYPE` 的扩展规范（基础规范 8.1.8）——这是命令设计中典型的"信封 vs 信纸"分离。

## 工作流程

```text
   Host                              Controller
     |  Directive Send (host->ctrl)     |
     |--------------------------------->|
     |  DTYPE + DOPER + DSPEC           | 解析三联标签 -> 派给对应 Directive Type
     |  NUMD dwords @ Data Pointer      | 处理 -> 把 payload 写进控制器
     |                                  |
     |<-- CQE --------------------------|
     |                                  |
     |  Directive Receive (ctrl->host)  |
     |--------------------------------->|
     |  DTYPE + DOPER + DSPEC           | 解析 -> 取出对应结构
     |  NUMD dwords @ Data Pointer      | 通过 Data Pointer 返回
     |                                  |
     |<-- payload ----------------------|
     |<-- CQE --------------------------|
```

简化说明：上图强调"信封"职责——命令通用，负载专用。

### 通用命令字段（CDW10/CDW11/CDW12/CDW13）

| 字段 | 含义 |
|------|------|
| Data Pointer | 传输缓冲区的起始地址（PRP 或 SGL） |
| `NUMD` | 0 基 dword 计数，决定本次传输长度 |
| `DTYPE` | 选择 Directive Type（功能大类） |
| `DOPER` | 在该 Type 内选择具体操作 |
| `DSPEC` | Directive-Type 依赖的二级参数 |
| CDW12–CDW13 | 可选，含义由所选 `DTYPE + DOPER` 决定 |

`Receive` 与 `Send` **共用同一组字段布局**，唯一区别是数据方向。

## 初学者案例

**场景：用 Directive Receive 拉取当前 Directive 状态**

1. 主机提交 `Directive Receive`：
   - `DTYPE=00h`（假设这是某 Directive Type 编号）
   - `DOPER=01h`（假设是"读状态"操作）
   - `DSPEC=0`（该 Type 内部不使用）
   - `NUMD=10`（预留 10 个 dword = 40 字节）
2. 控制器解析三联标签，把"该 Type 的状态结构"放进 Data Pointer。
3. 如果 `NUMD` **小于**结构大小 → 只传前 `NUMD` 个 dword。
4. 如果 `NUMD` **大于**结构大小 → 传完整结构，**不补零、不追加**。
5. 命令在 Admin CQ 投递 CQE，返回成功或 Directive-Type 专属错误码。

> 故障速查：拿到"短 payload" 先检查 `NUMD` 而不是怀疑控制器——`Receive` 的"截断 vs 完整返回"是**预期行为**。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 命令对偶 | `Directive Send`（主→控）与 `Directive Receive`（控→主）**只差方向** |
| 字段同构 | CDW10–CDW11 在 Send/Receive 上**完全一致**，CDW12–13 由 Directive Type 决定 |
| 通用不解义 | 通用命令**不**为 `DOPER/DSPEC/CDW12-13` 赋予含义，含义由 Directive Type 自定 |
| 长度行为 | `Receive`：`NUMD` 小于结构 → 截断前缀；`NUMD` 大于结构 → 返回完整结构不补零 |
| CQ 出口 | 完成后通过 **Admin Completion Queue** 返回 CQE |
| 状态码是 Type 级 | 错误码集合由所选 `DTYPE` 决定，基础命令不强制 |
| 字段位宽 | `DTYPE` 8 bit、`DOPER` 8 bit、`DSPEC` 16 bit、`NUMD` 32 bit |
| 长度对齐 | `NUMD` 是 dword 计数，Data Pointer 按 dword 对齐解读 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| Directive Send vs I/O Write | Send 是 Admin 命令、经 Admin CQ 走；I/O Write 是 I/O 命令、经 I/O CQ 走 |
| Directive Receive vs Get Log Page | 两者都是"读控制器数据"：Receive 走 Directive-Type 协议，Get Log Page 走 LID 注册表 |
| `DTYPE` vs Feature Identifier | `DTYPE` 是 Directive 内部功能号，与 `Get/Set Features` 的 FID 无关 |
| `DOPER` vs Command Opcode | `DOPER` 是 Directive Type 内的子操作，与 Admin/I/O 命令的 opcode 字段不冲突 |
| `DSPEC` vs NSID | `DSPEC` 是 Directive-Type 专属标识；NSID 选择命名空间，**不要混用** |
| `NUMD` 截断 vs 错误 | 截断是**正常**返回（短 payload）；错误一般指 Type 专有错误码 |
| CDW12–13 含义 | 在 `DOPER` 不需要时是**保留**，**不应当成"传什么都行"** |
| Directive 完成 vs 子命令完成 | Directive 是"包"级完成，不是包内每一项的完成 |

## 进阶细节

- **三联标签的解析顺序**：控制器先看 `DTYPE` 选定功能大类，再看 `DOPER` 选操作，最后用 `DSPEC` 定位该 Type 内的对象。`DSPEC` 解释权完全归 Directive Type。
- **`NUMD` 长度与 payload 的契约**：
  - Receive 路径：控制器**只保证传**实际结构大小，不会因 `NUMD` 偏大而追加填充字节。
  - Send 路径：主机负责按 `NUMD` 提供足够数据；不足视为发送不足。
- **完成状态码扩展点**：Command Specific Status Values 由 Directive Type 在 8.1.8 节定义；基础命令本身只承诺通用状态码（Invalid Opcode、Invalid Field in Command 等）。
- **Command Effects**：Send/Receive 的命令效果（CCIOC、CSE）由其在 Commands Supported and Effects 日志中的报告决定，主机可据此判断是否需要停止 I/O 再发送。
- **与扩展能力的关系**：Directives 在 8.1.8 节统一定义；新增 Directive Type 需要在那一节登记 `DTYPE`、DOPER 表、payload 结构、错误码集合。
- **典型应用示例**：
  - Streams Directive：`DTYPE` 选 Streams 后，DOPER 可能是"释放流资源"、"查询流写入指针"等。
  - Directive 的启用与否往往依赖 `Identify Controller` 中的能力位。
- **未启用 Directive Type 的行为**：若主机在 `DTYPE` 处选了未实现的类型，命令以**命令级错误**结束，不进入控制器内部处理。
- **Admin Completion Queue 的统一性**：无论 Send 还是 Receive，CQE 都从 Admin CQ 出，与普通 Admin 命令同一条提交/完成路径。

## 规范依据

- [Directive Receive 命令与数据指针/长度处理，PDF 第 212 页](../_source/pages/page-212.md)
- [Directive Receive/Send 通用字段布局（CDW10/CDW11/CDW12），PDF 第 213 页](../_source/pages/page-213.md)
- [Directive Send 命令完成与错误码，PDF 第 214 页](../_source/pages/page-214.md)

## 相关阅读

- [command-sets.md](command-sets.md) - Directive Type 与命令集绑定
- [command-effects-and-support.md](command-effects-and-support.md) - 命令效果的运行时报告
- [identify-command-model.md](identify-command-model.md) - 启用位的发现路径
- [common-io-control-commands.md](common-io-control-commands.md) - Streams 是 Directive 的典型用例
