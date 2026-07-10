---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 438
headings: []
---

# Source page 438

NVM Express® Base Specification, Revision 2.1

416
• If the value in the Requested Memory Range Tracking Granularity (RMRTG) field  is less than the
value in the Minimum Memory Range Granularity (MINMRG) field in the Identify Controller data
structure (refer to Figure 312), then the controller shall abort the command with a status code of
Invalid Field in Command.
• If the sum of:
o the value in the Requested Number of Memory Range Tracking Descriptors (RNMRTD)
field; and
o the number of Memory Range Tracking Descriptors  currently being tracked by the
controller processing the command for other controllers in the NVM subsystem,
is greater than  the value in the Controller Maximum Memory Range Tracking Descriptors
(CMMRTD) field in the Identify Controller data structure (refer to Figure 312), then the controller
processing the command shall abort the command with a status code of Invalid Field in Command.
• If the sum of the value in the Requested Number of Memory Range Tracking Descriptors
(RNMRTD) field plus the number of Memory Range Tracking Descriptors  currently being tracked
by all controllers in the NVM subsystem is greater than the value in the NVM subsystem Maximum
Memory Range Tracking Descriptors (NMMRTD) field in the Identify Controller data structure (refer
to Figure 312), then the controller shall abort the command with a status code of Invalid Field in
Command.
• If any controller in the NVM subsystem is already tracking host memory changes for the controller
specified by the CNTLID field, then the controller processing the command shall abort the
command with a status code of Invalid Controller Identifier.
• If:
o the Memory Range Tracking Length Limit (MRTLL) bit in the Identify Controller data
structure is set to ‘1’; and
o the length as specified by the Requested Memory Range Tracking Granularity field and
any Length field in a Track Memory Changes data structure is not a value that is a power
of 2,
then the controller shall abort the command with a status code of Invalid Field in Command.
If the TACT bit is cleared to ‘0’, then the data buffer shall be ignored by the controller.
Figure 466: Track Memory Changes – Management Operation Specific Field
Bits Description
15:01 Reserved
00
Tracking Action (TACT): If this bit is set to ‘1’, then the data buffer specifies the host memory
ranges associated with the controller specified by the Controller Identifier field. Any writes to
those host memory ranges by the controller specified by the Controller Identifier field shall be
tracked by the controller processing the command (i.e., tracking starts).
If this bit is cleared to ‘0’, then controller processing the command shall stop tracking host memory
changes by the controller specified by the Controller Identifier field.

Figure 467: Track Memory Changes – Command Dword 11
Bits Description
31:16 Reserved
15:00
Controller Identifier (CNTLID): This field specifies the identifier of the controller for which changes to host
memory is being tracked. If the value of this field is the controller identifier of the controller processing the
command, then the controller processing the command shall abort the command with a status code of Invalid
Controller Identifier.

If the controller specified by the CNTLID field is suspended (refer to section 5.1.17.1.1), then the controller
shall abort the command with a status code of Controller Suspended.
