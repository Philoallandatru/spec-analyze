---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 287
headings: ["5.1.12.1.22 Rotational Media Information Log (Log Page Identifier 16h)"]
---

# Source page 287

NVM Express® Base Specification, Revision 2.1

265
Figure 267: Boot Partition Log Specific Parameter Field
Bits Description
14:09 Reserved
08 Boot Partition Identifier  (BPID): This bit specifies the Boot Partition identifier for the Boot
Partition to return.

Figure 268: Boot Partition Log Page
Bytes Description
Boot Partition Header
00 Log Page Identifier (LID): This field shall be set to 15h.
03:01 Reserved
07:04
Boot Partition Information  (BPINFO): Contains defines the characteristics of Boot
Partitions.
Bits Description
31 Active Boot Partition ID (ABPID):  This bit indicates the identifier of the
active Boot Partition.
30:15 Reserved
14:00 Boot Partition Size (BPSZ): This field defines the size of the Boot Partition
Data field in multiples of 128 KiB.

15:08 Reserved
Boot Partition Data
BPSZ*128 KiB + 15:16 Boot Partition Data (BPD): Contains the contents of the specified Boot Partition.
5.1.12.1.22 Rotational Media Information Log (Log Page Identifier 16h)
This log page provides rotational media information (refer to section 8.1.23) for Endurance Groups that
store data on rotational media. The information provided is retained across power cycles and resets.
The Endurance Group Identifier is specified in the Log Specific Identifier field in Command Dword 11 of the
Get Log Page command as defined in Figure 217.
If the NVM subsystem does not contain any Endurance Groups that store data on rotational media, then
the Rotational Media Information Log should not be supported.
Figure 269: Rotational Media Information Log Page
Bytes Description
1:0 Endurance Group Identifier (ENDGID): The Endurance Group Identifier specified by the Get Log Page
command.
3:2 Number of Actuators (NUMA): Contains the number of actuators in this Endurance Group.
5:4
Nominal Rotational Speed (NRS):
Value Definition
0000h Not reported
0001h This value is prohibited to maintain backward compatibility with  other standards.
This value shall not be used.
FFFFh Reserved
All other values Nominal rotational speed in revolutions per minute while the current Power State is
0 (refer to section 5.1.25.1.2).

7:6 Reserved
11:8
Spinup Count (SPINC): Contains the total number of successful spinup events for this Endurance Group
over the lifetime of the Endurance Group. If the Spinup Count is less than FFFFFFFFh, then the controller
shall increment this count by one for each successful spinup event.
A successful spinup event occurs when the controller power state transitions from a non -operational
power state to an operational power state.
