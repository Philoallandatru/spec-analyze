---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 547
headings: []
---

# Source page 547

NVM Express® Base Specification, Revision 2.1

525
a. The MMHS submits a Track Send command to the MMC in the Source NVM Subsystem (refer
to section 5.1.27) specifying:
• the Select field set to the Track Memory Changes management operation (i.e., 0h);
• the Tracking Action (TACT) bit set to ‘1’;
• the Controller Identifier field specifying the identifier for the Migratable Controller in the
Source NVM Subsystem being migrated; and
• a Track Memory Changes data structure specifying the host memory to be tracked as
described by the set of host memory ranges.
b. If the Track Send command is successful, then the MMH S may query the host memory
modifications that have resulted from the Migratable Controller processing commands by
submitting a Track Receive command to the MMC in the Source NVM Subsystem specifying:
• the Select field set to the Track Memory Changes management operation (i.e., 0h); and
• the Controller Identifier field specifying the identifier for the Migratable Controller in the
Source NVM Subsystem being migrated.
3. Once the MMHS has copied the namespaces attached to the Migratable Controller  in the Source
NVM Subsystem, that MMHS determines a time to suspend that Migratable Controller. During this
period, that MMH S is migrating the user data in the namespaces attached to th at Migratable
Controller that are identified in the posted entries in the User Data Migration Queue. That MMH S
may handle the modified host memory in the host as reported by the Track Receive command.
4. The MMHS suspends the Migratable Controller in the Source NVM Subsystem by submitting a
Migration Send command to the MMC in the Source NVM Subsystem specifying:
• the Select field set to the Suspend management operation (i.e., 0h);
• the Suspend Type field set to Suspend;
• Delete User Data Migration Queue bit set to ‘1’, if the MMHS desires that the MMC delete
the User Data Migration Queue being used to log user data changes of the Migratable
Controller being migrated; and
• the Controller Identifier field (refer to Figure 166) set to the controller identifier for the
Migratable Controller to be suspended (i.e., the Migratable Controller being migrated).
Prior to completing that Migration Send command:
• the specified Migratable Controller being suspended:
o stops fetching commands (i.e.; from the Admin queue and I/O queues); and
o completes all previously fetched commands;
and
• the MMC processing that Migration Send command:
o if user data is being logged into a User Data Migration Queue that is associated with
the Migratable Controller being migrated, an entry is placed into the User Data
Migration Queue that indicates the user data logging has suspended, and then deletes
the User Data Migration Queue, if the Delete U ser Data Migration Queue bit is set to
‘1’.
5. Since the Migratable Controller in the Source NVM Subsystem is suspended, the MMH S migrates
the user data in the namespaces attached to the Migratable Controller that are identified in the
posted entries in the User Data Migration Queue. Refer to the appropriate I/O command set
specification to determine how the entries identify when entries have completed being posted. That
MMHS may handle the modified host memory in the host as reported by the Track Receive
command.
6. The MMHS is able to obtain the controller state of the Migratable Controller in the Source NVM
Subsystem by submitting one or more Migration Receive commands to the MMC in the Source
NVM Subsystem specifying:
