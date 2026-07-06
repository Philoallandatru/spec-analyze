---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 478
headings: ["6.4 Disconnect Command and Response"]
---

# Source page 478

NVM Express® Base Specification, Revision 2.1

456
Figure 548: Connect Response – Dword 0 Value Based on Status Code
Status Code  Definition of Dword 0
Successful
Completion
Bytes Description
01:00
Controller ID (CNTLID): Specifies the controller ID allocated to the host. If a
particular controller was specified in the CNTLID field of the Connect command,
then this field shall contain the same value.
03:02
Authentication and Security Requirements (AUTHREQ): Specifies the NVMe
in-band authentication and security requirements. The field is bit significant. If all
bits are cleared to ‘0’, then no requirements are specified.
Bits Description
15:03 Reserved
02
Authentication and Secure Channel Required (ASCR): If this
bit is set to ‘1’, then authentication using NVMe over Fabrics
Authentication protocols followed by secure channel
establishment is re quired and the ATR bit should be cleared to
‘0’. If this bit is cleared to ‘0’, then authentication using NVMe over
Fabrics Authentication protocols followed by secure channel
establishment is not required.
01
Authentication Transaction Required (ATR): If this bit is set to
‘1’, then authentication using NVMe over Fabrics  Authentication
protocols is required. If this bit is cleared to ‘0’, then authentication
using NVMe over Fabrics Authentication protocols is not required.
00 Obsolete.


Connect
Invalid
Parameters
Bytes Description
01:00
Invalid Parameter Offset (IPO): If an invalid parameter is reported, then this
field specifies the offset in bytes to the invalid parameter from the start of the
SQE or the data.
02
Invalid Attributes (IATTR): Specifies attributes of the invalid field parameter.
Bits Description
7:1 Reserved
0
Invalid Parameter Start (IPS):  If this bit is cleared to ‘0’, then the
invalid parameter is specified from the start of the SQE. If this bit is
set to ‘1’, then the invalid parameter is specified from the start of the
data.

03 Reserved

All Other
Status Values
Bytes Description
03:00 Reserved

6.4 Disconnect Command and Response
The Disconnect command is used to delete the I/O Queue on which the command is submitted. If a
Disconnect command is submitted on an Admin Queue, then the controller shall abort the command with a
status code of Invalid Queue Type. If the controller is not able to delete the I/O Queue, then the controller
shall abort the command with a status code of Controller Busy. The fields for the submission queue entry
are defined in Figure 549.
The NVMe Transport connection is not deleted upon issuance of a Disconnect command; the host and
controller may delete the NVMe Transport connection and associated resources after all NVMe Queues
(I/O Queues and/or Admin Queue) associated with that NVMe Tr ansport connection have been deleted
(refer to section 3.3.2.4).
The completion queue entry for the Disconnect command shall be the last entry submitted to the I/O Queue
Completion queue by the controller (i.e., no completion queue entries shall be submitted to the I/O Queue
Completion Queue after the completion queue entry for the Disconnect command). The controller shall
ensure that no further command processing is performed for any command on an I/O queue after sending
the completion queue entry for the Disconnect command.
