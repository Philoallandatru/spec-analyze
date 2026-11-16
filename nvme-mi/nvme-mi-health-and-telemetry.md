# 健康状态轮询、Admin 隧道与 BMC 排错

在数据中心运维中，BMC 最核心的日常任务是**以秒级或十秒级周期轮询服务器内所有 NVMe SSD 的健康与温度**，从而动态调节风扇转速（PID 调速）并在坏盘前发出预警 `[NVMe-MI §5]`。

---

## 高频带外健康命令

1. **`NVM Subsystem Health Status Poll`（MI Opcode `01h`）**：
   - 一次轻量级请求即可返回整个子系统的聚合健康状态：**Composite Temperature（复合温度）**、**Available Spare（剩余备用块百分比）**、**Percentage Used（寿命消耗）** 以及 **Critical Warnings（严重警告位图：温度/寿命/只读/掉电保护电容失效）**。
   - 若子系统包含多颗控制器（如 Dual-Port 双活 SSD），可配合 **`Controller Health Status Poll`（Opcode `02h`）** 分别查询每颗控制器的独立状态与错误计数。
2. **`MI Admin Tunneling`（`NMIMT = 2h`）**：
   - 当健康轮询发现异常（例如 `Critical Warning` 置位），BMC 可通过 Admin 隧道发送 **`Get Log Page`（SMART/Health `LID 02h` 或 Telemetry `LID 07h/08h`）**，抓取完整黑匣子日志，而无需打扰主机业务 OS。

```bash
# 在 Linux 中通过带内方式或 libnvme/nvme-cli 调试 NVMe-MI 健康状态（需内核/工具支持）
nvme smart-log /dev/nvme0
# 通过管理接口插件检查控制器/子系统状态（视具体平台 mi-cli/libnvme-mi 工具链而定）
nvme get-log /dev/nvme0 --log-id=0x02 --log-len=512
```

---

## BMC 联调常见坑与排错清单

| 常见现象 | 根因定位 | 排查手段 |
| :--- | :--- | :--- |
| **SMBus 读取大数据（如 Telemetry Log）超时或乱码** | MCTP over SMBus 基线 **MTU 仅 64 字节**，长日志需拆成数百个分片；若总线时钟拉伸（Clock Stretching）超时或任意一帧 **`PEC` 校验失败**，整条消息被丢弃 | 检查 BMC I2C/SMBus 控制器的 Clock Stretching 超时阈值，抓包确认 `SOM`/`EOM`/`PktSeq` 连续性与每帧尾 `PEC` |
| **主 12V 断电后 BMC 找不到盘，上电后又能找到** | 1) 盘仅接了 `12V` 未接 `3.3Vaux`，或盘固件不支持 `3.3Vaux` 唤醒管理模块；2) BMC 走的是 **PCIe VDM**（L0 依赖主供电）而非 SMBus | 确认背板 `3.3Vaux` 供电与 SMBus 走线，或仅在主机开机后通过 PCIe VDM 轮询 |
| **PCIe VDM 发送后石沉大海** | 1) VDM TLP 的 Vendor ID 未设为 `0x1AB4`；2) BMC 与 SSD 之间的 PCIe Switch 未开启 VDM 广播/按 ID 路由转发；3) `EID` 尚未通过 `Set Endpoint ID` 分配 | 用 PCIe 抓包确认 Switch 是否透传 `Type 1 VDM (0x1AB4)`，并确认先执行 MCTP `Set EID` |
| **热插拔换盘后 BMC 数据陈旧** | 换盘后新盘未重新分配 `EID`，或 BMC 未感知 `PRSNT#` 变化重新执行 MCTP 发现 | 监听背板 `PRSNT#` 中断，热插后重新触发 MCTP `Get Endpoint UUID` + `Set EID` |

---

## 关联与引申概念

- **SMART 与健康日志**：带外 `Health Poll` 与带内 `LID 02h` 的字段映射关系，详见 [错误与健康日志](../nvme/concepts/error-and-health-logs.md)。
- **遥测日志抓取**：当盘发生故障时通过带外/带内抓取 Telemetry，详见 [遥测捕获日志](../nvme/concepts/telemetry-capture-logs.md)。
- **MCTP 协议细节**：`0x1AB4`、64B MTU 与 `PEC` 的底层规则，详见 [MCTP 传输层与 PCIe VDM / SMBus 封装](mctp-transport-and-vdm.md)。
