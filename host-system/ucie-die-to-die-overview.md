# UCIe（Universal Chiplet Interconnect Express）芯粒互连概览

随着单颗芯片面积逼近光刻机掩膜极限，CPU、GPU 与 AI 芯片全面转向 **Chiplet（芯粒）** 先进封装架构。**UCIe** 是由 Intel、AMD、ARM、TSMC 等联合推出的开放式 **Die-to-Die（封装内芯粒间）互连标准**，它直接复用成熟的 **PCIe 6.0 与 CXL 3.x 协议栈**，把原本跨主板走线的总线搬进了毫米级的硅中介层（Silicon Interposer）里 `[UCIe 1.x/2.0 §1–2]`。

可以把 PCIe / CXL / UCIe 看成**同一套交通规则在三个不同尺度上的延伸**：
- **PCIe / CXL**：跨主板插槽 / 机柜线缆（厘米–米级，需要复杂均衡与高功耗 SerDes）。
- **UCIe**：同一块封装基板/硅中介层内的芯粒之间（微米–毫米级，超高密度、超低功耗、极低延迟）。

---

## UCIe 协议栈与 Flit 复用

```text
  ┌──────────────────────────────────────────────────────────┐
  │  Protocol Layer : 直接复用 PCIe 6.0 / CXL.io / CXL.mem/cache │
  ├──────────────────────────────────────────────────────────┤
  │  Die-to-Die Adapter : Flit 打包、CRC 校验与链路重传 (ARB/MUX) │
  ├──────────────────────────────────────────────────────────┤
  │  Physical Layer : Standard Package (有机基板) / Advanced (硅中介层) │
  └──────────────────────────────────────────────────────────┘
```

- **原生兼容软件生态**：由于上层直接复用 `PCIe` 与 `CXL` 协议，操作系统、BIOS 与现有驱动**几乎零感知**——一颗由 Compute Die + I/O Die（含 NVMe/PCIe Root Port）拼出来的 Chiplet CPU，在 OS 眼里依然是一颗标准的 PCIe/CXL 根复合体。
- **256B Flit 统一**：与 PCIe 6.0 / CXL 3.x 相同，UCIe 在 Adapter 层统一采用 **256B（或 68B）Flit** 进行流控与前向纠错。

---

## 关联与引申概念

- **CXL 协议复用**：UCIe 上层承载的核心一致性协议，详见 [CXL 3.x 架构与协议复用](cxl-architecture-overview.md)。
- **PCIe 6.0 物理层演进**：同样引入 256B Flit 与 FEC 的板级高速互连，详见 [PCS、PMA 与 PIPE 接口](../pcie/wiki/physical-layer/pcs-pma-pipe.md)。
