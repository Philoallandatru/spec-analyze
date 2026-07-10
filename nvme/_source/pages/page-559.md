---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 559
headings: ["8.1.17.4 Runtime D3 (RTD3) Transitions (PCIe only)", "8.1.17.5 Host Controlled Thermal Management"]
---

# Source page 559

NVM Express® Base Specification, Revision 2.1

537
Active power values in the power state descriptors are specified for a particular workload since they may
vary based on the workload of the NVM subsystem. The workload field indicates the conditions to observe
the energy values. If Active Power is indicated for a power state, a corresponding workload shall also be
indicated.
The workload values are described in Figure 626.
Figure 626: Workload Hints
Value Definition
000b No Workload: The workload is unknown or not provided.
001b
Workload 1: Extended Idle Period with a Burst of Random Writes. Workload #1 consists of five (5)
minutes of idle followed by thirty -two (32) random write commands of size 1  MiB submitted to a
single controller while all other controllers in the NVM subsystem are idle, and then thirty (30)
seconds of idle.
010b
Workload 2: Heavy Sequential Writes. Workload #2 consists of 80,000 sequential write commands
of size 128 KiB submitted to a single controller while all other controllers in the NVM subsystem are
idle. The submission queue(s) should be sufficiently large allowing the host to ensure there are
multiple commands pending at all times during the workload.
011b to 111b Reserved
 Runtime D3 (RTD3) Transitions (PCIe only)
In Runtime D3, main power is removed from the controller. Auxiliary power may or may not be provided.
RTD3 is used for additional power savings when the controller is expected to be idle for a period of time.
In this specification, RTD3 refers to the D3cold power state described in the PCI Express Base Specification.
RTD3 does not include the PCI Express D3 hot power state because main power is not removed from the
controller in the D3 hot power state. Refer to the PCI Express Base Specification for details on the D3 hot
power state and the D3cold power state.
To enable host software to determine when to use RTD3, the controller reports the RTD3  Entry Latency
(RTD3E) field and the RTD3 Resume Latency (RTD3R) field in the Identify Controller data structure in
Figure 312. The host may use the sum of these two values to evaluate whether the expected idle period is
long enough to benefit from a transition to RTD3.
The RTD3 Resume Latency is the expected elapsed time from the time power is applied until the controller
is able to:
a) process and complete I/O commands; and
b) access the resources (e.g., formatted storage) associated with attached namespace(s), if any, as
part of I/O command processing.
The latency reported is based on a normal shutdown with optimal controller settings preceding the RTD3
resume. The latency reported assumes that host software enables and initializes the controller and sends
a 4 KiB read operation.
If CSTS.ST is cleared to ‘0’, then t he RTD3 Entry Latency is the expected elapsed time from the time
CC.SHN is set to 01b by host software until CSTS.SHST is set to 10b by the controller. When CSTS.SHST
is set to 10b, the controller is ready for host software to remove power.
 Host Controlled Thermal Management
A controller may support host controlled thermal management (HCTM), as indicated in the Host Controlled
Thermal Management Attributes of the Identify Controller data structure in Figure 312. Host controlled
thermal management provides a mechanism for the host to configure a controller to automatically transition
between active power states or perform vendor specific thermal management actions in order to attempt to
meet thermal management re quirements specified by the host. If active power states transitions are used
to attempt to meet these thermal management requirements specified by the host, then those active power
states transitions are vendor specific.
