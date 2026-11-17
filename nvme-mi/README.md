# NVMe-MI 带外管理与 MCTP 传输（NVMe-MI & MCTP）

本模块基于 **NVMe Management Interface (NVMe-MI) Specification** 与 **DMTF MCTP（DSP0236 / DSP0238 / DSP0237）**，拆解服务器 **BMC（基板管理控制器）如何在不经过操作系统与主机 NVMe 驱动的情况下（带外 Out-of-Band）**，直接发现、监控温度/寿命、抓取遥测日志并控制 NVMe SSD。

> **20/80 裁剪原则**：聚焦 MCTP 分片与 PCIe VDM (`0x1AB4`) / SMBus 封装、`NMIMT` 消息类型、带内/带外通道对比，以及 BMC 联调最常碰到的健康轮询与 `PEC`/MTU 排错。

---

## 文档导航

1. **[MCTP 传输层与 PCIe VDM / SMBus 封装](mctp-transport-and-vdm.md)**
   - MCTP 通用包头（`EID`/`SOM`/`EOM`/`PktSeq`/`MsgTag`）、`Message Type 04h`、MCTP over PCIe VDM（`0x1AB4`）与 MCTP over SMBus/I3C（64B MTU 与 `PEC`）对比。
2. **[NVMe-MI 架构、带内/带外通道与消息类型](nvme-mi-architecture-and-messages.md)**
   - Out-of-Band（BMC）vs In-Band（Admin `1Dh`/`1Eh`）、`NMIMT` 五大消息族、Control Primitive 复位与 VPD/FRU 读取。
3. **[健康状态轮询、Admin 隧道与 BMC 排错](nvme-mi-health-and-telemetry.md)**
   - `NVM Subsystem Health Status Poll`、MI Admin Tunneling（带外执行 `Identify`/`Get Log Page`）以及常见分片/校验/时钟拉伸坑。
4. **[PLDM 固件升级与 Redfish 存储管理](pldm-and-redfish-storage.md)**
   - MCTP 之上的 `PLDM Type 2/5 (DSP0267)` 统一固件升级、`RDE (DSP0218)` 与 Redfish `Drive`/`Volume` 模型。

---

## 关联与引申概念

- **规范族定位**：NVMe-MI 作为独立于 Base 规范的带外管理成员，详见 [规范族与适用范围](../nvme/concepts/specification-family-and-scope.md)。
- **带内健康与遥测**：主机侧如何通过 Admin 命令读取相同健康信息，详见 [错误与健康日志](../nvme/concepts/error-and-health-logs.md) 与 [遥测捕获日志](../nvme/concepts/telemetry-capture-logs.md)。
- **物理边带引脚**：SMBus/I3C 与 `3.3Vaux` 在背板连接器上的位置，详见 [EDSFF 架构与连接器](../pcie-form-factors/edsff-e1-e3-architecture.md)。
