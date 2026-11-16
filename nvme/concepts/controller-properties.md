# 控制器属性（Controller Properties）

控制器属性（Controller Property）是控制器暴露的一组与传输无关（transport-independent）的 4 字节（dword）或 8 字节（qword）控制平面寄存器表，用于查询能力、配置运行参数与读取状态；传输绑定（PCIe MMIO 或 Fabrics Property Get/Set）仅决定访问路径，不改变属性语义。[PDF pp. 71-73](../_source/pages/page-071.md)

可以把它想成**智能设备的控制面板**：无论你是站在设备前直接按按钮（PCIe MMIO），还是打电话让前台代按（Fabrics Property Get/Set），操作的都是同一张面板上的旋钮。

## 属性空间布局与访问约定

```text
                     控制器属性空间（Controller Property Space）
   +---------------------------------------------------------------+
   | 00h CAP | 08h VS | 0Ch INTMS | 10h INTMC | 14h CC | 1Ch CSTS  |  <- 通用区 (< 1000h)
   | 20h NSSR| 24h AQA| 28h ASQ   | 30h ACQ   | ...                |
   +---------------------------------------------------------------+
   | PCIe: 1000h 起为传输私有区（Doorbell 等）                       |  <- 传输/厂商区 (>= 1000h)
   | Fabrics: 1000h-12FFh 为 Fabrics 区，1300h 起为厂商私有区        |
   +---------------------------------------------------------------+
     ^                                      ^
     | Property Get/Set (Message-Based)     | MMIO 读写 (Memory-Based)
```

[PDF pp. 71-73, 479-481](../_source/pages/page-071.md)

**严格访问规则**：
- **精确宽度与对齐**：主机必须从属性起始偏移按其声明宽度（dword/qword）发起单次访问；**禁止跨属性访问**。
- **保留位规则**：保留属性和保留位为只读，读出返回 `0h`；写 0 允许，但软件不应依赖“不支持的访问返回 0”。
- **Fabrics 访问**：Property Get/Set 发起 4 字节或 8 字节访问（Get 结果在响应字节 0-7，Set 值在命令字节 48-55）；偏移或宽度非法报 `Invalid Field in Command`。[PDF pp. 479-481](../_source/pages/page-479.md)

| 访问标记 | 含义 |
|---|---|
| `RO` / `RW` | 只读 / 读-写 |
| `RWC` / `RWS` | 写 `1` 清零 / 写 `1` 置位（写 `0` 无效） |
| `Impl Spec` / `HwInit` | 实现特定 / 硬件初始化默认值 |

## 核心能力属性：`CAP`、`VS` 与 `INTMS/INTMC`

### `CAP`（Controller Capabilities，偏移 `00h`）

主机在启用控制器前读取 `CAP`，并在其约束范围内配置 `CC.AMS`、`CC.MPS`、`CC.CSS` 及队列条目大小：[PDF pp. 74-79](../_source/pages/page-074.md)

## 规范依据

| 关键字段 | 含义与编码 |
|---|---|
| `MQES` | 0-based 单队列最大条目数（最小 `1h` = 2 条）。PCIe 限制 I/O SQ 和 CQ；Fabrics 仅限制 I/O SQ [PDF p. 77](../_source/pages/page-077.md) |
| `CQR` | `1` 要求队列物理连续（Fabrics 永远置 1；PCIe 可置 0） |
| `TO` | 最坏就绪等待时间，单位 500 ms，`FFh` = 127.5 秒；支持 `CRTO` 时可报告更长超时 [PDF pp. 76, 78-79](../_source/pages/page-076.md) |
| `DSTRD` | Doorbell 步长 = $2^{(2+\text{value})}$ 字节（`0h` 表示 4 字节，非 0 步长） |
| `MPSMIN` / `MPSMAX` | 主机内存页大小范围 = $2^{(12+\text{value})}$ 字节 |

> **常见排错**：写 `CC.EN=1` 后控制器迟迟不就绪，优先检查 `CC.AMS`/`CC.MPS`/`CC.CSS` 是否超出 `CAP` 允许的范围。

### `VS`（Version）与中断掩码（`INTMS`/`INTMC`）

- **`VS`（偏移 `08h`）**：编码规范版本（如 Revision 2.1 为 `MJR=2h, MNR=1h, TER=0h`）；带字母后缀版本（如 2.1a）共享同一编码。[PDF pp. 77-78](../_source/pages/page-077.md)
- **`INTMS` / `INTMC`（偏移 `0Ch` / `10h`）**：成对的引脚中断 / MSI 掩码视图，读回当前掩码状态；往 `INTMS` 写 `1` 屏蔽向量，往 `INTMC` 写 `1` 解除屏蔽（写 `0` 无效）。**启用 MSI-X 后禁止访问这两个寄存器**（MSI-X 使用独立掩码表）。[PDF p. 78](../_source/pages/page-078.md)

## 概念辨析

| 容易混淆 | 实际区别 |
|---|---|
| `CAP` vs `CC` | `CAP` 是设备声明的静态能力边界；`CC` 是主机选定的运行配置 |
| `INTMS` vs `INTMC` | 两者读返回值相同；写 `1` 效果相反（屏蔽 vs 解屏蔽） |
| `MQES` vs 实际深度 | `MQES` 为 0-based 编码，实际最大条目数为 `MQES + 1` |

## 关联与引申概念

- **控制器支持矩阵（[Controller Support Requirements](controller-support-requirements.md)）**：决定不同类型控制器（I/O、Discovery、Administrative）必须或禁止支持哪些属性。
- **控制器启用与重置（[Controller Enable, Shutdown, and Reset](controller-enable-shutdown-reset.md)）**：详细展开 `CC`、`CSTS`、`NSSR`、`NSSD`、`AQA`/`ASQ`/`ACQ` 的状态机与握手规则。
- **传输模型（[Transport Models](transport-models.md)）**：对比 PCIe MMIO 与 NVMe-oF Property Get/Set 的底层传输差异。
- **Doorbell（门铃寄存器）**：位于 PCIe `1000h` 起的传输私有区，步长由 `CAP.DSTRD` 决定，主机写 Doorbell 通知控制器有新 SQE 或已消费 CQE。

## 规范依据

- [属性映射与访问规则（Figure 33），PDF 第 71-73 页](../_source/pages/page-071.md)
- [CAP 能力字段（Figure 36），PDF 第 74-77 页](../_source/pages/page-074.md)
- [VS、INTMS、INTMC 与 CC 起始部分，PDF 第 77-79 页](../_source/pages/page-077.md)
- [Fabrics Property Get/Set 命令，PDF 第 479-481 页](../_source/pages/page-479.md)
