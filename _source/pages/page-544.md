---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 544
headings: ["8.1.12 Host Managed Live Migration"]
---

# Source page 544

NVM Express® Base Specification, Revision 2.1

522
A Controller Level Reset on a controller that is not performing the Host-Initiated Refresh operation shall not
impact that Host-Initiated Refresh operation.
 Host Managed Live Migration
The migration of a controller , by a host that is using  the Host Managed Live Migration capability involves
one or more NVM subsystem s, multiple hosts, and multiple controllers. In this section, to differentiate
between the multiple hosts and controllers, each is given a unique name as illustrated in  Figure 619. In an
NVM subsystem that supports Host Managed Live Migration, each controller that sets the HMLMS bit to ‘1’
is referred to as a Migration Management Controller (MMC) and each host associated with an MMC is
referred to as a Migration Management Host (MMH) . In that same NVM subsystem, all other controllers
(i.e., those that clear the HMLMS bit to ‘0’) are referred to as Migratable Controller s.
The Host Managed Live Migration capability allows an MMH to use an MMC to migrate a Migratable
Controller from a Source NVM Subsystem to a Migratable Controller in a Destination NVM Subsystem.
During that migration, a host is actively submitting commands to a Migratable Controller in the Source NVM
Subsystem and that controller is processing those commands.
The MMH that is managing the migration of a Migratable Controller from the Source NVM Subsystem is
responsible for ensuring that the Migratable Controller in the Destination NVM Subsystem is compatible
with controller state of the Migratable Controller from the Source NVM Subsystem. It is also the
responsibility of the MMH to transfer the migrating data between the NVM subsystems.
Figure 619: Host Managed Live Migration Host and Controller Naming
      NVM subsystem
(Destination NVM Subsystem)
Controller
(Migratable
Controller)
Host
Controller
(Managing Controller)
Migrating
Management Host
(MMH)
Named: MMHD
...
Controller
(Migratable
Controller)
Host
Attached
Namespaces
        NVM subsystem
(Source NVM Subsystem)
Controller
(Migratable
Controller)
Host
Attached
Namespaces
Migrating
Management
Controller
(MMC)
Migrating
Management Host
(MMH)
Named: MMHS
...
Controller
(Migratable
Controller)
Host
Attached
Namespaces
Migrating a migratable controller

An MMC is an I/O controller or an Administrative controller that supports the Host Managed Live Migration
capability and provides the ability for the MMH to use privileged actions (refer to section  3.10 to:
• suspend the processing of commands on the Migratable Controller in the Source NVM Subsystem
being migrated (refer to the Migration Send command in section 5.1.17.1.1);
• obtain the state of the Migratable Controller in the Source NVM Subsystem being migrated (refer
to the Migration Receive command in section 5.1.16.1.1);
• set that controller state in the Migratable Controller in the Destination NVM Subsystem (refer to
the Migration Send command in section 5.1.17.1.3); and
• resume the operation on the Migratable Controller in the Destination NVM Subsystem (refer to the
Migration Send command in section 5.1.17.1.2).
