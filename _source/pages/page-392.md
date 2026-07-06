---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 392
headings: ["5.1.24.1 Command Completion", "5.1.25 Set Features command"]
---

# Source page 392

NVM Express® Base Specification, Revision 2.1

370
protocol commands submitted by a Security Send command are retrieved with the Security Receive
command defined in section 5.1.23.
The association between a Security Send command and subsequent Security Receive command is
Security Protocol field dependent as defined in SPC-5.
The fields used are Data Pointer, Command Dword 10, and Command Dword 11 fields. All other command
specific fields are reserved.
Figure 379: Security Send – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.

Figure 380: Security Send – Command Dword 10
Bits Description
31:24
Security Protocol (SECP): This field specifies the security protocol as defined in SPC-5. The controller
shall abort the command with a status code of Invalid Field in Command  if a reserved value of the
Security Protocol is specified.
23:16 SP Specific 1 (SPSP1): The value of this field contains bits 15:08 of the Security Protocol Specific field
as defined in SPC-5.
15:08 SP Specific 0 (SPSP0): The value of this field contains bits 07:00 of the Security Protocol Specific field
as defined in SPC-5.
07:00 NVMe Security Specific Field (NSSF): Refer to Figure 378 for definition of this field for Security Protocol
EAh. For all other Security Protocols this field is reserved.

Figure 381: Security Send – Command Dword 11
Bits Description
31:00 Transfer Length (TL): The value of this field is specific to the Security Protocol Out command with the
INC_512 field cleared to 0h as defined in SPC-5.
 Command Completion
If the command is completed, then the controller shall post a completion queue entry to the Admin
Completion Queue indicating the status for the command.
 Set Features command
The Set Features command specifies the attributes of the Feature indicated.
The Set Features command uses the Data Pointer, Command Dword 10, and Command Dword 14. The
use of Command Dword 11, Command Dword 12, Command Dword 13, and Command Dword 15 fields is
Feature specific. If Command Dword 11, Command Dword 12, Command Dword 13, or Command Dword
15 fields are not used, then the Command Dwords are reserved.
Figure 382: Set Features – Data Pointer
Bits Description
127:00
Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field. If using PRPs, this field shall not be a pointer to a PRP List as the data buffer may not cross
more than one page boundary. If no data structure is used as part of the specified feature, then this field
is not used.
