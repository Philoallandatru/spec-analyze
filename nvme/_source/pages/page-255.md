---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 255
headings: []
---

# Source page 255

NVM Express® Base Specification, Revision 2.1

233
Figure 225: Persistent Event Log Specific Parameter Field
Bits Description
14:10 Reserved
09:08
Action (ACT): This field specifies the action the controller shall take during processing this Get
Log Page command.
Value Definition
00b
Read Log Data:  Return Persistent Event log page data starting at the address
indicated by the LPOU field and the LPOL field in the Get Log Page command. If the
controller does not have a persistent event log reporting context, then the controller
shall abort the command with a status code of Command Sequence Error.
01b
Establish Context and Read Log Data: The controller shall:
a) determine the length of the Persistent Event log page data;
b) determine the set of events to report in the Persistent Event log page data;
and
c) establish a persistent event log reporting context to store information
describing the Persistent Event log  page data to be reported and track
Persistent Event log page data accesses.
After establishing a persistent event log reporting context , the controller shall return
Persistent Event log page data starting at the address indicated by the LPOU field
and the LPOL field in the Get Log Page command.
If a persistent event log reporting context already exists, then the controller shall abort
the command with a status code of Command Sequence Error.
10b
Release Context: The controller shall:
• release the persistent event log reporting context, if any. It is not an error if
the controller does not have a persistent event log reporting context; and
• not return any Persistent Event log page data (i.e., the controller ignores the
NUMDU field, NUMDL field, LPOL field, and the LPOU field).
11b
Establish Context and Read 512 Bytes of Header: The controller shall:
a) determine the length of the Persistent Event log page data;
b) determine the set of events to report in the Persistent Event log page data;
and
c) if a reporting context does not already exist, then establish a persistent
event log reporting context to store information describing the Persistent
Event log page data to be reported and track Persistent Event log page data
accesses.
If a persistent event log reporting context did not already exist when the Get Log Page
command was processed, then the controller shall:
a) establish a persistent event log reporting context; and
b) after establishing the context, return 512 bytes of the Persistent Event Log
Header starting at offset 0h with the Reporting Context Exists bit cleared to
‘0’ with a status code of Successful Completion.
If a persistent event log reporting context already existed when the Get Log Page
command was processed, then the controller shall return 512 bytes of the Persistent
Event Log Header starting at offset 0h with the Reporting Context Exists bit set to ‘1’
with a status code of Successful Completion.
The 512 bytes of the Persistent Event Log Header shall be returned regardless of the
values in the LPOL, LPOU, NUMDL, and NUMDU fields (i.e., the controller shall
ignore the LPOL, LPOU, NUMDL, and NUMDU fields).  Hosts should ensure
allocation of at least 512 bytes in the data buffer to avoid possible data corruption.


The log page returned is defined in Figure 226.
