---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 468
headings: ["5.3.9.1.2 Delete (Management Operation 02h)"]
---

# Source page 468

NVM Express® Base Specification, Revision 2.1

446
Figure 531: Create Completion Queue Entry Dword 0 Data Structure
Bytes Description
01:00
Exported Port ID (EPID): If the Generate Exported Port ID bit is set to ‘1’ in the Create operation, then this
field specifies Exported Port ID to be used for this Exported Transport and associated with the Exported
NVM Subsystem identified by the Exported NVM Subsystem NVMe Qualified Name field in the Create data
structure. If the Generate Exported Port ID bit is cleared to ‘0’ in the Create operation, this field shall be set
to the Exported Port ID provided in the Create data structure.
03:02 Reserved
If the Exported NVM Subsystem NVMe Qualified Name field specified in the Create Data Structure does
not identify an existing Exported NVM Subsystem, then the controller shall abort the command with a status
code of Invalid Field in Command.
If the Generate Exported Port ID bit is cleared to ‘0’ in the Create data structure indicating an Exported Port
ID is specified in the Manage Exported Port data structure and used to identify the new Exported Port; and:
• an Exported Port ID is not specified in the Manage Exported Port data structure in the data buffer;
or
• an Exported Port ID specified in the data buffer does not uniquely identify an Exported NVM
Subsystem Port (i.e., if the specified Exported Port ID is used to identify an already existing
Exported NVM Subsystem Port),
then the controller shall abort the command with a status code of Invalid Field in Command.
5.3.9.1.2 Delete (Management Operation 02h)
The Delete operation of the Manage Exported Port command is used to remove an Exported Port from an
Exported NVM Subsystem and delete the Exported Port.
The Exported Port to be deleted is specified in the Exported Port ID (EPID) field in the Delete operation
data structure (refer to Figure 532) in the data buffer.
The Exported NVM Subsystem NQN to which this Exported Port ID is assigned is specified in the NVM
Subsystem NVMe Qualified Name (SUBNQN) field of the Delete operation data structure in the data buffer.
The Exported Port that is deleted by a Delete operation should not be in use (i.e., there should be no
association between any host and a controller through that Exported Port). If the Exported Port is in use,
then all associations between any host and a c ontroller through that port are terminated by the Delete
operation, and for any association that is terminated, the behavior of outstanding commands submitted via
that association and resources underlying that association (e.g., fabric connections) is unde fined.
Figure 532: Delete Data Structure
Bytes Description
255:00
NVM Subsystem NVMe Qualified Name (SUBNQN):  NVMe Qualified Name (NQN) that uniquely
identifies the Exported NVM subsystem which is associated with the Exported Port to be deleted in
command. Refer to section 4.7.
257:256 Exported Port ID (EPID): Specifies a particular NVM subsystem port to be deleted in this command.
319:258 Reserved
If:
• the Exported port ID field in the Delete operation data structure does not contain an Exported Port
ID associated with the Exported NVM Subsystem NQN (SUBNQN) specified in the Delete data
structure (refer to Figure 532); or
• the NVM Subsystem NVMe Qualified Name field in the Delete data structure (refer to Figure 532)
does not contain an NVM Subsystem NQN for an existing Exported NVM Subsystem that is
associated with the Exported Port ID specified in the Delete data structure,
then the controller shall abort the command with a status code of Invalid Field in Command.
