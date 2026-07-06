---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 167
headings: []
---

# Source page 167

NVM Express® Base Specification, Revision 2.1

145
Figure 103: Status Code – Command Specific Status Values
Value Description Commands Affected
01h Invalid Queue Identifier
Create I/O Submission Queue, Create I/O
Completion Queue, Delete I/O Completion
Queue, Delete I/O Submission Queue
02h Invalid Queue Size  Create I/O Submission Queue, Create I/O
Completion Queue
03h Abort Command Limit Exceeded Abort
04h Reserved
05h Asynchronous Event Request Limit Exceeded  Asynchronous Event Request
06h Invalid Firmware Slot Firmware Commit
07h Invalid Firmware Image Firmware Commit
08h Invalid Interrupt Vector Create I/O Completion Queue
09h Invalid Log Page Get Log Page
0Ah Invalid Format Format NVM, Namespace Management
0Bh Firmware Activation Requires Conventional Reset Firmware Commit, Sanitize
0Ch Invalid Queue Deletion Delete I/O Completion Queue
0Dh Feature Identifier Not Saveable Set Features
0Eh Feature Not Changeable Set Features
0Fh Feature Not Namespace Specific Set Features
10h Firmware Activation Requires NVM Subsystem Reset Firmware Commit, Sanitize
11h Firmware Activation Requires Controller Level Reset Firmware Commit, Sanitize
12h Firmware Activation Requires Maximum Time Violation Firmware Commit
13h Firmware Activation Prohibited Firmware Commit
14h Overlapping Range Firmware Commit, Firmware Image Download,
Set Features
15h Namespace Insufficient Capacity Namespace Management
16h Namespace Identifier Unavailable Namespace Management
17h Reserved
18h Namespace Already Attached Namespace Attachment
19h Namespace Is Private  Namespace Attachment
1Ah Namespace Not Attached Namespace Attachment
1Bh Thin Provisioning Not Supported Namespace Management
1Ch Controller List Invalid Namespace Attachment
1Dh Device Self-test In Progress Device Self-test
1Eh Boot Partition Write Prohibited Firmware Commit
1Fh Invalid Controller Identifier
Virtualization Management, Track Send, Track
Receive, Migration Send, Migration
Receive,Controller Data Queue
20h Invalid Secondary Controller State Virtualization Management
21h Invalid Number of Controller Resources Virtualization Management
22h Invalid Resource Identifier Virtualization Management
23h Sanitize Prohibited While Persistent Memory Region
is Enabled Sanitize
24h ANA Group Identifier Invalid Namespace Management
25h ANA Attach Failed Namespace Attachment
26h Insufficient Capacity Capacity Management
27h Namespace Attachment Limit Exceeded Namespace Attachment
28h Prohibition of Command Execution Not Supported Lockdown
29h I/O Command Set Not Supported Namespace Attachment, Namespace
Management
2Ah I/O Command Set Not Enabled Namespace Attachment
2Bh I/O Command Set Combination Rejected Set Features
2Ch Invalid I/O Command Set Identify
2Dh Identifier Unavailable Capacity Management
