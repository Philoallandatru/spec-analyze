---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 490
headings: ["7.5.1 Command Completion", "7.6 Reservation Register command"]
---

# Source page 490

NVM Express® Base Specification, Revision 2.1

468
Figure 569: Reservation Acquire – Command Dword 10
Bits Description
04
Dispersed Namespace Reservation Support (DISNSRS):  This bit specifies host support for
reservations on dispersed namespaces for a host that supports dispersed namespaces (i.e., for a host
that has set the Host Dispersed Namespace Support (HDISNS) field to 1h in the Host Behavior Support
feature).
If this bit is set to ‘1’, then the host supports reservations on dispersed namespaces (i.e., the host
supports receiving a value of FFFDh in the Controller ID (CNTLID) field of a Registered Controller data
structure or Registered Controller Extended data structure (refer to section 8.1.9.6)).
If this bit is cleared to ‘0’, then the host does not support reservations on dispersed namespaces (i.e.,
the host does not support receiving a value of FFFDh in the Controller ID (CNTLID) field of a Registered
Controller data structure or Registered Contr oller Extended data structure (refer to section 8.1.9.6)). If
the HDISNS field is set to 1h in the Host Behavior Support feature and the host submits the command
to a dispersed namespace with this bit cleared to ‘0’, then the controller aborts the command with a status
code of Namespace Is Dispersed as described in section 8.1.9.6.
03 Ignore Existing Key (IEKEY): If this bit is set to a ‘1’ , then the controller shall return an error of Invalid
Field in Command. If this bit is cleared to ‘0’, then the Current Reservation Key is checked.
02:00
Reservation Acquire Action (RACQA): This field specifies the action that is performed by the
command.
Value Definition Reference
000b Acquire 8.1.22.5
001b Preempt 8.1.22.7
010b Preempt and Abort 8.1.22.7
011b to 111b Reserved


Figure 570: Reservation Acquire Data Structure
Bytes Description
07:00 Current Reservation Key (CRKEY): The field specifies the current reservation key associated with
the host.
15:08
Preempt Reservation Key (PRKEY):  If the Reservation Acquire Action is set to 001b (i.e.,
Preempt) or 010b (i.e., Preempt and Abort), then this field specifies the reservation key to be
unregistered from the namespace. For all other Reservation Acquire Action values, this field is
reserved.


 Command Completion
When the command is completed, the controller shall post a completion queue entry to the associated I/O
Completion Queue indicating the status for the command.
7.6 Reservation Register command
The Reservation Register command is used to register, unregister, or replace a reservation key.
Figure 571: Reservation Type Encoding
Value Definition
0h Reserved
1h Write Exclusive Reservation
2h Exclusive Access Reservation
3h Write Exclusive - Registrants Only Reservation
4h Exclusive Access - Registrants Only Reservation
5h Write Exclusive - All Registrants Reservation
6h Exclusive Access - All Registrants Reservation
7h to FFh Reserved
