# 身份、名称与列表格式（Identity, Name, and List Formats）

NVMe 使用一套规范化的标识符家族来表达**厂商归属、人读设备信息、命名空间的全球唯一身份、子系统/主机名称（NQN）以及紧凑有序列表**。不同标识符在字节序、编码规范和唯一性作用域上有严格区分，混用会导致跨设备识别或迁移出错。[PDF pp. 183-189](../_source/pages/page-183.md)

可以把这套体系想象成**公司的"证件档案柜"**：`VID/SSVID/OUI/序列号` 是营业执照与出厂编号；`EUI64/NGUID/UUID` 是每个产品批次（命名空间）全球不撞号的身份证；`NQN` 是公司全名；而 `NSID` 和 `CNTLID` 只是公司内部的门牌号与工号（`CNTLID` 必须拼上子系统标识才全球唯一）。

## 标识符编码与字节序

| 标识符 | 宽度 / 形式 | 字节 / 字符串序 | 作用域与用途 |
|---|---|---|---|
| `VID`, `SSVID` | 各 16 位 | **小端（Little Endian）** | PCI SIG 分配的厂商与子系统厂商归属 |
| `IEEE OUI` | 24 位（3 字节） | Identify Controller 中为**小端** | 厂商组织唯一标识符；嵌入 `EUI64`/`NGUID` 时按**大端**存放 |
| Serial / Model Number | 定长 ASCII | 字符序，右侧空格填充 | 厂商分配的控制器/子系统序列号与型号 |
| `EUI64` | 8 字节 | **大端（Big Endian）** | 全球唯一命名空间标识符 |
| `NGUID` | 16 字节 | **大端（Big Endian）** | 含 OUI 的扩展命名空间唯一标识符（结构类似 NAA=6 WWN） |
| `UUID` | 16 字节 | RFC 9562 字节序 | 全球唯一命名空间标识符，或基于 UUID 的 NQN 组件 |
| `NQN` | ≤223 字节（含 null 结尾） | UTF-8 二进制串 | 主机或 NVM 子系统的全球永久唯一名称 |

> **易错点**：3 字节 `IEEE OUI` 在 Identify Controller 字段里是小端，但在 `EUI64` 和 `NGUID` 里是大端字节串的一部分；多版本 UUID 可共存，其中 UUIDv8 的唯一性需由同一管理域内的所有实现共同保证不冲突。[PDF pp. 183-186](../_source/pages/page-183.md)

## 命名空间持久身份与复用规则

新建的命名空间必须通过非零 `EUI64`、非零 `NGUID`，或 Namespace Identification Descriptor（`CNS 03h`，Type `3h`）中的 `UUID` **至少暴露一个全球唯一标识符**（若前两者均为 0，则必须提供有效的 UUID）。[PDF pp. 188-189](../_source/pages/page-188.md)

- **与 `NSID` 的区别**：`NSID` 只是子系统内的命令句柄，跨设备迁移应以 `EUI64`/`NGUID`/`UUID` 为准。
- **ID 复用控制（`UIDREUSE`）**：`UIDREUSE=0` 允许将已删除命名空间的非零 `EUI64`/`NGUID` 复用到新创建的命名空间；`UIDREUSE=1` 禁止此类复用。[PDF p. 189](../_source/pages/page-189.md)

## NVMe 限定名（NQN）与 UTF-8 边界

```text
1) 域权威形式 (Domain-authority form):
   nqn.yyyy-mm.<reversed-domain>:<authority-defined suffix>
   (日期须在权威方拥有该域名期间；反向域名不得为 org.nvmexpress)

2) UUID 形式 (UUID form):
   nqn.2014-08.org.nvmexpress:uuid:<RFC-9562 UUID text>
```

NQN 是以 null 结尾的 UTF-8 字符串，最长 223 字节，在主机或 NVM 子系统的整个生命周期内永久不变（仅允许 `-`、`.`、`:` 作为格式分隔符）。对于 NVMe 1.2.1 之前没有子系统 NQN 的旧控制器，主机可按 Figure 139 用标准前缀拼接 `VID`、序列号与型号构造传统 223 字节 NQN。[PDF pp. 187-188](../_source/pages/page-187.md)

**UTF-8 处理边界（Figure 140）**：Unicode 规范化、locale 敏感比较与 UTF-8 校验仅应发生在本地"配置入口"；NVMe 协议层在接收和比较 UTF-8 字符串（如 NQN、元数据）时**一律按原始二进制字节串处理**，不做规范化或本地化转换。[PDF pp. 189-190](../_source/pages/page-189.md)

## 有序列表与运行期标识 Feature

| 结构 / Feature | 格式与关键规则 |
|---|---|
| **Controller List**（Figure 137） | 16 位 `NUMCIDS` 头部（最大 2,047；0 表示空）+ 升序排列的 16 位 `CNTLID`；0 为填充/未用 |
| **Namespace List**（Figure 138） | 无头部；升序排列的 32 位 `NSID`；首个及之后未用条目填 0 |
| **Host Identifier（FID `81h`）** | 注册主机身份以关联预留（Reservations）与 Streams。PCIe 可选（64/128 位，默认 0）；Fabrics **强制 128 位**；均无 saved 值。非零值不能直接覆盖；在 ID=0 期间创建的资源即使后续注册了非零值，仍继续关联 0 身份 [PDF pp. 422-424](../_source/pages/page-422.md) |
| **Namespace Admin Label（FID `1Fh`）** | 必保存（mandatory-save）的 256 字节 null 结尾 UTF-8 人读标签。Set 要求 `SV=1`；外部管理工具的修改对 Get Features 可见；成功 Sanitize 后重置为全 0 [PDF pp. 414-415](../_source/pages/page-414.md) |

## 规范依据

- [VID/SSVID/OUI/SN/MN/EUI64/NGUID/UUID 字节布局（Figures 126–136），PDF pp. 183-186](../_source/pages/page-183.md)
- [Controller List 与 Namespace List（Figures 137–138），PDF pp. 186-187](../_source/pages/page-186.md)
- [NQN 形式、旧式构造（Figure 139）与唯一性范围，PDF pp. 187-189](../_source/pages/page-187.md)
- [UTF-8 入口/接收边界（Figure 140），PDF pp. 189-190](../_source/pages/page-189.md)
- [Namespace Admin Label（FID 1Fh）与 Host Identifier（FID 81h），PDF pp. 414-415, 422-424](../_source/pages/page-414.md)

## 关联与引申概念

- **Identify 命令模型**：Identify 命令（`CNS 01h/02h/03h/12h/13h`）是读取 NQN、EUI64/NGUID/UUID 以及有序列表的主要入口。详见 [识别命令模型](identify-command-model.md)。
- **命名空间标识符**：进一步区分 Active NSID、Allocated NSID 与持久标识的关系。详见 [命名空间标识符](namespace-identifiers.md)。
- **NVM 子系统**：NQN 在子系统发现与认证中的全局作用。详见 [NVM 子系统](nvm-subsystem.md)。
- **命令与功能锁定**：可通过锁定防止运行期修改 Host Identifier 等标识特性。详见 [命令与功能锁定](command-and-feature-lockdown.md)。
