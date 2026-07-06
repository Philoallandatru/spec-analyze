---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 258
headings: []
---

# Source page 258

NVM Express® Base Specification, Revision 2.1

236
Figure 226: Persistent Event Log Page
Bytes Description
511:480
Supported Events Bitmap  (SEB): This field contains a bitmap indicating support for the
persistent event log events. Each bit in the bitmap corresponds to the value for the event type
(refer to Figure 229) (e.g., bit 222 decimal, DEh, corresponds to event type value DEh, Vendor
Specific Event). A bit set to ‘1’ indicates that the corresponding event is supported. A bit cleared
to ‘0’ indicates that the corresponding event is not supported.
Bits Description Reference
255:224 Reserved
223 TCG Defined
TCG Storage
Interface Interactions
Specification
222 Vendor Specific Event Supported (VSES) 5.1.12.1.14.2.15
221:15 Reserved
14 Sanitize Media Verification Event Support
(SMVES) 5.1.12.1.14.2.14
13 Thermal Excursion Event Support (TEES) 5.1.12.1.14.2.13
12 Telemetry Log Create Event Support (TLCES) 5.1.12.1.14.2.12
11 Set Feature Event Support (SFES) 5.1.12.1.14.2.11
10 Sanitize Completion Event Support (SCES) 5.1.12.1.14.2.10
09 Sanitize Start Event Support (SSES) 5.1.12.1.14.2.9
08 Format NVM Completion Event Support (FNCES) 5.1.12.1.14.2.8
07 Format NVM Start Event Support (FNSES) 5.1.12.1.14.2.7
06 Change Namespace Event Support (CNES) 5.1.12.1.14.2.6
05 NVM Subsystem Hardware Error Event Support
(NSHEES) 5.1.12.1.14.2.5
04 Power-on or Reset Event Supported (PRES) 5.1.12.1.14.2.4
03 Timestamp Change Event Supported (TCES) 5.1.12.1.14.2.3
02 Firmware Commit Event Supported (FCES) 5.1.12.1.14.2.2
01 SMART / Health Log Snapshot Event Supported
(SHLSES) 5.1.12.1.14.2.1
00 Reserved

Persistent Event Log Events
(M-1)+512:512 Persistent Event 0: This field contains the first persistent event log entry (refer to Figure 227)
where M is the length of this persistent event.
… …
(TLL-1):(TLL-K)
Persistent Event N: This field contains the last persistent event log entry (refer to Figure 227)
where K is the length of this persistent event , TLL is the size specified in the Total Log Length
field, and N is the value of the TNEV field minus 1h.
The format of the Persistent Events in the Persistent Event log is shown in Figure 227.
Figure 227: Persistent Event Format
Bytes Description
Persistent Event Log Event Header
00 Event Type  (ET): This field indicates the event type for this entry. Refer to section
5.1.12.1.14.1 for the definition of the event types.
01
Event Type Revision (ETR): This field contains a number indicating the revision of the event
data structure for the event indicated by the Event Type field that this event data complies
with.
02
Event Header Length (EHL):  This field contains the length in bytes of the event header
information that follows. The total length of the event header in bytes is the value in this field
plus 3. The host should use the value in this field to calculate the offset to the start of the
Vendor Specific Information field.
