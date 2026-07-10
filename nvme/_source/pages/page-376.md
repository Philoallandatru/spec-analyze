---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 376
headings: []
---

# Source page 376

NVM Express® Base Specification, Revision 2.1

354
The Suspend management operation uses the Command Dword 11 field as defined in Figure 351 and does
not use the Management Operation Specific field.
Figure 351: Suspend – Command Dword 11
Bits Description
31
Delete User Data Migration Queue (DUDMQ): If this bit is set to ‘1’  and the command completes
successfully, then the User Data Migration Queue associated with the controller specified in the CNTLID
field, if any, is deleted (refer to the applicable I/O Command Set specification). If this bit is cleared to ‘0’,
then the User Data Migration Queue associated the controller specified in the CNTLID field is retained after
processing the command, if any.
30:24 Reserved
23:16
Suspend Type (STYPE): This field specifies the type of suspend.
Value Definition
0h
Suspend Notification: The specified controller is going to be suspended in the future with
a subsequent Migration Send command specifying a Suspend management operation and
a non-zero value in the Suspend Type field.
1h Suspend: Suspend the controller specified in the CNTLID field as defined in this section.
All Others Reserved

15:00 Controller Identifier (CNTLID):  This field specifies the identifier of the controller to which the Suspend
management operation is performed.
A Suspend Type field value of 0h (i.e., Suspend Notification) allows a host to notify a controller that the
controller specified by the Controller Identifier field is going to be requested to suspend with a subsequent
Migration Send command specifying a Su spend management operation and a non -zero value in the
Suspend Type field.
A Suspend Type field value of 1h (i.e., Suspend) specifies that the controller specified in the Controller
Identifier field (refer to  Figure 351) is to be suspended . That controller shall perform the following actions
and then stop initiating transport transactions:
1. Stop fetching commands until the specified controller is no longer suspended (refer to sectio n
8.1.12).
2. Complete the processing of all previously fetched commands, if any, except any outstanding
Asynchronous Event Request commands.
If:
• the command causes the specified controller to be suspended; and
• there is a User Data Migration Queue associated with that specified controller,
then:
• If the DUDMQ bit is set to ‘1’, then the controller processing the command before posting the
completion queue entry to the command shall delete that User Data Migration Queue; and
• after the specified controller completes the processing of all previously fetched commands, if any,
except any outstanding Asynchronous Event Request commands, the controller processing the
Migration Send command shall, at a vendor specific amount of time, post s an entry in that User
Data Migration Queue that indicates the specified controller is suspended as specified by the
applicable I/O Command Set specification.
It is not an error if a suspend operation occurs on a controller that is already suspended with the same
Suspend Type value.
If the result of this command causes the specified controller to be suspended, then the specified controller
remains suspended until:
• a Migration Send command specifying the Resume management operation (refer to section
5.1.17.1.2) on the specified controller is completed successfully; or
• a Controller Level Reset occurs on the controller processing this command.
