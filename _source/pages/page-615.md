---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 615
headings: ["8.2.5 Power Loss Signaling"]
---

# Source page 615

NVM Express® Base Specification, Revision 2.1

593
Regardless of the supported PMR write barrier mechanisms, a host may periodically read the PMRSTS
property to ensure that reads to the PMR have returned valid data. For example, if a read to the PMRSTS
property indicates that the PMR is operating normally is then followed by a series of reads, and finally a
second read to the PMRSTS property that indicates the PMR is unreliable, then one or more of the reads
between the two PMRSTS property reads may have returned invalid data. Such polling of the PMRSTS
property may be unnecessary if the host handles poisoned TLPs and/or poisoned TLP error reporting is
enabled.
The PMR write elasticity buffer size along with the PMR sustained write throughput allows a host to
determine the amount of time for a read associated with a Persistent Memory Region write barrier
mechanism to complete.
Support for PRPs, SGL Lists, Completion Queues, and Submission Queues in the Persistent Memory
Region is outside the scope of this specification. If the host attempts to use the Persistent Memory Region
for a PRP, SGL List, Completion Queue, or Submission Queue, the controller may abort the command with
a status code of Invalid Field in Command.
 Power Loss Signaling
Power Loss Signaling (PLS) is a capability that the host uses to inform all controllers in a domain of
impending power loss, and that each controller uses to inform the host that the controller is preparing for
that power loss.
There are two modes of Power Loss Signaling processing, Forced Quiescence Processing (refer to section
8.2.5.2) and Emergency Power Fail Processing (refer to section 8.2.5.3). All controllers in a domain support
the same modes (i.e., all controllers report the same value in the PLSFQ bit and all controllers report the
same value in the PLSEPF bit; refer to Figure 312).
Not more than one Power Loss Signaling mode is active at any time. The host uses the Power Loss
Signaling Config feature (refer to section 5.1.25.1.19) to select the mode of operation or to disable Power
Loss Signaling. All controllers in a domain use the same mode (i.e., the scope of the Power Loss Signaling
Config feature is domain). The selection persists across power cycles as defined in Figure 385.
Each controller contains two variables which are used by Power Loss Signaling to perform communication
between the host and controller:
• The Power Loss Notification (PLN) variable is set by the NVMe Transport and has two values,
Asserted and Deasserted.
• The Power Loss Acknowledge (PLA) variable is set by the controller and has four values, Asserted-
FQ, Asserted-EPF-Enabled, Asserted-EPF-Disabled, and Deasserted.
The values of the variables are described in Figure 659.
Figure 659: Power Loss Signaling Variables
Variable
Name Reset1 Description
PLN Deasserted
PLN Value: The PLN variable values are defined as follows:
Value Definition
Asserted A power loss is impending.
Deasserted A power loss is not impending.
