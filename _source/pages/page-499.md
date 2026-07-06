---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 499
headings: ["8.1.1.2 ANA Groups"]
---

# Source page 499

NVM Express® Base Specification, Revision 2.1

477
Figure 586: Namespace B optimized through Controller 1
NVMe Controller 1
NSID 2
NVMe Controller 2
Port x Port yNVM
Subsystem
NSID 3NSID 3
NS
C
NSID 2
NS
B
Media Access
Boundary

 ANA Groups
Namespaces that are members of the same ANA Group perform identical asymmetric namespace access
state transitions. The ANA Group maintains the same asymmetric namespace access state for all
namespaces that are members of that ANA Group (i.e., a change in t he asymmetric namespace access
state of one namespace only occurs as part of a change in the asymmetric namespace access state of all
namespaces that are members of that ANA Group). Namespaces that are members of the same ANA
Group shall be members of the same domain (refer to section 2.3.1). The method for assigning namespaces
to ANA Groups is outside the scope of this specification.
An ANA Group may contain zero or more namespaces, zero or more NVM Sets, or zero or more Endurance
Groups. The mapping of namespaces, NVM Sets, and Endurance Groups to ANA Groups is vendor
specific.
A valid ANA Group Identifier is a non-zero value that is less than or equal to ANAGRPMAX (refer to Figure
312).
The ANA Group Identifier (ANAGRPID) for each ANA Group shall be unique within the NVM subsystem. If
the ANA Group ID Change Support (ANAGIDCS) bit  in the ANACAP field in the Identify Controller data
structure is set to ‘1’, then the ANA Group Identifier shall not change while the namespace is attached to
any controller in the NVM subsystem. If the ANAGIDCS bit is cleared to ‘0’, then the ANA Group Identifier
may change while the namespace is attached to any controller in the NVM subsystem. If the ANA Group
Identifier changes, the controller shall issue the Asymmetric Namespace Access Change Notice as
described in 8.1.1.9.
If two or more participating NVM subsystems provide access to a dispersed namespace, and that dispersed
namespace is a member of an ANA Group on any participating NVM subsystem, then each participating
NVM subsystem shall contain an ANA Group whose members include that d ispersed namespace. The
ANAGRPID for a dispersed namespace may or may not be the same on each participating NVM subsystem.
Figure 587 shows the following four namespaces:
• the private namespace A in a first ANA Group;
• namespace B and namespace D, that are in the same second ANA Group; and
• namespace C that is in a third ANA Group.
