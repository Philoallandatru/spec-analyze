---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 372
headings: ["5.1.16 Migration Receive command"]
---

# Source page 372

NVM Express® Base Specification, Revision 2.1

350
Lockdown command specific status values are defined in Figure 339.
Figure 339: Lockdown – Command Specific Status Values
Value Description
28h Prohibition of Command Execution Not Supported: The command was aborted due to the specified
opcode or Feature Identifier not supporting being prohibited from execution by the command.
 Migration Receive command
The Migration Receive command is used to obtain information from the controller processing the command
that the host may use to manage a migratable controller (refer to section 8.1.12).
The Migration Receive command uses the Data Pointer field, Command Dword 10 field, Command Dword
12 field, Command Dword 13  field, Command Dword 14  field, and Command Dword 15  field. The use of
the Command Dword 11 field is specific to the management operation specified by the Select field. All other
command specific fields are reserved.
The Select field defined in Figure 340 specifies the management operation to be performed. Refer to section
5.1.16.1for a description of each management operation.
Figure 340: Migration Receive – Command Dword 10
Bits Description
31:16
Management Operation Specific (MOS): The definition of this field is specific to a management operation
(refer to the Select field). If a management operation does not define the use of this field, then this field is
reserved.
15:08 Reserved
07:00
Select (SEL): This field specifies the type of management operation to perform.
Value M/O1 Management Operation Reference Section
0h M Get Controller State 5.1.16.1.1
1h to FFh  Reserved
Notes:
1. O/M definition: O = Optional, M = Mandatory


Figure 341: Migration Receive – Command Dword 12
Bits Description
31:00
Offset Lower (OL): This field specifies the least -significant 32-bits of the offset , in bytes,  within the data
available to be returned and specifies the starting point for that data for what is actually returned to the host.
The offset is required to be dword aligned and if bits 1:0 are not cleared to 00b , then the controller shall
abort the command with a status code of Invalid Field in Command.
If the host specifies an offset (i.e., OL field and OU field) that is greater than the size of the returned data
requested (e.g., the returned data contains 100 bytes is requested starting at offset 200), then the controller
shall abort the command with a status of Invalid Field in Command.

Figure 342: Migration Receive – Command Dword 13
Bits Description
31:00
Offset Upper (OU): This field specifies the most -significant 32-bits of the offset , in bytes, within the data
available to be returned and specifies the starting point for that data for what is actually returned to the host.
Refer to the Offset Lower field definition in Figure 340.
If the controller supports selection of a UUID by the Migration Receive command (refer to Figure 210 and
section 8.1.28), then Command Dword 14 is used to specify a UUID Index value (refer to Figure 343).
