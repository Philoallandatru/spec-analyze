---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 409
headings: ["5.1.25.1.15 Sanitize Config (Feature Identifier 17h)"]
---

# Source page 409

NVM Express® Base Specification, Revision 2.1

387
Figure 407: Host Behavior Support – Data Structure
Bytes Description
05:04
Copy Descriptor Formats Enable (CDFE): The bits in this field that are used (i.e., are neither reserved
nor unused) enable the corresponding Copy Descriptor Formats (e.g., the Copy Descriptor Format 2h
Enable (CDF2E) bit enables Copy Descriptor Format 2h). Bits in this field that this specification defines
as unused (i.e., bits 1:0) correspond to Copy Descriptor Formats (refer to the NVM Command Set
Specification) that are always enabled if they are supported.
Bits Description
15:5 Reserved
4
Copy Descriptor Format 4h Enable (CDF4E):  If this bit is set to ‘1’ and the controller
supports Copy Descriptor Format 4h, then Copy Descriptor Format 4h is enabled. If this bit is
cleared to ‘0’, or the controller does not support Copy Descriptor Format 4h, then Copy
Descriptor Format 4h is disabled.
3
Copy Descriptor Format 3h Enable (CDF3E):  If this bit is set to ‘1’ and the controller
supports Copy Descriptor Format 3h, then Copy Descriptor Format 3h is enabled. If this bit is
cleared to ‘0’, or the controller does not support Copy Descriptor Format 3h, then Copy
Descriptor Format 3h is disabled.
2
Copy Descriptor Format 2h Enable (CDF2E):  If this bit is set to ‘1’ and the controller
supports Copy Descriptor Format 2h, then Copy Descriptor Format 2h is enabled. If this bit is
cleared to ‘0’, or the controller does not support Copy Descriptor Format 2h, then Copy
Descriptor Format 2h is disabled.
1:0
Not Used (NUSED): The controller shall ignore these bits when processing a Set Features
command that specifies this Feature and shall clear these bits to ‘0’ in the attributes returned
in the data buffer for a Get Features command that specifies this Feature.

511:06 Reserved
5.1.25.1.15 Sanitize Config (Feature Identifier 17h)
This Feature controls behavior of the Sanitize command and sanitize operations. The scope of this Feature
is the NVM subsystem.
The attributes are specified in Command Dword 11.
If a Get Features command is submitted for this Feature, the attributes specified in Figure 408 are returned
in Dword 0 of the completion queue entry for that command.
If this Feature is not saveable (refer to Figure 195), then the default value of the NODRM attribute shall be
cleared to ‘0’ (i.e., No-Deallocate Error Response Mode).
If the capabilities of the Sanitize Config Feature Identifier are both changeable and saveable (refer to
section 4.4), then the host is able to configure this Feature when initially provisioning a device.
Figure 408: Sanitize Config – Command Dword 11
Bits Description
31:01 Reserved
00
No-Deallocate Response Mode (NODRM): If the No-Deallocate Inhibited bit) is set to ‘1’ in the Sanitize
Capabilities field of the Identify Controller data structure (refer to Figure 312), then this bit defines the
response of the controller to a Sanitize command processed with the No-Deallocate After Sanitize
(NDAS) bit set to ‘1’ (refer to Figure 372).
If this bit is set to ‘1’ (i.e., No -Deallocate Warning Response Mode), then the controller shall process
such Sanitize commands, and if the resulting sanitize operation is completed successfully, then the SOS
field shall be set to 100b (i.e., Sanitized Unexpected Deallocate) in the Sanitize Status log page (refer to
Figure 291).
If this bit is cleared to ‘0’ (i.e., No-Deallocate Error Response Mode), then the controller shall abort such
Sanitize commands with a status code of Invalid Field in Command.
If the No-Deallocate Inhibited bit in the Sanitize Capabilities field of the Identify Controller data structure
(refer to Figure 312) is cleared to ‘0’, then this bit has no effect.
