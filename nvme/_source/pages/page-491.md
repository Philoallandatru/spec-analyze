---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 491
headings: []
---

# Source page 491

NVM Express® Base Specification, Revision 2.1

469
The command uses Command Dword 10 and a Reservation Register data structure in memory  (refer to
Figure 574). If the command uses PRPs for the data transfer, then PRP Entry 1 and PRP Entry 2 fields are
used. If the command uses SGLs for the data transfer, then the SGL Entry 1 field is used. All other command
specific fields are reserved.
Figure 572: Reservation Register – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the location of a data buffer where data is transferred from .
Refer to Figure 92 for the definition of this field.

Figure 573: Reservation Register – Command Dword 10
Bits Description
31:30
Change Persist Through Power Loss State (CPTPL): This field allows the Persist Through Power Loss
(PTPL) state associated with the namespace to be modified as a side effect of processing this command. If
the Reservation Persistence feature (refer to section 5.1.25.1.30) is saveable, then any change to the PTPL
state as a result of processing this command shall be applied to both the current value and the saved value
of that feature.
Value Definition
00b No change to PTPL state
01b Reserved
10b Clear PTPL state to ‘0’. Reservations are released and registrants are cleared on a
power on.
11b Set PTPL state to ‘1’. Reservations and registrants persist across a power loss.

29:05 Reserved
04
Dispersed Namespace Reservation Support (DISNSRS): This bit specifies host support for reservations
on dispersed namespaces for a host that supports dispersed namespaces (i.e., for a host that has set the
Host Dispersed Namespace Support (HDISNS) field to 1h in the Host Behavior Support feature).
If this bit is set to ‘1’, then the host supports reservations on dispersed namespaces (i.e., the host supports
receiving a value of FFFDh in the Controller ID (CNTLID) field of a Registered Controller data structure or
Registered Controller Extended data structure (refer to section 8.1.9.6)).
If this bit is cleared to ‘0’, then the host does not support reservations on dispersed namespaces (i.e., the
host does not support receiving a value of FFFDh in the Controller ID (CNTLID) field of a Registered
Controller data structure or Registered Contr oller Extended data structure (refer to section 8.1.9.6)). If the
HDISNS field is set to 1h in the Host Behavior Support feature and the host submits the command to a
dispersed namespace with this bit cleared to ‘0’, then the controller aborts the command with a status code
of Namespace Is Dispersed as described in section 8.1.9.6.
03
Ignore Existing Key (IEKEY) : If this bit is set to a ‘1’, then Reservation Register Action (RREGA) field
values that use the Current Reservation Key (CRKEY) shall succeed regardless of the value of the Current
Reservation Key field in the command (i.e., the current reservation key, if any, is not checked, and absence
of a current reservation key does not cause an error).
02:00
Reservation Register Action (RREGA): This field specifies the registration action that is performed by the
command.
Value Definition Reference
000b Register Reservation Key 8.1.22.3
001b Unregister Reservation Key 8.1.22.4
010b Replace Reservation Key 8.1.22.3
011b to 111b Reserved
