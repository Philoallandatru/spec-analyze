---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 164
headings: []
---

# Source page 164

NVM Express® Base Specification, Revision 2.1

142
Figure 102: Status Code – Generic Command Status Values
Value Description I/O Command
Set(s)1
0Ch
Command Sequence Error: The command was aborted due to a protocol violation in
a multi-command sequence (e.g., a violation of the Security Send and Security Receive
sequencing rules in the TCG Storage Synchronous Interface Communications protocol
(refer to TCG Storage Architecture Core Specification)).

0Dh
Invalid SGL Segment Descriptor:  The command includes an invalid SGL Last
Segment or SGL Segment descriptor. This may occur under various conditions,
including:
a) the SGL segment pointed to by an SGL Last Segment descriptor contains an
SGL Segment descriptor or an SGL Last Segment descriptor;
b) an SGL Last Segment descriptor contains an invalid length (i.e., a length of
0h or 1h that is not a multiple of 16); or
c) an SGL Segment descriptor or an SGL Last Segment descriptor contains an
invalid address (e.g., an address that is not qword aligned).

0Eh
Invalid Number of SGL Descriptors: There is an SGL Last Segment descriptor or an
SGL Segment descriptor in a location other than the last descriptor of a segment based
on the length indicated. This is also used for invalid SGLs in a command capsule.

0Fh
Data SGL Length Invalid: This may occur if the length of a data SGL is too short. This
may occur if the length of a data SGL is too long and the controller does not support
SGL transfers longer than the amount of data to be transferred as indicated in the SGL
Support field of the Identify Controller data structure.

10h
Metadata SGL Length Invalid: This may occur if the length of a metadata SGL is too
short. This may occur if the length of a metadata SGL is too long and the controller
does not support SGL transfers longer than the amount of data to be transferred as
indicated in the SGL Support field of the Identify Controller data structure.

11h
SGL Descriptor Type Invalid:  The type of an SGL Descriptor is a type that is not
supported by the controller, or the combination of type and subtype is not supported
by the controller.

12h Invalid Use of Controller Memory Buffer: The attempted use of the Controller
Memory Buffer is not supported by the controller. Refer to section 8.2.1.
13h
PRP Offset Invalid: The Offset field for a PRP entry is invalid. This may occur when
there is a PRP entry with a non-zero offset after the first entry or when the Offset field
in any PRP entry is not dword aligned (i.e., bits 1:0 are not cleared to 00b).

14h Atomic Write Unit Exceeded: See the applicable I/O Command Set specification for
the description. NVM, ZNS
15h
Operation Denied: The command was denied due to lack of access rights. Refer to
the appropriate security specification (e.g., TCG Storage Interface Interactions
specification). For media access commands, the Access Denied status code should
be used instead.

16h
SGL Offset Invalid: The offset specified in an SGL descriptor is invalid. This may
occur when using capsules for data transfers in NVMe over Fabrics implementations
and an invalid offset in the capsule is specified.

17h Reserved
18h Host Identifier Inconsistent Format: The NVM subsystem detected the simultaneous
use of 64-bit and 128-bit Host Identifier values on different controllers.
19h Keep Alive Timer Expired: The Keep Alive Timer expired.
1Ah
Keep Alive Timeout Invalid: The Keep Alive Timeout value specified is invalid. This
may be due to an attempt to specify a value of 0h on a transport that requires the Keep
Alive Timer feature to be enabled. This may be due to the value specified being too
large for the associated NVM e Transport as defined in the NVMe Transport binding
specification.

1Bh
Command Aborted due to Preempt and Abort:  The command was aborted due to
a Reservation Acquire command with the Reservation Acquire Action (RACQA) set to
010b (Preempt and Abort).

1Ch Sanitize Failed: The most recent sanitize operation failed and no recovery action has
been successfully completed.
