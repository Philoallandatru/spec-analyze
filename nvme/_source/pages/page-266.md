---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 266
headings: ["5.1.12.1.14.2.6 Change Namespace Event (Event Type 06h)"]
---

# Source page 266

NVM Express® Base Specification, Revision 2.1

244
Figure 238: Additional Hardware Error Information for correctable and uncorrectable PCIe
errors
Bytes Description
79:64
PCIe AER TLP Prefix Log Register (PCIEATPLR): Contains the contents of the PCIe AER
TLP Prefix Log Register ( refer to the Advanced Error Reporting Capability section in the
NVMe over PCIe Transport Specification), if supported, at the time of the event.

Figure 239: Additional Hardware Error Information for Controller Ready Timeout
Exceeded errors
Bytes Description
0
Controller State (CST): Indicates the state of the controller at the time the Controller Ready
Timeout Exceeded error occurred.
Bits Description
7:4 Reserved
3
Controller Not Ready (CNR): Indicates the controller was not ready to process
at least one command without error as described in section 3.5.4.1 within the
amount of time indicated by:
a) the Controller Ready With Media Timeout (CRTO.CRWMT)  field in
Controller Ready With Media mode (CC.CRIME is cleared to ‘0’); or
b) the Controller Ready Independent of Media Timeout (CRTO.CRIMT)
field in Controller Ready Independent of Media mode (CC.CRIME is
set to ‘1’),
since the controller was enabled.
2
Admin Command Media Not Ready  (ACMNR): Indicates media required to
process at least one Admin command was not ready within the amount of time
indicated by the Controller Ready With Media Timeout (CRTO.CRWMT)  field
since the controller was enabled by transitioning CC.EN from ‘0’ to ‘1’.
1
Namespace Not Ready (NNR): Indicates at least one namespace attached to
the controller was not ready within the amount of time indicated by the Controller
Ready With Media Timeout (CRTO.CRWMT)  field since the controller was
enabled by transitioning CC.EN from ‘0’ to ‘1’.
0
Controller Ready Independent of Media Enable (CRIME): Indicates the value
of the CC.CRIME bit when the Controller Ready Timeout Exceeded error
occurred.

3:1 Reserved
5.1.12.1.14.2.6 Change Namespace Event (Event Type 06h)
The Changed Namespace Event (refer to Figure 240) persists the host parameters used for successful
Namespace Management commands. This event type is specific to namespaces that are associated with
I/O Command Sets that specify logical blocks (e.g., the NVM Command Set and the Zoned Namespace
Command Set). The event contains a Persistent Event Log Event header and the Change Namespace
Event data.
The Changed Namespace Event shall set the Persistent Event Log Event Format Header:
• Event Type field to 06h; and
• Event Type Revision Field to 02h.
Figure 240: Change Namespace Event Data Format (Event Type 06h)
Bytes Description
03:00 Namespace Management CDW10 (NMCDW10): Contains the value from command Dword 10 of the
Namespace Management command that initiated the namespace change event (refer to Figure 367).
07:04 Reserved
