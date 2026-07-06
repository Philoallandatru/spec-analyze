---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 333
headings: []
---

# Source page 333

NVM Express® Base Specification, Revision 2.1

311
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
263 M M R
Number of Power States Support (NPSS): This field indicates the number of NVM
Express power states supported by the controller. This is a 0’s based value. Refer to
section 8.1.17.
Power states are numbered sequentially starting at power state 0. A controller shall
support at least one power state (i.e., power state 0) and may support up to 31 additional
power states (i.e., up to 32 total).
264 M M R
Admin Vendor Specific Command Configuration (AVSCC): This field indicates the
configuration settings for vendor specific Admin  command handling. Refer to section
8.1.26.
 Bits Description
7:1 Reserved
0
Vendor Specific Command Format (VSCF): If this bit is set to ‘1’, then all
vendor specific Admin commands use the format defined in Figure 93. If this
bit is cleared to ‘0’, then the format of all vendor specific Admin commands
is vendor specific.

265 O O R
Autonomous Power State Transition Attributes (APSTA): This field indicates the
attributes of the autonomous power state transition feature. Refer to section 8.1.17.2.
 Bits Description
7:1 Reserved
0
Autonomous Power Transition Support (APTS): If this bit is set to ‘1’, then
the controller supports autonomous power state transitions. If this bit is
cleared to ‘0’, then the controller does not support autonomous power state
transitions.

267:266 M M R
Warning Composite Temperature Threshold (WCTEMP): This field indicates the
minimum Composite Temperature field value (reported in the SMART / Health
Information log page in Figure 206) that indicates an overheating condition during which
controller operation continues. Immediate remediation is recommended (e.g., additional
cooling or workload reduction). The platform should strive to maintain a composite
temperature less than this value.
A value of 0h in this field indicates that no warning temperature threshold value is reported
by the controller. Implementations compliant with NVM Express Base Specification ,
Revision 1.2 or later shall report a non-zero value in this field.
It is recommended that implementations report a value of 0157h in this field.
269:268 M M R
Critical Composite Temperature Threshold (CCTEMP):  This field indicates the
minimum Composite Temperature field value (reported in the SMART / Health
Information log page in Figure 206) that indicates a critical overheating condition (e.g.,
may prevent continued normal operation, possibility of data loss, automatic device
shutdown, extreme performance throttling, or permanent damage).
A value of 0h in this field indicates that no critical temperature threshold value is reported
by the controller. Implementations compliant with NVM Express Base Specification ,
Revision 1.2 or later shall report a non-zero value in this field.
271:270 O O R
Maximum Time for Firmware Activation (MTFA): This field indicates the maximum time
the controller temporarily stops processing commands to activate the firmware image.
This field shall be valid if the controller supports firmware activation without a reset. This
field is specified in 100 millisecond units. A value of 0h indicates that the maximum time
is undefined.
For the amount of time to process a Firmware Commit command that specifies a value
of 011b in the Commit Action field (i.e., firmware activation without reset), refer to the
Maximum Processing Time for Firmware Activation Without Reset field.
