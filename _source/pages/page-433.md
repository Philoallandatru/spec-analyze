---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 433
headings: ["5.1.25.3.1 Asynchronous Event Configuration (Feature Identifier 0Bh) for Fabrics", "5.1.25.4 Command Completion", "5.1.26 Track Receive command"]
---

# Source page 433

NVM Express® Base Specification, Revision 2.1

411
5.1.25.3.1 Asynchronous Event Configuration (Feature Identifier 0Bh) for Fabrics
The Asynchronous Event Configuration feature is not Fabrics specific. Configuration of notifications for
Fabrics specific events is described in section 5.1.25.1.5.
 Command Completion
Upon completion of the Set Features command, the controller posts a completion queue entry to the Admin
Completion Queue. If a status code of Successful Completion is returned, the completion queue entry shall
not be posted until the controller has completed setting attributes associated with the Feature. Set Features
command specific status values are defined in Figure 456.
Figure 456: Set Features – Command Specific Status Values
Value Description
0Dh Feature Identifier Not Saveable: The Feature Identifier specified does not support a saveable value.
0Eh Feature Not Changeable: The Feature Identifier specified does not support a changeable value or the
value is not changeable at this time.
0Fh Feature Not Namespace Specific: The Feature Identifier does not have namespace scope. The scope
is defined in Figure 385. The Feature Identifier settings apply across all namespaces.
14h Overlapping Range: Command Set specific definition . Refer to each I/O Command Set specification
for applicability and details.
15h I/O Command Set Combination Rejected: This error indicates that the controller did not accept the
request to select the requested I/O Command Set Combination.
37h Invalid Controller Data Queue: This error indicates that the specified Controller Data Queue Identifier
is invalid for the controller processing the command.
 Track Receive command
The Track Receive command is used to obtain the information being tracked by the controller that was
enabled by a Track Send command (refer to section 5.1.27).
The Track Receive command uses the Data Pointer field, Command Dword 10 field, and Command Dword
12 field. The use of the Command Dword 11 field is specific to the management operation specified by the
Select field. All other command specific fields are reserved.
The Select field defined in Figure 457 specifies the management operation to be performed. Refer to section
5.1.26.1 for a description of each management operation.
Figure 457: Track Receive – Command Dword 10
Bits Description
31:08 Reserved
07:00
Select (SEL): This field specifies the type of management operation to perform.
Value M/O1 Management Operation Reference Section
0h O Tracked Memory Changes 5.1.26.1.1
1h to FFh  Reserved
Notes:
1. O/M definition: O = Optional, M = Mandatory


Figure 458: Track Receive – Command Dword 12
Bits Description
31:00
Number of Dwords (NUMDL):  This field specifies the number of dwords to return. If the host specifies a
size larger than defined by the specified management operation of the command, then the controller returns
the data specified by that management operation with undefined results for dwords beyond the end of the
data specified by that management operation. This is a 0’s based value.
