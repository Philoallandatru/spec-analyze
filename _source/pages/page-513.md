---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 513
headings: ["8.1.4 Capacity Management", "8.1.4.1 Overview"]
---

# Source page 513

NVM Express® Base Specification, Revision 2.1

491
is in a Write Locked Until Power Cycle state. Refer to section 8.1.21 for the specific behavior that the
controller exhibits under this condition.
 Capacity Management
 Overview
Capacity Management is a capability for organizing physical media into Endurance Groups and NVM Sets.
There are two different forms of Capacity Management, Fixed Capacity Management and Variable Capacity
Management. A controller that supports Capacity Management shall support at least one form.
Capacity Management commands shall not be supported by Exported NVM Subsystems.
The host uses Fixed Capacity Management to create Endurance Groups and NVM Sets by selecting a
configuration which explicitly allocates Media Units (refer to section 8.1.4.2) to Endurance Groups and NVM
Sets.
The host uses Variable Capacity Management to:
• create a single Endurance Group by specifying the desired capacity;
• create a single NVM Set by specifying the desired capacity;
• delete a single Endurance Group; and
• delete a single NVM Set.
The Capacity Adjustment Factor is the ratio between the capacity consumed by an Endurance Group from
the Unallocated NVM Capacity field in the Identify Controller data structure and the total NVM capacity in
that Endurance Group, i.e.:
𝐶𝑎𝑝𝑎𝑐𝑖𝑡𝑦 𝐴𝑑𝑗𝑢𝑠𝑡𝑚𝑒𝑛𝑡 𝐹𝑎𝑐𝑡𝑜𝑟 = 𝐼𝑁𝑇𝐸𝐺𝐸𝑅 ( 𝐶𝑎𝑝𝑎𝑐𝑖𝑡𝑦 𝐶𝑜𝑛𝑠𝑢𝑚𝑒𝑑
𝐸𝑛𝑑𝑢𝑟𝑎𝑛𝑐𝑒 𝐺𝑟𝑜𝑢𝑝 𝐶𝑎𝑝𝑎𝑐𝑖𝑡𝑦 ∗ 100)
(E.g., the value 200 indicates that creation of an Endurance Group with a total NVM capacity of 5 GiB
consumes 10 GiB of the Unallocated NVM Capacity indicated by the controller).
If an Endurance Group is created, then the controller performs the following actions as an atomic operation:
a) the value indicated by the Unallocated NVM Capacity (UNVMCAP) field of the Identify Controller
data structure is changed based on the requested capacity, the Capacity Adjustment Factor of the
created Endurance Group, and the granularity at which the controller allocates NVM capacity; and
b) the Endurance Group Identifier is added to the Endurance Group List.
If an Endurance Group is deleted, then the controller performs the following actions in sequence:
1) the Endurance Group Identifier is removed from the Endurance Group List;
2) if the Media Unit Status log page is supported, then the Endurance Group Identifier field is cleared
to 0h in all Media Unit Status Descriptors, if any, that indicate the deleted Endurance Group;
3) every NVM Set in the Endurance Group is deleted; and
4) the value indicated by the Unallocated NVM Capacity (UNVMCAP) field of the Identify Controller
data structure is increased by the value that was indicated by the Total Endurance Group Capacity
(TEGCAP) field of the Endurance Group Information log page of the deleted Endurance Group.
If any of the entities modified by the above sequence are accessed after the sequence begins and before
it completes, then the results are indeterminate.
If an NVM Set is created, then the controller performs the following actions as an atomic operation:
a) the NVM Set Identifier is added to the NVM Set List; and
b) the Unallocated Endurance Group Capacity indicated by the Endurance Group Information log
page (refer to Figure 218) is decreased by the amount of capacity allocated to the NVM Set; the
controller may allocate NVM capacity in units such that the requested size for an NVM Set may be
rounded up to the next unit boundary.
