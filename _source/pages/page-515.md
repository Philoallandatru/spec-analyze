---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 515
headings: ["8.1.4.3 Variable Capacity Management"]
---

# Source page 515

NVM Express® Base Specification, Revision 2.1

493
1) reading the Supported Capacity Configuration List log page (refer to Figure 256) and selecting the
desired configuration; and
2) issuing a Capacity Management command specifying the Select Capacity Configuration operation
and the Capacity Configuration Identifier of the desired configuration.
Following successful completion of the command, each Media Unit is allocated to one Endurance Group
and to one NVM Set. The resulting configuration of Media Units is reported by the Media Unit Status log
page (refer to section 5.1.12.1.16).
 Variable Capacity Management
Variable Capacity Management allows the dynamic creation and deletion of Endurance Groups and NVM
Sets.
A controller supporting Variable Capacity Management:
a) may support the Media Unit Status log page;
b) shall support Endurance Groups;
c) may support NVM Sets;
d) shall set the Variable Capacity Management bit to ‘1’ in the CTRATT field of the Identify Controller
data structure (refer to Figure 312);
e) shall support the Create Endurance Group operation of the Capacity Management command;
f) may support the Delete Endurance Group operation of the Capacity Management command; and
g) if NVM Sets are supported:
a. shall support the Create NVM Set operation of the Capacity Management command;
b. shall report non -zero values in the Total Endurance Group Capacity field and the
Unallocated Endurance Group Capacity field in the Endurance Group Information log page
(refer to Figure 218); and
c. may support the Delete NVM Set operation of the Capacity Management command.
If a controller supports the Delete Endurance Group operation of the Capacity Management command,
then it shall set the Delete Endurance Group bit to ‘1’ in the CTRATT field of the Identify Controller data
structure.
If a controller supports the Delete NVM Set operation of the Capacity Management command, then it shall
set the Delete NVM Set bit to ‘1’ in the CTRATT field of the Identify Controller data structure.
A typical sequence of operations for allocating capacity is:
1) determine the available capacities in each domain (refer to section 3.2.5);
2) create Endurance Group with desired capacity (refer to section 5.1.3);
3) create NVM Set with desired capacity in the Endurance Group (refer to section 5.1.3); and
4) create namespace with desired capacity in the NVM Set (refer to section 5.1.21).
A typical sequence of operations for deallocating capacity is:
1) delete namespace that contains formatted storage, if any, (refer to section 5.1.21);
2) delete NVM Set, if any, (refer to section 5.1.30); and
3) delete Endurance Group (refer to section 5.1.3).
If there is insufficient unallocated capacity in an Endurance Group for the controller to create an NVM Set,
then the host can delete one or more NVM Sets in that Endurance Group and create the new NVM Set
using some or all of the available capacity.
If there is insufficient unallocated capacity in a domain for the controller to create an Endurance Group,
then the host can delete one or more Endurance Groups in that domain and create a new Endurance Group
using some or all of the available capacity.
