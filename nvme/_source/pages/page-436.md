---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 436
headings: ["5.1.26.1.2 Command Completion", "5.1.27 Track Send command", "5.1.27.1 Track Send Management Operations", "5.1.27.1.1 Log User Data Changes (Management Operation 0h)"]
---

# Source page 436

NVM Express® Base Specification, Revision 2.1

414
5.1.26.1.2 Command Completion
Upon completion of the Track Receive command, the controller posts a completion queue entry to the
Admin Completion Queue indicating the status for the command. Section 5.1.26.1 describes completion
details each for management operation.
Track Receive command specific status values (i.e., SCT field set to 1h) are shown in Figure 462.
Figure 462: Track Receive – Command Specific Status Values
Value Definition
1Fh Invalid Controller Identifier: The controller for the specified Controller Identifier is already
tracking host memory changes.
 Track Send command
The Track Send command is used to manage the tracking of information by a controller (e.g., what to track
during the migration of a controller in an NVM subsystem).
The Track Send command uses the Command Dword 10  field. The use of the Data Pointer field  and
Command Dword 11 field is specific to the management operation specified by the Select field . All other
command specific fields are reserved.
The Select field defined in Figure 463 specifies the management operation to be performed. Refer to section
5.1.27.1 for a description of each management operation.
Figure 463: Track Send – Command Dword 10
Bits Description
31:16
Management Operation Specific (MOS): The definition of this field is specific to a management operation
(refer to the Select field). If a management operation does not define the use of this field, then this field is
reserved.
15:08 Reserved
07:00
Select (SEL): This field specifies the type of management operation to perform.
Value M/O1 Management Operation Reference Section
0h O Log User Data Changes 5.1.27.1.1
1h O Track Memory Changes 5.1.27.1.2
2h to FFh  Reserved
Notes:
1. O/M definition: O = Optional, M = Mandatory

 Track Send Management Operations
5.1.27.1.1 Log User Data Changes (Management Operation 0h)
The Log User Data Changes management operation of the Track Send command is used to start or stop
logging user data changes to namespaces attached to the controller associated with the User Data
Migration Queue (refer to section 5.1.4.1.1.1) specified in Controller Data Queue Identifier field in Command
Dword 11 (refer to Figure 465).
Refer to the applicable I/O command Set specification for additional requirements.
