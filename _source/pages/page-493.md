---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 493
headings: ["7.7.1 Command Completion", "7.8 Reservation Report command"]
---

# Source page 493

NVM Express® Base Specification, Revision 2.1

471
Figure 576: Reservation Release – Command Dword 10
Bits Description
02:00
Reservation Release Action (RRELA): This field specifies the reservation action that is performed by
the command.
Value Definition Reference
000b Release 8.1.22.6
001b Clear 8.1.22.8
010b to 111b Reserved


Figure 577: Reservation Release Data Structure
Bytes O/M Description
7:0 M Current Reservation Key (CRKEY):  The field specifies the current reservation key
associated with the host.
 Command Completion
When the command is completed, the controller shall post a completion queue entry to the associated I/O
Completion Queue indicating the status for the command.
7.8 Reservation Report command
The Reservation Report command returns a Reservation Status data structure to memory that describes
the registration and reservation status of a namespace.
If the namespace is not a dispersed namespace, then t he size of the Reservation Status data structure is
a function of the number of registrants of the namespace (i.e., there is a Registrant data structure and/or
Registrant Extended data structure for each such registrant). If the namespace is a dispersed namespace
that is able to be accessed by controllers in multiple participating NVM subsystems, then the size of the
Reservation Status data structure is a function of the number of registrants of the namespace in the NVM
subsystem containing the controller processing the command and the number of registrants of the
namespace in each separate participating NVM subsystem.  The controller returns the data structure in
Figure 581 if the host has selected a 64 -bit Host Identifier and the data structure in Figure 582 if the host
has selected a 128-bit Host Identifier (refer to section 5.1.25.1.28).
For controllers compliant with NVM Express Base Specification, Revision 2.0 and earlier, registrants of the
namespace that are not associated with any controller in the NVM subsystem may or may not be reported
by this command.
If a 64-bit Host Identifier has been specified and the Extended Data Structure bit is set to ‘1’ in Command
Dword 11, then the controller shall abort the command with the status code of Host Identifier Inconsistent
Format. If a 128-bit Host Identifier has been specified and the Extended Data Structure bit is cleared to ‘0’
in Command Dword 11, then the controller shall abort the command with the status code of Host Identifier
Inconsistent Format.
The command uses Command Dword 10  and Command Dword 11 . If the command uses PRPs for the
data transfer, then PRP Entry 1 and PRP Entry 2 fields are used. If the command uses SGLs for the data
transfer, then the SGL Entry 1 field is used. All other command specific fields are reserved.
Figure 578: Reservation Report – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the location of a data buffer where data is transferred to. Refer
to Figure 92 for the definition of this field.
