---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 273
headings: ["5.1.12.1.14.2.14 Sanitize Media Verification Event (Event Type 0Eh)", "5.1.12.1.14.2.15 Vendor Specific Event (Event Type DEh)"]
---

# Source page 273

NVM Express® Base Specification, Revision 2.1

251
• Event Type Revision field to 01h.
Figure 248: Thermal Excursion Event Data Format (Event Type 0Dh)
Bytes Description
0 Over Temperature (OTMP): Contains the absolute value of the difference (i.e., delta) in Kelvins between
the temperature indicated in the threshold field and temperature measured at the time of the event.
1
Threshold (THRESH): Contains an indicator for the temperature threshold crossing that is being reported.
Value Definition
High Temperature Transitions
1h The Composite Temperature has transitioned from a temperature that is less than
WCTEMP to a temperature that is greater than or equal to WCTEMP.
2h The Composite Temperature has transitioned from a temperature that is less than
CCTEMP to a temperature that is greater than or equal to CCTEMP.
3h
The Composite Temperature has transitioned from a temperature that is less than
TMT1 to a temperature is greater than or equal to TMT1 (i.e., vendor specific thermal
management actions that minimize the impact on performance, such as light throttling,
have started).
4h
The Composite Temperature has transitioned from a temperature that is less than
TMT2 to a temperature that is greater than or equal to TMT2 (i.e., vendor specific
thermal management actions that may impact performance, such as heavy throttling,
have started).
5h
The Composite Temperature has transitioned from a temperature where no vendor
specific thermal management actions are taken to a temperature that is greater than
or equal to a vendor specific temperature at which vendor specific thermal
management actions have started (e.g., self-throttling).
Normal Temperature Transitions
88h
The Composite Temperature has transitioned from a temperature that is greater than
or equal to WCTEMP or is less than or equal to an under temperature threshold to a
temperature that is between WCTEMP and that under temperature threshold (i.e., the
temperature has transitioned to a normal temperature).
89h
The Composite Temperature has transitioned from a temperature that is greater than
a temperature where thermal management actions are being performed and that is not
greater than or equal to WCTEMP to a temperature where thermal management
actions are stopped.
Low Temperature Transitions
B0h
The Composite Temperature has transition from a temperature that is greater than an
under temperature threshold to a temperature that is less than or equal to an under
temperature threshold.
F0h to FFh Vendor specific
All other
values Reserved

5.1.12.1.14.2.14 Sanitize Media Verification Event (Event Type 0Eh)
A Sanitize Media Verification event shall be recorded in the Persistent Event log page upon transition to
the Media Verification state (refer to section 8.1.24.3.6).
No Event Data (refer to Figure 227) is defined for this event.
The Sanitize Media Verification event shall set the Persistent Event Log Event Format Header:
• Event Type field to 0Eh;
• Event Type Revision field to 01h; and
• Event Length field to 0h.
5.1.12.1.14.2.15 Vendor Specific Event (Event Type DEh)
The Vendor Specific Event (refer to Figure 249) contains a set of Vendor Specific Event Descriptors that
describe an event that the vendor has determined is a significant event which should be reported to a host
in the persistent event log and that is not described by any of the other persistent event  log events.
