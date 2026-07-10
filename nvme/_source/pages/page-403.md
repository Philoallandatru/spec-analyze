---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 403
headings: ["5.1.25.1.8 Keep Alive Timer (Feature Identifier 0Fh)", "5.1.25.1.9 Host Controlled Thermal Management (Feature Identifier 10h)"]
---

# Source page 403

NVM Express® Base Specification, Revision 2.1

381
Figure 396: Timestamp – Data Structure for Get Features
Bytes Description
06
Timestamp Attribute (TSTMPS): This field indicates attributes associated with the timestamp.
Bits Description
07:04 Reserved
03:01
Timestamp Origin (TSTMPO):
Value Definition
000b The Timestamp field was initialized to 0h by a Controller Level Reset.
001b The Timestamp field was initialized with a Timestamp value using a Set
Features command.
010b to 111b Reserved

00
Synch (SYNC):
Value Definition
0b The controller counted time in milliseconds continuously since the Timestamp value
was initialized.
1b The controller may have stopped counting during vendor specific intervals after the
Timestamp value was initialized (e.g., non-operational power states).


07 Reserved
5.1.25.1.8 Keep Alive Timer (Feature Identifier 0Fh)
This Feature configures the controller Keep Alive Timer. Refer to section 3.9 for Keep Alive details. The
attributes are specified in Command Dword 11.
If a Get Features command is submitted for this Feature, the attributes specified in Figure 397 are returned
in Dword 0 of the completion queue entry for that command.
Figure 397: Keep Alive Timer – Command Dword 11
Bits Description
31:00
Keep Alive Timeout (KATO): This field specifies the timeout value for the Keep Alive  Timer feature in
milliseconds. The controller rounds up the value specified to the granularity indicated in the KAS field in
the Identify Controller data structure. If this field is cleared to 0h, then the Keep Alive Timer is disabled.
The default value for this field is 0h for NVMe transports that do not require use of the Keep Alive Timer
feature (e.g., NVMe over PCIe). For NVMe transports that require use of the Keep Alive Timer feature
(e.g., RDMA and TCP), the default value for this field is 1D4C0h (i.e., 120,000 milliseconds or 2 minutes)
rounded up to the granularity indicated in the KAS field.
Refer to the applicable NVMe Transport Binding specification for details.
5.1.25.1.9 Host Controlled Thermal Management (Feature Identifier 10h)
This Feature configures the controller settings for the host controlled thermal management feature, refer to
section 8.1.17.5. The host controlled thermal management feature uses Command Dword 11 with the
attributes shown in Figure 398.
If a Get Features command is submitted for this Feature, then the attributes shown in Figure 398 are
returned in Dword 0 of the completion queue entry for that command.
