---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 454
headings: []
---

# Source page 454

NVM Express® Base Specification, Revision 2.1

432

Figure 498: Extended Discovery Information Entry
Bytes O/M1 Description
00 M Transport Type (TRTYPE):  This field specifies the transport type as
defined in the Transport Type (TRTYPE) field in Figure 294.
01 M Address Family (ADRFAM):  This field specifies the address family as
defined in the Address Family (ADRFAM) field in Figure 294.
02 M2
Subsystem Type (SUBTYPE):  This field specifies the type of the NVM
subsystem that is indicated in this entry as defined in the Subsystem Type
(SUBTYPE) field in Figure 294.
03 M2
Transport Requirements (TREQ):  This field specifies requirements for
the NVMe Transport as defined in the Transport Requirements (TREQ)
field in Figure 294.
05:04 M2 Port ID (PORTID): This field specifies a particular NVM subsystem port as
defined in the Port ID (PORTID) field in Figure 294.
07:06 M2
Controller ID (CNTLID): This field specifies the controller ID as defined in
the Controller ID (CNTLID) field in Figure 294. This field shall specify a
controller ID that a host is able to use in a Connect command to the NVM
subsystem being registered.
09:08 M2
Admin Max SQ Size (ASQSZ):  This field specifies the maximum size of
an Admin Submission Queue as defined in the Admin Max SQ Size
(ASQSZ) field in Figure 294.
31:10  Reserved
63:32 M2
Transport Service Identifier (TRSVCID):  This field specifies the NVMe
Transport service identifier as an ASCII string as defined in the Transport
Service Identifier (TRSVCID) field in Figure 294.
255:64  Reserved
511:256 M
NVMe Qualified Name (NQN):  If the Entity Type (ETYPE) field in the
Discovery Information Management command data portion’s header is set
to 2h (i.e., a DDC is performing the register, de -register, or update task),
then this field specifies the NVMe Qualified Name (NQN) that uniquel y
identifies the NVM subsystem as defined in the NVM Subsystem Qualified
Name (SUBNQN) field in Figure 294.
If the ETYPE field in the Discovery Information Management command
data portion’s header is set to 1h or 3h (i.e., a host or CDC is performing
the register, de-register, or update task), then this field specifies the NQN
that uniquely identifies the host a s defined in the Host NVMe Qualified
Name (HOSTNQN) field in Figure 299.
767:512 M
Transport Address (TRADDR ): If the Entity Type (ETYPE) field in the
Discovery Information Management command data portion’s header is set
to 2h (i.e., a DDC is performing the registration, de -registration, or update
task), then this field specifies the address of a fabric interface on the NVM
subsystem that may be used for a Connect command as an ASCII string
as defined in the Transport Address (TRADDR) field in Figure 294.
If the ETYPE field in the Discovery Information Management command
data portion’s header is set to 1h or 3h (i.e., a host or CDC is performing
the registration, de-registration, or update task), then this field specifies the
address of a fabric interface o n the host that may be used for a Connect
command as an ASCII string as defined in the Transport Address
(TRADDR) field in Figure 299.
If the first byte (i.e., byte 512) of this field is NULL (i.e., cleared to 00h),
then the TRADDR value used by the controller shall be the remote IP
address associated with the connection used to transport the Discovery
Information Management command. If the host attempts to register or de-
register multiple discovery information entries with the first byte of this field
containing a NULL, then the controller shall abort the command with a
status code of Invalid Field in Command.
