---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 571
headings: []
---

# Source page 571

NVM Express® Base Specification, Revision 2.1

549
Figure 633: RPMB Device Configuration Block Data Structure
Bytes Type Description
0
Boot Partition 0 Write Locked (BPP0L): If this bit is set to ‘1’, then Boot Partition 0
(i.e., the BPID bit in Figure 181 is cleared to 0) is write locked. If this bit is cleared to
‘0’, then the Boot Partition 0 is write unlocked.

02 RW
Write Protection Control (WPC): This field specifies whether the controller processes or aborts
Set Features commands which enable certain namespace write protection states (refer to section
8.1.16 and section 5.1.25.1.31). If the controller does not support Namespace Write Protection,
then this field shall be cleared to 0h.
Bits Description
7:2 Reserved
1
Permanent Write Protect Control (PWPC): This bit cleared to ‘0’ specifies that the
controller shall fail a Set Features command which attempts to set the namespace
write protection state to Permanent Write Protect, as defined in section 8.1.16. This
bit set to ‘1’ specifies that the controller shall process a Set Features command which
attempts to set the namespace write protection state to Permanent Write Protect.
If the controller supports Namespace Write Protection, then this bit shall be cleared
to ‘0’ after a power cycle or a Controller Level Reset.
0
Write Protect Until Power Cycle Control (WPUPCC):  This bit cleared to ‘0’
specifies that the controller shall fail a Set Features command which attempts to set
the namespace write protection state to Write Protect Until Power Cycle, as defined
in section 8.1.16. This bit set to ‘1’ specifies that the controller shall process a Set
Features command which sets the namespace write protection state to Write Protect
Until Power Cycle.
If the controller supports Namespace Write Protection, then this bit shall be cleared
to ‘0’ after a power cycle or a Controller Level Reset.

511:03  Reserved

Figure 634: RPMB Request and Response Message Types
Request Message Types Description Requires
Data
RPMB Frame
Length
(bytes)
0001h Authentication key
programming request
The host is attempting to program the Authentication
Key for the selected RPMB target to the controller. No 256
0002h Reading of the Write
Counter value request
The host is requesting to read the current Write
Counter value from the selected RPMB target. No 256
0003h Authenticated data write
request
The host is attempting to write data to the selected
RPMB target. Yes M + 256
0004h Authenticated data read
request
The host is attempting to read data from the selected
RPMB target. No 256
0005h Result read request  The host is attempting to read the result code for any
of the other Message Types. No 256
0006h
Authenticated Device
Configuration Block write
request
The host is attempting to write Device Configuration
Block (DCB) to the selected RPMB target. This
request message type is only valid for RPMB target
0.
Yes 512 + 256
0007h
Authenticated Device
Configuration Block read
request
The host is attempting to read Device Configuration
Block (DCB) from the selected RPMB target. This
request message type is only valid for RPMB target
0.
No 256
0100h Authentication key
programming response
Returned as a result of the host requesting a Result
read request Message Type after programming the
Authentication Key.
No 256
