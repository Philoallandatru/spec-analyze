---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 356
headings: ["5.1.13.2.6 I/O Command Set specific Identify Controller data structure (CNS 06h)", "5.1.13.2.7 I/O Command Set specific Active Namespace ID list (CNS 07h)", "5.1.13.2.8 I/O Command Set Independent Identify Namespace data structure (CNS 08h)"]
---

# Source page 356

NVM Express® Base Specification, Revision 2.1

334
Each I/O Command Set specific Identify Namespace data structure specifies fields that define capabilities
used by a host to format or create a namespace. If the NSID field is set to FFFFFFFFh, then the controller
shall return an I/O Command Set specific Identify Namespace data structure for the I/O Command Set
specified in the CSI field (refer to Figure 311) that has:
• the capability fields as specified by the I/O Command Set specific Identify Namespace data
structure that requires a value that is the same value for all namespaces formatted using the User
Data Formats associated with the Number of LBA Formats field; and
• fields that are not capability fields set to the value 0h.
If the controller supports the Namespace Management capability (refer to section 8.1.15) and the NSID
field is set to FFFFFFFFh, then the controller shall return an I/O Command Set specific Identify Namespace
data structure for the I/O Command Set specified in the CSI field.
If the controller does not support the Namespace Management capability and the NSID field is set to
FFFFFFFFh, then the controller may abort the command with a status code of Invalid Namespace or
Format.
5.1.13.2.6 I/O Command Set specific Identify Controller data structure (CNS 06h)
An I/O Command Set specific Identify Controller data structure is returned to the host for the controller
processing the command. The specific Identify Controller data structure that is returned by this command
is specified by the Command Set Identifier (CSI) field (refer to Figure 311). Data structures for specific I/O
Command Sets are optionally defined by the I/O Command Set specifications. If the I/O Command Set
specified by the CSI field does not have an Identify Controller data structure, then the controller shall return
a zero filled data structure. If the host requests a data structure for an I/O Command Set that the controller
does not support, the controller shall abort the command with a status code of Invalid Field in Command.
5.1.13.2.7 I/O Command Set specific Active Namespace ID list (CNS 07h)
A list of 1,024 namespace IDs is returned to the host containing active NSIDs in increasing order that are
greater than the value specified in the Namespace Identifier (NSID) field of the command as specified by
the Command Set Identifier (CSI) field of th e command. Only namespaces associated with the I/O
Command Set specified by the CSI value are returned. For CSI values that are not supported or not enabled
the command is aborted with a status code of Invalid Field in Command.
The controller should abort the command with a status code of Invalid Namespace or Format if the NSID
field is set to FFFFFFFEh or FFFFFFFFh. The NSID field may be cleared to 0h to retrieve a Namespace
List including the namespace starting with NSID of 1h. The data structure returned is a Namespace List
(refer to section 4.6.2).
5.1.13.2.8 I/O Command Set Independent Identify Namespace data structure (CNS 08h)
If the Namespace Identifier (NSID) field specifies an active NSID, then the I/O Command Set Independent
Identify Namespace data structure (refer to Figure 319) is returned to the host for that specified namespace.
If the value in the NSID field specifies an inactive NSID , then the controller returns a zero filled data
structure.
The Reported column in Figure 319 specifies fields in the I/O Command Set Independent Identify
Namespace data structure that define namespace capabilities used by a host to format or create a
namespace. If the NSID field is set to FFFFFFFFh, then the controller shall return an I/O Command Set
Independent Identify Namespace data structure that:
• for fields in Figure 319 that indicate “Yes” in the Reported column, contain the same value for all
namespaces; and
• for fields in Figure 319 that indicate “No” in the Reported column, contain a value cleared to 0h.
If the controller supports the Namespace Management capability (refer to section 8.1.15) and the NSID
field is set to FFFFFFFFh, then the controller shall return an I/O Command Set Independent Identify
Namespace data structure. If the controller does not support the Namespace Management capability and
