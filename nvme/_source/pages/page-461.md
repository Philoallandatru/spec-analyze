---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 461
headings: ["5.3.7.1.2 Disassociate Namespace (Management Operation 02h)"]
---

# Source page 461

NVM Express® Base Specification, Revision 2.1

439
attached to a controller due to the processing of a Namespace Attachment command (refer to section
5.1.20).
The data pointer shall point to an Associate Namespace data structure (refer to Figure 518).
Figure 518: Associate Namespace Data Structure
Bytes Description
31:00 Exported Namespace ID (ENSID) : A valid and unassociated Exported NSID to be associated to an
Underlying Namespace and associated with the specified Exported NVM Subsystem.
253:32 Exported NVM Subsystem NQN  (ENSNQN): Identifies an existing Exported NVM Subsystem to be
associated with the Exported Namespace ID.
285:254
Underlying Namespace ID  (UNSID): An active Namespace ID to which Exported Namespace ID
(ENSID) is to be associated. Refer to Figure 65 for the definition of NSID types and relationship to
namespaces.
287:286 Underlying Controller ID (UCTRLID): Identifies the Underlying Controller used to access the specified
Underlying Namespace.
543:288 Underlying NVM Subsystem NQN  (UNSNQN): Identifies the Underlying NVM Subsystem which
provides the Underlying Namespace to be associated with the Exported Namespace ID.
575:544 Reserved
The controller shall abort the command with a status code of Invalid Field in Command if:
• a valid Exported Namespace ID is not specified in the Exported Namespace ID field in Figure 518;
• the Exported NVM Subsystem NQN field in Figure 518 does not refer to an existing Exported NVM
Subsystem;
• the Underlying Namespace ID field in Figure 518 does not refer to an active Underlying Namespace
ID;
• the Underlying Controller ID field in Figure 518 does not identify a controller contained in the
Underlying NVM Subsystem specified by the Underlying NVM Subsystem NQN field;
• the Underlying Namespace specified by the Underlying Namespace ID field shown in Figure 518
is not attached to the controller specified by the Underlying Controller ID in the Underlying
Controller ID field in Figure 518;
• the specified Underlying NVM Subsystem NQN field in Figure 518 does not refer to an existing
Underlying NVM Subsystem; or
• the specified Underlying Namespace ID is not an allocated namespace in the Underlying NVM
Subsystem NQN field in Figure 518.
Upon successful completion of a Manage Exported Namespace command with an Associate Namespace
operation:
• the Exported NSID which is associated with an attached Underlying Namespace is an allocated
namespace to the Exported NVM Subsystem. The Exported Namespace and the Underlying
Namespace contain the same user data (e.g., format, read, and write operations on the Exported
Namespace affect user data stored in the Underlying Namespace; similarly, format, read and write
operations on the Underlying Namespace affect user data stored in the Exported Namespace); and
• a Namespace Attribute Changed asynchronous event is reported as described in Figure 151.
5.3.7.1.2 Disassociate Namespace (Management Operation 02h)
The Disassociate Namespace operation of the Manage Exported Namespace command is used to remove
an association between an Exported Namespace ID (ENSID) and an Exported NVM Subsystem and delete
the Exported Namespace ID.
The data pointer shall point to a Disassociate Namespace data structure (refer to Figure 519).
