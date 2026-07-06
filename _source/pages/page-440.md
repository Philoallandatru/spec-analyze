---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 440
headings: ["5.1.27.2 Command Completion", "5.2 Memory-Based Transport Admin Commands (PCIe)"]
---

# Source page 440

NVM Express® Base Specification, Revision 2.1

418
• the configuration within the controller associated with the controller specified by the CNTLID field
changes resulting in that address range is no longer specifying host memory associated with the
controller specified by the CNTLID field (e.g., the address range overlaps with PMR or CMB), then
the behavior for tracking host memory changes for the controller specified by the CNTLID field is
undefined.
A host should not specify MSI or MSI -X registers in the address range. If the address range specified by
the Address field and the Length field contains addresses for MSI or MSI -X registers on the controller
specified by the CNTLID field, then the controll er may or may not report modification to those MSI or MSI -
X register addresses.
Figure 470: Track Memory Changes – Completion Queue Entry Dword 0
Bits Description
31:16 Reserved
15:00
Memory Range Tracking Granularity (MRTG): This field indicates the lowest granularity that the controller
is able track the memory changes with the specified Memory Range Tracking Descriptor list (refer to Figure
468).
This units for this field is defined by the RMRTG field.

Figure 471: Track Memory Changes – Completion Queue Entry Dword 1
Bits Description
31:00
Number of Memory Range Tracking Descriptors (NMRTD):  This field indicates the number of
Memory Range Tracking Descriptors for which the controller is able to track the memory changes
with the requested granularity specified in the RMRTG field (refer to Figure 468).
 Command Completion
Upon completion of the Track Send command, the controller posts a completion queue entry to the Admin
Completion Queue indicating the status for the command. Section 5.1.27.1 describes completion details
each for management operation.
The use of Dword 0 and Dword 1 of the completion queue entry is specific t o the management operation
specified in the Select field (refer to Figure 463). Refer to each management operation description in section
5.1.27.1 for details.
Track Send command specific status values (i.e., SCT field set to 1h) are shown in Figure 482.
Figure 472: Track Send – Command Specific Status Values
Value Definition
1Fh Invalid Controller Identifier: The controller for the specified Controller Identifier is already
tracking host memory changes.
37h
Invalid Controller Data Queue: This error indicates:
• that the specified Controller Data Queue Identifier is invalid for the controller processing
the command; or
• the controller associated with the Controller Data Queue specified by the Controller
Data Queue Identifier is already logging user data changes.
38h Not Enough Resources: This error indicates that there are not enough resources in the
controller to process the command successfully.
39h Controller Suspended: The requested operation is not allowed if the controller specified in the
Controller Identifier field is suspended by a Migration Send command (refer to section 5.1.17.1.1).
5.2 Memory-Based Transport Admin Commands (PCIe)
This section describes Admin commands that are specific to the Memory-based transport model.
