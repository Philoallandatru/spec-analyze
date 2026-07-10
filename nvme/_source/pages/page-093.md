---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 93
headings: ["3.1.4.23 Offset E04h: PMRCTL – Persistent Memory Region Control"]
---

# Source page 93

NVM Express® Base Specification, Revision 2.1

71
Figure 58: Offset E00h: PMRCAP – Persistent Memory Region Capabilities
Bits Type Reset Description
13:10 RO Impl
Spec
Persistent Memory Region Write Barrier Mechanisms (PMRWBM):  This field lists
mechanisms that may be used to ensure that previous writes to the Persistent Memory
Region have completed and are persistent when the Persistent Memory Region is ready
and operating normally. A bit in this field is set to ‘1’ if the corre sponding mechanism to
ensure persistence is supported. A bit in this field is cleared to ‘0’ if the corresponding
mechanism to ensure persistence is not supported.
At least one bit in this field shall be set to ‘1’.
Bits Description
3:2 Reserved
1
Completion of PMRSTS Read (CPMTSTSR): The completion of a read to
the PMRSTS property shall ensure that all prior writes to the Persistent
Memory Region have completed and are persistent.
0
Completion of Memory Read  (CMR): The completion of a memory read
from any Persistent Memory Region address ensures that all prior writes to
the Persistent Memory Region have completed and are persistent.

9:8 RO Impl
Spec
Persistent Memory Region Time Units (PMRTU): Indicates Persistent Memory Region
time units.
Value Persistent Memory Region Time Units
00b 500 milliseconds
01b minutes
10b to 11b Reserved

7:5 RO Impl
Spec
Base Indicator Register (BIR): This field indicates the Base Address Register (BAR)
that specifies the address and size of the Persistent Memory Region. Values 010b, 011b,
100b, and 101b are valid.
4 RO Impl
Spec
Write Data Support (WDS): If this bit is set to ‘1’, then the controller supports data and
metadata in the Persistent Memory Region for commands that transfer data from the
host to the controller (e.g., Write). If this bit is cleared to ‘0’, then data and metadata for
commands that transfer data from the host to the controller shall not be transferred to
the Persistent Memory Region.
If PMRCAP.CMSS is cleared to ‘0’, this bit shall be cleared to ‘0’.
3 RO Impl
Spec
Read Data Support (RDS): If this bit is set to ‘1’, then the controller supports data and
metadata in the Persistent Memory Region for commands that transfer data from the
controller to the host (e.g., Read). If this bit is cleared to ‘0’, then all data and metadata
for commands that transfer data from the controller to the host shall not be transferred
from the Persistent Memory Region.
If PMRCAP.CMSS is cleared to ‘0’, this bit shall be cleared to ‘0’.
2:0 RO 000b Reserved
 Offset E04h: PMRCTL – Persistent Memory Region Control
This optional property controls the operation of the Persistent Memory Region. If the controller does not
support the Persistent Memory Region feature, then this property shall be cleared to 0h.
This property shall not be reset by Controller Reset.
Figure 59: Offset E04h: PMRCTL – Persistent Memory Region Control
Bits Type Reset Description
31:1 RO 0h Reserved
0 RW 0b
Enable (EN): When set to ‘1’, then the Persistent Memory Region is ready to process
PCI Express memory read and write requests once PMRSTS.NRDY is cleared to ‘0’.
When cleared to ‘0’, then the Persistent Memory Region is disabled and PMRSTS.NRDY
shall be set to ‘1’ once the Persistent Memory Region is ready to be re-enabled.
