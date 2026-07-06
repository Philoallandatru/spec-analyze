---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 584
headings: ["8.1.22.2 Reservation Notifications"]
---

# Source page 584

NVM Express® Base Specification, Revision 2.1

562
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
Write Command Group
Capacity Management (Admin)
Flush
Format NVM (Admin)
Namespace Attachment (Admin)
Namespace Management (Admin)
Sanitize (Admin)
Security Send (Admin)
I/O Command Set specific Copy Commands
(destination)2,3
I/O Command Set specific Write Commands2
C C C C C A C A
Reservation Command Groups
Reservation Acquire - Acquire C C C C C C C C
Reservation Acquire - Preempt
Reservation Acquire - Preempt and Abort
Reservation Release
C A C A C A C A
All Other Commands Group
All other commands1 A A A A A A A A
Key:
A definition: A=Allowed, command processed normally by the controller
C definition: C=Conflict, command aborted by the controller with a status code of Reservation Conflict
Notes:
1. The behavior of a vendor specific command is vendor specific.
2. Refer to the applicable I/O Command Set specification
3. For an I/O Command Set specific Copy command, each source namespace is checked for reservation conflict as if accessed
by a read command and the destination namespace is checked for reservation conflict as if accessed by a write command,
as described in the applicable I/O command Set specification.
 Reservation Notifications
There are three types of reservation notifications: registration preempted, reservation released, and
reservation preempted. Conditions that cause a reservation notification to occur are described in the
following sections. A Reservation Notification log p age is created whenever an unmasked reservation
notification occurs on a namespace associated with the controller (refer to section 5.1.12.1.32). Reservation
notifications may be masked from generating a Reservation Notification log page on a per reservation
notification type and per namespace ID basis through the Reservation Notification Mask feature (refer to
section 5.1.25.1.29). A host may use the Asynchronous Event Request command (refer to section 5.1.2)
to be notified of the presence of one or more available Reservation Notification log pages (refer to section
5.1.12.1.32).
