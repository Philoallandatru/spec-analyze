---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 505
headings: ["8.1.2.5 Host ANA Transition Operation", "8.1.2.6 All Paths Down Condition", "8.1.3 Boot Partitions", "8.1.3.1 Reading from a Boot Partition"]
---

# Source page 505

NVM Express® Base Specification, Revision 2.1

483
page should be examined on each controller that provides access to the namespace and the host
should resume sending commands based on the new ANA state.
If the ANATT time interval expires, then the host should use a different controller for sending commands to
the namespaces in that ANA Group. The ANATT interval reported by the controller should prevent this type
of timer expiration from occurring.
 Host ANA Transition Operation
Receipt of an Asymmetric Namespace Access Change Notice from a controller may indicate:
a) that the ANA state reported in one or more ANA Group Descriptors has changed;
b) a new NSID has been added to one or more of the ANA Group Descriptors;
c) an NSID has been removed from one or more of the ANA Group Descriptors; and/or
d) the NSID of a namespace has moved from one ANA Group Descriptor to a different ANA Group
Descriptor (i.e., the ANAGRPID field in the Identify Namespace data structure for that namespace
has changed), if the ANA Group ID Change Support (ANAGIDCS) bit in the ANACAP field is cleared
to ‘0’ in the Identify Controller data structure (refer to Figure 312).
As a result of receiving an Asymmetric Namespace Access Change Notice, the host should read the ANA
log page (refer to section 5.1.12.1.13) to check for each of those possible changes.
 All Paths Down Condition
An all paths down condition occurs when there are no paths available on the host to access the namespaces
in an ANA Group (i.e., the NVM media). To determine whether an all paths down condition has occurred,
the host may examine the ANA log page on each co ntroller that provides access to the namespaces in a
particular ANA Group. All paths that are not in the ANA Persistent Loss state should be checked. If no paths
to the namespaces in that ANA Group become available (i.e., transition to the ANA Optimized st ate or the
ANA Non-Optimized state) for the duration of an ANATT time interval, then an all paths down condition has
occurred for the namespaces in that ANA Group.
 Boot Partitions
Boot Partitions provide an optional area of NVM storage that may be read  by the host  without the host
initializing queues or enabling the controller. The simplified interface to access Boot Partitions may be used
for platform initialization code (e.g., a bootloader that is executed from host ROM) to boot to a pre -OS
environment (e.g., UEF I) instead of storing the image on another  non-volatile storage medium (e.g., SPI
flash). Refer to section 8.1.3.1 for the procedure to read the contents of a Boot Partition.
A controller that supports Boot Partitions has two Boot Partitions of equal size using Boot Partition identifiers
0h and 1h. The two Boot Partitions allow the host to update one and verify the contents before marking the
Boot Partition active. Controllers in the NVM subsystem may share the same Boot Partitions.
The contents of Boot Partitions are only modified using the Firmware Image Download and Firmware
Commit commands (refer to section 8.1.3.2) and may be secured using either the Boot Partition Write
Protection Config feature or the Replay  Protected Memory Block to prevent unauthorized modifications
(refer to section 8.1.3.3).
 Reading from a Boot Partition
A Boot Partition is a continuous block of data as shown in Figure 589, that the host may read  via NVMe
properties.
