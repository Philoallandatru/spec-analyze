---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 331
headings: []
---

# Source page 331

NVM Express® Base Specification, Revision 2.1

309
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
259 M M M3
Asynchronous Event Request Limit (AERL): This field is used to convey the maximum
number of concurrently outstanding Asynchronous Event Request commands supported
by the controller (refer to section 5.1.2). This is a 0’s based value. It is recommended that
implementations support a minimum of four Asynchronous Event Request commands
outstanding simultaneously.
260 M M R
Firmware Updates (FRMW): This field indicates capabilities regarding firmware updates.
Refer to section 3.11 for more information on the firmware update process.
Bits Description
7:6 Reserved
5
Support Multiple Update Detection (SMUD):  If this bit is set to ‘1’, then the
controller is able to detect overlapping firmware/boot partition image update
command sequences (refer to section 3.11 and section 8.1.3.2). If this bit is
cleared to ‘0’, then the controller is not able to detect overlapping firmware/boot
partition image update command sequences.
4
Firmware Activation Without Reset (FAWR):  If this bit is set to ‘1’, then the
controller supports firmware activation without a reset. If this bit is cleared to
‘0’, then the controller requires a reset for firmware to be activated.
3:1
Number Of Firmware Slots (NOFS): This field indicates the number of
firmware slots supported by the domain that contains this controller. This field
shall specify a value from one to seven, indicating that at least one firmware
slot is supported and up to seven maximum. This corresponds to firmware slots
1 through 7
0
First Firmware Slot Read Only (FFSRO) : If this bit is set to ‘1’, then the first
firmware slot (i.e., slot 1) is read only. If this bit is cleared to ‘0’, then the first
firmware slot (i.e., slot 1) is read/write. Implementations may choose to have a
baseline read only firmware image.
