---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 322
headings: []
---

# Source page 322

NVM Express® Base Specification, Revision 2.1

300
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
77 M M M
Maximum Data Transfer Size (MDTS):  This field indicates the maximum data transfer
size for a command that transfers data between host-accessible memory (refer to section
1.5.44) and the controller. The host should not submit a command that exceeds this
maximum data transfer size. If a command is submitted that exceeds this transfer size,
then the command is aborted with a status code of Invalid Field in Command. The value
is in units of the minimum memory page size (CAP.MPSMIN) and is reported as a power
of two (2^n). A value of 0h indicates that there is no maximum data transfer size.
If the MEM bit is cleared to ‘0’ in the CTRATT field, then t his field includes the length of
metadata, if metadata is interleaved with the user data.
If the MEM bit is set to ‘1’, then this field excludes the length of metadata.
This field does not apply to commands that do not transfer data between host-accessible
memory and the controller (e.g., the Verify command, the Write Uncorrectable command,
and the Write Zeroes command); refer to the ONCS field for restrictions on these
commands and other commands that transfer data.
If SGL Bit Bucket descriptors are supported, their lengths shall be included in determining
if a command exceeds the Maximum Data Transfer Size for destination data buffers.
Their length in a source data buffer is not included for a Maximum Data Transfer S ize
calculation.
79:78 M M M Controller ID (CNTLID):  Contains the NVM subsystem unique controller identifier
associated with the controller.
83:80 M M M
Version (VER): This field contains the value reported in the Version property (i.e., VS
property) defined in section 3.1.4.2. Implementations compliant with NVM Express Base
Specification, Revision 1.2 or later shall report a non-zero value in this field.
87:84 M M R
RTD3 Resume Latency (RTD3R):  This field indicates the expected latency in
microseconds to resume from Runtime D3 (RTD3). Refer to section 8.1.17.4. A value of
0h indicates RTD3 Resume Latency is not reported.
91:88 M M R
RTD3 Entry Latency (RTD3E):  This field indicates the typical latency in microseconds
to enter Runtime D3 (RTD3). Refer to section 8.1.17.4. A value of 0h indicates RTD3
Entry Latency is not reported.
95:92 M M M
Optional Asynchronous Events Supported (OAES): This field indicates the optional
asynchronous events supported by the controller. A controller shall not send optional
asynchronous events before they are enabled by host software.
Bits Description
31
Discovery Log Page Change Notification  (DLPCN): If this bit is set to
‘1’, then the controller supports sending Discovery Log Page Change
Notifications. If this bit is cleared to ‘0’, then the controller does not support
the Discovery Log Page Change Notification events.
30:28 Reserved
27
Zone Descriptor Changed Notices  (ZDCN): If this bit is set to ‘1’, then
the controller supports the Zone Descriptor Changed Notices event and the
associated Changed Zone List log page (refer to the Zoned Namespace
Command Set specification). If this bit is cleared to ‘0’, then the controller
does not support the Zone Descriptor Changed Notices event nor the
associated Changed Zone List log page.
26:20 Reserved
19
Allocated Namespace Attribute Notices (ANSAN): If this bit is set to ‘1’,
then the controller supports the Allocated Namespace Attribute Notices
event and the associated Changed Allocated Namespace List log page. If
this bit is cleared to ‘0’, then the controller does not support the Allocated
Namespace Attribute Notices event nor the associated Changed Allocated
Namespace List log page.
18 Reserved
