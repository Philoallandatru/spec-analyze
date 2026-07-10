---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 397
headings: ["5.1.25.1.3.1 Temperature Threshold Hysteresis"]
---

# Source page 397

NVM Express® Base Specification, Revision 2.1

375
The over temperature threshold feature shall be implemented for Composite Temperature. The under
temperature threshold Feature shall be implemented for Composite Temperature if a non -zero Warning
Composite Temperature Threshold (WCT EMP) field value is reported in the Identify Controller data
structure (refer to Figure 312). The over temperature threshold and under temperature threshold features
shall be implemented for all implemented temperature sensors (i.e., all Temperature Sensor fields that
report a non-zero value).
The default value of the over temperature threshold feature for Composite Temperature is the value in the
WCTEMP field in the Identify Controller data  structure if the value of the WCTEMP field is non -zero;
otherwise, the default value is implementation specific. The default value of the under temperature threshold
feature for Composite Temperature is implementation specific.  The default value of the over temperature
threshold for all implemented temperature sensors is FFFFh. The default value of the under temperature
threshold for all implemented temperature sensors is 0h.
If a Get Features command is submitted for this Feature, the temperature threshold selected by Command
Dword 11 is returned in Dword 0 of the completion queue entry for that command.
 Temperature Threshold Hysteresis
This feature allows the host to specify the temperature hysteresis that a controller shall use to determine
the end of a temperature threshold event.
If the controller supports the use of the Temperature Threshold Hysteresis feature as indicated by at non -
zero value of the TMPTHMH field (refer to Figure 312), then the controller shall:
• support this feature for the Composite Temperature value (refer to Figure 389); and
• support the Temperature Threshold Hysteresis Recovery event (refer to Figure 312).
If hysteresis is supported for Threshold Temperature Select values other than Composite Temperature (i.e.,
0h) (refer to Figure 389); then, associated with each implemented temperature sensor is an over -
temperature threshold and an under -temperature threshold with a hysteresis value. If the host specifies
hysteresis for a Threshold Temperature Select that is not supported, the controller shall abort the command
with a status of Invalid Field in Command.
An over-temperature threshold hysteresis event begins when the value of a temperature field (refer to the
temperature values reported as described in section 5.1.25.1.3) transitions from less than its corresponding
temperature threshold to greater than or equal to that temperature threshold. That over -temperature
threshold hysteresis event ends when the value of that temperature field transitions from greater than or
equal to that temperature threshold to less than that temperature threshold minus the value of the
Temperature Threshold Hysteresis (TMPTHH) field.
An under-temperature threshold hysteresis event begins when the value of a temperature field transitions
from greater than its corresponding temperature threshold to less than or equal to that temperature
threshold. That under-temperature threshold hysteresis event ends when the value of that temperature field
transitions from less than or equal to that temperature threshold to greater than that temperature threshold
plus the value of the Temperature Threshold Hysteresis (TMPTHH) field.
When a temperature threshold hysteresis event has started, the Temperature Threshold Condition (TTC)
is set to ‘1’ of the Critical Warning field in the SMART / Health Information log page (refer to section
5.1.12.1.3) is set to ‘1’. This may trigger an initial asynchronous event.
At the end of a temperature threshold hysteresis event, a Temperature Threshold Hysteresis Recovery
event shall be triggered (refer to Figure 391), the TTC bit (refer to section 5.1.12.1.3) shall be cleared to ’0’
and the Warning Composite Temperature Time field shall stop accumulating the number of minutes.
If the value of the Temperature Threshold Hysteresis field is larger than the value supported by the controller
as specified by the TMPTHMH field in Figure 312, the controller shall abort the command with a status code
of Invalid Field in Command.
