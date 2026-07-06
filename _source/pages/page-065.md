---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 65
headings: ["3.1.3.3.3 Discovery Controller Initialization", "3.1.3.4 Command Support Requirements"]
---

# Source page 65

NVM Express® Base Specification, Revision 2.1

43
As a result of a Discovery controller updating Host Discovery log page(s), that Discovery controller shall
send a Host Discovery Log Page Change Asynchronous Event notification ( i.e., the Asynchronous Event
Information field set to F1h) to each entity that has requested asynchronous event notifications of this type
(refer to Figure 151).
3.1.3.3.3 Discovery Controller Initialization
The initialization process for Discovery controllers is described in section 3.5.2.1.
 Command Support Requirements
Figure 28 defines commands that are mandatory, optional, and prohibited for an I/O controller,
Administrative controller, and Discovery controller. I/O Command Set specific command support
requirements are described within individual I/O Command Set specifications.  Since an Administrative
controller does not support I/O queues, I/O Command Set specific commands that are not Admin
commands are not supported by an Administrative controller.
A host may utilize the Commands Supported and Effects log page to determine optional commands that
are supported by a controller.
Figure 28: Admin Command Support Requirements
Command
Combined
Opcode
Value
Controller Support Requirements1 Reference
I/O Administrative Discovery
Delete I/O Submission Queue 00h M 9 P P 5.2.4
Create I/O Submission Queue 01h M 9 P P 5.2.2
Get Log Page 02h M M M 5.1.12
Delete I/O Completion Queue 04h M 9 P P 5.2.3
Create I/O Completion Queue 05h M 9 P P 5.2.1
Identify 06h M M M 5.1.13
Abort 08h M O O 5.1.1
Set Features 09h M O 4 Note 6 5.1.25
Get Features 0Ah M O 4 Note 6 5.1.11
Asynchronous Event Request 0Ch M O 5 Note 6 5.1.2
Namespace Management 0Dh O O P 5.1.21
Firmware Commit 10h O O P 5.1.8
Firmware Image Download 11h O O P 5.1.9
Device Self-test 14h O O P 5.1.4
Namespace Attachment 15h O O P 5.1.20
Keep Alive 18h M 2 M 2 Note 6 5.1.14
Directive Send 19h O O P 5.1.7
Directive Receive 1Ah O O P 5.1.6
Virtualization Management 1Ch O O P 5.2.6
NVMe-MI Send 1Dh O O P 5.1.19
NVMe-MI Receive 1Eh O O P 5.1.16
Capacity Management 20h O O P 5.1.3
Discovery Information Management 21h P P M 7 5.3.3
Fabric Zoning Receive 22h P P M 7 5.3.5
Lockdown 24h O O P 5.1.15
Fabric Zoning Lookup 25h P P M 7 5.3.4
Clear Exported NVM Resource
Configuration 28h P O P 5.3.1
