---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 159
headings: ["4.1.2 Fabrics Command Common SQE"]
---

# Source page 159

NVM Express® Base Specification, Revision 2.1

137
Figure 93: Common Command Format – Vendor Specific Commands (Optional)
Bytes Description
03:00 Command Dword 0 (CDW0): This field is common to all commands and is defined in Figure 91.
07:04
Namespace Identifier (NSID): This field indicates the namespace ID that this command applies to.
If the namespace ID is not used for the command, then this field shall be cleared to 0h. Setting this
value to FFFFFFFFh causes the command to be applied to all namespaces attached to the controller
processing the command, unless otherwise specified.
The behavior of a controller in response to an inactive namespace ID for a vendor specific command
is vendor specific. Specifying an invalid namespace ID in a command that uses the namespace ID
shall cause the controller to abort the command with a status code of Invalid Namespace or Format,
unless otherwise specified.
15:08 Reserved
39:16 Meatadata and Data Pointers (MDPTR): Refer to Figure 92 for the definition of these fields.
43:40 Number of Dwords in Data Transfer (NDT):  This field indicates the number of dwords in the data
transfer.
47:44 Number of Dwords in Metadata Transfer (NDM):  This field indicates the number of dwords in the
metadata transfer.
51:48 Command Dword 12 (CDW12): This field is command specific Dword 12.
55:52 Command Dword 13 (CDW13): This field is command specific Dword 13.
59:56 Command Dword 14 (CDW14): This field is command specific Dword 14.
63:60 Command Dword 15 (CDW15): This field is command specific Dword 15.
 Fabrics Command Common SQE
The common submission queue entry for Fabrics commands is shown in Figure 94.
Figure 94: Fabrics Command – Submission Queue Entry Format
Bytes Description
03:00 Command Dword 0 (CDW0): Refer to Figure 95.
04
Fabrics Command Type (FCTYPE): This field specifies the Fabrics command transferred in the
capsule. The Fabrics command types are defined in Figure 540. If this field is set to a reserved value,
then the command shall be aborted with a status code of Invalid Field in Command.  The format of the
FCTYPE field is shown here:
Bits Description
07:02
Function (FN): This field contains a value that, in combination with the other fields in
the Fabrics Command Type data structure, creates a unique Combined Fabrics
Command Type.
01:00
Data Transfer Direction (DTD): This field indicates the direction of a data transfer, if
any. All options of the command shall transfer data as specified or transfer no
data. All commands, including vendor specific commands, shall follow this
convention.
Value Definition
00b No Data Transfer: No data is transferred.
01b Host to Controller Transfer:  Data is transferred from the host to the
controller.
10b Controller to Host Transfer: Data is transferred from the controller to the
host.
11b Reserved


23:05 Reserved
39:24
SGL Descriptor 1 (SGL1): This field contains a Transport SGL Data Block descriptor or a Keyed SGL
Data Block descriptor that describes the entire data transfer. Refer to section 4.3.2 for the definition of
SGL descriptors.
This field is used for Fabrics commands that transfer data. If a Fabrics command does not transfer data,
then this field is reserved.
