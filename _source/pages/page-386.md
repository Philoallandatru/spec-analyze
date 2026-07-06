---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 386
headings: ["5.1.21.1 Command Completion"]
---

# Source page 386

NVM Express® Base Specification, Revision 2.1

364
Figure 367: Namespace Management – Command Dword 10
Bits Description
31:04 Reserved
03:00
Select (SEL): This field selects the type of management operation to perform.
Value Definition
0h Create
1h Delete
2h to Fh Reserved


Figure 368: Namespace Management – Command Dword 11
Bits Description
31:24
Command Set Identifier (CSI):  For a create operation (i.e., SEL 0h), this field specifies the I/O
Command Set for the created namespace. A CSI value of 0h creates a namespace using the NVM
Command Set. For all other operations this field is reserved.
Values for this field are defined by Figure 311.
23:0 Reserved

Figure 369: Namespace Management – Data Structure for Create
Bytes Description
511:00 Specific to the I/O Command Set (SIOCS): Refer to the Namespace Management command
section of the applicable I/O Command Set specification.
1023:512 Reserved
4095:1024 Vendor Specific (VS)
 Command Completion
When the command is completed, the controller posts a completion queue entry to the Admin Completion
Queue indicating the status for the command.
Namespace Management command specific status values  (i.e., SCT field set to 1h)  are shown in Figure
370.
Figure 370: Namespace Management – Command Specific Status Values
Value Definition
0Ah
Invalid Format:  The User Data Format specified is not supported.  This may be due to various
conditions, including:
1) specifying an invalid User Data Format number;
2) enabling protection information when there are not sufficient resources (e.g., metadata per
LBA); or
3) the specified format is not available in the current configuration.
15h
Namespace Insufficient Capacity: Creating the namespace requires more unallocated capacity than
is currently available. The Command Specific Information field in the Error Information Log Entry data
structure (refer to Figure 205) specifies the total amount of unallocated NVM capacity required to create
the namespace in bytes.
16h Namespace Identifier Unavailable: The number of namespaces supported has been exceeded.
1Bh Thin Provisioning Not Supported: Thin provisioning is not supported by the controller.
24h
ANA Group Identifier Invalid:  The specified ANA Group Identifier (ANAGRPID) is not supported in
the submitted command. This may be due to various conditions, including:
a) specifying an ANAGRPID that does not exist;
b) the controller does not allow an ANAGRPID to be specified (i.e., the ANA Group ID Support
(ANAGIDS) bit in the ANACAP field is cleared to ‘0’); or
