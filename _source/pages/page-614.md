---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 614
headings: []
---

# Source page 614

NVM Express® Base Specification, Revision 2.1

592
The controller uses the PMR’s controller address range to reference PMR with addresses supplied by the
host. The PCI Express address range and the controller address range of the PMR may differ, but both
ranges have the same size, and equivalent offsets wi thin each range have a one -to-one correspondence.
The host configures the controller address range via the PMRMSCU and PMRMSCL properties.
The host enables the PMR’s controller memory space via the PMRMSCL.CMSE bit. When controller
memory space is enabled, if host supplies an address referencing the PMR’s controller address range, then
the controller directs memory read or write requests for this address to the PMR.
When the PMR’s controller memory space is disabled, the controller does not consider any host -supplied
address to reference the PMR’s controller address range, and memory read and write requests are directed
elsewhere (e.g., to memory other than the PMR).
The contents of data written to the PMR while the PMR is ready persists across power cycles, Controller
Level Resets, and disabling of the PMR. The mechanism used to make a write to the PMR persistent is
implementation specific. For example, in one impleme ntation this may mean that a write to non -volatile
memory has completed while in another implementation this may mean that the write has been stored in a
non-volatile write buffer and is written to non-volatile memory at some later point.
A PMR implementation has a maximum sustained write throughput. The PMR implementation may also
have an optional write elasticity buffer used to buffer writes from PMR PCIe write requests. When the PMR
sustained write throughput is less than the PCI Express  link throughput, then such a write elasticity buffer
allows PCIe write request burst throughput to exceed the PMR sustained write throughput without back
pressuring into the PCI Express fabric.
The time required to transfer data from the write elasticity buffer to nonvolatile media is the amount of data
written to the elasticity buffer divided by the Persistent Memory Region Sustained Write Throughput (refer
to section 3.1.4.26). The time to transfer the entire contents of the write elasticity buffer is the Persistent
Memory Region Elasticity Buffer Size (refer to section 3.1.4.25) divided by the Persistent Memory Region
Sustained Write Throughput.
The host enables the PMR by setting PMRCTL.EN to ‘1’. Once enabled, the controller indicates that the
PMR is ready by clearing PMRSTS.NRDY to ‘0’. It is not necessary to enable the controller to enable the
PMR. Restoring and saving the contents of the PMR may take time to complete. When the host modifies
the value of PMRCTL.EN, the host should wait for at least the time interval specified in PMRCAP.PMRTO
for PMRSTS.NRDY to reflect the change.
When the PMR is not ready, PMR reads complete successfully and return an undefined value while PMR
writes complete normally, but do not update memory (i.e., the contents of the PMR address written remains
unchanged). The undefined value returned by a PMR r ead following a sanitize operation is such that
recovery of any previous user data from any cache or the non-volatile storage media is not possible.
When the PMR becomes read -only or unreliable, then a critical warning is reported in the SMART/Health
Information Log which may be used to trigger an NVMe interface asynchronous event. Since reporting of
an asynchronous event may occur an unspecified amount of time after the PMR health status has changed,
the host should assume that all operations to the PMR have been affected since the last time normal
operation was reported in PMRSTS.HSTS.
PMRCAP.PMRWBM enumerates supported PMR write barrier mechanisms. At least one mechanism shall
be supported. An implementation may optionally support a mechanism where a PCI Express read of any
size to the PMR, including a “zero -length read,” ensures that a ll previous memory writes (i.e., Posted PCI
Express requests) to the PMR have completed and are persistent. An implementation may optionally
support a write barrier mechanism that utilizes a read of the PMRSTS property. When supported, a read of
the PMRSTS property allows a host to:
• ensure that previously issued memory writes to the PMR have completed; and
• determine whether the PMR updates associated with those writes have completed without error
and are persistent.
A PMR memory write error may be the result of a poisoned PCI Express TLP, an NVM subsystem internal
error, or a PMR health status issue.
