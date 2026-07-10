---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 375
headings: ["5.1.17 Migration Send command", "5.1.17.1 Migration Send Management Operations", "5.1.17.1.1 Suspend (Management Operation 0h)"]
---

# Source page 375

NVM Express® Base Specification, Revision 2.1

353
Figure 348: Migration Receive – Command Specific Status Values
Value Definition
1Fh Invalid Controller Identifier: The specified controller is not in a condition to set the controller state.
 Migration Send command
The Migration Send command is used to manage the migration of a controller (refer to section 8.1.12).
The Migration Send command uses the Command Dword 10 and Command Dword 14. The use of the
Data Pointer field , Command Dword 11  field, Command Dword 12  field, Command Dword 13  field, and
Command Dword 15 field is specific to the management operation specified by the Select field . All other
command specific fields are reserved.
The Select field defined in Figure 349 specifies the management operation to be performed. Refer to section
5.1.17.1 for a description of each management operation.
Figure 349: Migration Send – Command Dword 10
Bits Description
31:16
Management Operation Specific (MOS): The definition of this field is specific to a management operation
(refer to the Select field). If a management operation does not define the use of this field, then this field is
reserved.
15:08 Reserved
07:00
Select (SEL): This field specifies the type of management operation to perform.
Value M/O1 Management Operation Reference Section
0h M Suspend 5.1.17.1.1
1h M Resume 5.1.17.1.2
2h M Set Controller State 5.1.17.1.3
3h to FFh  Reserved
Notes:
1. O/M definition: O = Optional, M = Mandatory

If the controller supports selection of a UUID by the Migration Send command (refer to Figure 210 and
section 8.1.28), then Command Dword 14 is used to specify a UUID Index value (refer to Figure 350).
Figure 350: Migration Send – Command Dword 14
Bits Description
31:07 Reserved
06:00 UUID Index (UIDX): Refer to Figure 658.
 Migration Send Management Operations
5.1.17.1.1 Suspend (Management Operation 0h)
The Suspend management operation of the Migration Send command allows a host to specify the type of
suspend to be done on the specified controller.
