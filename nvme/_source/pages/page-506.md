---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 506
headings: ["8.1.3.2 Writing to a Boot Partition"]
---

# Source page 506

NVM Express® Base Specification, Revision 2.1

484
Figure 589: Boot Partition Overview

To read the contents of a Boot Partition using NVMe properties, the host allocates a Boot Partition Memory
Buffer in host memory for the controller to copy contents from a Boot Partition. The host initializes the Boot
Partition Memory Buffer Base Address. The host sets the Boot Partition ID, Boot Partition Read Size,  and
Boot Partition Read Offset to initiate the Boot Partition read operation. The host may continue reading from
the Boot Partition until the entire Boot Partition has been read.
A portion of the Boot Partition may be read by the host any time the NVM subsystem is powered (i.e.,
whether or not CC.EN is set to ‘1’). The host shall not modify transport specific properties (described in the
applicable NVMe Transport binding specification), reset, or shutdown the controller while a Boot Partition
read is in progress.
To read data from a Boot Partition, the host follows these steps:
1. Initialize the transport (e.g., PCIe link), if necessary;
2. Determine if Boot Partitions are supported by the controller (CAP.BPS);
3. Determine which Boot Partition is active (BPINFO.ABPID) and the size of the Boot Partition
(BPINFO.BPSZ);
4. Allocate a physically contiguous memory buffer in the host to store the contents of a Boot Partition;
5. Initialize the address (BPMBL.BMBBA) into the memory buffer where the contents should be
copied;
6. Initiate the transfer of data from a Boot Partition by writing to the Boot Partition Read Select
(BPRSEL) property. This includes setting the Boot Partition identifier (BPRSEL.BPID), size of Boot
Partition Read Size (BPRSEL.BPRSZ) and Boot Partition Read Offset (BPRSEL.BPROF). The
controller sets the Boot Read Status (BPINFO.BRS) field while transferring the Boot Part ition
contents to indicate a Boot Partition read operation is in progress; and
7. Wait for the controller to completely transfer the requested portion of the Boot Partition, indicated
in the status field (BPINFO.BRS). If BPINFO.BRS is set to 10b, the requested Boot Partition data
has been transferred to the Boot Partition Memory Buffer. If BPINFO.BRS is set to 11b, there was
an error transferring the requested Boot Partition data and the host may request the Boot Partition
data again.
In constrained memory environments, the host may read the contents of a Boot Partition with a small Boot
Partition Memory Buffer by reading a small portion of a Boot Partition, moving the data out of the Boot
Memory Buffer to another memory location, and then reading another portion of the Boot Partition until the
entire Boot Partition has been read.
If the Boot Partition log page is supported (refer to section  5.1.12.1.1), then the Boot Partition can be
accessed through the Boot Partition log page (refer to section 5.1.12.1.21).
 Writing to a Boot Partition
Boot Partition contents may be modified by the host using the Firmware Image Download and Firmware
Commit commands while the controller is enabled (CC.EN set to ‘1’).
The process for updating a Boot Partition is:
