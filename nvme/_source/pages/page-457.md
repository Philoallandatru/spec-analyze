---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 457
headings: ["5.3.3.1 Command Completion", "5.3.4 Fabric Zoning Lookup command", "5.3.4.1 Command Completion", "5.3.5 Fabric Zoning Receive command"]
---

# Source page 457

NVM Express® Base Specification, Revision 2.1

435
 Command Completion
Upon completion of the Discovery Information Management command, the controller posts a completion
queue entry to the Admin Completion Queue indicating the status of the command. Discovery Information
Management command specific status values are defined in Figure 501.
Figure 501: Discovery Information Management – Command Specific Status Values
Value Definition
2Fh
Invalid Discovery Information: The discovery information provided in one or more extended discovery
information entries is not applicable for the type of entity selected in the Entity Type (ETYPE) field of the
Discovery Information Management command data portion’s header. Refer to Figure 500 for restrictions,
if any, that apply to each field in the extended discovery information entry based upon the type of
extended discovery information entry being registered, de-registered, or updated.
32h
Insufficient Discovery Resources:  The number of discovery information entries provided in the data
portion of the Discovery Information Management command for a registration task (i.e., TAS field cleared
to 0h) exceeds the available capacity for new discovery information entries on the CD C or DDC. This
may be a transient condition.
 Fabric Zoning Lookup command
The Fabric Zoning Lookup (FZL) command is used to lookup a key associated with a Zoning data structure
in the CDC. The FZL command uses the Data Pointer field, as shown in Figure 502.
Figure 502: Fabric Zoning Lookup (FZL) – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.
 Command Completion
Upon completion of the Fabric Zoning Lookup (FZL) command, the controller posts a completion queue
entry to the Admin Completion Queue indicating the status of the command. Command specific status
values for the FZL command are defined in Figure 503.
Figure 503: Fabric Zoning Lookup (FZL) – Command Specific Status Values
Value Definition
30h Zoning Data Structure Locked: The requested Zoning data structure is locked on the CDC.
31h Zoning Data Structure Not Found: The requested Zoning data structure does not exist on the CDC.
33h Requested Function Disabled: Fabric Zoning is not enabled on the CDC.
34h ZoneGroup Originator Invalid: The DDC is not allowed to access the specified ZoneGroup.
The key associated with the Zoning data structure that matches the specified FZL data (refer to section
8.3.2.3.8) is returned in the Completion Queue Entry Dword 0, as shown in Figure 504.
Figure 504: Fabric Zoning Lookup (FZL) – Completion Queue Entry Dword 0
Bits Description
31:00 Zoning Data Key (ZDK): The key associated with the Zoning data structure that matches the specified FZL
data.
 Fabric Zoning Receive command
The Fabric Zoning Receive (FZR) command is used to receive a Zoning data structure. The FZR command
uses the Data Pointer, Command Dword 10, Command Dword 11, and Command Dword 12 fields, as
shown in Figure 505, Figure 506, Figure 507, and Figure 508 respectively.
