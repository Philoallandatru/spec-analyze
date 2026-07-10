---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 247
headings: []
---

# Source page 247

NVM Express® Base Specification, Revision 2.1

225
Figure 218: Endurance Group Information Log Page
Bytes Description
00
Endurance Group Critical Warning (EGCW): This field indicates critical warnings for the state of
the Endurance Group. Each bit corresponds to a critical warning type; multiple bits may be set to ‘1’.
If a bit is cleared to ‘0’, then that critical warning does not apply. Critical warnings may result in an
asynchronous event notification to the host. Bits in this field represent the state at the time the Get
Log Page command is processed and may not reflect the state at the time a related asynchronous
event notification, if any, occurs or occurred.
If a bit is set to ‘1’ in all Endurance Groups in the NVM subsystem, then the corresponding bit shall
be set to ‘1’ in the Critical Warning field of the SMART / Health Information log page (refer to Figure
206).
Bits Description
7:4 Reserved
3
Endurance Group Read-Only (EGRO): If this bit is set to ‘1’, then all namespaces in
the Endurance Group have been placed in read only mode for reasons other than a
change in the write protect state of the namespace. The controller shall not set this bit
to '1' if the read -only condition on the Endurance Group is a result of a change in the
write protection state of all namespaces in the Endurance Group.
2
Endurance Group Degraded Reliability (EGDR): If this bit is set to ‘1’, then the
Endurance Group reliability has been degraded due to significant media related errors
or any internal error that degrades NVM subsystem reliability.
1 Reserved
0 Endurance Group Available Spare Below (EGASB): If this bit is set to ‘1’, then the
available spare capacity of the Endurance Group has fallen below the threshold.

01
Endurance Group Features (EGFEAT): This field defines features of the Endurance Group.
Bits Description
7:1 Reserved
0
Endurance Group Media ( EGRMEDIA): If this bit is set to ‘1’ , then the Endurance
Group stores data on rotational media (refer to section 8.1.23). If this bit is cleared to
‘0’, then the Endurance Group does not store data on rotational media.

02 Reserved
03 Available Spare (AVSP): Contains a normalized percentage (0% to 100%) of the remaining spare
capacity available for the Endurance Group.
04
Available Spare Threshold (AVSPT): If the Available Spare falls below the threshold indicated in
this field, an asynchronous event completion may occur. The value is indicated as a normalized
percentage (0% to 100%). The values 101 to 255 are reserved.
05
Percentage Used (PUSED): Contains a vendor specific estimate of the percentage of life used for
the Endurance Group based on the actual usage and the manufacturer’s prediction of NVM life.  A
value of 100 indicates that the estimated endurance of the NVM in the Endurance Group has been
consumed, but may not indicate an NVM failure.  The value is allowed to exceed 100.  Percentages
greater than 254 shall be represented as 255.  This value shall be updated once per power-on hour
when the controller is not in a sleep state.
Refer to the JEDEC JESD218 B-02 standard for SSD device life and endurance measurement
techniques.
07:06
Domain Identifier (DID): This field indicates the identifier of the domain that contains this Endurance
Group. If the NVM subsystem supports multiple domains, this field shall be set to a non -zero value.
If this field is cleared to 0h, then the NVM subsystem does not support multiple domains.
31:08 Reserved
47:32
Endurance Estimate (EE): This field is an estimate of the total number of data bytes that may be
written to the Endurance Group over the lifetime of the Endurance Group assuming a write
amplification of 1 (i.e., no increase in the number of write operations performed by the device beyond
the number of write operations requested by a host). This value is reported in billions (i.e., a value of
1 corresponds to 1,000,000,000 bytes written) and is rounded up (e.g., one indicates the number of
bytes written is f rom 1 to 1,000,000,000, three indicates the number of bytes written is from
2,000,000,001 to 3,000,000,000).
A value of 0h indicates that the controller does not report an Endurance Estimate.
