---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 412
headings: ["5.1.25.1.20 Flexible Data Placement (Feature Identifier 1Dh)"]
---

# Source page 412

NVM Express® Base Specification, Revision 2.1

390
If a Set Features command is processed while the controller is in the FQ Processing state, then that
command is aborted as described in section 8.2.5.2.
Figure 414: Power Loss Signaling Config – Command Dword 11
Bits Description
31:02 Reserved
01:00
Power Loss Signaling Mode (PLSM): Specifies the Power Loss Signaling mode of operation.
Value Definition
00b Power Loss Signaling not enabled
01b Power Loss Signaling with Emergency Power Fail enabled
10b Power Loss Signaling with Forced Quiescence enabled
11b Reserved

5.1.25.1.20 Flexible Data Placement (Feature Identifier 1Dh)
This Feature controls operation of the Flexible Data Placement capability (refer to section 8.1.10) in the
specified Endurance Group (refer to Figure 415).
The attribute is specified in Command Dword 12. Effects of enabling and disabling the Flexible Data
Placement are described in section 8.1.10.
If a Get Features command specifying this Feature is successfully completed, then the attributes described
in Figure 416 are returned in Dword 0 of the completion queue entry for that command.
Figure 415: Flexible Data Placement – Command Dword 11
Bits Description
31:16 Reserved
15:00 Endurance Group Identifier (ENDGID): If Endurance Groups are supported, then this field specifies
the identifier for the Endurance Group (refer to section 3.2.3) used for this Feature.

Figure 416: Flexible Data Placement – Command Dword 12
Bits Description
31:16 Reserved
15:08
Flexible Data Placement Configuration Index (FDPCIDX): This field specifies the index into the FDP
Configuration Descriptor List (refer to Figure 279) identifying a valid FDP configuration to be applied to
the Endurance Group when Flexible Data Placement is enabled (refer to section 8.1.10).
07:01 Reserved
00
Flexible Data Placement Enable (FDPE):  This bit specifies if the Flexible Data Placement (i.e.,
capability) is enabled or disabled. If this bit is set to ‘1’, then Flexible Data Placement is enabled. If this
bit is cleared to ‘0’, then Flexible Data Placement is disabled.
This Feature shall be saveable (refer to section 4.4). The default value of this Feature shall be 0h.
The value of this Feature is only allowed to change if the SV bit is set to ‘1’ in the Set Features command.
Therefore, if the Save and Select Feature Support (SSFS) bit is set to ‘1’ in the ONCS field in the Identify
Controller data structure (refer to Figure 312) and the SV bit is cleared to ’0’, then the controller shall abort
the command with a status code of Invalid Field in Command.
If the value of this Feature changes, then the controller is allowed to modify any fields associated with
specifying any User Data Format information accessible using the Identify command (i.e., any field
associated with the Format Index (refer to section 1.6.2) that includes CNS values 00h, 05h, 08h, 09h,
0Ah). The host should issue Identify commands in this condition to access the values that may have been
modified.
