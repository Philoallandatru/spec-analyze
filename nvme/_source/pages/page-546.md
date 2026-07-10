---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 546
headings: ["8.1.12.1 Process for Migrating a Controller"]
---

# Source page 546

NVM Express® Base Specification, Revision 2.1

524
A controller indicates support for tracking changes to host memory (refer to section  1.5.46) due to the
processing of commands by a controller by setting the Track Host Memory Changes Support (THMCS) bit
to ‘1’ in the TRATTR field.
 Process for Migrating a Controller
The controller This section provides an example sequence of steps to allow the MMH S to migrate a
Migratable Controller in the Source NVM Subsystem and the attached namespaces to a Migratable
Controller in the Destination NVM Subsystem (refer to Figure 619). Additionally, the steps include obtaining
the host memory modifications made to the host due to the Migratable Controller in the Source NVM
Subsystem processing commands during the migration.
In this example the MMC in the Source NVM Subsystem has:
• the Track User Data Changes Support (TUDCS) bit set to ‘1’ in the Tracking Attributes (TRATTR)
field in the Identify Controller data structure (refer to Figure 312); and
• the Track Host Memory Changes Support (THMCS) bit set to ‘1’ in the Tracking Attributes
(TRATTR) field in the Identify Controller data structure.
If attached namespaces are not required to be migrated by the MMH S, then those steps may be ignored.
If host memory modifications are not required to be obtained by the MMH S, then those steps may be
ignored.
1. The MMH S copies (i.e., migrates) the user data in namespaces attached to the Migratable
Controller in the Source NVM Subsystem while the host  is submitting commands and the
Migratable Controller is processing those submitted commands.
a. The MMHS creates a User Data Migration Queue in the MMC in the Source NVM Subsystem
by submitting a Controller Data Queue command to th at MMC (refer to section 5.1.4)
specifying:
• the Select field set to the Create Queue management operation (i.e., 1h);
• the Queue Type field set to User Data Migration Queue; and
• the Controller Identifier field (refer to Figure 166) set to the controller identifier for the
Migratable Controller in the Source NVM Subsystem.
b. If the Controller Data Queue command is successful, the completion queue entry contains the
Controller Data Queue Identifier for the created User Data Migration Queue. The MMH S
causes the MMC in the Source NVM Subsystem to post user data modifications to namespaces
attached to the Migratable Controller by submitting a Track Send command to th at MMC
specifying:
• the Select field set to Log User Data Changes management operation (i.e., 0h);
• the Logging Action bit set to ‘1’ (i.e., start logging); and
• the Controller Data Queue Identifier field set to the Controller Data Queue Identifier from
the completion queue entry for the Controller Data Queue command.
It is the responsibility of the MMHS to make sure that the User Data Migration Queue is sized
properly and there may be more restrictions on that MMHS as specified by the appropriate I/O
command set (e.g., the NVM Command Set requires the MMH S to not allow the User Data
Migration Queue to become full). Refer to section 8.1.6 for a description of the User Data
Migration Queue (i.e., a Controller Data Queue).
c. If the Track Send command is successful, then the MMH S starts copying the namespaces
attached to the Migratable Controller in the Source NVM Subsystem to the Destination NVM
Subsystem. Refer to the applicable I/O Command Set specifications for capabilities that may
reduce the amount of user date that is required to copied (e.g., the Get LBA Status command
in the NVM Command Set Specification).
2. The MMHS causes the MMC in the Source NVM Subsystem to start tracking the modifications to
host memory in the host due to the Migratable Controller processing submitted commands.
