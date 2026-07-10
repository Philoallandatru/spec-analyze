---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 458
headings: ["5.3.5.1 Command Completion"]
---

# Source page 458

NVM Express® Base Specification, Revision 2.1

436
Figure 505: Fabric Zoning Receive (FZR) – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.

Figure 506: Fabric Zoning Receive (FZR) – Command Dword 10
Bits Description
31:00
Zoning Data Key (ZDK):  If the FZR command is issued by a DDC, then this field specifies the key
identifying a Zoning data structure in the Zoning database of the CDC. If the FZR command is issued by
the CDC, then this field specifies the Transaction ID of the current Zoning operation

Figure 507: Fabric Zoning Receive (FZR) – Command Dword 11
Bits Description
31:00
Zoning Data Offset (ZDO): This field specifies the byte offset within a Zoning data structure to store the
transferred data.
The offset shall be dword aligned, indicated by bits 1:0 being cleared to 00b. The controller is not required
to check that bits 1:0 are cleared to 00b. The controller may return a status code of Invalid Field in
Command if bits 1:0 are not cleared to 00b. If the controller does no t return a status code of Invalid Field
in Command, then the controller shall operate as if bits 1:0 are cleared to 00b.
If an offset greater than the size of the requested Zoning data structure is specified, then the controller
shall abort the command with a status code of Invalid Field in Command.

Figure 508: Fabric Zoning Receive (FZR) – Command Dword 12
Bits Description
31:29 Reserved
28
ZDK Context (ZDKC): This bit specifies the content of the ZDK field. If this bit is set to ‘1’, then the ZDK
field contains a Transaction ID. If this bit is cleared to ‘0’, then the ZDK field contains a Zoning data key.
A CDC receiving a FZR command with this bit set to ‘1’ shall abort the command with a status code of
Invalid Field in Co mmand. A DDC receiving a FZR command with this bit cleared to ‘0’ shall abort the
command with a status code of Invalid Field in Command.
27:00 Number of Dwords (NUMD): This field specifies the number of dwords to transfer. If this field is cleared
to 0h, then the controller shall abort the command with a status code of Invalid Field in Command.
 Command Completion
Upon completion of the Fabric Zoning Receive (FZR) command, the controller posts a completion queue
entry to the Admin Completion Queue indicating the status of the command. Command specific status
values for the FZR command are defined in Figure 509.
Figure 509: Fabric Zoning Receive (FZR) – Command Specific Status Values
Value Definition
30h Zoning Data Structure Locked: The requested Zoning data structure is locked on the CDC.
31h Zoning Data Structure Not Found: The requested Zoning data structure does not exist on the CDC.
33h Requested Function Disabled: Fabric Zoning is not enabled on the CDC.
34h ZoneGroup Originator Invalid: The DDC is not allowed to access the specified ZoneGroup.
The last fragment indication is returned in Dword 0 of the completion queue entry, as defined in Figure 510.
