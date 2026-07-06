---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 443
headings: ["5.2.2.1 Command Completion"]
---

# Source page 443

NVM Express® Base Specification, Revision 2.1

421
Figure 479: Create I/O Submission Queue – Command Dword 11
Bits Description
31:16
Completion Queue Identifier (CQID): This field specifies the identifier of the I/O Completion Queue
to utilize for any command completions entries associated with this Submission Queue.
If the value specified:
a) is 0h (i.e., the Admin Completion Queue), then the controller shall abort the command with
a status code of Invalid Queue Identifier;
b) is outside the range supported by the controller, then the controller shall abort the command
with a status code of Invalid Queue Identifier; or
c) is within the range supported by the controller and does not specify an I/O Completion Queue
that has been created, then the controller shall abort the command with a status code of
Completion Queue Invalid.
15:03 Reserved
02:01
Queue Priority (QPRIO): This field specifies the priority class to use for commands within this
Submission Queue. This field is only used when the weighted round robin with urgent priority class is
the arbitration mechanism selected. This field shall be ignored by the controller if weighted round robin
with urgent priority class is not used. Refer to section 3.4.4.
Value Definition
00b Urgent
01b High
10b Medium
11b Low

00
Physically Contiguous (PC): If this bit is set to ‘1’, then the Submission Queue is physically
contiguous and PRP Entry 1 (PRP1) is the address of a contiguous physical buffer. If this bit is cleared
to ‘0’, then the Submission Queue is not physically contiguous and PRP Entry 1 (PRP1) is a PRP List
pointer. If this bit is cleared to ‘0’ and CAP.CQR is set to ‘1’, then the controller should return an error
of Invalid Field in Command.
If the:
• queue is located in the Controller Memory Buffer;
• PC is cleared to ‘0’; and
• CMBLOC.CQPDS is cleared to ‘0’,
then the controller shall abort the command with Invalid Use of Controller Memory Buffer status.

Figure 480: Create I/O Submission Queue – Command Dword 12
Bits Description
31:16 Reserved
15:00
NVM Set Identifier (NVMSETID): This field indicates the identifier of the NVM Set to be associated with
this Submission Queue.
If this field is cleared to 0h or the SQ Associations capability is not supported (refer to section 8.1.25),
then this Submission Queue is not associated with any specific NVM Set.
If this field is set to a non -zero value that is not specified in the NVM Set List (refer to Figure 317) and
the SQ Associations capability is supported (refer to section 8.1.25), then the controller shall abort the
command with a status code of Invalid Field in Command.
The host should not submit commands for namespaces associated with other NVM Sets in this
Submission Queue (refer to section 8.1.25).
 Command Completion
Upon completion of the Create I/O Submission Queue command, the controller posts a completion queue
entry to the Admin Completion Queue.
Create I/O Submission Queue command specific status values are defined in Figure 481.
