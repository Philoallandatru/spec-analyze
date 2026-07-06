---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 494
headings: []
---

# Source page 494

NVM Express® Base Specification, Revision 2.1

472
Figure 579: Reservation Report – Command Dword 10
Bits Description
31:00
Number of Dwords (NUMD): This field specifies the number of dwords of the Reservation Status data
structure to transfer. This is a 0’s based value.
If this field corresponds to a length that is less than the size of the Reservation Status data structure,
then only that specified portion of the data structure is transferred. If this field corresponds to a length
that is greater than the size of the Reservation Status data structure, then the entire contents of the data
structure are transferred and no additional data is transferred.

Figure 580: Reservation Report – Command Dword 11
Bits Description
31:02 Reserved
01
Dispersed Namespace Reservation Support (DISNSRS):  This bit specifies host support for
reservations on dispersed namespaces for a host that supports dispersed namespaces (i.e., for a host
that has set the Host Dispersed Namespace Support (HDISNS) field to 1h in the Host Behavior Support
feature).
If this bit is set to ‘1’, then the host supports reservations on dispersed namespaces (i.e., the host
supports receiving a value of FFFDh in the Controller ID (CNTLID) field of a Registrant data structure or
Registrant Extended data structure (refer to section 8.1.9.6)).
If this bit is cleared to ‘0’, then the host does not support reservations on dispersed namespaces (i.e.,
the host does not support receiving a value of FFFDh in the Controller ID (CNTLID) field of a Registrant
data structure or Registrant Extended data structure (refer to section 8.1.9.6)). If the HDISNS field is set
to 1h in the Host Behavior Support feature and the host submits the command to a dispersed namespace
with this bit cleared to ‘0’, then the controller aborts the command with a status code of Namespace Is
Dispersed as described in section 8.1.9.6.
00
Extended Data Structure (EDS): If this bit is set to ‘1’ , then the controller returns the extended data
structure defined in Figure 582. If this bit is cleared to ‘0’, then the controller returns the data structure
defined in Figure 581.

Figure 581: Reservation Status Data Structure
Bytes Description
Header
03:00
Generation (GEN): This field contains a 32 -bit wrapping counter that is incremented any time any
one the following occur:
• a Reservation Register command completes successfully on any controller associated with
the namespace;
• a Reservation Release command with Reservation Release Action (RRELA) set to 001b (i.e.,
Clear) completes successfully on any controller associated with the namespace; and
• a Reservation Acquire command with Reservation Acquire Action (RACQA) set to 001b
(Preempt) or 010b (Preempt and Abort) completes successfully on any controller associated
with the namespace.
If the value of this field is FFFFFFFFh, then the field shall be cleared to 0h when incremented (i.e.,
rolls over to 0h).
04
Reservation Type (RTYPE): This field indicates whether a reservation is held on the namespace. A
value of 0h indicates that no reservation is held on the namespace. A non -zero value indicates a
reservation is held on the namespace and the reservation type is defined in Figure 571.
06:05
Number of Registrants (REGSTRNT ): This field indicates the number of registrants of the
namespace. This indicates the number of Registrant data structures or Registrant E xtended data
structures contained in this data structure.
Note: This field was formerly named Number of Registered Controllers (REGCTL).
08:07 Reserved
