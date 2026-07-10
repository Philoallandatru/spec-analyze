---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 202
headings: []
---

# Source page 202

NVM Express® Base Specification, Revision 2.1

180
Figure 152: Asynchronous Event Information – I/O Command Specific Status
Value Definition
01h
Sanitize Operation Completed: Indicates that a sanitize operation has completed (including any
associated additional media modification, refer to the No-Deallocate Modifies Media After Sanitize
field in Figure 312) without unexpected deallocation of all media allocated for user data (refer to
section 5.1.25.1.15) and status is available in the Sanitize Status log page  (refer to section
5.1.12.1.33). To clear this event, the host reads the Sanitize Status log page with the Retain
Asynchronous Event bit cleared to ‘0’.
02h
Sanitize Operation Completed With Unexpected Deallocation:  Indicates that a sanitize
operation for which No-Deallocate After Sanitize (refer to Figure 372) was requested has completed
with the unexpected deallocation of all media allocated for user data (refer to section 5.1.25.1.15)
and status is available in the Sanitize Status log page (refer to section 5.1.12.1.33). To clear this
event, the host reads the Sanitize Status log page with the Retain Asynchronous Event bit cleared
to ‘0’.
03h
Sanitize Operation Entered Media Verification State: Indicates that sanitize processing was
successful, the sanitize operation entered the Media Verification state (refer to section 8.1.24), and
status is available in the Sanitize Status log page (refer to section 5.1.12.1.33).
04h to FFh Reserved

Figure 153: Asynchronous Event Information – Immediate
Value Description
00h
NVM Subsystem Normal Shutdown:  This controller has started performing a normal NVM
Subsystem Shutdown that is due to:
• the value 4E726D6Ch ("Nrml") has been written to the NSSD.NSSC field within the NVM
subsystem or Domain; or
• an NVMe -MI Shutdown command (refer to the NVM Express Management Interface
Specification) being processed.
Refer to section 3.6.3.
01h Temperature Threshold Hysteresis Recovery:  Indicates the end of a temperature threshold
hysteresis event (Refer to section 5.1.25.1.3.1).
02h to FFh Reserved

Figure 154: Asynchronous Event Information – One Shot
Value Definition
00h
Controller Data Queue Tail Pointer:  Indicates that an entry in a Controller Data Queue was
posted at the tail pointer location specified by the Controller Data Queue feature (refer to section
5.1.25.1.23). Figure 155 defines the Event Specific Parameter field in completion queue entry
Dword 1 field for this event.
01h
Controller Data Queue Full Error: The controller detected that a Controller Data Queue became
full. Figure 155 defines the Event Specific Parameter field in Completion Queue Entry Dword  1
for this event.
02h to FFh Reserved

Figure 155: Controller Data Queue Tail Pointer – Event Specific Parameter
Bits Description
31:16 Reserved
15:00 Controller Data Queue Identifier (CDQID): This field indicates the identifier for the Controller
Data Queue associated with this event (refer to Figure 168).
