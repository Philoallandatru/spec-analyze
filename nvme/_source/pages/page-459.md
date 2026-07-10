---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 459
headings: ["5.3.6 Fabric Zoning Send command"]
---

# Source page 459

NVM Express® Base Specification, Revision 2.1

437
Figure 510: Fabric Zoning Receive (FZR) – Completion Queue Entry Dword 0
Bits Description
31
Last Fragment (LF): This bit specifies if the transferred data buffer contains the last fragment of this Zoning
data structure. If this bit is set to ‘1’, then the transferred data buffer contains the last fragment. If this bit is
cleared to ‘0’, then the transferred data buffer does not contain the last fragment.
30:00 Reserved
 Fabric Zoning Send command
The Fabric Zoning Send (FZS) command is used to send a Zoning data structure. The FZS command uses
the Data Pointer, Command Dword 10, Command Dword 11, and Command Dword 12 fields, as shown in
Figure 511, Figure 512, Figure 513, and Figure 514 respectively.
Figure 511: Fabric Zoning Send (FZS) – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.

Figure 512: Fabric Zoning Send (FZS) – Command Dword 10
Bits Description
31:00
Zoning Data Key (ZDK):  If the FZS command is issued by a DDC, then this field specifies the key
identifying a Zoning data structure in the Zoning database of the CDC. If the FZS command is issued
by the CDC, then this field specifies the Transaction ID of the current Zoning operation.

Figure 513: Fabric Zoning Send (FZS) – Command Dword 11
Bits Description
31:00
Zoning Data Offset (ZDO): This field specifies the byte offset within a Zoning data structure to store the
transferred data.
The offset shall be dword aligned, indicated by bits 1:0 being cleared to 00b. The controller is not required
to check that bits 1:0 are cleared to 00b. The controller may return a status code of Invalid Field in
Command if bits 1:0 are not cleared to 00b. If the controller does not return a status code of Invalid Field
in Command, then the controller shall operate as if bits 1:0 are cleared to 00b.

Figure 514: Fabric Zoning Send (FZS) – Command Dword 12
Bits Description
31
Last Fragment (LF): This bit specifies if the transferred data buffer contains the last fragment of this
Zoning data structure. If this bit is set to ‘1’, then the transferred data buffer contains the last fragment . If
this bit is cleared to ‘0’, then the transferred data buffer does not contain the last fragment.
30:29 Reserved
28
ZDK Context (ZDKC): This bit specifies the content of the ZDK field. If this bit is set to ‘1’, then the ZDK
field contains a Transaction ID. If this bit is cleared to ‘0’, then the ZDK field contains a Zoning data key.
A CDC receiving a FZS command with this bit set to ‘1’ shall abort the command with a status code of
Invalid Field in Command. A DDC receiving a FZS command with this bit cleared to ‘0’ shall abort the
command with a status code of Invalid Field in Command.
27:00 Number of Dwords (NUMD): This field specifies the number of dwords to transfer. If this field is cleared
to 0h, then the controller shall abort the command with a status code of Invalid Field in Command.
