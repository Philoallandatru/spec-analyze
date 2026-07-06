---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 265
headings: []
---

# Source page 265

NVM Express® Base Specification, Revision 2.1

243
Figure 237: Additional Hardware Error Information for Unexpected Power Loss Errors
Bytes Description
15:0 Unexpected Power Loss  (UPL): This field shall indicate the value from the Unexpected
Power Losses field in the SMART / Health Information log at the time of the event.
16
Unexpected Power Loss  Information (UPLI): This field contains additional information
about the unexpected power loss.
Bits Description
7:1 Reserved
0
Unexpected Power Loss  Due to Out -Of-Band Activity  (UPLOA): If the
shutdown was unsafe because media was not in a shutdown state when main
power was lost because an Admin command that accesses media as defined
by Figure 84 was processed via the out -of-band mechanism with the Ignore
Shutdown bit set to ‘1’ (refer to the NVM Express Management Interface
Specification) while shutdown processing was reported as in progress or was
reported as complete (i.e., while the value of the CSTS.SHST field was set to
01b or 10b), then this bit shall be set to ‘1’; otherwise, this bit shall be cleared
to ‘0’.


Figure 238: Additional Hardware Error Information for correctable and uncorrectable PCIe
errors
Bytes Description
01:00 PCIe Device Status Register (PCIEDSR): Contains the contents of the PCI Device Status
Register (refer to the PCI Express Base Specification) at the time of the event.
02
PCIe AER Support (PCIEAS):
Bits Description
7:1 Reserved
0
PCIe AER Supported (PCIEAERS): If this bit is set to ‘1’, then the PCIe AER
(refer to the PCI Express specification) is supported and that the PCIe AER
Error Status field, PCIe AER Error Mask field, PCIe AER Header Log Register
field, and the PCIe AER TLP Prefix Log Register field are reported. If this bit is
cleared to ‘0’, then the PCIe AER is not supported and that the PCIe AER Error
Status field, PCIe AER Error Mask field, PCIe AER Header L og Register field,
and PCIe AER TLP Prefix Log Register field are not reported (i.e., bytes 79:16
are not reported).

15:03 Reserved
31:16
PCIe AER Error Status (PCIEAES): Contains the contents of:
a) the PCIe AER Correctable Error Status Register (refer to the Advanced Error
Reporting Capability section in the NVMe over PCIe Transport Specification) at the
time of the event if the error is a correctable error; or
b) The PCIe AER Uncorrectable Error Status Register ( refer to the Advanced Error
Reporting Capability section in the NVMe over PCIe Transport Specification), at the
time of the event if the error is an uncorrectable error.
47:32
PCIe AER Error Mask (PCIEAEM): Contains the contents of:
a) the PCIe AER Correctable Error Mask Register ( refer to the Advanced Error
Reporting Capability section in the NVMe over PCIe Transport Specification) at the
time of the event if the error is a correctable error; or
b) the PCIe AER Uncorrectable Error Mask Register ( refer to the Advanced Error
Reporting Capability section in the NVMe over PCIe Transport Specification) at the
time of the event if the error is an uncorrectable error.
63:48
PCIe AER Header Log Register  (PCIEAHLR): Contains the contents of the PCIe AER
Header Log Register (refer to the Advanced Error Reporting Capability section in the NVMe
over PCIe Transport Specification), if supported, at the time of the event.
