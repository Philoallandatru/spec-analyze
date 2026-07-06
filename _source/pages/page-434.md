---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 434
headings: ["5.1.26.1 Track Receive Management Operations", "5.1.26.1.1 Tracked Memory Changes (Management Operation 0h)"]
---

# Source page 434

NVM Express® Base Specification, Revision 2.1

412
 Track Receive Management Operations
5.1.26.1.1 Tracked Memory Changes (Management Operation 0h)
The Tracked Memory Changes management operation of the Track Receive command is used to report
host memory changes by the controller specified by the Controller Identifier (CNTLID) field (refer to Figure
459) processing any and all commands (e.g., Admin commands, I/O command) as a result of tracking host
memory modifications being started by a Tracked Memory Changes management operation of the Track
Send command (refer to section 5.1.27.1.2).
The data returned is a Tracked Memory Change data structure defined by Figure 460. The identifier for the
controller that is tracked is specified in the Controller Identifier field in the Command Dword 11 field as
defined in Figure 459.
Figure 459: Tracked Memory Changes – Command Dword 11
Bits Description
31:16 Reserved
15:00 Controller Identifier (CNTLID ): This field specifies the identifier of the controller for which the tracked
memory changes, if any, are to be returned.
If the Track Receive command for this management operation completes successfully, then the controller
shall remove the tracked memory changes that were reported in the returned data. The reported memory
ranges identify the host memory that has changed due  to the specified controller processing commands
since:
• Track Memory Changes was enabled (refer to 5.1.27.1.2) and no tracked memory changes have
been reported by the controller; or
• the controller last reported tracking change while the memory tracking remained enabled.
If tracked memory changes occur for the same memory address at a high frequency, the controller reporting
of the Tracking Memory Changed Descriptors for that address may be delayed by a vendor specific amount
of time so that Tracking Memory Changed Descrip tors for other addresses are able to be reported during
that vendor specific amount of time.
If the Suspended bit is set to ‘1’ and the More To Report (MTR) bit is cleared to ‘0’ in the returned Track
Memory Changed data structure, then all host memory changes have been reported to the host. If the
controller specified by the CNTLID field remains suspended, then the controller has no more tracked host
memory changes to report to the host until the controller specified by the CNTLID field is resumed.
If the Suspended bit is cleared to ‘0’ and the More To Report (MTR) bit is cleared to ‘0’ in the returned Track
Memory Changed data structure, then no additional tracked host memory changes are available to report
to the host in a subsequent commands at the time of processing this command.
Figure 460: Tracked Memory Change Data Structure
Bytes Description
Header
00
Version (VER): This field indicates the version of this data structure.
This field shall be cleared to 0h.
