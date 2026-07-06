---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 263
headings: []
---

# Source page 263

NVM Express® Base Specification, Revision 2.1

241
An NVM Subsystem Hardware Error event shall be recorded in the Persistent Event Log when a supported
NVM subsystem hardware error event is detected. Which of the NVM subsystem hardware error events
are supported is vendor specific. The NVM subsystem hardware error event shall set the Persistent Event
Log Event Format Header:
• Event Type field to 05h; and
• Event Type Revision Field to 02h.
All detected NVM Subsystem Hardware Error events supported by the NVM subsystem shall be logged
unless otherwise specified (e.g., suppressed due to reoccurrence frequency (refer to section 5.1.12.1.14)).
NVM Subsystem Hardware Error event fields reporting information that is not available (e.g., due to a PCIe
optional feature that is not implemented) shall be cleared to 0h unless otherwise specified in the NVM
Subsystem Hardware Error Event code description.
The NVM Subsystem Hardware Error Event data is specified in Figure 235.
Figure 235: NVM Subsystem Hardware Error Event Format (Event Type 05h)
Bytes Description
01:00 NVM Subsystem Hardware Error Event Code  (NSHEEC): This field contains a code (refer to
Figure 236) indicating the type of NVM subsystem hardware error that is being reported.
03:02 Reserved
M+3:04
Additional Hardware Error Information (AHEI): This field contains additional information about
the hardware error event indicated in the NVM Subsystem Hardware Error Event Code field (refer
to Figure 236). Where M is the number of bytes of additional hardware error information.
This field is omitted if the subsystem hardware error being reported does not contain additional
hardware error information (i.e., if the number of bytes of additional hardware error information, M,
is 0h).

Figure 236: NVM Subsystem Hardware Error Event Codes
Value Definition
00h Reserved
01h
PCIe Correctable Error: Indicates that the NVM subsystem has detected that a PCIe correctable
error occurred.
Refer to Figure 238 for the format of the Additional Hardware Error Information field.
02h
PCIe Uncorrectable Non fatal Error: Indicates that the NVM subsystem has detected that a PCIe
uncorrectable non-fatal error occurred.
Refer to Figure 238 for the format of the Additional Hardware Error Information field.
03h
PCIe Uncorrectable Fatal Error:  Indicates that the NVM subsystem has detected that a PCIe
uncorrectable fatal error occurred.
Refer to Figure 238 for the format of the Additional Hardware Error Information field.
04h
PCIe Link Status Change: Indicates that a change in the values reported in the PCI Express Link
Status register (refer to the PCI Express Link Status section in the NVMe over PCIe Transport
Specification) have changed due to an attempt to correct unreliable link operation.
The Additional Hardware Error Information field shall be set to the contents of the PCI Express Link
Status register at the time of the event.
05h
PCIe Link Not Active: Indicates that the Data Link Control and Management State Machine (refer
to PCI Express Base Specification) has transitioned out of the DL_Active state without a
corresponding event (e.g., without an indication from the host that the link is to be disabled).
This NVM subsystem hardware error event does not contain additional hardware error information.
