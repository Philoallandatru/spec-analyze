---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 101
headings: ["3.2.3 Endurance Groups"]
---

# Source page 101

NVM Express® Base Specification, Revision 2.1

79
• the total capacity of the NVM Set; and
• the unallocated capacity for the NVM Set.
An NVM Set Identifier is a 16 -bit value that specifies the NVM Set with which an action is associated. An
NVM Set Identifier is unique with the NVM subsystem. An NVM Set Identifier may be specified in NVM Set
aware Admin commands (refer to Figure 68). An NVM Set Identifier value of 0h is reserved and is not a
valid NVM Set Identifier. Unless otherwise specified, if the host specifies an NVM Set Identifier cleared to
0h for a command that requires an NVM Set Identifier, then that command shall abort with a status code of
Invalid Field in Command.
Each NVM Set is associated with exactly one Endurance Group (refer to section  3.2.3).
The NVM Set with which a namespace  that contains formatted storage  is associated is reported in the
Identify Namespace data structure (refer to the applicable NVMe I/O Command Set specification). When a
host creates a namespace that contains formatted storage using the Namespace Management command,
the host specifies the NVM Set Identifier of the NVM Set that the namespace is to be created in. The
namespace that is created inherits attributes from the NVM Set (e.g., the optimal write size to the NVM).
If NVM Sets are supported, then all controllers in the NVM subsystem shall:
• Indicate support for NVM Sets in the Controller Attributes field in the Identify Controller data
structure;
• Support the NVM Set Identifier in all commands that use the NVM Set Identifier;
• Support the NVM Set List for the Identify command;
• Indicate the NVM Set Identifier with which any namespace that contains formatted storage is
associated in the Identify Namespace data structure for that namespace;
• Support Endurance Groups; and
• For each NVM Set, indicate the associated Endurance Group as an attribute.
If support for NVM Sets is not reported (i.e., the NVM Sets bit is cleared to ‘0’ in the CTRATT field; refer to
Figure 312), then the NVM Set Identifier field shall be cleared to 0h in all commands and data structures
that support an NVM Set Identifier field.
 Endurance Groups
Endurance may be managed within a single NVM Set (refer to section 3.2.2) or across a collection of NVM
Sets. Each NVM Set is associated with an Endurance Group (refer to Figure 317). If two or more NVM Sets
have the same Endurance Group Identifier, then endurance is managed by the NVM subsystem across
that collection of NVM Sets. If only one NVM Set is associated with a specific Endurance Group Identifier,
then endurance is managed locally to that NVM Set.
If NVM Sets are not supported, then endurance is managed by the NVM subsystem:
• within each Endurance Group if Endurance Groups are supported; or
• within the domain if Endurance Groups are not supported.
An Endurance Group shall be part of only one domain (refer to section 3.2.5).
An Endurance Group Identifier is a 16-bit value that specifies the Endurance Group with which an action is
associated. An Endurance Group Identifier is unique within the NVM subsystem. An Endurance Group
Identifier value of 0h is reserved and is not a valid Endurance Group Identifier. Unless otherwise specified,
if the host specifies an Endurance Group Identifier cleared to 0h for a command that requires an Endurance
Group Identifier, then that command shall abort with a status code of Invalid Field in Command.
The information that describes an Endurance Group is indicated in the Endurance Group Information log
page (refer to section 5.1.12.1.10).
Figure 69 shows Endurance Groups added to the example in Figure 67. In this example, the endurance of
NVM Set A and NVM Set B are managed together as part of Endurance Group Y, while the endurance of
NVM Set C is managed only within NVM Set C which is the only NVM Set that is part of Endurance Group
Z.
