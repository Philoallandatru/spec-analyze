---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 272
headings: ["5.1.12.1.14.2.12 Telemetry Log Create Event (Event Type 0Ch)", "5.1.12.1.14.2.13 Thermal Excursion Event (Event Type 0Dh)"]
---

# Source page 272

NVM Express® Base Specification, Revision 2.1

250
Figure 246: Set Feature Event Data Format
Bytes Description
Memory Buffer Count +
(Dword Count * 4) + 7:
Memory Buffer Count +
(Dword Count * 4) + 4
Command Completion Dword 0 (CCDW0): If the Logged Command Completion
Dword 0 bit is set to ‘1’, then this field contains the Dword  0 value from the Set
Features command completion. If the Logged Command Completion Dword  0 bit
is cleared to ‘0’, then this field is not logged.
5.1.12.1.14.2.12 Telemetry Log Create Event (Event Type 0Ch)
A Telemetry Log Create event may be created if the controller determines that a  Telemetry Host-Initiated
log page (refer to section 5.1.12.1.8) or that a Telemetry C ontroller-Initiated log page (refer to section
5.1.12.1.9) has been generated.
The Telemetry Log Create Event shall set the Persistent Event Log Event Format Header:
• Event Type field to 0Ch; and
• Event Type Revision Field to 01h.
Figure 247: Telemetry Log Create Event Data Format (Event Type 0Ch)
Bytes Description
511:00
Telemetry Initiated Log  (TIL): Contains a copy of the values from the first 512 bytes of the
Telemetry Host-Initiated log page (refer to Figure 214) or the first 512 bytes of the Telemetry
Controller-Initiated log page (refer to Figure 216).
5.1.12.1.14.2.13 Thermal Excursion Event (Event Type 0Dh)
A Thermal Excursion event shall be recorded in the Persistent Event Log if the Composite Temperature
has transitioned from a temperature that is less than:
a) the WCTEMP, if any, (refer to  Figure 312) to a temperature that is greater than or equal to the
WCTEMP, if any; or
b) the CCTEMP, if any, (refer to  Figure 312), to a temperature that is greater than or equal to the
CCTEMP, if any,
unless recording of the event causes a vendor specific frequency of threshold reports for this threshold to
be exceeded.
A Thermal Excursion event may be recorded in the Persistent Event Log if the Composite Temperature has
transitioned from a temperature:
a) that is less than TMT1 (refer to section 5.1.25.1.9), if any, to a temperature that is greater than or
equal to TMT1, if any (i.e., light throttling has started);
b) that is less than TMT2 (refer to section 5.1.25.1.9), if any, to a temperature that is greater than or
equal to TMT2, if any (i.e., heavy throttling has started);
c) that is less than a vendor specific temperature where thermal throttling occurs due to self-throttling
to a temperature that is greater than that vendor specific temperature (i.e., self -throttling has
started);
d) outside of a temperature threshold to a value that is within all temperature thresholds (i.e., the
temperature returns to normal);
e) at which thermal throttling is occurring to a temperature at which thermal throttling is stopped; or
f) that is greater than an under temperature threshold (refer to section 5.1.25.1.3) to a temperature
that is less than or equal to an under temperature threshold,
unless recording of the event causes a vendor specific frequency of threshold reports for this threshold to
be exceeded.
The Thermal Excursion event shall set the Persistent Event Log Event Format Header;
• Event Type field to 0Dh; and
