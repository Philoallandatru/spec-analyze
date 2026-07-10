---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 156
headings: []
---

# Source page 156

NVM Express® Base Specification, Revision 2.1

134
Figure 91: Command Dword 0
Bits Description
07:00
Opcode (OPC): This field specifies the opcode of the command to be executed as shown here:
Bits Description
07:02 Function (FN): This field contains a value that, in combination with the other fields in the
Opcode data structure, creates a unique combined opcode value.
01:00
Data Transfer Direction (DTD): This field indicates the direction of a data transfer, if any. All
options of the command shall transfer data as specified or transfer no data. All
commands, including vendor specific commands, shall follow this convention.
Value Definition
00b No Data Transfer: No data is transferred.
01b Host to Controller Transfer:  Data is transferred from the host to the
controller.
10b Controller to Host Transfer: Data is transferred from the controller to the
host.
11b Bi-Directional Transfer: Data is transferred from the host to the controller
and from the controller to the host.


The Common Command Format is defined in Figure 92. Any additional I/O Command Set defined in the
future may use an alternate command size or format.
SGLs shall not be used for Admin commands in NVMe over PCIe implementations.
Figure 92: Common Command Format
Bytes Description
03:00 Command Dword 0 (CDW0): This field is common to all commands and is defined in Figure 91.
07:04
Namespace Identifier (NSID): This field specifies the namespace that this command applies to. If the
namespace identifier is not used for the command, then this field shall be cleared to 0h. The value FFFFFFFFh
in this field is a broadcast value (refer to section 3.2.1.2), where the scope (e.g., the NVM subsystem, all
attached namespaces, or all namespaces in the NVM subsystem) is dependent on the command. Refer to
Figure 141 and Figure 555 for commands that support the use of the value FFFFFFFFh in this field.
Specifying an inactive namespace identifier (refer to section 3.2.1.4) in a command that uses the namespace
identifier shall cause the controller to abort the command with a status code of Invalid Field in Command,
unless otherwise specified. Specifying an invalid namespace identifier (refer to section 3.2.1.2) in a command
that uses the namespace identifier shall cause the controller to abort the command with a status code of Invalid
Namespace or Format, unless otherwise specified.
If the namespace identifier is used for the command (refer to Figure 141), the value FFFFFFFFh is not
supported for that command, and the host specifies a value of FFFFFFFFh, then the controller shall abort the
command with a status code of Invalid Field in Command, unless otherwise specified.
If the namespace identifier is not used for the command and the host specifies a value from 1h to FFFFFFFFh,
then the controller shall abort the command with a status code of Invalid Field in Command, unless otherwise
specified.
11:08 Command Dword 2 (CDW2): This field is command specific Dword2.
15:12 Command Dword 3 (CDW3): This field is command specific Dword3.
