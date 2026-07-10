---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 198
headings: []
---

# Source page 198

NVM Express® Base Specification, Revision 2.1

176
Figure 148: Asynchronous Event Request – Completion Queue Entry Dword 1
Bits Description
31:00 Event Specific Parameter (EVNTSP): This field specifies information for a specific event. If this field is
not defined by a specific event, then this field is reserved for that specific event.

Figure 149: Asynchronous Event Information – Error Status
Value Definition
00h Write to Invalid Doorbell Register : Host software wrote the doorbell of a queue that was not
created.
01h
Invalid Doorbell Write Value:  Host software attempted to write an invalid doorbell value. Some
possible causes of this error are:
• the value written was out of range of the corresponding queue’s base address and size;
• the value written is the same as the previously written doorbell value;
• the number of commands that would be added as part of a doorbell write would exceed
the number of available entries;
• host software attempts to add a command to a full Submission Queue; and
• host software attempts to remove a completion queue entry  from an empty Completion
Queue.
02h Diagnostic Failure: A diagnostic failure was detected. This may include a self-test operation.
03h
Persistent Internal Error: A failure occurred that is persistent and the controller is unable to isolate
to a specific set of commands. If this error is indicated, then the CSTS.CFS bit may be set to ‘1’ and
the host should perform a reset as described in section3.7.
04h Transient Internal Error: A transient error occurred that is specific to a particular set of commands;
controller operation may continue without a reset.
05h Firmware Image Load Error: The firmware image could not be loaded. The controller reverted to
the previously active firmware image or a baseline read-only firmware image.
06h to FFh Reserved

Figure 150: Asynchronous Event Information – SMART / Health Status
Value Definition
00h
NVM subsystem Reliability: NVM subsystem reliability has been compromised. This may be due
to significant media errors, an internal error, the media being placed in read only mode, or a volatile
memory backup device failing. This status value shall not be used if the read-only condition on the
media is due to a change in the write protection state of a namespace (refer to section 8.1.16.1).
01h Temperature Threshold: A temperature is greater than or equal to an over temperature threshold
or less than or equal to an under temperature threshold (refer to section 5.1.25.1.3).
02h Spare Below Threshold: Available spare capacity has fallen below the threshold.
03h to FFh Reserved
