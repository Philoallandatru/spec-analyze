---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 347
headings: []
---

# Source page 347

NVM Express® Base Specification, Revision 2.1

325
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
543:540 O O R
Maximum Number of Allowed Namespaces (MNAN): This field indicates the maximum
number of namespaces supported by the NVM subsystem. If this field is cleared to 0h,
then the maximum number of namespaces supported by the NVM subsystem is less than
or equal to the value in the NN field. If the controlle r supports Asymmetric Namespace
Access Reporting, then this field shall be set to a non-zero value that is less than or equal
to the NN value.
559:544 O R R
Maximum Domain Namespace Attachments (MAXDNA): Indicates the maximum of
the sum of the number of namespaces attached to each I/O controller in the Domain. If
this field is cleared to 0h, then no maximum is specified.
The value of this field shall be the same value for all I/O controllers in the Domain.
563:560 O R R
Maximum I/O Controller Namespace Attachments (MAXCNA): Indicates the
maximum number of namespaces that are allowed to be attached to this I/O controller. If
this field is cleared to 0h, then no maximum is specified.
The value of this field shall be less than or equal to the number of namespaces supported
by the NVM subsystem (refer to the MNAN field).
567:564 O R R
Optimal Aggregated Queue Depth (OAQD): Indicates the recommended maximum
total number of outstanding I/O commands across all I/O queues on the controller for
optimal operation. The host may use this value to limit the number of commands
outstanding at one time across all I/O queues on the controller.
If this field is cleared to 0h, then the Optimal Aggregated Queue Depth is not reported.
568 O R R
Recommended Host-Initiated Refresh Interval (RHIRI):  If the Host -Initiated Refresh
capability is supported (i.e., the HIRS bit in the DSTO field is set to ‘1’), then this field
indicates the recommended time interval in days from last power down to the time at
which the host should initiate the Host-Initiated Refresh operation.
If this field is cleared to 0h, then this field is not reported.
If the HIRS bit in the DSTO field is cleared to ‘0’, then This field shall be cleared to 0h.
569 O R R
Host-Initiated Refresh Time (HIRT): If the Host-Initiated Refresh capability is supported
(i.e., the HIRS bit in the DSTO field is set to ‘1’), then this field indicates the nominal
amount of time in minutes that the controller takes to complete the Host-Initiated Refresh
operation.
If this field is cleared to 0h, then this field is not reported.
If the HIRS bit in the DSTO field is cleared to ‘0’, then This field shall be cleared to 0h.
571:570 O O P
Controller Maximum Memory Range Tracking Descriptors (CMMRTD): This field
indicates the maximum number of Memory Range Tracking Descriptors (refer to  Figure
469) the controller supports (refer to section 5.1.27.1.2).
If the THMCS bit is cleared to ‘0’, then this field shall be cleared to 0h and the host should
ignore this field.
573:572 O O P
NVM Subsystem Maximum Memory Range Tracking Descriptors (NMMRTD): This
field indicates the maximum number of Memory Range Tracking Descriptors (refer to
Figure 469) the NVM subsystem supports (refer to section 5.1.27.1.2).
If the THMCS bit is cleared to ‘0’, then this field shall be cleared to 0h and the host should
ignore this field.
574 O O P
Minimum Memory Range Tracking Granularity (MINMRTG): This field indicates the
minimum value supported in the Requested Memory Range Tracking Granularity
(RMRTG) field (refer to Figure 469) of the Track Memory Ranges data structure.
If the THMCS bit is cleared to ‘0’, then this field shall be cleared to 0h and the host should
ignore this field.
