# PLDM 固件升级与 Redfish 存储管理（DMTF）

在现代服务器带外管理体系中，**MCTP** 只是“快递传输车”，车上除了拉 **NVMe-MI（`Type 04h`）** 外，还会拉 **DMTF PLDM（Platform Level Data Model，`Type 01h`）** 来实现跨厂商的传感器监控、**统一固件升级（FW Update）**，并通过 **Redfish** RESTful API 暴露给数据中心运维平台 `[DSP0240 / DSP0267 / DSP0218]`。

---

## PLDM 常见消息类型与核心场景

| PLDM 类型 | 规范编号 | 核心职责 |
| :--- | :--- | :--- |
| **Type 0 (`Messaging Control`)** | `DSP0240` | 发现端点支持的 PLDM 类型与版本 |
| **Type 2 (`Platform Monitoring & Control`)** | `DSP0248` | 标准化温度、电压、功耗传感器（PDR）与告警上报 |
| **Type 5 (`Firmware Update`)** | **`DSP0267`** | **统一带外固件升级**：BMC 将固件包分块下发至 SSD/NIC/Retimer，完成校验与激活，无需依赖各家私有工具 |
| **Type 6 (`RDE`, Redfish Device Enablement)** | **`DSP0218`** | 让设备自己托管紧凑二进制 JSON（BEJ）格式的 Redfish 模型，BMC 无需为每款新硬件写适配代码即可直接透传给 Redfish |

---

## Redfish 存储模型：`Storage` → `Drive` vs `Volume`

运维人员通过 HTTPS 访问 BMC Redfish 接口（`/redfish/v1/Systems/1/Storage`）时，存储资源被清晰拆分为物理与逻辑两层：
- **`Drive`（物理盘）**：对应一块真实的 NVMe SSD，展示序列号、温度、剩余寿命（`PredictedMediaLifeLeftPercent`）、固件版本与定位灯（`IndicatorLED`）。
- **`Volume`（逻辑卷 / 命名空间）**：对应从一块或多块盘切分出的 **NVMe Namespace** 或硬件 RAID 卷，展示容量、块大小与挂载状态。

```bash
# 通过 Redfish REST API 查询服务器某块存储盘的健康与固件版本
curl -s -k -u root:password https://<BMC-IP>/redfish/v1/Systems/1/Storage/1/Drives/NVMe0 | grep -E "Model|FirmwareVersion|Health"
```

---

## 关联与引申概念

- **MCTP 底层传输**：PLDM 与 NVMe-MI 共用的 MCTP 分片与 PCIe VDM / SMBus 通道，详见 [MCTP 传输层与 PCIe VDM / SMBus 封装](mctp-transport-and-vdm.md)。
- **带内固件升级**：主机 OS 通过 Admin 命令升级固件的流程，详见 [固件更新生命周期](../nvme/concepts/firmware-update-lifecycle.md)。
