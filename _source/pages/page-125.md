---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 125
headings: ["3.5.2 Message-based Controller Initialization (Fabrics)"]
---

# Source page 125

NVM Express® Base Specification, Revision 2.1

103
8. The host determines any I/O Command Set specific configuration information as follows:
a. If the CAP.CSS.IOCSS bit is set to ‘1’, then the host does the following:
i. Issue the Identify command specifying the Identify I/O Command Set data structure (CNS
1Ch); and
ii. Issue the Set Features command with the I/O Command Set Profile Feature Identifier (FID
19h) specifying the index of the I/O Command Set Combination (refer to Figure 327) to be
enabled;
and
b. For each I/O Command Set that is enabled (Note: the NVM Command Set is enabled if the
CC.CSS field is set to 000b):
i. Issue the Identify command specifying the I/O Command Set specific Active Namespace
ID list (CNS 07h) with the appropriate Command Set Identifier (CSI) value of that I/O
Command Set; and
ii. For each NSID that is returned:
1. If the enabled I/O Command Set is the NVM Command Set or an I/O Command Set
based on the NVM Command Set (e.g., the Zoned Namespace Command Set) issue
the Identify command specifying the Identify Namespace data structure (CNS 00h);
and
2. Issue the Identify command specifying each of the following data structures (refer to
Figure 310): the I/O Command Set specific Identify Namespace data structure, the I/O
Command Set specific Identify Controller data structure, and the I/O Command Set
independent Identify Namespace data structure;
9. If the controller implements I/O queues, then the host should determine the number of I/O
Submission Queues and I/O Completion Queues supported using the Set Features command with
the Number of Queues feature identifier. After determining the number of I/O  Queues, the NVMe
Transport specific interrupt registers (e.g., MSI and/or MSI-X registers) should be configured;
10. If the controller implements I/O queues, then the host should allocate the appropriate number of
I/O Completion Queues based on the number required for the system configuration and the number
supported by the controller. The I/O Completion Queues are alloc ated using the Create I/O
Completion Queue command;
11. If the controller implements I/O queues, then the host should allocate the appropriate number of
I/O Submission Queues based on the number required for the system configuration and the number
supported by the controller. The I/O Submission Queues are alloc ated using the Create I/O
Submission Queue command; and
12. To enable asynchronous notification of optional events, the host should issue a Set Features
command specifying the events to enable. To enable asynchronous notification of events, the host
should submit an appropriate number of Asynchronous Event Request commands. This step may
be done at any point after the controller signals that the controller is ready (i.e., CSTS.RDY is set
to ‘1’).
After performing these steps, the controller shall be ready to process Admin or I/O commands issued by
the host.
For exit of the D3 power state (refer to the PCI Express Base Specification), the initialization steps outlined
should be followed.
 Message-based Controller Initialization (Fabrics)
The host selects the NVM subsystem with which to create a host to controller association. The host first
establishes an NVMe Transport connection with the NVM subsystem. Next the host forms an association
with a controller and creates the Admin Queue using  the Fabrics Connect command. Finally, the host
configures the controller and creates I/O Queues. Figure 82 is a ladder diagram that describes the queue
creation process for an Admin Queue or an I/O Queue.
