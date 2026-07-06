---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 482
headings: ["7 I/O Commands", "7.1 Cancel command"]
---

# Source page 482

NVM Express® Base Specification, Revision 2.1

460
7 I/O Commands
An I/O command is a command submitted to an I/O Submission Queue. Figure 555 lists the I/O commands
that are defined for use  in all I/O Command Sets . The following subsections provide defin itions for each
command. Refer to section 3.1.3.4 for mandatory, optional, and prohibited  I/O commands for the various
controller types. The following subsections describe the definition for each of these commands.
The user data format and any end-to-end protection information is I/O Command Set specific. Refer to each
I/O Command Set specification for applicability and additional details, if any.  Refer to the referenced I/O
Command Set specification for all I/O Command Set specific commands described in Figure 555.
Commands shall only be submitted by the host when the controller is ready as indicated in the Controller
Status property (CSTS.RDY) and after appropriate I/O Submission Queue(s) and I/O Completion Queue(s)
have been created.
The submission queue entry (SQE) structure and the fields that are common to all I/O commands are
defined in section 4.1. The completion queue entry (CQE) structure and the fields that are common to all
I/O commands are defined in section 4.2. The command specific fields in the SQE and CQE structures (i.e.,
SQE Command Dwords 10-15, CQE Dword 0, and CQE Dword 1) for I/O commands supported across all
I/O Command Sets are defined in this section.
Figure 555: Opcodes for I/O Commands
Opcode by Field Combined
Opcode1 Command2 Reference (07:02)  (01:00)
Function Data Transfer3
0000 00b 00b 00h Flush4 7.2
0000 11b 01b 0Dh Reservation Register 7.6
0000 11b 10b 0Eh Reservation Report 7.8
0001 00b 01b 11h Reservation Acquire 7.5
0001 00b 10b 12h I/O Management Receive 7.3
0001 01b 01b 15h Reservation Release 7.7
0001 10b 00b 18h Cancel4 7.1
0001 11b 01b 1Dh I/O Management Send 7.4
0111 11b 11b5 7Fh Fabric Commands5 6
Vendor Specific
1xxx xxb NOTE 3 80h to FFh Vendor specific
Notes:
1. Opcodes not listed are I/O Command Set specific or reserved. Refer to Figure 91 for Opcode details.
2. All I/O commands use the Namespace Identifier (NSID) field. The value FFFFFFFFh is not supported in this field
unless footnote 4 in this figure indicates that a specific command does support that value.
3. 00b = no data transfer; 01b = host to controller; 10b = controller to host; 11b = bidirectional. Refer to the Transfer
Direction field in Figure 91.
4. This command may support the use of the Namespace Identifier (NSID) field set to FFFFFFFFh.
5. All Fabrics commands use the opcode 7Fh with the direction of data transfer specified as shown in Figure 540.
Refer to section 6 for details.
7.1 Cancel command
The Cancel command is used to request an abort for specified commands submitted on the same I/O
Submission Queue to which the Cancel command is submitted. The Cancel command may apply to a single
namespace or to multiple namespaces. The Cancel command may be deeply queued. Some of the
commands to abort may have already completed, currently be processing, or be deeply queued. As a result,
the performance of the Cancel command may be impacted.
