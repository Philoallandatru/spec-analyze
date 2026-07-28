# 主机元数据与管理地址（Host Metadata and Management Addresses）

## 一句话说明

NVMe 用两类 Feature 把"管理代理地址"和"主机环境描述"挂到控制器上：管理地址 Feature 存 URI，主机元数据 Feature 存 4 KiB 的带类型描述符集合，让远程管理和主机平台/命名空间注解能跨子系统传递。

## 生活化类比

把这套特性想成**设备的"名片 + 档案袋"**：

- **管理地址（78h/79h）** = 名片上"找我请打这个号码"：
  - `78h` Embedded Mgmt Controller Address：系统/机箱级（BMC 之类）打过来的号码
  - `79h` Host Mgmt Agent Address：主机软件里的管家号码
- **主机元数据（7Dh/7Eh/7Fh）** = 档案袋里的"产品说明书 + 维修记录 + 标签贴"：
  - `7Dh` Enhanced Controller Metadata：可以贴多张同类型（比如多条故障记录）
  - `7Eh` Controller Metadata：每种只能贴一张（兼容老格式）
  - `7Fh` Namespace Metadata：某控制器下某命名空间的专属标签
- **GDHM** = "请帮我预填一摞厂商默认表"
- **命令与功能锁定** = 名片和档案袋"上锁"——非授权不许改

## 工作流程

```text
  管理地址（Feature 78h / 79h，Figure 428/429）：
    Set/Get Features 走 512 字节数据缓冲：
      +--------+-------------------------------------+
      | 03:00  | 511:04                              |
      +--------+-------------------------------------+
      | 保留   | URI (RFC 3986)                      |
      +--------+-------------------------------------+

  主机元数据（Feature 7Dh / 7Eh / 7Fh）：
    Set/Get Features 走 4 KiB Host Metadata 数据结构：
      +------+------+----------------------------------+
      | 00   | 01   | x:02 ... 4095:z                  |
      +------+------+----------------------------------+
      | NMED | Resv | Metadata Element Descriptor 0..N |
      +------+------+----------------------------------+
      其中每个描述符（Figure 433）：
        +----------+-------------------------------+
        | 31:16    | 31+(ELEN*8):32                |
        +----------+-------------------------------+
        | ELEN     | EVAL (Element Value)          |
        | Element  |  Element Type 在更前面的位     |
        | Length   |                               |
        +----------+-------------------------------+

  CDW11 控制：
    Get Features: bit 0 = GDHM（生成默认）
    Set Features: bit 14:13 = EA（Element Action）
      00b = Add or Replace Entry
      01b = Delete Entry Multiple
      10b = Add Entry Multiple
      11b = 保留
```

## 初学者案例

**场景：写 4 KiB 元数据缓冲区后，控制器说 `Invalid Field in Command`。**

1. 你调用 `nvme set-feature -f 0x7D` 写 Enhanced Controller Metadata。
2. 在 4 KiB 缓冲里放了 3 个描述符，CDW11 `EA=00b`（Add or Replace Entry）。
3. 控制器返回 `Invalid Field in Command`。
4. 原因：Enhanced Controller Metadata **不允许** `EA=00b` 直接替换；必须用 `EA=10b`（Add Entry Multiple）才能添加描述符；替换场景下"加多"才是合法。
5. 把 CDW11 改成 `EA=10b` 重发，成功。
6. 想删除某个 Element Type，把 `EA=01b`，并把要删的描述符填在缓冲里；不存在的描述符按幂等成功（不报错）。

> 排错提示：`EA` 的合法值随 Feature 变化：7Dh 不能用 00b 替换；7Eh/7Fh 替换/添加用 00b。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| 管理地址 Feature | `78h` Embedded Mgmt Controller Address；`79h` Host Management Agent Address |
| 管理地址缓冲 | 512 字节；前 4 字节保留；剩余 508 字节为 RFC 3986 URI |
| 管理地址锁定 | 命令与功能锁定可禁止未授权修改；`78h` 可限制为仅 Admin SQ；`79h` 可限制为带外管理端点 |
| 元数据 Feature | `7Dh` Enhanced Controller Metadata；`7Eh` Controller Metadata；`7Fh` Namespace Metadata |
| 缓冲大小 | 4 KiB Host Metadata 数据结构 |
| 元素操作（EA） | `00b` Add/Replace；`01b` Delete Multiple；`10b` Add Multiple；`11b` 保留 |
| 元素添加/替换 | 7Eh/7Fh 支持 00b；7Dh **禁用** 00b（直接 `Invalid Field in Command`） |
| 元素添加多个 | 仅 7Dh 支持 10b 一次性添加多条同 Element Type |
| 元素删除 | 7Dh/7Eh/7Fh 都支持 01b；幂等；不存在的描述符视作成功 |
| 描述符唯一性 | 7Eh/7Fh：每 Element Type 最多一条；7Dh：允许同 Element Type 多条 |
| Capabilities 报告 | 三个 FID 用 `SEL=011b` Get 时：Saveable=0，Changeable=1 |
| GDHM | Get Features CDW11 bit 0；置 1 时控制器用厂商默认字符串替换"默认"返回值 |
| GDHM 持久性 | 替换的默认**不**跨 Controller Level Reset |
| 原子性 | 每次 Set Features 对 4 KiB 内所有描述符原子生效 |
| 大小约束 | 超过 4 KiB 的结构视为非法 |
| 默认值 | `SEL=001b` 读默认；Set/Get 后**已修改的**默认（GDHM 产生的）继续生效直至 Controller Level Reset |
| 元素类型目录 | 7Eh 的元素类型目录在规范 5.1.25.1.26.2；7Fh 同上 |
| 文本处理 | 元数据中字符串遵守规范 4.8 的 UTF-8 边界（不规范化/不本地化） |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| 78h vs 79h | 78h=嵌入式/系统级；79h=主机软件级 |
| 7Dh vs 7Eh | 7Dh 允许多条同类型；7Eh 每种一条 |
| 7Eh vs 7Fh | 7Eh 控制器级；7Fh 控制器+命名空间级 |
| EA=00b vs 10b | 00b=Add/Replace（不支持 7Dh）；10b=Add Multiple（仅 7Dh） |
| EA=01b vs 10b | 01b=Delete Multiple；10b=Add Multiple（不是 Add Replace） |
| GDHM 1 vs 0 | 1=生成新默认；0=不修改默认（Get 返回当前已修改默认） |
| 4 KiB vs 8 KiB | 元数据缓冲固定 4 KiB；管理地址 512 字节 |
| 默认 vs 当前 | GDHM 改的是"默认"返回值；不是 current |
| 描述符删除 vs 设为零 | 01b 是删除；0h 描述符只是"未用占位" |
| 命名空间元数据 vs 命名空间标签 | 元数据在控制器内；Namespace Admin Label (`1Fh`) 是另一 Feature |

## 进阶细节

- **Figure 428 Embedded Mgmt Controller Address 缓冲**：512 字节，前 4 字节保留，508 字节 RFC 3986 URI。
- **Figure 429 Host Mgmt Agent Address 缓冲**：512 字节，前 4 字节保留，508 字节 RFC 3986 URI。
- **Figure 430 Get Features CDW11**：`00`=GDHM（其余位保留）。
- **Figure 431 Set Features CDW11**：
  - `31:15` 保留
  - `14:13` EA（00b Add/Replace；01b Delete Multiple；10b Add Multiple；11b 保留）
  - `12:00` 保留
- **Figure 432 Host Metadata Data Structure**（4 KiB）：
  - 字节 0：NMED（描述符数量）
  - 字节 1：保留
  - 字节 2 起：Metadata Element Descriptor 列表（每个变长）
- **Figure 433 Metadata Element Descriptor**：
  - `31+(ELEN*8):32` EVAL（Element Value）
  - `31:16` ELEN（Element Length in bytes）
  - 更前的位是 Element Type
  - 删除时 `ELEN=0`、`EA=01b`
- **EA=00b 在 7Dh 的失败**（规范 5.1.25.1.26.1）：`Add or Replace Entry` 对 Enhanced 不合法，回 `Invalid Field in Command`，**不**改变 Feature 值。
- **EA=10b 在 7Eh/7Fh**（隐含）：不是设计目标；控制器按规范解释，替换/添加用 00b。
- **GDHM 实现差异**：可返回 0 个 / 子集 / 全部 Element Type 的厂商默认字符串，行为实现定义。
- **GDHM 持久范围**：仅跨当前"默认"读取的语义；Controller Level Reset 后丢失。
- **Capabilities**：`SEL=011b` 时 3 个 FID 的 CQE DW0：Saveable=0；Changeable=1；NSSPEC 视作用域。
- **命名空间元数据作用域**：必须用 `NSID` 指定"哪个控制器 + 哪个命名空间"；`Get` 不到该 NSID 时回 `Invalid Namespace or Format`。
- **7Dh 元素类型**：与 7Eh/7Fh 类似但允许多条同 Type。
- **描述符缓存**：Set/Get 都用同一 4 KiB 结构；Get 读出所有当前描述符；Set 提供完整替换或部分增删（取决于 EA）。

## 规范依据

- [Embedded Mgmt Controller Address 78h + Figure 428，PDF 第 416 页](../_source/pages/page-416.md)
- [Host Mgmt Agent Address 79h + Host Metadata 系列 FID，PDF 第 417 页](../_source/pages/page-417.md)
- [GDHM 默认值生成，PDF 第 418 页](../_source/pages/page-418.md)
- [EA 元素操作与 4 KiB 上限，PDF 第 419 页](../_source/pages/page-419.md)
- [Enhanced Controller Metadata 7Dh + Figure 432/433，PDF 第 420 页](../_source/pages/page-420.md)
- [Controller Metadata 7Eh 元素目录，PDF 第 421 页](../_source/pages/page-421.md)
- [Namespace Metadata 7Fh，PDF 第 422 页](../_source/pages/page-422.md)
- [Management Address List 描述符类型 1h/2h，PDF 第 549-550 页](../_source/pages/page-549.md)

## 相关阅读

- [特性值与作用域](feature-values-and-scope.md) - 7Dh-7Fh 都是 Feature
- [通用控制器特性](common-controller-features.md) - 主机元数据 FID 详解
- [命令与功能锁定](command-and-feature-lockdown.md) - 元数据锁定场景
- [控制器虚拟化资源](controller-virtualization-resources.md) - 多控制器元数据行为
