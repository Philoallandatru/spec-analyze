---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 410
headings: ["5.1.25.1.16 Endurance Group Event Configuration (Feature Identifier 18h)", "5.1.25.1.17 I/O Command Set Profile (Feature Identifier 19h)"]
---

# Source page 410

NVM Express® Base Specification, Revision 2.1

388
5.1.25.1.16 Endurance Group Event Configuration (Feature Identifier 18h)
This Feature controls the events that trigger adding an Endurance Group Event Aggregate Log Change
Notices event to the Endurance Group Event Aggregate log  for the specified Endurance Group. This
Feature may be used to disable reporting events in the case of a persistent condition (refer to section 5.1.2).
If the condition for an event is true when the corresponding notice is enabled, then an event is sent to the
host. The attributes are specified in Command Dword 11.
If a Get Features command is submitted for this Feature, the Endurance Group Critical Warnings field in
Command Dword 11 is not used and the attributes specified in Figure 409 are returned in Dword 0 of the
completion queue entry for that command.
Figure 409: Asynchronous Event Configuration – Command Dword 11
Bits Description
31:24 Reserved
23:16
Endurance Group Critical Warnings  (EGCW): This field determines whether an event entry for an
Endurance Group (refer to section  3.2.3) is added to the Endurance Group Event Aggregate log page
(refer to section 5.1.12.1.15) for the corresponding Critical Warning specified in the Endurance Group
Information log page (refer to Figure 218). If a bit is set to ‘1’, then an entry is added when the
corresponding critical warning bit is set to ‘1’ in the Endurance Group Information log page. If a bit is
cleared to ‘0’, then an entry is not added when the corresponding critical warning bit is set to ‘1’ in the
Endurance Group Information log page.
15:00
Endurance Group Identifier (ENDGID):  This field indicates the Endurance Group for which
asynchronous events are being configured. If this field is cleared to 0h, then the Endurance Group Critical
Warnings field is not used.
If a bit is set to ‘1’ in the Endurance Group Critical Warnings field which corresponds to a reserved bit in the
Critical Warning field of the Endurance Group Information log page (refer to Figure 218), then the Set
Features command shall be aborted with a status code of Invalid Field in Command.
If the Endurance Group Identifier specifies an Endurance Group that does not exist, then the Set Features
or Get Features command shall be aborted with a status code of Invalid Field in Command.
5.1.25.1.17 I/O Command Set Profile (Feature Identifier 19h)
This Feature specifies the I/O Command Sets that may be used by the controller when all supported I/O
Command Sets (110b) are selected in CC.CSS. This Feature shall be implemented if the CAP.CSS.IOCSS
bit is set to ‘1’. When CC.CSS is set to any value other than 110b, then this Feature has no effect and the
I/O Command Sets that may be used by the controller are specified by CC.CSS. If CC.CSS is set to any
value other than 110b and the controller receives a Set Features command for this Feature, then this
command has no effect and returns a status code of Successful Completion.
When all supported I/O Command Sets (110b) is selected in CC.CSS, the value of this Feature specifies
the index of the I/O Command Set Combination in the Identify I/O Command Set data structure that is used.
Refer to section  5.1.13.2.19 for more information. The Index is specified in the I/O Command Set
Combination Index field of Command Dword 11 (refer to  Figure 410). If any namespace attached to the
controller uses an I/O Command Set that is not supported by the specified I/O Command Set combination,
then the controller shall abort the command with a status code of I/O Command Set Combination Rejected.
Upon successful completion of a Set Features command for this Feature, the controller transitions to using
the specified I/O Command Set Combination.
Figure 410: I/O Command Set Profile – Command Dword 11
Bits Description
31:09 Reserved
