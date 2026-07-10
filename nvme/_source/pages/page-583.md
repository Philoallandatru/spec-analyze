---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 583
headings: []
---

# Source page 583

NVM Express® Base Specification, Revision 2.1

561
Figure 645: Command Behavior in the Presence of a Reservation
Reservation Type
Reservation
Holder Registrant Non-
Registrant Reservation Holder Definition
Read Write Read Write Read Write
Write Exclusive Y Y Y N Y N One Reservation Holder
Exclusive Access Y Y N N N N One Reservation Holder
Write Exclusive -
Registrants Only Y Y Y Y Y N One Reservation Holder
Exclusive Access -
Registrants Only Y Y Y Y N N One Reservation Holder
Write Exclusive - All
Registrants Y Y Y Y Y N All Registrants are Reservation
Holders
Exclusive Access - All
Registrants Y Y Y Y N N All Registrants are Reservation
Holders
The differences between these reservation types are: the type of access that is excluded (i.e., writes or all
accesses), whether registrants have the same access rights as the reservation holder, and whether
registrants are also considered to be reservation holders. These differences are summarized in Figure 645
and the specific behavior for each NVM Express command is shown in Figure 646.
Reservations and registrations persist across all Controller Level Resets and all NVM Subsystem Resets
except reset due to power loss. A reservation may be optionally configured to be retained across a reset
due to power loss using the Persist Through Powe r Loss State (PTPLS). A Persist Through Power Loss
State (PTPLS) is associated with each namespace that supports reservations and may be modified as a
side effect of a Reservation Register command (refer to section 7.6) or a Set Features command (refer to
section 5.1.25).
Figure 646: Command Behavior in the Presence of a Reservation
NVMe Command
Write
Exclusive
Reservation
Exclusive
Access
Reservation
Write Exclusive
Registrants Only
or
Write Exclusive
All Registrants
Reservation
Exclusive Access
Registrants Only
or
Exclusive Access
All Registrants
Reservation
Non-Registrant
Registrant
Non-Registrant
Registrant
Non-Registrant
Registrant
Non-Registrant
Registrant
Read Command Group
Security Receive (Admin)
I/O Command Set specific Copy Commands
(source)2,3
I/O Command Set specific Read Commands2
A A C C A A C A
