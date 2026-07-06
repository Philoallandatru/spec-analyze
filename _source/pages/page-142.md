---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 142
headings: ["3.8.2 Media Unit Organization Examples", "3.8.2.1 Simple NVM Subsystem"]
---

# Source page 142

NVM Express® Base Specification, Revision 2.1

120
The capacity in an NVM Set is able to be allocated to one or more namespaces, and each namespace that
contains formatted storage exists entirely in that NVM Set (refer to section 3.2.2). Not all of the capacity in
the NVM Set is required to be allocated to a namespace.
If the controller supports NVM Sets, then the capacity in an Endurance Group is able to be allocated to one
or more NVM Sets and each NVM Set exists entirely in that Endurance Group (refer to section 3.2.3). Not
all of the capacity in an Endurance Group is required to be allocated to an NVM Set.
If the controller supports Endurance Groups and does not indicate support for NVM Sets, then in all data
structures that contain an NVMSETID field, the NVMSETID field shall be cleared to 0h.
If the controller does not support Endurance Groups, then in all data structures that contain an ENDGID
field, the ENDGID field shall be cleared to 0h.
If the controller supports Endurance Groups, then the capacity in a domain is able to be allocated to one or
more Endurance Groups, and each Endurance Group exists entirely in that domain (refer to section 3.2.5).
Not all of the capacity in a domain is required to be allocated to an Endurance Group.
NVM subsystems may report the composition of Endurance Groups and NVM Sets in terms of Media Units.
Each Media Unit is allocated to exactly one Endurance Group. If NVM Sets are supported, each Media Unit
is allocated to exactly one NVM Set. Data is transferred to and from Media Units via Channels. Each Media
Unit is connected to one or more Channels. Each Channel is connected to one or more Media Units.
A host uses Capacity Management (refer to section 8.1.4) to allocate:
a) Domain capacity to Endurance Groups;
b) Endurance Group capacity to NVM Sets;
c) Media Units to Endurance Groups; and
d) Media Units to NVM Sets,
as part of creating those entities.
A host uses the Namespace Management create operation (refer to section 8.1.15) to allocate capacity to
namespaces that contain formatted storage.
 Media Unit Organization Examples
Allocation of Media Units is used to organize the physical NVM resources in an NVM subsystem to meet
particular performance goals.
The following examples show an NVM subsystem with all resources in a single domain. The domain has
four Channels, with four Media Units attached to each Channel.
 Simple NVM Subsystem
Figure 86 shows an example of a single domain NVM subsystem where endurance is managed across all
media units. The performance goal is maximum bandwidth, which is achieved by allowing each read or
write operation to simultaneously access all Media Units. All Media Units are in the same Endurance Group
and in the same NVM Set.
