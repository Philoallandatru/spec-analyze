---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 492
headings: ["7.6.1 Command Completion", "7.7 Reservation Release command"]
---

# Source page 492

NVM Express® Base Specification, Revision 2.1

470
Figure 574: Reservation Register Data Structure
Bytes Description
07:00
Current Reservation Key (CRKEY):  If the Reservation Register Action is 001b (i.e., Unregister
Reservation Key) or 010b (i.e., Replace Reservation Key), then this field contains the current
reservation key associated with the host. For all other Reservation Register Action values, this field
is reserved.
The controller ignores the value of this field when the Ignore Existing Key (IEKEY) bit is set to ‘1’.
15:08
New Reservation Key (NRKEY):  If the Reservation Register Action  field is cleared to 000b (i.e.,
Register Reservation Key) or 010b (i.e., Replace Reservation Key), then this field contains the new
reservation key associated with the host. For all other Reservation Register Action values, this field
is reserved.
 Command Completion
When the command is completed, the controller shall post a completion queue entry to the associated I/O
Completion Queue indicating the status for the command.
7.7 Reservation Release command
The Reservation Release command is used to release or clear a reservation held on a namespace.
The command uses Command Dword 10 and a Reservation Release data structure in memory. If the
command uses PRPs for the data transfer, then PRP Entry 1 and PRP Entry 2 fields are used. If the
command uses SGLs for the data transfer, then the SGL Entry 1 fie ld is used. All other command specific
fields are reserved.
Figure 575: Reservation Release – Data Pointer
Bits Description
127:00 Data Pointer (DPTR):  This field specifies the location of a data buffer where data is transferred from .
Refer to Figure 92 for the definition of this field.

Figure 576: Reservation Release – Command Dword 10
Bits Description
31:16 Reserved
15:08
Reservation Type (RTYPE): If the Reservation Release Action  field is cleared to 000b (i.e., Release),
then this field specifies the type of reservation that is being released. The reservation type in  this field
shall match the current reservation type . If the reservation type in this field does not match the current
reservation type, then the controller should return a status code of Invalid Field in Command. This field
is defined in Figure 571.
07:05 Reserved
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
03 Ignore Existing Key (IEKEY): If this bit is set to a ‘1’, then the controller shall return an error of Invalid
Field in Command. If this bit is cleared to ‘0’, then the Current Reservation Key is checked.
