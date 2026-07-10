---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 396
headings: ["5.1.25.1.2 Power Management (Feature Identifier 02h)", "5.1.25.1.3 Temperature Threshold (Feature Identifier 04h)"]
---

# Source page 396

NVM Express® Base Specification, Revision 2.1

374
If a Get Features command is submitted for this Feature, the attributes specified in Figure 386 are returned
in Dword 0 of the completion queue entry for that command.
Figure 386: Arbitration & Command Processing – Command Dword 11
Bits Description
31:24 High Priority Weight (HPW): This field defines the number of commands that may be executed from
the high priority service class in each arbitration round. This is a 0’s based value.
23:16 Medium Priority Weight (MPW): This field defines the number of commands that may be executed from
the medium priority service class in each arbitration round. This is a 0’s based value.
15:08 Low Priority Weight (LPW): This field defines the number of commands that may be executed from the
low priority service class in each arbitration round. This is a 0’s based value.
07:03 Reserved
02:00
Arbitration Burst (AB): Indicates the maximum number of commands that the controller may fetch at
one time from a particular Submission Queue.  The value is expressed as a power of two (e.g., 000b
indicates one, 011b indicates eight). A value of 111b indicates no limit.
5.1.25.1.2 Power Management (Feature Identifier 02h)
This Feature allows the host to configure the  controller power state . The attributes are specified in
Command Dword 11 (refer to Figure 387).
Upon successful completion of a Set Features command for this Feature, the controller shall be in the
Power State specified. For a transition to a non-operational power state, the device may exceed the power
indicated for that non -operational power state as defined in section 8.1.17.1 (e.g., while completing this
command). If enabled, autonomous power state transitions continue to occur from the new state.
If a Get Features command is submitted for this Feature, the attributes described in Figure 388 are returned
in Dword 0 of the completion queue entry for that command.
Figure 387: Power Management – Command Dword 11
Bits Description
31:08 Reserved
07:05 Workload Hint (WH):  This field indicates the type of workload expected. This hint may be used to
optimize performance. Refer to section 8.1.17.3 for more details.
04:00
Power State (PS): This field indicates the new power state into which the controller  is requested to
transition. This power state shall be one supported by the controller as indicated in the Number of Power
States Supported (NPSS) field in the Identify Controller data structure. If the power state specified is not
supported, the controller shall abort th e command and should return an error of Invalid Field in
Command.

Figure 388: Power Management – Completion Queue Entry Dword 0
Bits Description
31:08 Reserved
07:05 Workload Hint (WH): This field indicates the type of workload. Refer to section 8.1.17.3 for more details.
04:00 Power State (PS): This field indicates the current power state of the controller, or the power state into
which the controller is transitioning.
5.1.25.1.3 Temperature Threshold (Feature Identifier 04h)
A controller may report up to nine temperature values in the SMART / Health Information log page (i.e., the
Composite Temperature and Temperature Sensor 1 through Temperature Sensor 8 ; refer to Figure 206).
Associated with each implemented temperature sensor is an over temperature threshold and an under
temperature threshold. When a temperature is greater than or equal to its corresponding over temperature
threshold or less than or equal to its correspondi ng under temperature threshold, then the Temperature
Threshold Condition (TTC) bit is set to ‘1’ of the Critical Warning field in the SMART / Health Information
log page (refer to section 5.1.12.1.3). This may trigger an asynchronous event.
