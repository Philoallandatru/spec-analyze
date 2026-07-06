---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 274
headings: []
---

# Source page 274

NVM Express® Base Specification, Revision 2.1

252
The Vendor Specific Event Descriptors follow the format shown in Figure 250 and contain vendor specific
data of the type indicated in the Vendor Specific Event Data Type field of the Vendor Specific Event
Descriptor.
If a UUID Index is specified in the Get Log Page command (refer to section 5.1.12), then the controller shall
return:
a) Vendor specific events based on that UUID index; and
b) Vendor specific events defined by the NVM subsystem manufacturer.
The controller shall set the Vendor Specific Event Format Header:
a) Event Type field to DEh; and
b) Event Type Revision field to 01h.
The Vendor Specific Event data is specified in Figure 249.
Figure 249: Vendor Specific Event Format (Event Type DEh)
Bytes Description
Vendor Specific Event Descriptor List
M-1:0 Vendor Specific Event Descriptor 0: Contains the first vendor specific event descriptor (refer to
Figure 250). Where M is the length of this vendor specific event descriptor.
… …
EL-VSIL-1:
EL-VSIL-K 1
Vendor Specific Event Descriptor N: Contains the last vendor specific event descriptor (refer to
Figure 250). Where K is the length of this vendor specific event descriptor (refer to Figure 250).
Notes:
1. Refer to Figure 227 for the definitions of EL and VSIL.
The format of the Vendor Specific Event Descriptor is shown in Figure 250.
Figure 250: Vendor Specific Event Descriptor Format
Bytes Description
01:00
Vendor Specific Event Code  (VSEC): Contains a vendor specific code that uniquely identifies
the type of event that is described in the data that follows. All vendor specific events of the same
type should report the same Vendor Specific Event Code field value.
02 Vendor Specific Event Data Type (VSEDT): Contains a code indicating the type of data reported
in the Vendor Specific Event Data field (refer to Figure 251).
03 UUID Index (UIDX): UUID Index (refer to Figure 658) at the time of this event for the vendor that
defined this event.
05:04 Vendor Specific Event Data Length (VSEDL) : Contains the length in bytes of the Vendor
Specific Event Data field.
Vendor Specific Event Data, if any (i.e., VSEDL > 0)
VSEDL+5:06 Vendor Specific Event Data (VSED): Contains vendor specific data that is associated with this
event and is of the type specified in the Vendor Specific Event Data Type field.
The Vendor Specific Event Data Types that are able to be reported in a Vendor Specific Event Descriptor
are shown in Figure 251.
Figure 251: Vendor Specific Event Data Type Codes
Value Description
00h Reserved
