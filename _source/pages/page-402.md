---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 402
headings: ["5.1.25.1.7 Timestamp (Feature Identifier 0Eh)"]
---

# Source page 402

NVM Express® Base Specification, Revision 2.1

380
5.1.25.1.7 Timestamp (Feature Identifier 0Eh)
This Feature enables the host to set a timestamp value in the controller.  A controller indicates support for
the Timestamp feature through the Optional NVM Command Support (ONCS) field in the Identify Controller
data structure. The Timestamp field value (refer to Figure 395) in a Set Features command sets a timestamp
value in the controller. After the current value for this Feature is set, the controller updates that value as
time passes. A Get Features command that requests the current value reports the timestamp value in the
controller at the time the Get Features command is processed (e.g., the value set with a Set Features
command for the current value plus the elapsed time since being set).
Note: If the Timestamp feature is saveable (refer to  Figure 195) and the host saves a value , then the
timestamp value restored after a subsequent power on or reset event is the value that was saved (refer to
section 4.4). As a result, the timestamp may appear to move backwards in time.
The accuracy of a Timestamp value after initialization may be affected by vendor specific factors, such as
whether the controller continuously counts after the timestamp is initialized, or whether the controller stops
counting during certain intervals (e.g., non-operational power states). If the controller stops counting during
such intervals, then the Synch bit in the Timestamp – Data Structure for Get Features (refer to Figure 396)
shall be set to ‘1’.
If the controller maintains (i.e., continues to update) the timestamp value across any type of Controller Level
Reset (e.g., across a Controller Reset), then the controller shall also preserve the Timestamp Origin field
(refer to Figure 396) across that type of Controller Level Reset.
If the controller does not maintain the value of the timestamp across the most recent Controller Level Reset,
then the Timestamp field is cleared to 0h due to that Controller Level Reset.
Timestamp values should not be used for security applications.  Other application use of the Timestamp
feature is outside the scope of this specification.
If a Set Features command is issued for this Feature, the data structure specified in Figure 395 is transferred
in the data buffer for that command, specifying the Timestamp value.
Figure 395: Timestamp – Data Structure for Set Features
Bytes Description
05:00 Timestamp (TSTMP): Number of milliseconds that have elapsed since midnight, 01-Jan-1970, UTC. Refer
to ISO 8601 for format requirements.
07:06 Reserved
If a Get Features command is issued for this Feature, the data structure specified in Figure 396 is returned
in the data buffer for that command.
Figure 396: Timestamp – Data Structure for Get Features
Bytes Description
05:00
Timestamp (TSTMP):
If the Timestamp Origin field cleared to 000b, then this field is set to the time in milliseconds since the last
Controller Level Reset.
If the Timestamp Origin field is set to 001b, then this field is set to the last Timestamp value set by the host, plus
the time in milliseconds since the Timestamp was set. If the sum of the Timestamp value set by the host and the
elapsed time exceeds 2^48, the value returned should be reduced modulo 2^48.
If the Synch bit is set to ‘1’, then the Timestamp value may be reduced by vendor specific time intervals not
counted by the controller.
