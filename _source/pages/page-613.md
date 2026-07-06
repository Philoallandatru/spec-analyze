---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 613
headings: ["8.2.2 Doorbell Stride for Software Emulation", "8.2.3 Host Memory Buffer", "8.2.4 Persistent Memory Region"]
---

# Source page 613

NVM Express® Base Specification, Revision 2.1

591
Memory Write Request to the SQ Tail doorbell property shall not have the Relaxed Ordering bit set to ‘1’
(refer to the PCI Express Base Specification), to ensure that prior writes to the CMB have completed.
 Doorbell Stride for Software Emulation
The doorbell stride, specified in CAP.DSTRD (refer to Figure 36), may be used to separate doorbells by a
number of bytes in memory space. The doorbell stride is a number of bytes equal to (2 ^ (2 + CAP.DSTRD)).
This is useful in software emulation of an NVM Express controller. In this case, a software thread is
monitoring doorbell notifications. The software thread may be made more efficient by monitoring one
doorbell per discrete cacheline or utilize the monitor/mwait CPU instructions. For hardware implementations
of the NVM Express interface, the expected doorbell stride value is 0h.
 Host Memory Buffer
The Host Memory Buffer (HMB) feature allows the controller to utilize an assigned portion of host memory
exclusively. The use of the host memory resources is vendor specific. Host software may not be able to
provide any or a limited amount of the host memo ry resources requested by the controller. The controller
shall function properly without host memory resources. Refer to section 5.1.25.2.4.
The controller may indicate limitations for the minimum usable descriptor entry size and the maximum
number of descriptor entries (refer to the HMMINDS and HMMAXD fields in the Identify Controller data
structure, Figure 312). If the host does not create the Host Memory Buffer within the indicated limits, then
the host memory allocated for use by the controller may not be fully utilized (e.g., descriptor entries beyond
the maximum number of entries indicated may be ignored by the controller).
During initialization, host software may provide a descriptor list that describes a set of host memory address
ranges for exclusive use by the controller. The host memory resources assigned are for the exclusive use
of the controller (host software should not modify the ranges) until host software requests that the controller
release the ranges and the controller completes the Set Features command. The controller is responsible
for initializing the host memory resources. Host software should request that th e controller release the
assigned ranges prior to a shutdown event, a Runtime D3 event, or any other event that requires host
software to reclaim the assigned ranges. After the controller acknowledges that the ranges are no longer
in use, host software may  reclaim the host memory resources. In the case of Runtime D3, host software
should provide the host memory resources to the controller again and inform the controller that the ranges
were in use prior to the RTD3 event and have not been modified.
The host memory resources are not persistent in the controller across a Controller Level Reset. Host
software should provide the previously allocated host memory resources to the controller after that reset
completes. If host software is providing previously allocated host memory resources (with the same
contents) to the controller, the Memory Return bit (refer to Figure 447) is set to ‘1’ in the Set Features
command.
The controller shall ensure that there is no data loss or data corruption in the event of a surprise removal
while the Host Memory Buffer feature is being utilized.
 Persistent Memory Region
The Persistent Memory Region (PMR) is an optional region of general purpose PCI Express read/write
persistent memory that may be used for a variety of purposes. The controller indicates support for the PMR
by setting CAP.PMRS (refer to section 3.1.4.1) to ‘1’ and indicates whether the controller supports
command data and metadata transfers to or from the PMR by setting support flags in the PMRCAP property.
When command data and metadata transfers to or from PMR are supported, all data and metadata
associated with a particular command shall be either entirely located in the Persistent Memory Region or
outside the Persistent Memory Region.
The PMR’s PCI Express address range is used for external memory read and write requests to the PMR.
The PCI Express address range and size of the PMR is defined by the PCI Base Address Register (BAR)
indicated by PMRCAP.BIR. The PMR consumes the entire add ress region exposed by the BAR and
supports all the required features of the PCI Express programming model (i.e., it in no way restricts what is
otherwise permitted by PCI Express).
