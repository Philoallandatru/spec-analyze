# 身份、名称与列表格式（Identity, Name, and List Formats）

## 一句话说明

NVMe 用一套统一的"标识符家族"表示厂商归属、人读信息、命名空间全球唯一身份、子系统/主机名称以及紧凑有序列表；不同标识符有不同的字节序、唯一性范围和用途，不能混用。

## 生活化类比

把这套标识符想成**一个公司的"证件档案柜"**：

- **VID/SSVID/IEEE OUI/序列号/型号** = 公司营业执照 + 厂牌 + 出厂编号（PCI 厂商分配，决定责任方）。
- **EUI64 / NGUID / UUID** = 每个"产品批次"（命名空间）的全球唯一身份证号，跨公司也不会撞。
- **NQN** = 公司全名（"nqn.2014-08.com.example:xxx"），跨公司也要唯一。
- **Controller List / Namespace List** = 公司的"门店清单"和"产品清单"，按编号升序排列。
- **Host Identifier / Namespace Admin Label** = "员工编号"和"产品备注名"，运行期可改但有约束。

## 工作流程

```text
  标识符家族（按用途分组）：

  厂商/展示 ─────────────── 字节序
    VID / SSVID (16-bit)     小端（PCI 分配）
    IEEE OUI (24-bit)        小端（Identify Controller 用）
    Serial Number / Model    字符序 + 空格右补
    EUI64 (8B)               大端（IEEE 分配的扩展唯一标识）
    NGUID (16B)              大端（OUI + Vendor 扩展 + 扩展标识）

  全球唯一身份 ────────────  字节序
    UUID (16B)               RFC 9562 顺序
    NQN (≤223B null-term)    UTF-8 字符串（不规范化/不本地化）

  控制器全球身份 = 子系统标识 + CNTLID

  列表结构 ───────────────── 字节序
    Controller List          NUMCIDS (16-bit, ≤2047) + CNTLID 升序
    Namespace List           NSID (32-bit) 升序，无头部
```

## 初学者案例

**场景：跨厂商迁移命名空间，UUID 怎么保证不撞？**

1. 你买一块新 SSD，配置工具调用 `nvme create-ns`。
2. 控制器选 UUIDv5（基于 NQN + 命名空间名）还是 UUIDv4（随机）由厂商实现决定。
3. 规范允许**多版本 UUID 共存**；UUIDv8（实验/厂商自定）唯一性是"在该组实现间共同负责"。
4. 如果厂商用 UUIDv8 但生成算法不同，**可能撞**——需要部署前约定算法（参考 RFC 9562 Appendix B）。
5. 配置工具应记录命名空间的 EUI64/NGUID/UUID，作为迁移时的"硬身份证"。
6. 若 `NSFEAT.UIDREUSE=0`，删除命名空间后其 NGUID/EUI64 可被新命名空间**复用**——可能引发数据归属混淆，要慎重。

> 排错提示：跨设备识别一定要用 UUID/EUI64/NGUID，**不要**用 NSID（NSID 只是子系统的临时句柄）。

## 必须记住的规则

| 规则 | 要点 |
|------|------|
| VID/SSVID | 16-bit，PCI SIG 分配，小端 |
| IEEE OUI | 24-bit，Identify Controller 用，**小端**；嵌入 EUI64/NGUID 时按大端存放 |
| EUI64 | 8 字节 IEEE EUI-64，**大端** |
| NGUID | 16 字节（OUI + Vendor Specific + 扩展），**大端**，类似 NAA=6 的 WWN |
| UUID | 16 字节，按 RFC 9562 顺序 |
| 多 UUID 版本 | 可共存；UUIDv8 唯一性由该组实现共同负责 |
| 命名空间全球身份 | 创建时**至少**一个非零 EUI64 / 非零 NGUID / Namespace Identification Descriptor 含 UUID (Type=3h) |
| UUID 强制 | 若 EUI64 与 NGUID 都为 0，**必须**支持 Namespace UUID |
| UIDREUSE=0 | 允许删除的 NS 的非零 NGUID/EUI64 被新 NS 复用 |
| UIDREUSE=1 | 禁止上述复用 |
| NQN 长度 | ≤223 字节 + null 终止 |
| NQN 编码 | UTF-8（RFC 3629），只能含 `-` `.` `:` 三种格式字符 |
| NQN 形式 1（域权威） | `nqn.yyyy-mm.<reversed-domain>:<authority-suffix>`；日期必须处于权威方拥有该域名的时段；反向域不得是 `org.nvmexpress` |
| NQN 形式 2（UUID） | `nqn.2014-08.org.nvmexpress:uuid:<UUID text>` |
| NQN 永久性 | 对 host / NVM 子系统整个生命周期永久 |
| NQN 比较 | 协议级按二进制字符串比较，不做规范化 / 本地化处理 |
| 旧子系统 NQN | 1.2.1 之前的控制器：可由主机从 `nqn.2014-08.org.nvmexpress:` + VID + SN + MN 拼出（Figure 139） |
| Controller List | 16-bit `NUMCIDS` 头，最多 2047 项；CNTLID 升序；0=未用；NUMCIDS=0 表示空 |
| Namespace List | 无头部；32-bit NSID 升序；未用条目为 0 |
| Host Identifier 81h | PCIe 可选（64/128-bit）；Fabrics 必选 128-bit；0 表示无关联；非 0 不能直接覆盖 |
| Host Identifier 持久性 | 无 saved value；Streams/Reservations 在 Host ID=0 时仍关联 0 身份 |
| Namespace Admin Label 1Fh | 256 字节 UTF-8，null 终止；**必保存**；Set 必须 `SV=1`；Sanitize 成功后重置为全 0 |

## 容易混淆的地方

| 容易混 | 实际区别 |
|--------|----------|
| VID vs IEEE OUI | VID 是 PCI SIG 16-bit；OUI 是 IEEE 24-bit（仅在 Identify 字段） |
| OUI 在 EUI64/NGUID 里的位置 | 嵌入大端；Identify 字段里小端——不要混用 |
| EUI64 vs NGUID | EUI64=8B；NGUID=16B（OUI + 扩展） |
| NSID vs UUID | NSID=子系统内句柄；UUID=全球身份 |
| CNTLID 全局唯一 | 不！CNTLID 仅子系统内唯一；加子系统标识才全球唯一 |
| NQN 域形式 vs UUID 形式 | 域形式给"有人事/有域"；UUID 形式给"无权威/无需人读" |
| UTF-8 字符串处理 | 协议层只比二进制；规范化/本地化只发生在"配置输入"环节 |
| Host ID 0 vs 后续非 0 | 0 期间创建的资源**仍**绑 0 身份，**不会**因后来非 0 而变 |
| UIDREUSE vs Delete NS | UIDREUSE=0 允许**非零** NGUID/EUI64 复用；0 永远不能复用 |
| NSID 0 vs NSID 广播 | NSID=0 是"未指定"；NSID=FFFFFFFFh 是"广播" |

## 进阶细节

- **EUI64/NGUID 字节序**（规范 4.5.4-4.5.5）：大端，IEEE 风格。
- **NGUID 与 WWN 关系**（Figure 136）：NGUID 结构与 NAA=6 的 WWN 类似；前 1B=`06h`，接着 OUI（3B），再是 Vendor Specific + 扩展。
- **UUID 字节序**：按 RFC 9562（混合字节序：前 3 部分 little-endian-like，后 2 部分 big-endian-like）。
- **UUIDv8 一致性**（规范 4.5.6）：同一管理域内所有实现用相同算法才能保证不撞；RFC 9562 Appendix B 给示例算法。
- **NQN 例子**：
  - `nqn.2014-08.com.example:nvme:nvm-subsystem-sn-d78432`
  - `nqn.2014-08.com.example:nvme.host.sys.xyz`
  - `nqn.2014-08.org.nvmexpress:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6`
- **NQN 旧式构造**（Figure 139）：1.2.1 之前控制器，host 端用 `nqn.2014-08.org.nvmexpress:` + 16-hex-char VID + 20-char SN + 40-char MN 拼成 223 字节 NQN。
- **Controller List 最大 2047**（Figure 137）：NUMCIDS 是 16-bit，但规范限 ≤2047；0=空。
- **Namespace List 无头部**（Figure 138）：第一个条目放最低 NSID，剩余条目按升序，未用条目=0。
- **Host Identifier 协议差异**（规范 5.1.25.1.28）：
  - PCIe：可选；支持 64/128-bit；默认 0；无 saved
  - Fabrics：必选 128-bit；无 saved
  - 同一子系统内一致非零宽度（不能跨控制器混用）
- **Host ID 写入限制**：非零当前值**不能**直接覆盖；需先特殊流程（部分实现允许控制器 Reset 清零）。
- **Namespace Admin Label**（Feature `1Fh`，规范 5.1.25.1.22）：
  - 256B UTF-8，null 终止
  - saved=current；Set 必须 `SV=1`
  - 外部管理改后，Get Features 立即可见
  - Sanitize 成功后 reset 到全 0
- **UTF-8 边界**（规范 4.8）：协议级对 NVMe 字符串按二进制存储/比较；Unicode 规范化、locale 敏感比较、UTF-8 校验只发生在"配置入口"边界，**不要**在收到 NVMe 字符串时应用。
- **CNTLID 子系统唯一**：跨子系统可能重复；全球身份必须结合子系统标识。

## 规范依据

- [VID/SSVID/OUI/SN/MN/EUI64/NGUID/UUID 定义，PDF 第 183-186 页](../_source/pages/page-183.md)
- [Controller List + Namespace List Figures 137-138，PDF 第 186-187 页](../_source/pages/page-186.md)
- [NQN 形式与约束 Figures 137-138 之后，PDF 第 187-188 页](../_source/pages/page-187.md)
- [唯一标识符与控制器/NS 身份作用域，PDF 第 188 页](../_source/pages/page-188.md)
- [命名空间 EUI64/NGUID/UUID 强制条件，PDF 第 189 页](../_source/pages/page-189.md)
- [UTF-8 处理边界 Figure 140，PDF 第 189-190 页](../_source/pages/page-189.md)
- [Host Identifier Feature 81h，PDF 第 422-424 页](../_source/pages/page-422.md)
- [Namespace Admin Label Feature 1Fh，PDF 第 414-415 页](../_source/pages/page-414.md)

## 相关阅读

- [识别命令模型](identify-command-model.md) - 由 Identify 读出这些标识
- [NVM 子系统](nvm-subsystem.md) - NQN 子系统命名规范
- [命名空间标识符](namespace-identifiers.md) - 命名空间标识体系
- [命令与功能锁定](command-and-feature-lockdown.md) - Host Identifier 关联锁定
