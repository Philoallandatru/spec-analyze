---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 94
headings: ["3.1.4.24 Offset E08h: PMRSTS – Persistent Memory Region Status"]
---

# Source page 94

NVM Express® Base Specification, Revision 2.1

72
 Offset E08h: PMRSTS – Persistent Memory Region Status
This optional property provides the status of the Persistent Memory Region. If the controller does not
support the Persistent Memory Region feature, then this property shall be cleared to 0h.
This property shall not be reset by Controller Reset.
Figure 60: Offset E08h: PMRSTS – Persistent Memory Region Status
Bits Type Reset Description
31:13 RO 0h Reserved
12 RO 0b
Controller Base Address Invalid (CBAI): This field indicates whether the controller has
failed to enable the Persistent Memory Region’s controller memory space because the
controller 64-bit base address specified by PMRMSCU.CBA and PMRMSCL.CBA  are
invalid. If PMRCAP.CMSS is set to ‘1’, PMRMSCL.CMSE is set to ‘1’, and the controller
64-bit base address specified by PMRMSCU.CBA and PMRMSCL.CBA is invalid, this
bit shall be set to ‘1’. Otherwise, this bit shall be cleared to ‘0’.
11:9 RO 000b
Health Status (HSTS): If the Persistent Memory Region is ready, then this field indicates
the health status of the Persistent Memory Region. This field is always cleared to 000b
when the Persistent Memory Region is not ready.
The health status values are defined as:
Value Definition
000b Normal Operation:  The Persistent Memory Region is operating
normally.
001b
Restore Error:  The Persistent Memory Region is operating
normally and is persistent; however, the contents of the Persistent
Memory Region may not have been restored correctly (i.e., may not
contain the contents prior to the last power cycle, NVM Subsystem
Reset, Controller Level Reset, or Persistent Memory Region
disable).
010b
Read Only:  The Persistent Memory Region is read only. PCI
Express memory write requests do not update the Persistent
Memory Region. PCI Express memory read requests to the
Persistent Memory Region return correct data.
011b
Unreliable: The Persistent Memory Region has become unreliable.
PCI Express memory reads may return invalid data or generate
poisoned PCI Express TLP(s). Persistent Memory Region memory
writes may not update memory or may update memory with
undefined data. The Persis tent Memory Region may also have
become non-persistent.
100b to 111b Reserved

8 RO 0b
Not Ready (NRDY): This bit indicates if the Persistent Memory Region is ready for use.
If this bit is cleared to ‘0’ and the PMRCTL.EN is set to ‘1’, then the Persistent Memory
Region is ready to accept and process PCI Express memory read and write requests. If
this bit is set to ‘1’ or the PMRCTL.EN bit is cleared to ‘0’, then the Persistent Memory
Region is not ready to process PCI Express memory read and write requests.
7:0 RO 0h
Error (ERR): When the Persistent Memory Region is ready and operating normally, this
field indicates whether previous memory writes to the Persistent Memory Region have
completed without error. If this field is cleared to 0h, then previous writes to the Persistent
Memory Region have completed without error and that the values written are persistent.
A non-zero value in this field indicates the occurrence of an error that may have caused
one or more of the previous writes to not have completed successfully. The meaning of
any particular non-zero value is vendor specific.
Once this field takes on a non -zero value, it maintains a non -zero value until the PCI
Function is reset.
