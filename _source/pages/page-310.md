---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 310
headings: []
---

# Source page 310

NVM Express® Base Specification, Revision 2.1

288
Figure 294: Discovery Log Page Entry Data Structure
Bytes Description
07:06
Controller ID (CNTLID): Specifies the controller ID. If the NVM subsystem uses a dynamic
controller model, then this field shall be set to FFFFh. If the NVM subsystem uses a static
controller model, then this field may be set to a specific controller ID (values 0h to FFEFh are
valid). If the NVM subsystem uses a static controller model and the value indicated is FFFEh,
then the host should remember the Controller ID returned as part of the Fabrics Connect
command in order to re-establish an association in the future with the same controller.
09:08 Admin Max SQ Size (ASQSZ): Specifies the maximum size of an Admin Submission Queue.
This applies to all controllers in the NVM subsystem. The value shall be a minimum of 32 entries.
11:10
Entry Flags (EFLAGS): This field Indicates additional information related to the current entry.
Bits Description
15:3 Reserved
2
No CDC Connectivity (NCC): If this bit is set to ‘1’, then no DDC that describes this
entry is currently connected to the CDC. If this bit is cleared to ‘0’, then at least one
DDC that describes this entry is currently connected to the CDC. If the Discovery
controller returning this log page is not a CDC, then this bit shall be cleared to ‘0’
and should be ignored by the host.
1
Explicit Persistent Connection Support for Discovery (EPCSD): If this bit is set
to ‘1’, then Explicit Persistent Connections are supported for the Discovery controller
described by this entry. If this bit is cleared to ‘0’, then support for Explicit Persistent
Connections is not reported.
For entries with the SUBTYPE field set to 2h, this bit shall be cleared to ‘0’.
0
Duplicate Returned Information (DUPRETINFO): If this bit is set to ‘1’, then using
the content of this entry to access this Discovery Service returns the same
information that is returned by using the content of other entries in this log page that
also have this bit set to ‘1’. If this bit is cleared to ‘0’, then using the content of this
entry to access this Discovery Service may or may not return different information
than is returned by using the content of any other entry in this log page.
For entries with the SUBTYPE field set to a value other than 3h, this bit shall be
cleared to ‘0’.

31:12 Reserved
63:32
Transport Service Identifier (TRSVCID): Specifies the NVMe Transport service identifier as an
ASCII string. The NVMe Transport service identifier is specified by the associated NVMe
Transport binding specification.
255:64 Reserved
511:256
NVM Subsystem Qualified Name (SUBNQN): NVMe Qualified Name (NQN) that uniquely
identifies the NVM subsystem. Refer to section 4.7.
For a Discovery subsystem, if that Discovery subsystem has a unique Discovery Service NQN
(i.e., the NVM Subsystem NVMe Qualified Name (SUBNQN) field in that Discovery subsystem’s
Identify Controller data structure contains a unique Discovery Service NQN value), then the value
returned shall be that unique Discovery Service NQN. If the Discovery subsystem does not have
a unique Discovery Service NQN, then the value returned shall be the well -known Discovery
Service NQN (i.e., nqn.2014-08.org.nvmexpress.discovery).
767:512
Transport Address (TRADDR): Specifies the address of the NVM subsystem that may be used
for a Connect command as an ASCII string. The Address Family field describes the reference for
parsing this field. Refer to section 1.4.2 for ASCII string requirements.  For the definition of this
field, refer to the appropriate NVMe Transport binding specification.
1023:768
Transport Specific Address Subtype (TSAS): Specifies NVMe Transport specific information
about the address. For the definition of this field, refer to the appropriate NVMe Transport binding
specification.
