---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 486
headings: ["7.3 I/O Management Receive command", "7.3.1 I/O Management Receive Operations", "7.3.1.1 Reclaim Unit Handle Status (Management Operation 01h)"]
---

# Source page 486

NVM Express® Base Specification, Revision 2.1

464
7.3 I/O Management Receive command
The I/O Management Receive command is used to receive information from the controller used by the host
to manage I/O. The behavior of the command is dependent on the specified operation as defined in the
Management Operation field in Figure 561.
The command uses the Data Pointer, Command Dword 10, and Command Dword 11 fields. All other
command specific fields are reserved. If the command uses PRPs for the data transfer, then the PRP Entry
1 and PRP Entry 2 fields are used. If the command uses SGLs  for the data transfer, then the SGL Entry 1
field is used. All other command specific fields are reserved.
If the Number of Dwords (NUMD) field corresponds to a length that is less than the size of the data structure
to be returned, then only that specified portion of the data structure is transferred. If the NUMD field
corresponds to a length that is greater than the size of the associated data structure, then the entire contents
of the data structure are transferred and no additional data is transferred.
Figure 560: I/O Management Receive – Data Pointer
Bits Description
127:00 Data Pointer (DPTR):  This field specifies the location of a data buffer where data is transferred from.
Refer to Figure 92 for the definition of this field.

Figure 561: I/O Management Receive – Command Dword 10
Bits Description
31:16
Management Operation Specific (MOS): This definition of this field is specific the value of the MO field.
If this field is not defined for the management operation specified by the MO field, then this field is
reserved.
15:08 Reserved
07:00
Management Operation (MO): This field specifies the management operation to perform.
Value Definition
00h No action
01h Reclaim Unit Handle Status: For each Placement Handle of the namespace, the
controller shall return a Reclaim Unit Handle Status Descriptor for each Reclaim Group.
FFh Vendor specific
All others Reserved


Figure 562: I/O Management Receive – Command Dword 11
 Bits Description
31:00 Number of Dwords (NUMD): This field specifies the number of dwords to transfer. This is a 0’s based
value.
 I/O Management Receive Operations
 Reclaim Unit Handle Status (Management Operation 01h)
The Reclaim Unit Handle Status management operation is used to provide information about Reclaim Unit
Handles (refer to section 1.5.84) that are accessible by the specified namespace. The returned data
contains one or more Reclaim Unit Handle Status Descriptor data structures (refer to the applicable I/O
Command Set specification). The information contained in each Reclaim Unit Handle Status Descriptor:
• is the information at the time the controller processes that Reclaim Unit Handle Status Descriptor;
and
• may or may not contain the information reflecting any outstanding command that affects the
reported Reclaim Unit Handle Status Descriptor.
If Flexible Data Placement is disabled in the Endurance Group containing the specified namespace, then
the command shall be aborted with a status code of FDP Disabled.
