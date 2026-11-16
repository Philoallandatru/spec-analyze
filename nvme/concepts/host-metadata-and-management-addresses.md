# 主机元数据与管理地址（Host Metadata and Management Addresses）

NVMe 定义了两类用于系统集成与带外/远程管理的 Feature：**管理地址（Management Address，FID `78h` / `79h`）**用于发布管理代理的 URI（如 BMC 或主机管理软件）；**主机元数据（Host Metadata，FID `7Dh` / `7Eh` / `7Fh`）**用于以 4 KiB 带类型描述符集合的形式存放主机平台、操作系统和命名空间注解。[PDF pp. 416-422](../_source/pages/page-416.md)

可以把它们想象成设备的**"名片夹 + 档案袋"**：名片夹（管理地址）写明"找系统级管家或主机级管家请连哪个 URI"；档案袋（主机元数据）则用来装操作系统版本、驱动信息、故障记录和命名空间卷标等描述性标签。

## 管理地址（Management Address）

管理地址使用 512 字节数据缓冲（前 4 字节保留，后 508 字节为符合 RFC 3986 的 URI）：[PDF pp. 416-417](../_source/pages/page-416.md)

| Feature | FID | 来源与指向 | 访问控制（Lockdown） |
|---|---|---|---|
| **Embedded Management Controller Address** | `78h` | 系统/硬件提供，指向机箱或主板级管理控制器（如 BMC） | 可被命令与功能锁定（Lockdown）限制为仅允许 Admin SQ 路径访问 |
| **Host Management Agent Address** | `79h` | 主机软件提供，指向 OS 或虚拟化层中的管理代理 | 可被 Lockdown 限制为仅允许带外管理端点（Management Endpoint）访问 |

> 注：NVM 子系统内代理与 Fabric 接口管理器的 URI 则通过 **Management Address List 描述符**（类型 `1h` 与 `2h`）发布，而非上述两个 Feature。[PDF pp. 549-550](../_source/pages/page-549.md)

## 主机元数据（Host Metadata）

三种主机元数据 Feature 均**可更改（Changeable=1）但不可保存（Saveable=0）**，每次读写传输固定 **4 KiB** 数据结构（首字节为描述符数量 `NMED`，字节 1 保留，字节 2 起为变长描述符列表；超 4 KiB 视为非法），且 Set Features 对缓冲内所有描述符**原子生效**。[PDF pp. 417-420](../_source/pages/page-417.md)

| Feature | FID | 作用域 | 同类型是否允许重复 | 典型用途 |
|---|---|---|---|---|
| **Enhanced Controller Metadata** | `7Dh` | 控制器 | **允许** | 现代推荐方式；适合记录多条同类条目（如多次故障记录） |
| **Controller Metadata** | `7Eh` | 控制器 | 禁止（每类最多 1 条） | 兼容旧版管理控制器（如 OS、BIOS、CPU、驱动、固件信息）；若支持 `7Dh` 应优先用 `7Dh` |
| **Namespace Metadata** | `7Fh` | 控制器 + 命名空间 | 禁止（每类最多 1 条） | 绑定特定控制器与 NSID 的 OS 卷标、启动管理器标签与限定符 |

### 元素操作（`EA`）与默认值生成（`GDHM`）

Set Features 通过 CDW11 bits `14:13` 的 **EA（Element Action）** 控制描述符增删改，不同 FID 的合法 `EA` 不同：[PDF pp. 418-420](../_source/pages/page-418.md)

| `EA` 值 | 操作含义 | `7Dh`（Enhanced） | `7Eh` / `7Fh` |
|---|---|---|---|
| `00b` | Add or Replace Entry（添加或替换） | **禁止**（返回 `Invalid Field in Command`） | **允许**（无则建，有则替） |
| `01b` | Delete Entry Multiple（删除多项） | **允许**（幂等，不存在不报错） | **允许**（幂等） |
| `10b` | Add Entry Multiple（添加多项） | **允许**（支持同类型多条） | 非设计目标 |

- **默认值生成（`GDHM`）**：Get Features 时置 CDW11 bit 0（`GDHM=1`）可请求控制器生成实现定义的默认字符串并更新"默认"返回值；该生成的默认值在后续 `SEL=001b` 读取中保持有效，但**不跨 Controller Level Reset**。[PDF pp. 418-420](../_source/pages/page-418.md)
- **命名空间元数据查询**：FID `7Fh` 必须通过 `NSID` 指定目标命名空间，若找不到对应 NSID 则返回 `Invalid Namespace or Format`。

## 规范依据

- [Embedded Mgmt Controller Address（78h）与 Host Mgmt Agent Address（79h），PDF pp. 416-417](../_source/pages/page-416.md)
- [GDHM 默认值生成与 EA 原子操作（4 KiB 限制），PDF pp. 418-419](../_source/pages/page-418.md)
- [Enhanced Controller（7Dh）、Controller（7Eh）与 Namespace Metadata（7Fh），PDF pp. 420-422](../_source/pages/page-420.md)
- [Management Address List 描述符类型 1h/2h，PDF pp. 549-550](../_source/pages/page-549.md)

## 关联与引申概念

- **命令与功能锁定（Command and Feature Lockdown）**：可锁定 `78h` / `79h` 等管理 Feature，防止未授权路径篡改 URI。详见 [命令与功能锁定](command-and-feature-lockdown.md)。
- **UTF-8 处理边界**：主机元数据中的字符串在协议层按原始二进制字节存储与比较，不做 Unicode 规范化或本地化排序。详见 [身份、名称与列表格式](identity-name-and-list-formats.md)。
- **Namespace Admin Label（FID `1Fh`）**：另一项以命名空间为作用域、必保存的 256 字节人读标签特性，与不可保存的 `7Fh` 用途不同。详见 [特性值与作用域](feature-values-and-scope.md) 与 [通用控制器特性](common-controller-features.md)。
