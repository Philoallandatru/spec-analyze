---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 127
headings: ["3.5.2.1 Discovery Controller Initialization"]
---

# Source page 127

NVM Express® Base Specification, Revision 2.1

105
ii. Issue the Set Features command with the I/O Command Set Profile Feature Identifier (FID
19h) specifying the index of the I/O Command Set Combination (refer to Figure 327) to be
enabled;
and
b. For each I/O Command Set that is enabled  (Note: the NVM Command Set is enabled if the
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
Figure 311): the I/O Command Set specific Identify Namespace data structure, the I/O
Command Set specific Identify Controller data structure, and the I/O Command Set
independent Identify Namespace data structure;
9. The host should determine:
a. the maximum I/O Queue size using CAP.MQES; and
b. the number of I/O Submission Queues and I/O Completion Queues supported using the
response from the Set Features command with the Number of Queues feature identifier;
10. The host should use the Connect command (refer to section 6.3) to create I/O Submission and
Completion Queue pairs; and
11. To enable asynchronous notification of optional events, the host should issue a Set Features
command specifying the events to enable. The host may submit one or more Asynchronous Event
Request commands to be notified of asynchronous events as described by section 5.1.2. This step
may be done at any point after the controller signals that the controller is ready (i.e., CSTS.RDY is
set to ‘1’).
The association may be removed if step 5 (i.e., setting CC.EN to ‘1’) is not completed within 2 minutes after
establishing the association.
 Discovery Controller Initialization
The initialization process for Discovery controllers is described in Figure 83.
