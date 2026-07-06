---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 227
headings: ["5.1.12.1 Common Log Specific Information", "5.1.12.1.1 Supported Log Pages (Log Page Identifier 00h)"]
---

# Source page 227

NVM Express® Base Specification, Revision 2.1

205
 Figure 202: Get Log Page – Log Page Identifiers
Log Page
Identifier CSI8 Scope Log Page Name Reference
Section
82h to BEh I/O Command Set Specific
BFh Refer to the Zoned Namespace Command Set Specification
C0h to FFh Vendor specific 5
Notes:
1. For namespace identifiers of 0h or FFFFFFFFh.
2. For namespace identifiers other than 0h or FFFFFFFFh.
3. The Single Device Self-test Operation (SDSO) bit is cleared to ‘0’ in the DSTO field in the Identify Controller data
structure (refer to Figure 312).
4. The SDSO bit is set to ‘1’.
5. Selection of a UUID may be supported. Refer to section 8.1.28.
6. For NVM subsystems that support multiple domains (refer to the MDS bit in the Identify Controller data structure,
Figure 312), Domain scope information is returned.
7. The scope is defined in the header of the log page (refer to the Telemetry Host -Initiated Scope field and the
Telemetry Controller-Initiated Scope field).
8. If multiple I/O Command Sets are supported, then the CDW14.CSI field (refer to Figure 201) is used by the log
page: Y = Yes, N = No. If Yes, then refer to the definition of the log page for details on usage.
 Common Log Specific Information
Refer to section 3.1.3.5 for mandatory, optional, and prohibited log pages for the various controller types.
Log pages that indicate a scope of NVM subsystem return information that is global to the NVM subsystem.
Log pages that indicate a scope of Domain return information that is global to the Domain. Log pages that
indicate a scope of controller return information that is specific to the controller that is processing the
command. Log pages that indicate a scope of Namespace return information that is specific to the specified
namespace. For log pages t hat indicate multiple scopes, support for multiple domains o r the namespace
identifier that is specified determines which information is returned. The definition of any individual field
within a log page may indicate a different scope that is specific to that individual field.
For log pages with a scope of NVM subsystem or controller (as shown in Figure 202), the controller shall
abort commands that specify namespace identifiers other than 0h or FFFFFFFFh with a status code of
Invalid Field in Command. Otherwise, the rules for namespace identifier usage in Figure 92 apply.
5.1.12.1.1 Supported Log Pages (Log Page Identifier 00h)
An NVM subsystem may support several interfaces for submitting a Get Log Page command such as an
Admin Submission Queue, PCIe VDM Management Endpoint, or 2-Wire Management Endpoint (refer to
the NVM Express Management Interface Specification for details on Management Endpoints) and may
have zero or more instances of each of those interfaces. The log pages supported on each instance of each
interface may be differ ent. This log page is used to describe the log pages that are supported on the
interface to which the Get Log Page command was submitted and attributes specific to each log page. The
log page is defined in Figure 203. The attributes of each log page are described in a LID Supported and
Effects data structure defined in Figure 204.
If the UUID Selection Supported bit is set to ‘1’ for the Get Log Page command in the Commands Supported
and Effects log page (refer to section 5.1.12.1.6), then the log page data reflects the log pages that are
supported based on the value of the UUID Index field (refer to section 8.1.28).
For controllers that implement I/O Queues, the log pages that the controller supports are dependent on the
I/O Command Set that is based on:
• the I/O Command Set selected in CC.CSS, if CC.CSS is not set to 110b; and
• the Command Set Identifier (CSI) field in CDW 14, if CC.CSS is set to 110b.
