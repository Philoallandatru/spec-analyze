---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 281
headings: ["5.1.12.1.18 Feature Identifiers Supported and Effects (Log Page Identifier 12h)"]
---

# Source page 281

NVM Express® Base Specification, Revision 2.1

259
Figure 259: Channel Configuration Descriptor
Bytes Description
Media Unit Configuration Descriptor List
NOTE 1 Media Unit 0 Configuration Descriptor:  This field contains the Media Unit Configuration Descriptor
(refer to Figure 260) for the first Media Unit attached to this Channel, if any.
NOTE 1 Media Unit 1 Configuration Descriptor: This field contains the Media Unit Configuration Descriptor for
the second Media Unit attached to this Channel, if reported.
… …
NOTE 1 Media Unit CHMUS -1 Configuration Descriptor:  This field contains the Media Unit Configuration
Descriptor for the last Media Unit attached to this Channel, if any.
Notes:
1. Media Unit Configuration Descriptors may be different lengths.
The Media Unit Configuration Descriptor is defined in Figure 260.
Figure 260: Media Unit Configuration Descriptor
Bytes Description
1:0 Media Unit Identifier (MUID): This field indicates the identifier of this Media Unit.
5:2 Reserved
7:6
Media Unit Descriptor Length (MUDL):  This field contains the length in bytes of the descriptor
information that follows. The total length of the Media Unit Configuration Descriptor in bytes is the value
in this field plus 8.
This field shall be cleared to 0h.
5.1.12.1.18 Feature Identifiers Supported and Effects (Log Page Identifier 12h)
An NVM subsystem may support several interfaces for submitting a Get Log Page command such as an
Admin Submission Queue, PCIe VDM Management Endpoint, or 2-Wire Management Endpoint (refer the
NVM Express Management Interface Specification for details on Management Endpoints) and may have
zero or more instances of each of those interfaces. The feature identifiers (FIDs) supported on each
instance of each interface  may be different. This log page describes the FIDs that are supported on the
interface to which the Get Log Page command was submitted and the effects of those features on the state
of the NVM subsystem. The log page is defined in Figure 261. Each Feature Identifier’s effects are
described in a FID Supported and Effects data structure defined in Figure 262.
If the UUID Selection Supported bit is set to ‘1’ for the Get Log Page command in the Commands Supported
and Effects log page (refer to section 5.1.12.1.6), then the log page data reflects the FIDs that are supported
based on the value of the UUID Index field (refer to section 8.1.28).
If the Feature Identifiers Supported and Effects log page is supported and the Feature is supported, then
the scope, as defined in Figure 385, shall be indicated in the FID Scope field (FSP) for that Feature (refer
to Figure 262).
For controllers that implement I/O Queues, t he features that the controller supports are dependent on the
I/O Command Set that is based on:
• the I/O Command Set selected in CC.CSS, if CC.CSS is not set to 110b; and
• the Command Set Identifier (CSI) field in CDW 14, if CC.CSS is set to 110b.
If CC.CSS is set to 110b, I/O Command Sets that have not been enabled by the I/O Command Set Profile
(FID 19h) (refer to section 5.1.25.1.17) are treated as unsupported.
Figure 261: Feature Identifiers Effects Log Page
Bytes Description
03:00 Feature Identifier Supported 0 (FIS0): Contains the FID Supported and Effects data structure (refer
to Figure 262) for FID 0h.
