# 控制器属性（Controller Properties）

## 一句话说明

控制器属性是控制器上**与传输无关**的寄存器式属性表，承载能力、配置、状态、版本等控制平面信息；传输绑定（PCIe MMIO / Fabrics Property Get/Set）只决定"怎么访问"，不改变属性本身。

## 生活化类比

把控制器属性想成**酒店房间的智能控制面板**：

- **面板上的旋钮/按钮** = CAP、VS、CC、CSTS 等属性（每格大小、位置、读/写规则都写死）
- **面板总布局** = 属性映射表（`1000h` 以下是通用区域，以上是各传输厂商私有区）
- **走廊里的 MMIO 通道** = PCIe 内存映射访问（住宿客人直接用卡刷）
- **前台的电话转接** = Fabrics Property Get/Set 报文（电话告知服务员"请把第 23 号开关切到 5"）
- **客人和服务员最终看到的开关状态完全一致**——只是"动手"的方式不同

客人（主机）永远只能整格（dword/qword）操作一个属性，不能跨两格去掰一半；保留位就像"被胶带封死的旋钮"，读出来是 `0h`，但你不该指望它给你"写 0 也保平安"。

## 工作流程

```text
                       控制器属性空间（控制器全局）
   +-----------------------------------------------------------+
   | 00h CAP | 08h VS | 0Ch INTMS | 10h INTMC | 14h CC | ...   |
   | 1Ch CSTS| 20h NSSR| 24h AQA | 28h ASQ  | 30h ACQ | ...   |
   +-----------------------------------------------------------+
   ^ 通用区 1000h 以下                                            ^
   |                                                             |
   |  PCIe: 1000h 起跳到 transport-specific 区域                  |
   |  Fabrics: 1000h-12FFh 给 Fabrics, 1300h 起厂商私有           |
   +-------------------------------------------------------------+

   主机 ─► 一次精确宽度访问（dword/qword） ─► 单个属性 ─► 控制器解析

   关键约束：
   - 单次访问不能跨越两个属性
   - 保留属性/保留位只读，返回 0h
   - 写 0 到保留位是允许的，但读不可依赖"返回 0"
```

简化说明：上图省略了 PCIe 特有寄存器（DBstr / MSI-X table 等）和 Fabrics 私有字段，仅表达"通用属性表 + 传输私有区拼接"的核心结构。

## 初学者案例

**场景：上电后主机怎么知道这块 NVMe 设备能跑多快？**

1. 主机通过 PCIe BAR 或 Property Get 读取 `CAP` 寄存器。
2. 从 `CAP.MQES` 知道每个 I/O 提交队列最大条目数（0 基编码，`1h` 表示 2 条）。
3. 从 `CAP.MPSMIN` / `CAP.MPSMAX` 算出主机页面大小范围：`2^(12+value)` 字节。
4. 从 `CAP.DSTRD` 算出 Doorbell 步长：`2^(2+value)` 字节（`0h` = 4 字节）。
5. 从 `CAP.TO` 知道最坏就绪时间（500 ms 为单位，封顶 127.5 秒）。
6. 主机在 `CC` 里挑选 `CC.AMS`（仲裁）、`CC.MPS`（页大小）、`CC.CSS`（命令集），都必须在 `CAP` 限定的合法区间内。
7. 一切合法后再 `CC.EN=1`，控制器按 `CAP.TO`（或 `CRTO`）进入 ready 状态。

> 故障速查：写 `CC.EN=1` 后控制器一直不进入 ready，**90% 是 `CC.AMS/MPS/CSS` 不在 `CAP` 允许的范围内**，而不是控制器坏。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 传输无关性 | 同一份属性表对 PCIe 和 Fabrics 都有效，区别只在于访问手段 |
| 单属性访问边界 | 一次访问必须整 dword/qword 命中单个属性，跨属性访问不被支持 |
| 保留位行为 | 保留属性与保留位**只读、返回 0h**，但软件不应依赖"写 0 后读 0"的语义 |
| 宽度锁定 | 主机必须用属性声明的宽度和起始偏移访问，否则是越界/不支持 |
| 配置守能力 | `CC` 中选择的值必须落在 `CAP` 报告的合法区间内 |
| 时序前置 | I/O 队列条目大小必须在创建 I/O 队列前完成初始化 |
| 中断掩码对 | `INTMS` 写 1 屏蔽、`INTMC` 写 1 解除屏蔽，写 0 无效；两者都用于引脚/MSI，不可用于 MSI-X |
| 版本编码 | `VS` 编码主/次/第三级版本号；Revision 2.1 编码为 `MJR=2h, MNR=1h, TER=0h` |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 控制器属性 vs 命令寄存器 | 属性是查询/配置寄存器，命令是提交 I/O 或 Admin 任务的入口 |
| `CAP` vs `CC` | `CAP` 是能力（设备报告它**能**做什么），`CC` 是配置（主机告诉它**这次**怎么做） |
| `INTMS` vs `INTMC` | 写 1 含义相反：`INTMS` 屏蔽位，`INTMC` 解除屏蔽位；都是只读当前掩码状态 |
| `CAP.TO` vs `CRTO` | `CAP.TO` 是 500 ms 单位最多 127.5 秒；支持 `CRTO` 的控制器可在那里报告更长的就绪超时 |
| `MQES` vs 实际队列深度 | `MQES` 是 0 基编码（`1h` = 2 条），实际深度 = `MQES + 1` |
| Fabrics Property Get vs PCIe MMIO | 访问机制不同，但作用到的是**同一份**属性表 |
| `DSTRD=0h` 含义 | 编码为 `0h` 仍然表示步长 4 字节，不是"步长为 0" |

## 进阶细节

- **属性映射表（Figure 33）**：`00h` CAP、`08h` VS、`0Ch` INTMS、`10h` INTMC、`14h` CC、`1Ch` CSTS、`20h` NSSR（可选）、`24h` AQA、`28h` ASQ、`30h` ACQ 等。通用区到 `0FFFh` 截止，往上交给传输。
- **CAP 关键位编码**：
  - `MPSMIN` / `MPSMAX`：主机页大小 = `2^(12+value)` 字节。
  - `DSTRD`：Doorbell 步长 = `2^(2+value)` 字节，`0h` = 4 字节。
  - `TO`：最坏就绪时间 = `(value+1) × 500 ms`，`FFh` = 127.5 秒。
  - `MQES`：0 基最大队列条目数，`1h` = 2 条。
  - `CQR`：置 1 时队列需物理连续；消息型传输（Fabrics）永远置 1。
- **CQR 与传输的耦合**：PCIe 控制器可置 0（允许非连续队列），Fabrics 控制器必须置 1（队列连续）。
- **MQES 范围语义差异**：PCIe 下 `MQES` 同时限制 I/O SQ/CQ；Fabrics 下只限制主机创建的 I/O SQ。
- **VS 版本语义**：字母版本（如 2.1a）与对应的数字版本共享同一编码，便于前后兼容。
- **中断模型边界**：`INTMS`/`INTMC` 只用于引脚中断和 MSI；一旦切到 MSI-X，必须走 MSI-X 自带的中断掩码表。
- **Fabrics Property 报文细节**：Get 在响应字节 0–7 返回值；Set 在命令字节 48–55 嵌入写入值；非法属性/偏移/宽度返回 `Invalid Field in Command`。
- **就绪超时来源**：修订版 ≤ 2.0 控制器一般只看 `CAP.TO`；2.1 起 `CRTO` 允许报告超过 127.5 秒的真实就绪时长，主机应同时读两者。

## 规范依据

- [Controller Properties 总述与属性映射 Figure 33，PDF 第 71 页](../_source/pages/page-071.md)
- [属性访问规则与传输差异，PDF 第 71–73 页](../_source/pages/page-071.md)
- [CAP 字段定义 Figure 36，PDF 第 74–77 页](../_source/pages/page-074.md)
- [VS / INTMS / INTMC 与 CC 起始部分，PDF 第 77–78 页](../_source/pages/page-077.md)
- [CC 使能与队列大小前置条件，PDF 第 78–79 页](../_source/pages/page-078.md)

## 相关阅读

- [controller-support-requirements.md](controller-support-requirements.md) - 揭示属性支持与否的矩阵
- [controller-enable-shutdown-reset.md](controller-enable-shutdown-reset.md) - CC/CSTS 直接联动
- [transport-models.md](transport-models.md) - PCIe MMIO 与 Fabrics 两种访问路径
- [identify-command-model.md](identify-command-model.md) - Identify 数据也走属性映射
