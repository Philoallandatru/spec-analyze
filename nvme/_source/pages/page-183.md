---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 183
headings: ["4.5 Identifier Format and Layout (Informative)", "4.5.1 PCI Vendor ID (VID) and PCI Subsystem Vendor ID (SSVID)"]
---

# Source page 183

NVM Express® Base Specification, Revision 2.1

161
• the Get Features command shall return the current setting of the requested feature value for
the controller.
For feature values that apply to the namespace scope:
a) if the NSID field is set to an active namespace identifier (refer to section 3.2.1.4), then:
• the Set Features command shall set the specified feature value of the specified namespace;
and
• the Get Features command shall return the current setting of the requested feature value for
the specified namespace;
b) if the NSID field is set to FFFFFFFFh, then:
• for the Set Features command, the controller shall:
o if the MDS bit is set to ‘1’ in the Identify Controller data structure, abort the command with
Invalid Field in Command; or
o if the MDS bit is cleared to ‘0’ in the Identify Controller data structure , unless otherwise
specified, set the specified feature value for all namespaces attached to the controller
processing the command;
and
• for the Get Features command, the controller shall, unless otherwise specified in section
5.1.25, abort the command with a status code of Invalid Namespace or Format;
and
c) if the NSID field is set to any other value, then the Set Features command and the Get Features
command is aborted by the controller as described in Figure 92.
For feature values that apply to NVM subsystem scope, Domain scope , NVM Set  scope, or Endurance
Group scope:
a) the NSID field should be cleared to 0h; and
b) if the NSID field is set to a non -zero value, then the Set Features command and the Get Features
command are aborted by the controller as described in Figure 92.
There are mandatory and optional Feature Identifiers defined in section 3.1.3.6. If a Get Features command
or Set Features command is processed that specifies a Feature Identifier that is not supported, then the
controller shall abort the command with a status code of Invalid Field in Command.
4.5 Identifier Format and Layout (Informative)
This section provides guidance for proper implementation of various identifiers defined in the Identify
Controller, Identify Namespace, and Namespace Identification Descriptor data structures.
 PCI Vendor ID (VID) and PCI Subsystem Vendor ID (SSVID)
The PCI Vendor ID (VID, bytes 01:00) and PCI Subsystem Vendor ID (SSVID, bytes 03:02) are defined in
the Identify Controller data structure. The values are assigned by the PCI SIG. Each identifier is a 16 -bit
number in little endian format.
Example:
• VID = ABCDh; and
• SSVID = 1234h.
