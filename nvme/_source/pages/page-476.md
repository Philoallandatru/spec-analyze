---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 476
headings: []
---

# Source page 476

NVM Express® Base Specification, Revision 2.1

454
Figure 545: Connect Command – Submission Queue Entry
Bytes Description
46
Connect Attributes (CATTR): This field indicates attributes for the connection.
Bits Description
7:5 Reserved
4
Connecting Entity (CONNENT):  Indicates the type of entity performing the Connect
command. If this bit is set to ‘1’, then the entity performing the Connect command is a
Discovery controller. If this bit is cleared to ‘0’, then the entity performing the Connect
command is a host.
3
Individual I/O Queue Deletion Support (INDIVIOQDELS): Indicates support for deleting
individual I/O Queues. If this bit is set to ‘1’, then the host supports the deletion of individual
I/O Queues. If this bit is cleared to ‘0’, then the host does not support the deletion of
individual I/O Queues.
2
Disable SQ Flow Control (DISSQFC):  If this bit is set to ‘1’, then the host is requesting
that SQ flow control be disabled. If this bit is cleared to ‘0’, then SQ flow control shall not
be disabled.
1:0
Priority Class (PRIOCLASS): Indicates the priority class to use for commands within this
Submission Queue. This field is only used when the weighted round robin with urgent
priority class is the arbitration mechanism selected (refer to CC.AMS in Figure 41, the field
is ignored if weighted round robin with urgent priority class is not used. Refer to section
3.4.4. This field is only valid for I/O Queues and shall be cleared to 00b for Admin Queue
connections.
Value Definition
00b Urgent
01b High
10b Medium
11b Low


47 Reserved
51:48
Keep Alive Timeout (KATO): In the Connect command for the Admin Queue, this field has the same
definition as the Keep Alive Timeout  (KATO) field of the Keep Alive Timer  feature (refer to section
5.1.25.1.8). Upon successful completion of the Connect command , the controller shall set the KATO field
in the Keep Alive Timer feature. Refer to section 3.9.2 for a description of activating the Keep Alive Timer.
In the Connect command for an I/O Queue, this field is reserved.
53:52
NVM Set Identifier (NVMSETID):  This field indicates the identifier of the NVM Set to be associated with
this Submission Queue. This field is only valid for I/O Queues.
In the Connect command for an Admin Queue, the:
• host should clear this field to 0h;
• controller shall ignore this field; and
• Submission Queue is not associated with any NVM Set.
In the Connect command for an I/O queue, if the SQ Associations capability is not supported (refer to
section 8.1.25) or this field is cleared to 0h, then this Submission Queue is not associated with any specific
NVM Set.
If the SQ Associations capability is supported (refer to section 8.1.25) and this field contains a non -zero
value that is not indicated in the NVM Set List (refer to Figure 317), then the controller shall abort the
command with a status code of Invalid Field in Command.
The host should not submit commands for namespaces associated with other NVM Sets in this Submission
Queue (refer to section 8.1.25).
63:54 Reserved
