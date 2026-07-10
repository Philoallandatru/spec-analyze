---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 324
headings: []
---

# Source page 324

NVM Express® Base Specification, Revision 2.1

302
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
19
Flexible Data Placement Support (FDPS): If this bit is set to ‘1’, then the
controller supports the Flexible Data Placement capability (refer to section
8.1.10). If this bit is cleared to ‘0’, then the controller does not support the
Flexible Data Placement capability.
If Fixed Capacity Management is supported (i.e., the FCM bit is set to ‘1’),
then this bit shall be cleared to ‘0’.
18
Reservations and Host Identifier Interaction (RHII): This bit indicates
the reservations and Host Identifier interaction support for the controller.
Value Definition
0b Not Reported
1b The Host Identifier is required to be set to a non -zero value
for a host to use reservations (refer to section 8.1.22).

17
HMB Restrict Non-Operational Power State Access (HMBR): If this bit
is set to ‘1’, then the controller supports restricting HMB access in non -
operational power states as defined in section 5.1.25.2.4. If this bit is
cleared to ‘0’, then the controller does not support restricting HMB access
in non-operational power states as defined by section 5.1.25.2.4.
16
MDTS and Size Limits Exclude Metadata (MEM):  If this bit is set to ‘1’,
then:
• The controller reported MDTS values do not include interleaved
metadata.
• The controller reported VSL, WZSL, and WUSL values in the I/O
Command Set specific Identify Controller data structure (refer to
the NVM Command Set Specification) do not include interleaved
metadata.
If this bit is cleared to ‘0’, then:
• The controller reported MDTS values include interleaved
metadata.
• The controller reported VSL, WZSL, and WUSL values in the I/O
Command Set specific Identify Controller data structure include
interleaved metadata.
15
Extended LBA Formats Supported (ELBAS): If this bit is set to ‘1’, then
the controller supports the I/O command set specific extended protection
information formats (refer to the Protection Information Formats section of
the applicable I/O command set specification).
If this bit is cleared to ‘0’ , then the controller does not support the I/O
command set specific extended protection information formats (refer to the
Protection Information Formats section of the NVM Command Set
Specification).
Refer to the LBA Format Extension Enable (LBAFEE) field in the Host
Behavior Support feature (refer to section  5.1.25.1.14) for details for host
software to enable the controller to operate on namespaces using the
protection information formats.
NOTE: This bit field applies to all I/O Command Sets . The original name
has been retained for historical continuity.
14
Delete NVM Set  (DNVMS): If this bit is set to ‘1’, then the controller
supports the Delete NVM Set operation (refer to section 8.1.4.3). If this bit
is cleared to ‘0’, then the controller does not support the Delete NVM Set
operation.
