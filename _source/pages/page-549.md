---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 549
headings: ["8.1.14 Management Addresses"]
---

# Source page 549

NVM Express® Base Specification, Revision 2.1

527
The Key Per I/O capability does not have any effect on host accesses to RPMBs and Boot Partitions as
these features are not addressed through I/O commands that specify a namespace.
A controller that supports the Key Per I/O capability shall set the KPIOS bit to ‘1’ in the Identify Controller
data structure (refer to Figure 312).
A namespace that supports the Key Per I/O capability shall set the KPIOSNS bit to ‘1’ in the I/O Command
Set Independent Identify Namespace data structure (refer to Figure 319).
The Key Per I/O capability uses the Command Extension Type (CETYPE) and Command Extension Value
(CEV) fields in all read and write commands. Definition of the CETYPE fields are shown in Figure 620.
Figure 620: CETYPE Definition
Value Definition CEV Field Definition
0h Reserved
1h Key Per I/O Tag (KPIOTAG):  This command
is using the Key Per I/O capability.
Key Tag (KEYTAG) : Specifies a namespace -specific
16-bit encryption key tag that identifies the encryption
key used to encrypt or decrypt the data of the
command.
The same Key Tag value on different namespaces may
or may not identify the same encryption key.
Refer to the Maximum Key Tag field in the I/O
Command Set Independent Identify Namespace data
structure (refer to Figure 319) for the supported values.
2h to Eh Reserved
Fh Vendor Specific
If:
• the Key Per I/O capability is enabled in a namespace (i.e., the KPIOENS bit set to ‘1’);
• an I/O command supports the CETYPE field; and
• the CETYPE field in that I/O command is set to a value that is reserved,
then the controller shall abort that command with a status code of Invalid Field in Command.
 Management Addresses
Various entities on a network are able to request a management agent to perform management operations
on NVM subsystems. A controller in an NVM subsystem use the Management Addresses capability to
indicate the network addresses of those management agents. When an NVM subsystem is provisioned in
a storage system, the management addresses are established in the controller.
Management agents are able to be located in various networked entities, including:
• NVM subsystems;
• Fabric interface managers;
• Embedded management controllers; and
• Host software.
Each management address is represented as a uniform resource indicator (URI; refer to RFC 3986).
The address of a management agent contained in an NVM subsystem (e.g., an Ethernet-attached SSD) is
indicated in the Management Address List log page (refer to section 5.1.12.1.24), in a Management Address
Descriptor indicating a Management Address Type of 1h. The method by which the address is determined
is outside the scope of this specification.
The address of a management agent contained in a fabric interface manager is indicated in the
Management Address List log page (refer to section 5.1.12.1.24), in a Management Address Descriptor
indicating a Management Address Type of 2h. The method by which the address is determined is outside
the scope of this specification.
