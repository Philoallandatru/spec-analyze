---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 206
headings: ["5.1.3.4 Command Completion", "5.1.4 Controller Data Queue command"]
---

# Source page 206

NVM Express® Base Specification, Revision 2.1

184
 Command Completion
Upon completion of the Capacity Management command, the controller posts a completion queue entry to
the Admin Completion Queue. Capacity Management command specific status values are defined in Figure
159.
Figure 159: Capacity Management – Command Specific Status Values
Value Description
26h
Insufficient Capacity: The requested operation requires more free space than is currently available. The
Command Specific Information field in the Error Information Log Entry data structure (refer to Figure 205)
specifies the total amount of NVM capacity in bytes required to create the Endurance Group or NVM Set.
2Dh Identifier Unavailable: The number of Endurance Groups or NVM Sets supported has been exceeded.
Dword 0 of the completion queue entry contains the identifier of the Endurance Group or NVM Set created,
if any. Completion queue entry Dword 0 is defined in Figure 160.
Figure 160: Capacity Management – Completion Queue Entry Dword 0
Bits Description
31:16 Reserved
15:00
Created Element Identifier (CELID): This field indicates the identifier of the NVM Set that was created
if the Create NVM Set operation was used.
This field indicates the identifier of the Endurance Group Identifier if the Create Endurance Group
operation was used.
This field is reserved for all other operations.
 Controller Data Queue command
The Controller Data Queue command is used to manage queues in host memory that a controller posts a
specific type of data (refer to section 8.1.6).
The Controller Data Queue command uses the Command Dword 10  field. The use of the PRP1 field,
Command Dword 11 field, and Command Dword 12 field is specific to the management operation specified
by the Select field. All other command specific fields are reserved.
The Select field defined in Figure 161 determines which management operation is to be performed by this
command and refer to section 5.1.4.1 for a description of each management operation.
