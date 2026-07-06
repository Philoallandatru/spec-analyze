---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 188
headings: ["4.7.1 Unique Identifier", "4.7.1.1 Unique Identifier Overview", "4.7.1.2 NVM Subsystem Unique Identifier", "4.7.1.3 Controller Unique Identifier", "4.7.1.4 Namespace Unique Identifier"]
---

# Source page 188

NVM Express® Base Specification, Revision 2.1

166
• The string “nqn.2014-08.org.nvmexpress:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6”.
NVMe hosts, controllers and NVM subsystems process (e.g., compare for equality) UTF-8 NVMe Qualified
Names without any text processing or text comparison logic that is specific to the Unicode character set or
locale (refer to section 4.8).
 Unique Identifier
 Unique Identifier Overview
Unique identifiers include the following:
a) NVM subsystem unique identifiers (refer to section 4.7.1.2);
b) controller unique identifiers (refer to section 4.7.1.3); and
c) namespace unique identifiers (refer to section 4.7.1.4).
Unique identifiers are either globally unique or unique only within the NVM subsystem. NVM subsystem
unique identifiers are globally unique. Controller unique identifiers (i.e., the CNTLID) are unique only within
the NVM subsystem. Namespace unique identifiers are globally unique.
 NVM Subsystem Unique Identifier
The NVM Subsystem NVMe Qualified Name specified in the Identify Controller data structure (refer to
Figure 312) should be used (e.g., by host software) as the unique identifier for the NVM subsystem. If the
controller is compliant with an NVM Express Specification prior to revision 1.2.1 (i.e., revisions where the
NVM Subsystem NQN was not defined), then the PCI Vendor ID, Serial Number, and Model Number fields
in the Identify Controller data structure and the NQN Starting String “nqn.2014.08.org.nvmexpress:” may
be combined by the host to form a globally unique value that identifies the NVM subsyst em (e.g., for host
software that uses NQNs). The method shown in Figure 139 should be used by the host to construct an
NVM Subsystem NQN for older NVM subsystems that do not provide an NQN in the Identify Controller data
structure. The mechanism used by the vendor to assign Serial Number and Model Number values to ensure
uniqueness is outside the scope of this specification.
Figure 139: NQN Construction for Older NVM Subsystems
Bytes Description
26:00 NQN Starting String (NSS): Contains the 27 letter ASCII string ”nqn.2014-08.org.nvmexpress:”.
30:27 PCI Vendor ID (VID): Contains the company vendor identifier that is assigned by the PCI SIG as a
hexadecimal ASCII string.
34:31 PCI Subsystem Vendor ID (SSVID): Contains the company vendor identifier that is assigned by the
PCI SIG for the subsystem as a hexadecimal ASCII string.
54:35 Serial Number (SN): Contains the serial number for the NVM subsystem that is assigned by the vendor
as an ASCII string.
94:55 Model Number (MN): Contains the model number for the NVM subsystem that is assigned by the
vendor as an ASCII string.
222:95 Padding (PAD): Contains spaces (ASCII character 20h).
 Controller Unique Identifier
An NVM subsystem may contain multiple controllers. All controllers contained in the NVM subsystem share
the same NVM subsystem unique identifier. The Controller ID (CNTLID) value returned in the Identify
Controller data structure may be used to uniquely identify a controller within an NVM subsystem. The
Controller ID value when combined with the NVM subsystem identifier forms a globally unique value that
identifies the controller. The mechanism used by the vendor to assign Controller ID values is outside th e
scope of this specification.
 Namespace Unique Identifier
The Identify Namespace data structure (refer to the applicable I/O Command Set specification) contains
the IEEE Extended Unique Identifier (EUI64) and the Namespace Globally Unique Identifier (NGUID) fields.
The Namespace Identification Descriptor data str ucture (refer to Figure 315) contains the Namespace
