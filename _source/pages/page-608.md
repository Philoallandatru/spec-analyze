---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 608
headings: ["8.1.28 Universally Unique Identifiers (UUIDs) for Vendor Specific Information", "8.1.28.1 UUIDs for Vendor Specific Information Introduction", "8.1.28.2 UUIDs for Vendor Specific Information Requirements"]
---

# Source page 608

NVM Express® Base Specification, Revision 2.1

586
Figure 657: Telemetry Log Example – Data Area 2 Populated
Block Number Telemetry Host-Initiated Data Areas
1












1,000

Data Area 1
(empty)







Data Area 2 *







Data Area 3 *
* Data Area 2, and Data Area 3 contain the same telemetry data in blocks 1 through 1,000.
 Universally Unique Identifiers (UUIDs) for Vendor Specific Information
 UUIDs for Vendor Specific Information Introduction
Several commands send or receive information that contains fields described as Vendor Specific or that is
specified by a command field containing a value in a vendor specific range. Examples include the Set
Features command, which may specify a vendor specific feature identifier, and the Identify command, which
may retrieve a data structure having a vendor specific area.
The vendor specific information may have different definitions (e.g., a vendor specific log page identifier
with the contents of the page defined differently by different entities, such as an NVM subsystem vendor
and an NVM subsystem customer). By associating each definition of the information with a UUID specified
by the defining entity, a command is able to specify the particular definition of the information.
A command specifies a particular definition of the information by specifying an index into a list of UUIDs
supported by the controller (refer to section 5.1.13.2.14). The NVMe Invalid UUID (refer to section 8.1.28.2)
is used to replace a previously valid UUID in the UUID List (refer to Figure 320). This is done to keep the
values in the list at a static index, as that index is used by the Host to access the UUID List contents.
NVM subsystem vendors and customers communicate (by means outside the scope of this specification)
the UUID used for each definition of the information.
 UUIDs for Vendor Specific Information Requirements
A UUID list is a list of non -zero UUID values, terminated by a 0h UUID value. Each non -zero UUID value
may be either a valid UUID or the NVMe Invalid UUID. The NVMe Invalid UUID is the hexadecimal value
FFFFFFFF_FFFFFFFF_7FFFFFFF_FFFFFFFFh. A valid UUID is any non-zero value other than the NVMe
Invalid UUID.
