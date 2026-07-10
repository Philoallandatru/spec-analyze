---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 416
headings: ["5.1.25.1.24 Embedded Management Controller Address (Feature Identifier 78h)"]
---

# Source page 416

NVM Express® Base Specification, Revision 2.1

394

Figure 425: Controller Data Queue – Command Dword 12
Bits Description
31:00 Head Pointer (HP): This field specifies the slot of the head pointer for the specified CDQ.

Figure 426: Controller Data Queue – Command Dword 13
Bits Description
31:00
Tail Pointer Trigger (TPT): If the ETPT bit is set to ‘1’, then this field specifies a slot in the CDQ that when
posted with an entry causes the controller to issue a Controller Data Queue Tail Pointer event.
For a Set Features command, if the ETPT bit is cleared to ‘0’, then this field shall be ignored by the controller.
For a Get Features command, this field shall be ignored by the controller.
If the CDQ is empty (refer to section 3.3.1.4) and the Head Pointer field specifies a value that is not the
same value as the current value, then the controller shall abort the command with a status code of Invalid
Field in Command.
If the CDQ is not empty and the Head  Pointer field specifies a slot not associated with an entry that was
posted by the controller within the specified CDQ, then the controller shall abort the command with a status
code of Invalid Field in Command.
If the Enable Tail Pointer Trigger bit is set to ‘1’ and the Tail Pointer Trigger field specifies a slot not
associated with an entry within the specified CDQ, then the controller shall abort the command with a
status code of Invalid Field in Command.
If the value in the Controller Data Queue Identifier field specifies a CDQ that does not exist in the controller
processing the command, then the controller shall abort the command with a status code of Invalid
Controller Data Queue.
Figure 427: Controller Data Queue – Data Structure
Bytes Description
03:00 Head Pointer (HP): This field indicates the slot of the head pointer for the specified CDQ.
07:04
Tail Pointer Trigger (TPT):  If the ETPT bit is set to ‘1’, then this field indicates the slot in the CDQ that
when posted with an entry causes the controller to issue a Controller Data Queue Tail Pointer event.
If the ETPT bit is cleared to ‘0’, then this field shall be cleared to 0h and should be ignored by the host.
511:08 Reserved
5.1.25.1.24 Embedded Management Controller Address (Feature Identifier 78h)
This Feature configures a URI (refer to RFC 3986) containing the address of a management agent provided
by the system (e.g., a Management Controller (refer to the NVM Express Management Interface
Specification) or an enclosure manager) for management of the NVM subsystem.
If a Set Features command is issued for this Feature, the data structure specified in Figure 428 is transferred
in the data buffer for that command.
If a Get Features command is issued for this Feature, the data structure specified in Figure 428 is returned
in the data buffer for that command.
Figure 428: Set Features – Command Specific Status Values
Bytes Description
03:00 Reserved
511:04 Embedded Management Controller Address (EMCA): A URI (refer to RFC 3986), containing the
address of a management agent provided by the system.
