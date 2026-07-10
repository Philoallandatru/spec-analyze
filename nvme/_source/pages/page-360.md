---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 360
headings: []
---

# Source page 360

NVM Express® Base Specification, Revision 2.1

338
Figure 319: Identify – I/O Command Set Independent Identify Namespace Data Structure
Bytes O/M1 Description Reported2
14 M
Namespace Status (NSTAT):  This field indicates the status of the namespace
with the specified NSID.
Bits Description
7:3 Reserved
2:1
I/O Impacted (IOI):  This field indicates whether I/O performance is
currently degraded (e.g., I/O command processing is delayed or slow).
Value Definition
00b I/O performance degradation is not reported
01b Reserved
10b I/O performance is not currently degraded
11b I/O performance is currently degraded

0
Namespace Ready (NRDY):  A value of ‘1’ indicates that the
namespace is ready (refer to section 3.5.3). A value of ‘0’ indicates that
the namespace is not ready.

No
15 O
Key Per I/O Status  (KPIOS): This field indicates the namespace Key Per I/O
capability status.
Bits Description
7:2 Reserved
1
Key Per I/O Supported in Namespace (KPIOSNS): If this bit is set to
‘1’, then the Key Per I/O capability is supported by the namespace. If
this bit is cleared to ‘0’, then the Key Per I/O capability is not supported
by the namespace.
0
Key Per I/O Enabled in Namespace (KPIOENS):  If this bit is set to
‘1’, then the Key Per I/O capability is enabled on the namespace. The
mechanism to enable the Key Per I/O capability on the namespace is
outside the scope of this specification (refer to section 8.1.11). If this bit
is cleared to ‘0’, then the Key Per I/O capability is disabled on the
namespace.
If the KPIOSNS bit is cleared to ‘0’, then this bit shall be cleared to ‘0’.

Yes
17:16 O
Maximum Key Tag (MAXKT): This field indicates the maximum Key Tag field
value allocated to th is namespace in an I/O command that supports a Key Tag
field.
If the KPIOENS bit is set to ‘1’, then the valid Key Tag field values for this
namespace are 0h to the value of this field.
If the KPIOENS bit is cleared to ‘0’, then this field is reserved.
No
19:18  Reserved
23:20 O
Reachability Group Identifier (RGRPID): For NSID other than FFFFFFFFh, this
field indicates the Reachability Group Identifier of the reachability group (refer to
section 8.1.19.1) of which the namespace is a member. Each namespace that is
attached to a controller that supports Reachability Group Reporting (refer to the
RRSUP bit in the CRCAP field in Figure 312)) shall report a valid RGRPID. If the
controller does not support Reachability Group Reporting, then this field shall be
cleared to 0h.
If the value in this field changes and Reachability Group Change Notices are
supported and enabled, then the controller shall issue a Reachability Group
Change Notice.
No
4095:24  Reserved
Notes:
1. O/M definition: O = Optional, M = Mandatory.
2. Identifies fields that report information for the Identify command when querying the capabilities of LBA formats.
