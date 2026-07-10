---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 231
headings: []
---

# Source page 231

NVM Express® Base Specification, Revision 2.1

209
Critical warnings regarding the health of the NVM subsystem may be indicated via an asynchronous event
notification to the host.  The warnings that results in an asynchronous event notification to the host are
configured using the Set Features command; refer to section 5.1.25.1.5.
Performance may be calculated using parameters returned as part of the SMART / Health Information log.
Specifically, the number of Read or Write commands, the amount of data read or written, and the amount
of controller busy time enables both I/Os per second and bandwidth to be calculated.
The log page returned is defined in Figure 206.
Figure 206: SMART / Health Information Log Page
Bytes Description
00
Critical Warning  (CW): This field indicates critical warnings for the state of the controller. Each bit
corresponds to a critical warning type; multiple bits may be set  to ‘1’. If a bit is cleared to ‘0’, then that
critical warning does not apply. Critical warnings may result in an asynchronous event notification to the
host. Bits in this field represent the state at the time the Get Log Page command is processed and may
not reflect the state at the time a related asynchronous event notification, if any, occurs or occurred.
Bits Description
7:6 Reserved
5 Persistent Memory Region Read -Only (PMRRO): If this bit is set to ‘1’, then the
Persistent Memory Region has become read-only or unreliable (refer to section 8.2.4).
4
Volatile Memory Backup Failed (VMBF): If this bit is set to ‘1’, then the volatile memory
backup device has failed. This field is only valid if the controller has a volatile memory
backup solution.
3
All Media Read-Only (AMRO): If this bit is set to ‘1’, then all of the media has been placed
in read only mode.  The controller shall not set this bit to '1' if the read -only condition on
the media is a result of a change in the write protection state of a namespace (refer to
section 8.1.16.1).
2
NVM Subsystem Degraded Reliability (NDR): If this bit is set to ‘1’, then the NVM
subsystem reliability has been degraded due to significant media related errors or any
internal error that degrades NVM subsystem reliability.
1
Temperature Threshold Condition (TTC): If this bit is set to ‘1’, then a temperature is:
a) greater than or equal to an over temperature threshold; or
b) less than or equal to an under temperature threshold,
(refer to section 5.1.25.1.3).
0 Available Spare Capacity Below Threshold (ASCBT): If this bit is set to ‘1’, then the
available spare capacity has fallen below the threshold.

02:01
Composite Temperature (CTEMP): Contains a value corresponding to a temperature in Kelvin s that
represents the current composite temperature of the controller and namespace(s) associated with that
controller. The manner in which this value is computed is implementation specific and may not represent
the actual temperature of any physical point in the NVM subsystem. The value of this field may be used
to trigger an asynchronous event (refer to section 5.1.25.1.3).
Warning and critical overheating composite temperature threshold values are reported by the WCTEMP
and CCTEMP fields in the Identify Controller data structure in Figure 312.
03 Available Spare  (AVSP): Contains a normalized percentage (0 % to 100%) of the remaining spare
capacity available.
04
Available Spare Threshold (AVSPT): When the Available Spare falls below the threshold indicated in
this field, an asynchronous event completion may occur . The value is indicated as a normalized
percentage (0% to 100%). The values 101 to 255 are reserved.
05
Percentage Used (PUSED): Contains a vendor specific estimate of the percentage of NVM subsystem
life used based on the actual usage and the manufacturer’s prediction of NVM life. A value of 100
indicates that the estimated endurance of the NVM in the NVM subsystem has been consumed, but may
not indicate an NVM subsystem failure. The value is allowed to exceed 100.  Percentages greater than
254 shall be represented as 255.  This value shall be updated once per power -on hour (when the
controller is not in a sleep state).
Refer to the JEDEC JESD218 B-02 standard for SSD device life and endurance measurement
techniques.
