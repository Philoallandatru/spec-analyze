---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 448
headings: ["5.3 Message-Based Transport Admin Commands (Fabrics)", "5.3.1 Clear Exported NVM Resource Configuration command", "5.3.1.1 Command Completion", "5.3.2 Create Exported NVM Subsystem command"]
---

# Source page 448

NVM Express® Base Specification, Revision 2.1

426
Figure 491: Virtualization Management – Command Specific Status Values
Value Definition
20h Invalid Secondary Controller State:  The action requested for the secondary controller is invalid
based on the current state of the secondary controller and its primary controller.
21h
Invalid Number of Controller Resources:  The specified number of Flexible Resources is invalid
(e.g., the Number of Controller Resources (NR) is greater than VQ Resources Flexible Total
(VQFRT) (refer to Figure 330), the Number of Controller Resources (NR) is greater than VQ
Resources Flexible Secondary Maximum (VQFRSM) (refer to Figure 330)).
22h
Invalid Resource Identifier: At least one of the specified resource identifiers was invalid (e.g., the
Number of Controller Resources (NR) is greater than the number of remaining available flexible
resources).
Dword 0 of the completion queue entry contains information about the controller resources that were
modified as part of the Primary Controller Flexible Allocation and Secondary Controller Assign actions.
Dword 0 of the completion queue entry is defined in Figure 492.
Figure 492: Virtualization Management – Completion Queue Entry Dword 0
Bits Description
31:16 Reserved
15:00 Number of Controller Resources Modified (NRM): This field indicates the number of controller resources
that were allocated or assigned. The value may be smaller or larger than the number requested.
5.3 Message-Based Transport Admin Commands (Fabrics)
This section describes Admin commands that are specific to the Message-Based transport model.
 Clear Exported NVM Resource Configuration command
The Clear Exported NVM Resource Configuration command is used to delete all Exported NVM resource
configuration information including removing all Exported NVM Resources (refer to section 1.5.35 for a list
of Exported NVM Resources).
All command specific fields are reserved.
The Clear Exported NVM Resource Configuration command shall not be supported by  Exported NVM
Subsystems.
The Clear Exported NVM Resource Configuration command shall not affect Underlying Namespaces or
Underlying NVM Subsystems.
If a Clear Exported NVM Resource Configuration command is issued without first disconnecting all hosts
from all Exported NVM Subsystems, then the controller shall abort the command with a status code of
Command Sequence Error.
 Command Completion
Upon completion of the Clear Exported NVM Resource Configuration command, the controller shall post a
completion queue entry to the Admin Completion Queue indicating the status for the command. Refer to
section 8.3.3 for usages.
 Create Exported NVM Subsystem command
The Create Exported NVM Subsystem command is used to create a new Exported NVM Subsystem. The
Create Exported NVM Subsystem command uses the Data Pointer and Command Dword 10. All other
command specific fields are reserved.
The Create Exported NVM Subsystem command shall not be supported by Exported NVM Subsystems.
