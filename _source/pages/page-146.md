---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 146
headings: ["3.9 Keep Alive"]
---

# Source page 146

NVM Express® Base Specification, Revision 2.1

124
information for each Endurance Group, the host uses the Get Log Page command to access the Endurance
Group Information log page (refer to section 5.1.12.1.10).
For an NVM subsystem that supports NVM Sets (refer to section 3.2.2), the host may use the Identify
command to access the NVM Set List data structure (refer to section 5.1.13.2.4) to determine the NVM
Sets that are accessible by the controller and the capacity information for each of those NVM Sets.
For the management of Endurance Groups, NVM Sets, and namespaces  that contain formatted storage ,
Figure 89 describes the effects of the support of NVM Sets, Endurance Groups, and domains on which
capacity information is used for each management operation.
Figure 89: Capacity Information Field Usage
Entity being
created / deleted
NVM Sets
supported
Endurance
Groups
supported
Domains
supported Capacity information used 5
Endurance Group 11 n/a Yes
No NVM subsystem 7
Yes Domain 8
NVM Set 11 Yes Yes 6 n/a Endurance Group 9
Namespace that
contains formatted
storage 12
No
No
No NVM subsystem 7
Yes Domain 8
Yes n/a Endurance Group 9
Yes Yes 6 n/a NVM Set 10
Notes:
5. This information described in this column is used by the host for creating the entity (e.g., to determine if there
is sufficient available capacity) and this information is altered by the controller as a result of the creation or
deletion of the entity (e.g., unallocated capacity decreased as a result of entity creation, or unallocated capacity
increased as a result of entity deletion).
6. NVM Set support requires support for Endurance Groups as described in section 3.2.2.
7. Capacity information in the Identify Controller data structure (i.e., TNVMCAP field, UNVMCAP field, and
MEGCAP fields (refer to Figure 312)).
8. Capacity information in the Domain Attributes Entry (i.e., Total Domain Capacity field, Unallocated Domain
Capacity field, and Max Endurance Group Domain Capacity field (refer to Figure 324)).
9. Capacity information in the Endurance Group Information log page (i.e., TEGCAP field, UEGCAP field (refer
to Figure 218)).
10. Capacity information in the NVM Set Attributes Entry (i.e., Total NVM Set Capacity field, and Unallocated NVM
Set Capacity field (refer to Figure 318)).
11. Endurance Groups and NVM Sets are created and deleted using the Capacity Management command (refer
to section 5.1.3)
12. Namespaces are created and deleted using the Namespace Management command (refer to section 8.1.15).
Namespaces are deleted using the Capacity Management command.
3.9 Keep Alive
The Keep Alive capability uses the  Keep Alive Timer on a controller  as a watchdog timer to detect
communication failures (e.g., transport failure, host failure, or controller failure) between a host and a
controller. If the Keep Alive Timer feature is supported (i.e., the KAS field is set to a non -zero value (refer
to Figure 312)), the controller shall support the Keep Alive command.
The term Keep Alive Timeout Time (KATT) refers to the time indicated by the value of the Keep Alive
Timeout field on a controller (refer to Figure 397).
The NVMe Transport binding specification for the associated NVMe Transport defines:
• the minimum Keep Alive Timeout value, if any;
• the maximum Keep Alive Timeout value, if any; and
• if the Keep Alive Timer feature is required to be supported and enabled.
