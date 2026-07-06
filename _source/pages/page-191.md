---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 191
headings: ["5 Admin Command Set"]
---

# Source page 191

NVM Express® Base Specification, Revision 2.1

169
5 Admin Command Set
The Admin Command Set defines the commands that may be submitted to the Admin Submission Queue.
Section 5.1 describes Admin commands  that are common to all transport models . Section 5.2 describes
Admin commands that are specific to the Memory -based transport model . Section 5.3 describes Admin
commands that are specific to the Message-based transport model.
The submission queue entry (SQE) structure and the fields that are common to all Admin com mands are
defined in section 4.1. The completion queue entry (CQE) structure and the fields that are common to all
Admin commands are defined in section 4.2. The command specific fields in the SQE and CQE structures
(i.e., SQE Command Dwords 10 to 15, CQE Dword 0, and CQE Dword 1) for the Admin Command Set are
defined in this section.
Admin commands should not be impacted by the state of I/O queues (e.g., a full I/O Completion Queue
should not delay or stall the Delete I/O Submission Queue command).
Figure 141 defines all Admin commands. Refer to Figure 28 for mandatory, optional, and prohibited
commands for the various controller types.
Figure 141: Opcodes for Admin Commands
Opcode by Field
Combined
Opcode 1
Namespace
Identifier
Used 2
Command Reference 8
(07:02)  (01:00)
Function
Data
Transfer3
0000 00b 00b 00h No Delete I/O Submission Queue 5.2.4
0000 00b 01b 01h No Create I/O Submission Queue  5.2.2
0000 00b 10b 02h Yes11 Get Log Page 5.1.12
0000 01b 00b 04h No Delete I/O Completion Queue 5.2.3
0000 01b 01b 05h No Create I/O Completion Queue 5.2.1
0000 01b 10b 06h NOTE 6 Identify 5.1.13
0000 10b 00b 08h No Abort 5.1.1
0000 10b 01b 09h Yes Set Features 5.1.25
0000 10b 10b 0Ah Yes Get Features 5.1.11
0000 11b 00b 0Ch No Asynchronous Event Request 5.1.2
0000 11b 01b 0Dh Yes Namespace Management 5.1.21
0001 00b 00b 10h No Firmware Commit 5.1.8
0001 00b 01b 11h No Firmware Image Download 5.1.9
0001 01b 00b 14h Yes Device Self-test 5.1.4
0001 01b 01b 15h Yes 4 Namespace Attachment 5.1.20
0001 10b 00b 18h No Keep Alive 5.1.14
0001 10b 01b 19h Yes 5 Directive Send 5.1.7
0001 10b 10b 1Ah Yes 5 Directive Receive 5.1.6
0001 11b 00b 1Ch No Virtualization Management 5.2.6
0001 11b 01b 1Dh No NVMe-MI Send 5.1.19
0001 11b 10b 1Eh No NVMe-MI Receive 5.1.18
0010 00b 00b 20h No Capacity Management 5.1.3
0010 00b 01b 21h No Discovery Information Management 5.3.3
0010 00b 10b 22h No Fabric Zoning Receive 5.3.5
0010 01b 00b 24h No Lockdown 5.1.15
0010 01b 01b 25h No Fabric Zoning Lookup 5.3.4
0010 10b 00b 28h No Clear Exported NVM Resource Configuration10 5.3.1
0010 10b 01b 29h No Fabric Zoning Send 5.3.6
0010 10b 10b 2Ah No Create Exported NVM Subsystem10 5.3.2
