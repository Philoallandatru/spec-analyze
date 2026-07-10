---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 367
headings: ["5.1.13.3.2 Secondary Controller list (CNS 15h)"]
---

# Source page 367

NVM Express® Base Specification, Revision 2.1

345
Figure 330: Identify – Primary Controller Capabilities Structure
Bytes Description
04
Controller Resource Types (CRT): This field indicates the controller resources types supported. If a
primary controller supports a controller resource type, then all associated secondary controllers shall
support that controller resource type.
Bits Description
7:2 Reserved
1 VI Resources Support (VIRS): If this bit is set to ‘1’, then VI Resources are supported. If this
bit is cleared to ‘0’, then VI Resources are not supported. Refer to section 8.2.6.2.
0 VQ Resources Support (VQRS): If this bit is set to ‘1’, then VQ Resources are supported. If
this bit is cleared to ‘0’, then VQ Resources are not supported. Refer to section 8.2.6.1.

31:05 Reserved
35:32 VQ Resources Flexible Total (VQFRT): This field indicates the total number of VQ Flexible Resources
for the primary and its secondary controllers.
39:36 VQ Resources Flexible Assigned (VQRFA): This field indicates the total number of VQ Flexible
Resources Assigned to the associated secondary controllers.
41:40
VQ Resources Flexible Allocated to Primary (VQRFAP): This field indicates the total number of VQ
Flexible Resources currently allocated to the primary controller. This value may change after a Controller
Level Reset other than a Controller Reset, if a new value was set using the Virtualization Management
command. The default value of this field is implementation specific.
43:42 VQ Resources Private Total (VQPRT): This field indicates the total number of VQ Private Resources
for the primary controller.
45:44 VQ Resources Flexible Secondary Maximum (VQFRSM):  This field indicates the maximum number
of VQ Flexible Resources that may be assigned to a secondary controller.
47:46
VQ Flexible Resource Preferred Granularity (VQGRAN): This field indicates the preferred granularity
of assigning and removing VQ Flexible Resources. Assigning and removing VQ Resources in this
granularity minimizes any wasted internal implementation resources.
63:48 Reserved
67:64 VI Resources Flexible Total (VIFRT): This field indicates the total number of VI Flexible Resources for
the primary and its secondary controllers.
71:68 VI Resources Flexible Assigned (VIRFA): This field indicates the total number of VI Flexible
Resources Assigned to the associated secondary controllers.
73:72
VI Resources Flexible Allocated to Primary (VIRFAP): This field indicates the total number of VI
Flexible Resources currently allocated to the primary controller. This value may change after a Controller
Level Reset other than a Controller Reset, if a new value was set using the Virtualization Management
command. The default value of this field is implementation specific.
75:74 VI Resources Private Total (VIPRT): This field indicates the total number of VI Private Resources for
the primary controller.
77:76 VI Resources Flexible Secondary Maximum (VIFRSM): This field indicates the maximum number of
VI Flexible Resources that may be assigned to a secondary controller.
79:78
VI Flexible Resource Preferred Granularity (VIGRAN): This field indicates the preferred granularity
of assigning and removing VI Flexible Resources. Assigning and removing VI Resources in this
granularity minimizes any wasted internal implementation resources.
4095:80 Reserved
5.1.13.3.2 Secondary Controller list (CNS 15h)
A Secondary Controller List (refer to Figure 331) is returned to the host for up to 127 secondary controllers
associated with the primary controller processing this command. The list contains entries for controller
identifiers greater than or equal to the value specified in the Controller Identifier (CD W10.CNTID) field.
All secondary controllers are represented, including those that are in an Offline state due to SR -IOV
configuration settings (e.g., VF Enable is cleared to ‘0’ or NumVFs specifies a value that does not enable
the associated secondary controller).
