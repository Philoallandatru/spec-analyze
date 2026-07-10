---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 207
headings: ["5.1.4.1 Controller Data Queue Management Operations", "5.1.4.1.1 Create Controller Data Queue (Management Operation 0h)"]
---

# Source page 207

NVM Express® Base Specification, Revision 2.1

185
Figure 161: Controller Data Queue – Command Dword 10
Bits Description
31:16
Management Operation Specific (MOS): The definition of this field is specific to a management operation
(refer to the Select field). If a management operation does not define the use of this field, then this field is
reserved.
15:08 Reserved
07:00
Select (SEL): This field specifies the type of management operation to perform.
Value Management Operation Reference Section
0h Create Controller Data Queue 5.1.4.1.1
1h Delete Controller Data Queue 5.1.4.1.2
2h to BFh Reserved
C0h to FFh Vendor Specific

 Controller Data Queue Management Operations
5.1.4.1.1 Create Controller Data Queue (Management Operation 0h)
The Create Controller Data Queue management operation of the Controller Data Queue command is used
to create a queue in host memory to which a controller posts the information specified by the Queue Type
field (refer to Figure 165).
The Create Controller Data Queue management operation uses the PRP Entry 1  field, Command Dword
11 field, and Command Dword 12 field.
If a PRP List is provided to describe the Controller Data Queue, then the PRP List shall be maintained by
the host at the same location in host physical memory and the values in the PRP List shall not be modified
until the Controller Data Queue is deleted (refer to section 5.1.4.1.2) or the controller is reset. If the PRP
List values are modified, then the behavior is undefined.
If the number of memory ranges specified by the PRP Entry 1 field is greater than the value defined by the
Maximum CDQ Memory Ranges (MCMR) field in the Identify Controller data structure (refer to Figure 312),
then the controller shall abort the command with a status code of Invalid Field in Command.
If the number of memory ranges specified by the PRP Entry 1 field plus the number of memory ranges that
exists for all of the Controller Data Queues in the NVM subsystem is greater than the value defined by the
NVM Subsystem Maximum Controller CDQ Memory Ranges (NMCMR) field in the Identify Controller data
structure (refer to Figure 312), then the controller shall abort the command with a status code of Invalid
Field in Command.
If the current number of User Data Migration Queues that exist in the controller is equal to the value in the
Maximum Controller User Data Migration Queues (MCUDMQ) field in the Identify Controller data structure,
then the controller shall abort the command with a status code of Invalid Field in Command.
If the current number of User Data Migration Queues that exist in the NVM subsystem is equal to the value
in the Maximum NVM Subsystem User Data Migration Queues (MNSUDMQ) field in the Identify Controller
data structure, then the controller shall abort the command with a status code of Invalid Field in Command.
