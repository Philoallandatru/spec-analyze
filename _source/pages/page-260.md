---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 260
headings: ["5.1.12.1.14.1 Persistent Event Log Page LID Specific Parameter Field", "5.1.12.1.14.2 Persistent Event Log Events", "5.1.12.1.14.2.1 SMART / Health Log Snapshot Event (Event Type 01h)"]
---

# Source page 260

NVM Express® Base Specification, Revision 2.1

238
 Persistent Event Log Page LID Specific Parameter Field
Figure 228 specifies the format for the LID Specific Parameter field in the Supported Log Pages log page
(refer to section 5.1.12.1.1) for the Persistent Event log page.
Figure 228: Persistent Event LID Specific Parameter Field
Bits Description
15:1 Reserved
0
Establish Context and Read 512 Bytes of Header Supported (ECRH): If this bit is cleared to ‘0’, then
the controller does not support the Establish Context and Read 512 Bytes of Header action (refer to Figure
225).
If this bit is set to ‘1’, then the controller supports:
• the Establish Context and Read 512 Bytes of Header action; and
• the Generation Number field in the Persistent Event log page.
If the Persistent Event log page is supported by the controller, then i mplementations compliant with NVM
Express Base Specification, Revision 2.0 and later shall set this bit to ‘1’.
 Persistent Event Log Events
The values that may be reported in the Event Type field (refer to section 5.1.12.1.14) of the event header
for events in the Persistent Event log are defined in Figure 229.
Figure 229: Persistent Event Log Event Types
Type O/M1 Event Reference Section
00h  Reserved
01h NOTE 2 SMART / Health Log Snapshot 5.1.12.1.14.2.1
02h M Firmware Commit 5.1.12.1.14.2.2
03h M Timestamp Change 5.1.12.1.14.2.3
04h M Power-on or Reset 5.1.12.1.14.2.4
05h M NVM Subsystem Hardware Error 5.1.12.1.14.2.5
06h NOTE 3 Change Namespace 5.1.12.1.14.2.6
07h NOTE 3 Format NVM Start 5.1.12.1.14.2.7
08h NOTE 3 Format NVM Completion 5.1.12.1.14.2.8
09h NOTE 3 Sanitize Start 5.1.12.1.14.2.9
0Ah NOTE 3 Sanitize Completion 5.1.12.1.14.2.10
0Bh O Set Feature 5.1.12.1.14.2.11
0Ch O Telemetry Log Created 5.1.12.1.14.2.12
0Dh O Thermal Excursion 5.1.12.1.14.2.13
0Eh O Sanitize Media Verification Event 5.1.12.1.14.2.14
0Fh to DDh  Reserved
DEh O Vendor Specific Event 5.1.12.1.14.2.15
DFh O TCG Defined 5.1.12.1.14.2.16
E0h to FFh  Reserved
Notes:
1. O/M definition: O = Optional, M = Mandatory.
2. Mandatory for NVMe over PCIe implementations, Optional for NVMe over Fabrics implementations.
3. Mandatory if the command used to initiate the activity reported by the event is supported.
5.1.12.1.14.2.1 SMART / Health Log Snapshot Event (Event Type 01h)
NVM subsystems that support the Persistent Event Log shall create a SMART / Health Log Snapshot Event:
a) If virtualization management is not implemented, then for every controller in the NVM subsystem;
or
b) If virtualization management is implemented, then for every primary controller,
at least once every 24 power on hours at a time determined by the controller.
