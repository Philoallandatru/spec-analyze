---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 415
headings: ["5.1.25.1.23 Controller Data Queue (Feature Identifier 21h)"]
---

# Source page 415

NVM Express® Base Specification, Revision 2.1

393
Figure 423: Namespace Admin Label – Data Structure
Bytes Description
255:0
Namespace Admin Label (NSAL): This field contains the Namespace Admin Label for the namespace
as a null-terminated UTF-8 string. A Namespace Admin Label is intended to assist a human administrator
in identifying a namespace (e.g., based on the contents of the data stored in the namespace (e.g., “Q4
2018 Financial Records”) or the application intended to use the namespace (e.g., “CRM data”)).
5.1.25.1.23 Controller Data Queue (Feature Identifier 21h)
This Feature allows a host to update the status of the head pointer of a Controller Data Queue (CDQ) and
specify the configuration of a Controller Data Queue Tail event. The CDQ is specified by the Controller
Data Queue Identifier (CDQID) field in Command Dword 11 (refer to Figure 424).
The Head Pointer field specifies the current slot of the head pointer in the queue (refer to section 8.1.6).
The controller uses this value to determine if CDQ entries have been freed by the host.
If the Enable Tail Pointer Trigger (ETPT) bit is set to ‘1’, then when the slot specified by the Tail Pointer
Trigger (TPT) field for the specified CDQ is posted with an entry, the controller shall generate a Controller
Data Queue Tail Pointer event to the host (refer to section 8.1.6).
If the Set Features command is successful and there is a pending Controller Data Queue Tail Pointer event
for the specified Controller Data Queue specified by the CDQID field, then the controller shall clear that
pending event.
A controller should report Controller Data Queue Tail Pointer events in the order of occurrence to avoid
reporting the same Controller Data Queue when a Controller Data Queue Tail Pointer event for that
Controller Data Queue is being repeatedly triggered.
If the Enable Tail Pointer Trigger (ETPT) bit is set to ‘1’ in the current value of this Feature (refer to section
4.4) and the controller processes a Set Features command for this Feature that specifies:
• the Enable Tail Pointer Trigger (ETPT) bit set to ‘1’; and
• a slot in the Tail Pointer Trigger (TPT) field,
then prior to posting the completion for that Set Features command, the controller may post an entry into
the specified CDQ in the slot specified by the value of the TPT field  that had been set prior to processing
that Set Features command . To detect this condition, after receiving the completion of that Set Features
command, a host should examine the CDQ to determine if the requested tail pointer trigger has already
occurred. If the requested tail pointer trigger has occurred, then a subse quent Set Feature s command
should be submitted by the host to disable the Tail Pointer Trigger event or request a different tail pointer
trigger.
If a Get Features command is submitted for this Feature, the attributes described in Figure 424 are returned
in Dword 0 and the attributes described in Figure 426 are returned in the data buffer for that command.
Figure 424: Controller Data Queue – Command Dword 11
Bits Description
31
Enable Tail Pointer Trigger (ETPT): If this bit is set to ‘1’, then the controller is to generate a Controller
Data Queue Tail Pointer event when that controller posts the entry into the slot specified by the TPT field
for the specified CDQ. If this bit is cleared to ‘0’, then there is no request for a Controller Data Queue Tail
Pointer event to be sent by the controller.
If a Controller Data Queue Tail Pointer event is generated for the specified Controller Data Queue, then
the current value of this bit for this Feature shall be cleared to ’0’ (i.e., the current value for this bit only
enables a single occurrence of a Controller Data  Queue Tail Pointer event for the specified Controller
Data Queue).
For a Get Features command, this field shall be ignored by the controller.
30:16 Reserved
15:00 Controller Data Queue Identifier (CDQID): This field contains the identifier associated with the CDQ.
