---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 321
headings: ["5.1.13.2 Common Identify Data Structures", "5.1.13.2.1 Identify Controller Data Structure (CNS 01h)"]
---

# Source page 321

NVM Express® Base Specification, Revision 2.1

299
 Common Identify Data Structures
5.1.13.2.1 Identify Controller Data Structure (CNS 01h)
The Identify Controller data structure (refer to Figure 312) is returned to the host for the controller processing
the command.
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
Controller Capabilities and Features
01:00 M M R
PCI Vendor ID (VID): Contains the company vendor identifier that is assigned by the PCI
SIG. This is the same value as reported in the ID register in  the PCI Header section of
the NVMe over PCIe Transport Specification.
03:02 M M R
PCI Subsystem Vendor ID (SSVID): Contains the company vendor identifier that is
assigned by the PCI SIG for the subsystem. This is the same value as reported in the SS
register in the PCI Header section of the NVMe over PCIe Transport Specification.
23:04 M M O5
Serial Number (SN): Contains the serial number for the NVM subsystem that is assigned
by the vendor as an ASCII string. Refer to section 1.4.2 for ASCII string requirements.
For a Discovery subsystem, this field should not be used to construct a unique identifier.
For subsystems that are not Discovery subsystems,  refer to section 4.7.1 for unique
identifier requirements.
63:24 M M O5
Model Number (MN): Contains the model number for the NVM subsystem that is
assigned by the vendor as an ASCII string. Refer to section 1.4.2 for ASCII string
requirements.
For a Discovery subsystem, this field should not be used to construct a unique identifier.
For subsystems that are not Discovery subsystems , r efer to section 4.7.1 for unique
identifier requirements.
71:64 M M M
Firmware Revision (FR): Contains the currently active firmware revision , as an ASCII
string, for the domain of which this controller is a part . This is the same revision
information that may be retrieved with the Get Log Page command, refer to section
5.1.12.1.4.
72 M M R
Recommended Arbitration Burst (RAB):  This is the recommended Arbitration Burst
size. The value is in commands and is reported as a power of two (2^n). This is the same
units as the Arbitration Burst size. Refer to section 3.4.4.
75:73 M M R
IEEE OUI Identifier (IEEE): Contains the Organization Unique Identifier (OUI) for the
controller vendor. The OUI shall be a valid IEEE/RAC assigned identifier that may be
registered at http://standards.ieee.org/develop/regauth/oui/public.html.
76 O O R
Controller Multi -Path I/O and Namespace Sharing Capabilities (CMIC):  This field
specifies multi-path I/O and namespace sharing capabilities of the controller and NVM
subsystem.
Bits Description
7:4 Reserved
3
Asymmetric Namespace Access Reporting Support (ANARS): If this bit is
set to ‘1’, then the NVM subsystem supports Asymmetric Namespace Access
Reporting (refer to section 8.1.1). If this bit is cleared to ‘0’, then the NVM
subsystem does not support Asymmetric Namespace Access Reporting.
2
Function Type (FT): If this bit is  set to ‘1’, then the controller is associated
with an SR-IOV Virtual Function. If this bit is cleared to ‘0’, then the controller
is associated with a PCI Function or a Fabrics connection.
1
Multiple Controllers (MCTRS): If this bit is set to ‘1’, then the NVM subsystem
may contain two or more controllers. If this bit is cleared to ‘0’, then the NVM
subsystem contains only a single controller. As described in section 2.4.1, an
NVM subsystem that contains multiple controllers may be used by multiple
hosts, or may provide multiple paths for a single host.
0
Multiple Ports (MPORTS): If this bit is  set to ‘1’, then the NVM subsystem
may contain more than one NVM subsystem port. If this bit is cleared to ‘0’,
then the NVM subsystem contains only a single NVM subsystem port.
