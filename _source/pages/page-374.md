---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 374
headings: ["5.1.16.1.2 Command Completion"]
---

# Source page 374

NVM Express® Base Specification, Revision 2.1

352
Figure 345: Get Controller State Management Operation – Management Operation Specific
Bits Description
15:08 Reserved
7:0
Controller State Version Index (CSVI): A non-zero value in this field specifies the index to a specific entry
in the Controller State Version Supported list of the Supported Controller State Formats data structure
(refer to section 5.1.13.2.21). The contents of that entry specify the format of the Controller State field in
the returned data.
If this field is cleared to 0h, then no Controller State field is returned in the returned data.
The Get Controller State management operation of the Migration Receive command uses Command Dword
11 as defined in Figure 346.
Figure 346: Get Controller State Management Operation – Command Dword 11
Bits Description
31:24
Controller State UUID Index Parameter (CSUIDXP): This field is vendor specific.
If the Controller State UUID Index field is cleared to 0h, then the controller shall ignore this field.
23:16
Controller State UUID Index (CSUUIDI): A non-zero value in this field specifies the index to a specific entry
in the Vendor Specific Controller State UUID Supported list of the Supported Controller State Formats data
structure (refer to section 5.1.13.2.21). The contents of that entry specify the format of the Vendor Specific
field in the returned data.
If this field is cleared to 0h, then no Vendor Specific field is returned in the returned data.
15:00 Controller Identifier (CNTLID): This field specifies the identifier of the controller on which the management
operation is performed.
If the CSVI field (refer to Figure 346) specifies a non -zero index that is not defined by the Supported
Controller State Formats data structure (refer to Figure 329), then the controller shall abort the command
with a status code of Invalid Field in Command.
If the CSVI field is cleared to 0h, then the Controller State Size field in the returned data shall be cleared to
0h.
If the CSUUIDI field specifies a non-zero index that is not defined by the Supported Controller State Formats
data structure, then the controller shall abort the command with a status code of Invalid Field in Command.
If the CSUUIDI field is cleared to 0h, then the Vendor Specific Size field in the returned data shall be cleared
to 0h.
If a status code of Successful Completion is returned, then the completion queue entry Dword 0 contains
additional information about the command as defined in Figure 347.
Figure 347: Get Controller State Management Operation – Completion Queue Entry Dword 0
Bits Description
31:01 Reserved
00
Controller Suspended (CSUP): If this bit is set to ‘1’, then the controller associated with the returned
data structure was suspended for the entire duration of the processing of this command. If this bit is
cleared to ‘0’, then the controller associated with this data structure was not suspended for the entire
duration of the processing of this command.
5.1.16.1.2 Command Completion
The Upon completion of the Migration Receive command, the controller posts a completion queue entry to
the Admin Completion Queue indicating the status for the command. Section 5.1.16.1 describes completion
details for each management operation.
Migration Receive command specific status values (i.e., SCT field set to 1h) are shown in Figure 348.
