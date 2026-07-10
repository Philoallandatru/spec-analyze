---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 230
headings: ["5.1.12.1.3 SMART / Health Information (Log Page Identifier 02h)"]
---

# Source page 230

NVM Express® Base Specification, Revision 2.1

208
Figure 205: Error Information Log Entry Data Structure
Bytes Description
23:16
Logical Block Address (LBA): This field indicates I/O Command Set specific data about the error
condition, if applicable . The description is described in the applicable I/O Command Set
specification.
NOTE: The original field name has been retained for historical continuity.
27:24 Namespace Identifier (NSID): This field indicates the NSID of the namespace that the error is
associated with, if applicable.
28
Vendor Specific Information Available  (VSIA): If there is additional vendor specific error
information available, this field provides the log page identifier associated with that page. A value of
0h indicates that no additional information is available. Valid values are in the range of 80h to FFh.
29
Transport Type (TRTYPE): This field indicates the Transport Type of the transport associated with
the error. The values in this field are the same as the TRTYPE values in the Discovery Log Page
Entry (refer to section  5.1.12.3.1). If the error is not transport related, this field shall be cleared to
0h. If the error is transport related, this field shall be set to the type of the transport as defined in the
TRTYPE field within Figure 294.
30
Command Set Indicator (CSI): This field contains command set indicator for the command that
the error is associated with. This field is valid if the Log Page Version field is greater than or equal
to 1h.
31
Opcode (OPC): This field contains opcode for the command that the error is associated with. This
field is valid if the Log Page Version field is greater than or equal to 1h.
This field, the CSI field, and the Submission Queue ID field uniquely indicate the command and the
command set.
39:32 Command Specific Information (CSINFO): This field contains the command specific information.
If used, the command definition specifies the information returned.
41:40
Transport Type Specific Information (TTSI): This field indicates additional transport type specific
error information. If multiple errors exist, then this field indicates additional information about the first
error. This field is transport type dependent (refer to TRTYPE) as follows:
Transport
Type Description
All other values Reserved
3h
This field indicates, the offset, in bytes, from the start of the Transport Header
to the start of the field that is in error. If multiple errors exist, then this field
indicates the lowest offset that is in error.

62:42 Reserved
63 Log Page Version (LPVER): This field shall be set to 1h.
5.1.12.1.3 SMART / Health Information (Log Page Identifier 02h)
This log page is used to provide SMART and general health information.  The information provided is over
the life of the controller and is retained across power cycles  unless otherwise specified . To request the
controller log page, the namespace identifier specified is FFFFFFFFh  or 0h. For compatibility with
implementations compliant with NVM Express Base Specification , Revision 1.4 and earlier, hosts should
use a namespace identifier of FFFFFFFFh to request the controller log page . The controller may also
support requesting the log page on a per namespace basis, as indicated by the SMART Support bit of the
LPA field in the Identify Controller data structure in Figure 312.
If the log page is not supported on a per namespace basis and:
• if a namespace identifier other than 0h or FFFFFFFFh is specified by the host, then the controller
shall abort the command with a status code of Invalid Field in Command; and
• if a namespace identifier of 0h or FFFFFFFFh is specified by the host , then the controller returns
the controller log page.
There is no namespace specific information defined in the SMART / Health  Information log page in this
revision of the specification, thus the controller log page and namespaces specific log page contain identical
information.
