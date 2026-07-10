---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 259
headings: []
---

# Source page 259

NVM Express® Base Specification, Revision 2.1

237
Figure 227: Persistent Event Format
Bytes Description
03
Event Header Additional Information (EHAI): This field indicates if additional
information is present in this event header.

Bits Description
7:2 Reserved
1:0
Port Identifier Type (PIT): This field indicates the type of port identifier reported
in the Port Identifier (PELPID) field. Implementations that are compliant with NVM
Express Base Specification Revision 2.0 and later shall set this field to a non-zero
value.
Value Definition
00b The Port Identifier Type is not reported and the Port Identifier
(PELPID) field does not apply.
01b This event is associated with an NVM subsystem port.
10b This event is associated with a  Management Endpoint (refer to
the NVM Express Management Interface Specification).
11b This event is not associated with any port.

05:04
Controller Identifier (CNTLID): This field contains the NVM subsystem unique controller
identifier for the controller that created this event. If the event is controller specific, then the
event data is associated with that controller. If the event is not controller specific, then this is
the controller that the NVM subsystem selected for creating the event.
13:06 Event Timestamp (ETSTP): This field contains a timestamp of the time when this event
occurred using the Timestamp data structure defined in Figure 395.
15:14
Port Identifier (PELPID): If the PIT field in the EHAI field is cleared to 0 0b or set to 11b,
then this field shall be cleared to 0h.
If the PIT field in the EHAI field is not cleared to 0 0b or set to 11b, then this field contains a
Port Identifier of the type indicated in the PIT field.
If the PIT field in the EHAI field is set to 01b, then this field shall contain the Port Identifier of
the NVM subsystem port associated with this event as defined in the Primary Controller
Capabilities data structure (refer to section 5.1.13.3.1).
If the PIT field in the EHAI field is set to 10b, then:
• the least -significant byte of this field shall contain the Port Identifier of the
Management Endpoint associated with this event  (refer to the NVM Express
Management Interface Specification); and
• the most-significant byte of this field shall be cleared to 0h.
19:16 Reserved
21:20
Vendor Specific Information Length (VSIL): This field indicates the length in bytes of the
Vendor Specific Information. If no Vendor Specific Information is present, then this field shall
be cleared to 0h. The length of the Vendor Specific Information is included in the Event
Length field (bytes 2 3:22). Information associated with this event that is not able to be
described in the event data structure fields may be reported in Vendor Specific Information
fields in this event.
23:22
Event Length (EL): This field indicates the length in bytes of the vendor specific information,
if any, and the persistent event log event data that follows. The total length of the event in
bytes is the value in this field plus the value in the Event Header Length field plus 3.
Vendor Specific Information, if any
EHL+2+VSIL:EHL+3
Vendor Specific Information  (VSI): This field contains the vendor specific information, if
any. This field is omitted if there is no vendor specific information (i.e., if VSIL is cleared to
0h).
Persistent Event Log Event Data
EHL+EL+2:
EHL+3+VSIL
Event Data (ED): This field contains persistent event log event data, if any (refer to section
5.1.12.1.14.1).
