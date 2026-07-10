---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 161
headings: ["4.2.2 Fabrics Command Common CQE", "4.2.3 Status Field Definition"]
---

# Source page 161

NVM Express® Base Specification, Revision 2.1

139
Figure 97: Completion Queue Entry: DW 2
Bits Description
15:00
SQ Head Pointer (SQHD): Indicates the current Submission Queue Head pointer for the Submission
Queue indicated in the SQ Identifier field. This is used to indicate to the host the submission queue entries
that have been consumed and may be re-used for new entries.
Note: The value returned is the value of the SQ Head pointer when the completion queue entry was
created. By the time host software consumes the completion queue entry, the controller may have an SQ
Head pointer that has advanced beyond the value indicated.

Figure 98: Completion Queue Entry: DW 3
Bits Description
31:17 Status (STATUS): Indicates the status for the command that is being completed. Refer to section 4.2.3.
16
Phase Tag (P): Identifies whether a completion queue entry is new. Refer to section 4.2.4.
This is a reserved bit in NVMe over Fabrics implementations.
15:00
Command Identifier (CID): Indicates the identifier of the command that is being completed. This identifier
is assigned by host software when the command is submitted to the Submission Queue. The combination
of the SQ Identifier and Command Identifier uniquely identifies the command  that is being completed. The
maximum number of requests outstanding for a Submission Queue at one time is 65,535.
 Fabrics Command Common CQE
The common completion queue entry for Fabrics commands is shown in Figure 99.
Figure 99: Fabrics Response – Completion Queue Entry Format
Bytes Description
07:00 Fabrics Response Type Specific (FRTS):  The definition of this field is Fabrics response type
specific.
09:08
SQ Head Pointer (SQHD): This field indicates the current Submission Queue Head pointer for the
associated Submission Queue. This field is reserved if SQ flow control is disabled for the queue pair
(refer to section 6.3).
11:10 Reserved
13:12 Command Identifier (CID): This field indicates the identifier of the command that is being completed.
15:14
Status Info (STS): This field indicates the status for the associated Fabrics command.
Bits Description
15:01 Status (STATUS): This field as defined in section 4.2.3.
00 Reserved

 Status Field Definition
The Status field defines the status for the command indicated in the completion queue entry, defined in
Figure 100.
A value of 0h for the Status field indicates a successful command completion, with no fatal or non-fatal error
conditions. Unless otherwise noted, if a command fails to complete successfully for multiple reasons, then
the particular status code returned is chosen by the vendor.
