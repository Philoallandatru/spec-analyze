---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 350
headings: []
---

# Source page 350

NVM Express® Base Specification, Revision 2.1

328
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
1806 R R O
Discovery Controller Type (DCTYPE): This field indicates what type of Discovery
controller the controller is.
Value Definition
0h Discovery controller type is not reported
1h DDC
2h CDC
3h to FFh Reserved

2047:1807    Reserved
Power State Descriptors
2079:2048 M M R Power State 0 Descriptor (PSD0): This field indicates the characteristics of power state
0. The format of this field is defined in Figure 313.
2111:2080 O O R Power State 1 Descriptor (PSD1): This field indicates the characteristics of power state
1. The format of this field is defined in Figure 313.
2143:2112 O O R Power State 2 Descriptor (PSD2): This field indicates the characteristics of power state
2. The format of this field is defined in Figure 313.
…
3071:3040 O O R Power State 31 Descriptor (PSD31): This field indicates the characteristics of power
state 31. The format of this field is defined in Figure 313.
Vendor Specific
4095:3072 O O O Vendor Specific (VS)
Notes:
1. O/M/R definition: O = Optional, M = Mandatory, R = Reserved.
2. Mandatory for message-based controllers. For memory-based controllers this field is reserved.
3. Mandatory for Discovery controllers that support explicit persistent connections. For Discovery controllers that do not support explicit
persistent connections this field is reserved.
4. For Discovery controllers, the TBKAS bit is optional, and all other bits are reserved.
5. If a Discovery controller returns a non -null value for either the Serial Number (SN) or Model Number (MN) fields, that Discovery
controller shall return a non-null value for both of those fields.
Figure 313 defines the power state descriptor that describes the attributes of each power state.  For more
information on how the power state descriptor fields are used, refer to section 8.1.17 on power
management.
Figure 313: Identify – Power State Descriptor Data Structure
Bits Description
255:220 Reserved
219:216
Emergency Power Fail Vault Time Scale (EPFVTS):  This field indicates the scale for the Emergency
Power Fail Vault Time field as defined in Figure 314.
If the EPFVT field is cleared to 0h, then this field:
• shall be cleared to 0h; and
• is ignored by the host.
215:212
Forced Quiescence Vault Time Scale (FQVTS):  This field indicates the scale for the Forced
Quiescence Vault Time field as defined in Figure 314.
If the FQVT field is cleared to 0h, then this field:
• shall be cleared to 0h; and
• is ignored by the host.
