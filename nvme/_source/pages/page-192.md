---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 192
headings: []
---

# Source page 192

NVM Express® Base Specification, Revision 2.1

170
Figure 141: Opcodes for Admin Commands
Opcode by Field
Combined
Opcode 1
Namespace
Identifier
Used 2
Command Reference 8
(07:02)  (01:00)
Function
Data
Transfer3
0010 11b 01b 2Dh No Manage Exported NVM Subsystem10 5.3.8
0011 00b 01b 31h Yes Manage Exported Namespace10 5.3.7
0011 01b 01b 35h No Manage Exported Port10 5.3.9
0011 10b 01b 39h No Send Discovery Log Page 5.3.10
0011 11b 01b 3Dh No Track Send 5.1.27
0011 11b 10b 3Eh No Track Receive 5.1.26
0100 00b 01b 41h No Migration Send 5.1.17
0100 00b 10b 42h No Migration Receive 5.1.16
0100 01b 01b 45h No Controller Data Queue 5.1.4
0111 11b 00b 7Ch No Doorbell Buffer Config 5.2.5
0111 11b 11b9 7Fh No Fabrics Commands9 6
1000 00b 00b 80h Yes Format NVM 5.1.10
1000 00b 01b 81h NOTE 7 Security Send 5.1.24
1000 00b 10b 82h NOTE 7 Security Receive 5.1.23
1000 01b 00b 84h No Sanitize 5.1.22
1000 01b 01b 85h Yes Load Program CP
1000 01b 10b 86h Yes4 Get LBA Status NVM, ZNS
1000 10b 00b 88h Yes Program Activation Management CP
1000 10b 01b 89h  Yes Memory Range Set Management CP
Vendor Specific
11xx xxb NOTE 3 C0h to
FFh  Vendor specific
Notes:
1. Opcodes not listed are reserved.
2. A subset of commands use the Namespace Identifier (NSID) field. If the Namespace Identifier field is used, then the
value FFFFFFFFh is supported in this field unless otherwise indicated in footnotes in this figure that a specific command
does not support that value or supports that value only under specific conditions. When this field is not used, the field
is cleared to 0h as described in Figure 92.
3. 00b = no data transfer; 01b = host to controller; 10b = controller to host; 11b = bidirectional. Refer to the Data Transfer
Direction field in Figure 91.
4. This command does not support the use of the Namespace Identifier (NSID) field set to FFFFFFFFh.
5. Support for the Namespace Identifier field set to FFFFFFFFh depends on the Directive Operation (refer to section
8.1.8).
6. Use of the Namespace Identifier field depends on the CNS value in the Identify Command as described in Figure 310.
7. The use of the Namespace Identifier is Security Protocol specific.
8. Section 5.1 contains commands common to all transport models, section 5.2 contains commands specific to the
Memory-based transport model, and section 5.3 contains commands specific to the Message-based transport model.
NVM = NVM Command Set specific, ZNS = Zoned Namespace Command Set specific, CP = Computational Programs
Command Set specific.
9. All Fabrics commands use the opcode 7Fh with the data transfer direction specified as shown in Figure 540. Refer to
section 6 for details.
10. Support for this command is prohibited in NVM subsystems that use a Memory-Based Transport Model (e.g., the PCIe
transport) for any controller.
11. Use of the Namespace Identifier field is specified further in section 5.1.12.1 and Figure 202.
Figure 142 lists the Admin commands that are allowed while a sanitize operation is in progress and the
Admin commands that should be allowed during the processing of a Format NVM command.
