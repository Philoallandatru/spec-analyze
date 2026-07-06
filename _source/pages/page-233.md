---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 233
headings: []
---

# Source page 233

NVM Express® Base Specification, Revision 2.1

211
Figure 206: SMART / Health Information Log Page
Bytes Description
159:144
Unexpected Power Losses (UPL): Contains a count of the number of unexpected power losses. This
count shall be incremented if, and only if, main power is lost when:
a) the controller does not report it is ready to be powered off (i.e., the CSTS.SHST field is not set
to 10b); or
b) media is not in  the shutdown state (refer to Figure 85) because an Admin command that
accesses media as defined by  Figure 84 was processed via the out -of-band mechanism with
the Ignore Shutdown bit set to ‘1’ (refer to the NVM Express Management Interface
Specification) while shutdown processing was reported as in progress or was reported as
complete (i.e., the CSTS.SHST field was set to 01b or set to 10b).
Note that this field was previously named Unsafe Shutdowns.
If power is lost when the CSTS.SHST field is not set to 10b, then subsequent controller initialization (i.e.,
the amount of time from when the CC.EN bit transitions from ‘0’ to ‘1’ until the CSTS.RDY bit transitions
from ‘0’ to ‘1’) may take longer for any  NVM subsystem, and data corruption may occur for any NVM
subsystem that is not protected against power loss.
If the Controller Power Scope (i.e., CAP.CPS) field is cleared to 00b (i.e., Not Reported) or set to 01b
(i.e., Controller scope), then the controller reports that the controller is ready to be powered off when the
controller is shutdown (i.e., CSTS.SHST field is set to 10b).
If the CAP.CPS field is set to 10b  (i.e., Domain scope), then the controller reports that the domain is
ready to be powered off when all the controllers in that domain are shutdown (e.g., NVM Subsystem
Shutdown processing is complete).
If the CAP.CPS field is set to 11b (i.e., NVM subsystem scope), then the controller reports that the NVM
subsystem is ready to be powered off  when all controllers in the NVM subsystem are shutdown (e.g.,
NVM Subsystem Shutdown processing is complete).
175:160
Media and Data Integrity Errors (MDIE): Contains the number of occurrences where the controller
detected an unrecovered data integrity error. Errors such as uncorrectable ECC, CRC checksum failure,
or LBA tag mismatch are included in this field.  Errors introduced as a result of a Write Uncorrectable
command (refer to the NVM Command Set Specification) may or may not be included in this field.
191:176 Number of Error Information Log Entries  (NEILE): Contains the number of Error Information Log
Entries over the life of the controller.
195:192
Warning Composite Temperature Time (WCTT): If the Temperature Threshold Hysteresis Attributes
(TMPTHHA) field cleared to 0h (refer to Figure 312), then this field contains the amount of time in minutes
that the controller is operational and the Composite Temperature is greater than or equal to the Warning
Composite Temperature Threshold (WCTEMP) field and less than the Critical Composite Temperature
Threshold (CCTEMP) field in the Identify Controller data structure in Figure 312.
If the Temperature Threshold Maximum Hysteresis (TMPTHMH) and Temperature Threshold Hysteresis
(TMPTHH) fields are non-zero, then hysteresis time is included in the time specified in this field (refer to
section 5.1.25.1.3.1).
If the value of the WCTEMP or CCTEMP field is 0h, then this field is always cleared to 0h regardless of
the Composite Temperature value.
199:196
Critical Composite Temperature Time  (CCTT): Contains the amount of time in minutes that the
controller is operational and the Composite Temperature is greater than or equal to the Critical
Composite Temperature Threshold (CCTEMP) field in the Identify Controller data structure in Figure
312.
If the value of the CCTEMP field is 0h, then this field is always cleared to 0h regardless of the Composite
Temperature value.
201:200 Temperature Sensor 1 (TSEN1): Contains the current temperature reported by temperature sensor 1.
This field is defined by Figure 207.
203:202 Temperature Sensor 2 (TSEN2): Contains the current temperature reported by temperature sensor 2.
This field is defined by Figure 207.
205:204 Temperature Sensor 3 (TSEN3): Contains the current temperature reported by temperature sensor 3.
This field is defined by Figure 207.
