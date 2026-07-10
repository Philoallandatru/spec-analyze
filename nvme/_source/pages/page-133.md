---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 133
headings: ["3.6.1 Memory-based Controller Shutdown (PCIe)"]
---

# Source page 133

NVM Express® Base Specification, Revision 2.1

111
Figure 85 does not include transition conditions for a controller that is becoming ready or is undergoing
reset. During these transitions, the CC.EN bit and the CSTS.RDY bit have different values (refer to Figure
41 and Figure 42). The media may or may not be usable during these transitions.
Figure 85 does not include the NVMe-MI effects of processing an Admin command that requires access to
the media (refer to Figure 84) and specifies the Ignore Shutdown bit set to ‘1’ is processed by the controller
via the out-of-band mechanism (refer to the NVM Express Management Interface Specification). Processing
of such a command causes the media to become usable, after which the media may or may not be returned
to its previous condition.
 Memory-based Controller Shutdown (PCIe)
It is recommended that the host perform an orderly shutdown of the controller by following the procedure
in this section when a power-off or shutdown condition is imminent.
The host should perform the following actions in sequence for a normal controller shutdown:
1. If the controller is enabled (i.e., CC.EN (refer to Figure 41) is set to ‘1’):
a. Stop submitting any new I/O commands to the controller and allow any outstanding
commands to complete;
b. If the controller implements I/O queues, then the host should delete all I/O Submission
Queues, using the Delete I/O Submission Queue command (refer to section 5.2.4). A result
of the successful completion of the Delete I/O Submission Queue command is that any
remaining commands outstanding are aborted;
c. If the controller implements I/O queues, then the host should delete all I/O Completion
Queues, using the Delete I/O Completion Queue command (refer to section 5.2.3);
and
2. The host should set the Shutdown Notification (CC.SHN) field (refer to Figure 41) to 01b to indicate
a normal controller shutdown operation. The controller indicates when shutdown processing is
completed by updating the Shutdown Status (CSTS.SHST) field to 10b and the Shutdown Type
(CSTS.ST) field (refer to Figure 42) is cleared to ‘0’.
The host should perform the following actions in sequence for an abrupt shutdown:
1. If the controller is enabled (i.e., CC.EN is set to ‘1’), then stop submitting any new I/O commands
to the controller; and
2. The host should set the Shutdown Notification (CC.SHN) field (refer to Figure 41) to 10b to indicate
an abrupt shutdown operation. The controller indicates when shutdown processing is completed
by updating the Shutdown Status (CSTS.SHST) field (refer to Figure 42) to 10b and CSTS.ST
(refer to Figure 42) is cleared to ‘0’.
For entry to the D3 power state (refer to the PCI Express Base Specification), the shutdown steps outlined
for a normal controller shutdown should be followed.
It is recommended that the host wait a minimum of the RTD3 Entry Latency reported in the Identify
Controller data structure (refer to Figure 312) for the shutdown operations to complete; if the value reported
in RTD3 Entry Latency is 0h, then the host should wait for a minimum of one second. While shutdown
processing is in progress on a controller, it is not recommended to disable that controller via the CC.EN bit.
This causes a Controller Reset which may impact the time required to complete shutdown processing.
While shutdown processing is in progress on a controller, that controller may abort any command with a
status code of Commands Aborted due to Power Loss Notification.
The controller is ready to be powered off (e.g., the media is in the shutdown state (refer to Figure 85)) when
the CSTS.ST bit is cleared to ‘0’, and the CSTS.SHST field indicates that controller shutdown processing
is complete (i.e., the CSTS.SHST field is set to 10b), regardless of the value of the CC.EN bit. The controller
remains ready to be powered off (e.g., the media remains in the shutdown state) until:
A. the controller is enabled (i.e., the CC.EN bit transitions from ‘0’ to ‘1’);
B. the controller is reset by a Controller Level Reset; or
