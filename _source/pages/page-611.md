---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 611
headings: ["8.2 Memory-Based Transport Extended Capabilities (PCIe)", "8.2.1 Controller Memory Buffer"]
---

# Source page 611

NVM Express® Base Specification, Revision 2.1

589
2) Sets the Feature Identifier field of the Get Features command to F1h; and
3) Sets the UUID Index field of the Get Features command to the index of UUID V.
8.2 Memory-Based Transport Extended Capabilities (PCIe)
This section describes extended capabilities that are specific to the Memory-based transport model.
 Controller Memory Buffer
The Controller Memory Buffer (CMB) is a region of general purpose read/write memory on the controller.
The controller indicates support for the CMB by setting CAP.CMBS to ‘1’. The host indicates intent to use
the CMB by setting CMBMSC.CRE to ‘1’. If this bit is set to ‘1’, the controller indicates the properties of the
CMB via the CMBLOC and CMBSZ properties (refer to section 3.1.4).
The CMB may be used for a variety of purposes. The controller indicates which purposes the memory may
be used for by setting support flags in the CMBSZ property.
The CMB’s PCI Express address range is used for external memory read and write requests to the CMB.
The PCI Express base address of the CMB is defined by the PCI Base Address Register (BAR) indicated
by CMBLOC.BIR, and the offset indicated by CMBLOC.OFST. The size of the CMB is indicated by
CMBSZ.SZ.
The controller uses the CMB’s controller address range to reference CMB with addresses supplied by the
host. The PCI Express address range and the controller address range of the CMB may differ, but both
ranges have the same size, and equivalent offsets wi thin each range have a one -to-one correspondence.
The host configures the controller address range via the CMBMSC property.
The host enables the CMB’s controller memory space via the CMBMSC.CMSE bit. When controller memory
space is enabled, if the host supplies an address referencing the CMB’s controller address range, then the
controller directs memory read or write requests for this address to the CMB.
When the CMB’s controller memory space is disabled, the controller does not consider any host -supplied
address to reference the CMB’s controller address range, and memory read and write requests are directed
elsewhere (e.g., to memory other than the CMB).
To prevent possible misdirection of the controller’s memory requests, before host software enables the
CMB’s controller memory space, the host should configure the CMB’s controller address range so that the
addresses do not overlap any address that host software intends to use for DMA.
In versions prior to NVM Express Base Specification, Revision 1.4, for a controller that supports the CMB,
the CMB’s controller address range is fixed to be equal to its PCI Express address range, and the CMB’s
controller memory space is always enabled whenever the controller is enabled. To prevent misdirection of
controller memory requests when such a controller is assigned to a virtual machine, host software (on the
hypervisor or host OS) should not enable translation of the CMB’s PCI Express address rang e and should
ensure that this address range does not overlap any range of pre -translated addresses that the virtual
machine may use for DMA.
A host may configure the CMBMSC property so that CMB operates when the controller is assigned to a
virtual machine that only supports NVM Express Base Specification, Revision 1.3 and earlier. To prevent
that virtual machine from unintentionally clearing the CMBMSC property to 0h, the contents of the CMBSMC
property are preserved across Controller Reset and Function Level Reset (refer to the NVM Express NVMe
over PCIe Transport Specification).
Submission Queues in host memory require the controller to perform a PCI Express read from host memory
in order to fetch the submission queue entries. Submission Queues in controller memory enable host
software to directly write the entire submission queue  entry to the controller's internal memory space,
avoiding one read from the controller to the host. This approach reduces latency in command execution
and improves efficiency in a PCI Express fabric topology that may include multiple switches. Similarly, PRP
Lists or SGLs require separate fetches across the PCI Express fabric, which may be avoided by writing the
PRP or SGL to the Controller Memory Buffer. Completion Queues in the Controller Memory Buffer may be
used for peer to peer or other applications. For writes of small amounts of data, it may be advantageous to
