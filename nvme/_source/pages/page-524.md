---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 524
headings: ["8.1.8.3 Streams (Directive Type 01h, Optional)"]
---

# Source page 524

NVM Express® Base Specification, Revision 2.1

502
Controller Level Reset, then that change shall be made to the enable state of that Directive for that
namespace attached to any other controller associated with that Host Identifier.
If the Host Identifier value is 0h and the Host Identifier is required to be set to a non -zero value before a
host enables a Directive (i.e., the Directive Type is set to Streams and the SRNZID bit in the NVM
subsystem Stream Capability field (refer to Figure 607) is set to ‘1’) then the Directive Send command shall
be aborted with a status code of Host Identifier Not Initialized.
Refer to the sections defining each Directive Type for restrictions on enabling that directive.
Figure 601: Enable Directive – Command Dword 12
Bits Description
31:16 Reserved
15:08 Directive Type (DTYPE): This field specifies the Directive Type to enable or disable. If this field specifies
the Identify Directive (i.e., 00h), then a status code of Invalid Field in Command shall be returned.
07:01 Reserved
00
Enable Directive (ENDIR): If this bit is set to ‘1’ and the Directive Type is supported, then the Directive
is enabled, unless otherwise specified. If this bit is cleared to ‘0’, then the Directive is disabled. If this bit
is set to ‘1’ for a Directive that is not supported, then a status code of Invalid Field in Command shall be
returned.

Figure 602: Enable Directive – Command Specific Status Values
Value Description
7Fh Stream Resource Allocation Failed: The controller failed to enable the Streams Directive for the
specified namespace and Host Identifier due to insufficient resources.
 Streams (Directive Type 01h, Optional)
The Streams Directive enables the host to indicate (i.e., by using the stream identifier) to the controller that
the specified user data in a User Data Out command (e.g., logical blocks in a write command) are part of
one group of associated data. This information may be used by the controller to store related data in
associated locations or for other performance enhancements.
The controller provides information in response to the Return Parameters operation about the configuration
of the controller that indicates Stream Write Size, Stream Granularity Size, and stream resources at the
NVM subsystem and namespace levels.
Data that is aligned to and in multiples of the Stream Write Size (SWS) provides optimal performance of
the write commands to the controller. The SWS unit of granularity is defined independently for each I/O
Command Set. The Stream Granularity Size indicates the size of the media that is prepared as a unit for
future allocation for write commands and is a multiple of the Stream Write Size. The controller may allocate
and group together a stream in Stream Granularity Size (SGS) units. Refer to Figure 603.
Figure 603: Directive Streams – Stream Alignment and Granularity
SGS (first) SGS (last)...
Complete Stream
Stream Granularity (SGS)
SWS
(first)
SWS
(last)...

One example of this is if the host issues an NVM Command Set Dataset Management command (refer to
the Dataset Management command section of the NVM Command Set Specification)  to deallocate logical
blocks that are associated with a stream, that host should specify a starting LBA and length that is aligned
