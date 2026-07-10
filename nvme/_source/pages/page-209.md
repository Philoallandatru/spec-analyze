---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 209
headings: ["5.1.4.1.1.1 User Data Migration Queue (Queue Type 0h)", "5.1.4.1.2 Delete Controller Data Queue (Management Operation 1h)", "5.1.4.2 Command Completion"]
---

# Source page 209

NVM Express® Base Specification, Revision 2.1

187
 User Data Migration Queue (Queue Type 0h)
The User Data Migration Queue of the Controller Data Queue command is used to create a queue for
logging of user data modified during the migration of the controller specified by the Controller Identifier field
as defined by Figure 166.
If a User Data Migration Queue is associated with the same controller specified by the Controller Identifier
field (refer to Figure 166), then the controller shall abort the command with a status code of Invalid Field in
Command.
If the number of User Data Migration Queues that exist in the controller is equal to the value defined in the
Maximum Controller User Data Migration Queues (MCUDMQ) field in the Identify Controller data structure
(refer to Figure 312), then the controller shall abort this command with a status code of Not Enough
Resources.
If the number of User Data Migration Queues that exist in the NVM subsystem is equal to the value defined
in the Maximum NVM Subsystem User Data Migration Queues (MNSUDMQ) field in the Identify Controller
data structure (refer to Figure 312), then the controller shall abort this command with a status code of Not
Enough Resources.
Figure 166: User Data Migration Queue – Create Queue Specific
Bits Description
15:00 Controller Identifier (CNTLID): This field specifies the controller associated with the information contained
in the User Data Migration Queue.
Refer to the applicable I/O Command Set specification for the requirements and formats of the entries for
the User Data Migration Queue.
5.1.4.1.2 Delete Controller Data Queue (Management Operation 1h)
The Delete Controller Data Queue management operation of the Controller Data Queue command is used
to delete a CDQ.
The Delete Controller Data Queue management operation uses the Command Dword 11 field.
Figure 167: Delete Controller Data Queue – Command Dword 11
Bits Description
31:16 Reserved
15:00 Controller Data Queue Identifier (CDQID):  This field specifies the identifier (refer to Figure 168) of the
existing User Data Migration Queue to delete.
If the value in the Controller Data Queue Identifier field specifies a CDQ that does not exist in the controller
processing the command, then the controller shall abort the command with a status code of Invalid
Controller Data Queue.
 Command Completion
Upon completion of the Controller Data Queue command, the controller posts a completion queue entry to
the Admin Completion Queue indicating the status for the command. Section 5.1.4.1 describes completion
details for each management operation.
If a status code of Successful Completion is returned  for the Create Controller Data Queue management
operation, then Dword 0 of the completion queue entry contains the identifier of the Controller Data Queue
created as defined in Figure 168. The indicated identifier is unique to the controller that processed the
command.
