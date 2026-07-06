---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 223
headings: ["5.1.11.2 Command Completion", "5.1.12 Get Log Page command"]
---

# Source page 223

NVM Express® Base Specification, Revision 2.1

201
A Select field set to 011b (i.e., supported capabilities) returns the capabilities supported for this Feature
Identifier. The capabilities supported are returned in Dword 0 of the completion  queue entry of the Get
Features command (refer to Figure 195).
 Command Completion
Upon completion of the Get Features command, the controller posts a completion queue entry to the Admin
Completion Queue. If the Select field is not set to 011b, then Dword 0 of the completion queue entry may
contain feature-dependent information (refer to section 5.1.25).
If the Select field is set to 011b, then Figure 195 describes the contents of Dword 0 of the completion queue
entry.
Figure 195: Completion Queue Entry Dword 0 when Select is set to 11b
Bits Description
31:3 Reserved
2 Changeable (CHANG): If this bit is set to ‘1’, then the feature values are changeable. If this bit is cleared to
‘0’, then the feature values are not changeable. Changing a feature value is described in section 5.1.25.
1 NS Specific (NSSPEC): If this bit is set to ‘1’, then the Feature Identifier has a namespace scope. If this bit
is cleared to ‘0’, then the Feature Identifier has a scope as defined in Figure 385.
0 Saveable (SVBL): If this bit is set to ‘1’, then the feature values are saveable. If this bit is cleared to ‘0’, then
the feature values are not saveable.
If the controller supports any changeable value of any attribute of a feature, then the controller reports that
feature as changeable (i.e., the controller sets the Changeable bit to '1' in Completion Queue Entry Dword
0 for a Get Features command that specifies that featur e and has the Select field set to 011b), even if the
feature has been set to a value or values that are not changeable.
For some features (e.g., Namespace Write Protection Config (refer to section 5.1.25.1.31), Boot Partition
Write Protection Config  (refer to section 5.1.25.1.32)), the changeability of feature values is value -
dependent (e.g., the Permanent Write Protect (i.e., 011b) value of the Write Protection State in the
Namespace Write Protection Config feature is not changeable).
 Get Log Page command
The Get Log Page command returns a data buffer containing the log page requested.  The Get Log Page
command may be impacted by the ANA state (refer to section 8.1.1.10).
The Get Log Page command uses the Data Pointer, Command Dword 10, Command Dword 11, Command
Dword 12, Command Dword 13 , and Command Dword 14  fields. All other command specific fields are
reserved.
There are mandatory and optional Log  Page Identifiers defined in section 3.1.3.5. If a Get Log Page
command is processed that specifies a Log Page Identifier that is not supported, then the controller shall
abort the command with a status code of Invalid Log Page with the exception defined in Figure 305.
The controller indicates support for the Log Page Offset and extended Number of Dwords (32 bits rather
than 12 bits) in the Log Page Attributes field of the Identify Controller data structure. If extended data is not
supported, then bits 27:16 of the Number of Dwords Lower field specify the Number of Dwords to transfer.
If the Log Page Offset is supported, then
• a byte offset shall be supported (i.e., Offset Type field cleared to ‘0’) for all log pages; and
• for each log page that has the IOS bit set to ‘1’ in the LID Supported and Effects data structure
(refer to Figure 204) for the specified LID in the Supported Log Pages log page an index offset shall
be supported (i.e., Offset Type field set to ‘1’).
If the IOS bit is cleared to ‘0’ in the LID Supported and Effects data structure for the specified LID in the
Supported Log Pages log page and a Get Log Page command specifies Offset  Type field set to ‘1’, then
that command shall be aborted with a status code of Invalid Field in Command.
