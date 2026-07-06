---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 475
headings: []
---

# Source page 475

NVM Express® Base Specification, Revision 2.1

453
b) the SQHD field is set to FFFFh in the response to that Connect command.
If SQ flow control is disabled, then the SQHD field is reserved in Fabrics response capsules for all command
completions on that queue pair after the response that completes the Connect command.
SQ flow control is enabled and shall be used for a created queue pair if:
a) the DISSQFC bit is cleared to ‘0’ in the Connect command that creates the queue pair; or
b) the SQHD field is not set to FFFFh in the response to that Connect command.
If SQ flow control is enabled, then the controller shall use the SQHD field in Fabrics response capsules for
all command completions on that queue pair, except for command completions that omit the SQHD value
due to use of the SQHD pointer update optimization described in section 3.3.2.7.
Figure 545: Connect Command – Submission Queue Entry
Bytes Description
03:00 Command Dword 0 (CDW0): Refer to Figure 95.
04 Fabrics Command Type (FCTYPE): Set to 01h to specify a Connect command.
23:05 Reserved
39:24 SGL Descriptor 1 (SGL1): Refer to Figure 94.
41:40
Record Format (RECFMT): Specifies the format of the Connect command capsule. The format of the
record specified in this definition shall be 0h. If the NVM subsystem does not support the value specified,
then a status code of Incompatible Format shall be returned.
43:42
Queue ID (QID): Specifies the Queue Identifier for the Admin Queue or I/O Queue to be created. The
identifier is used for both the Submission and Completion Queue. The identifier for the Admin Submission
Queue and Admin Completion Queue is 0h. The identifier for an I/O Submission and Completion Queue is
in the range 1 to 65,534.
If the value in this field specifies the Queue ID of a queue that already exists, then the controller shall abort
the command with a status code of Invalid Queue Identifier.
45:44
Submission Queue Size (SQSIZE): This field indicates the size of the Submission Queue to be created.
If the size is 0h or larger than the controller supports, then a status code of Connect Invalid Parameters
shall be returned. The maximum size of the Admin Submission Queue is specified in the Discovery Log
Page Entry for the NVM subsystem. Refer to Figure 294. This is a 0’s based value.
