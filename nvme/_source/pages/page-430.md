---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 430
headings: []
---

# Source page 430

NVM Express® Base Specification, Revision 2.1

408
acknowledges that it is safe for the host software to modify the host memory buffer contents. Refer to
section 8.2.3.
A host is able to restrict access to the host memory buffer (HMB) while the controller is in a non-operational
power state that was configured by the host (refer to section 5.1.25.1.2). If this HMB non-operational power
state access restriction is enabled by the host (refer to Figure 447) and the host configures a non -
operational power state, then the controller does not access the HMB until the controller transitions to an
operational power state except for HMB access required to process Admin commands and background
operations initiated by Admin commands. Enabling or disabling Non -Operational Power State Permissive
Mode (refer to section 5.1.25.1.10) shall have no effect on HMB non -operational power state access
restriction.
Enabling or disabling HMB non -operational power state access restriction should not affect the Entry
Latency (ENLAT) for non-operational power states (refer to section 8.1.17) that are reported in the power
state descriptors in Identify Controller data structure (e.g., if HMB non -operational power state access
restriction is enabled, the controller may consume additional time beyond the applicable Entry Latency
value in order to retrieve necessary data from the HMB before the controller transitions to a non-operational
power state).
If HMB non -operational power state access restriction is enabled and the controller autonomously
transitions from an operational power state to a non -operational power state, then HMB access by the
controller is not restricted and that access should be min imized (e.g., access ceases as soon as possible
after that transition and does not resume until after the controller transitions to an operational power state).
If HMB non-operational power state access restriction is enabled and the host configures a non-operational
power state while the controller is in a non -operational power state, then HMB access by the controller is
restricted.
If a Get Features command is issued for this Feature, then the completion queue entry indicates whether
HMB non-operational power state access restriction is enabled and whether HMB non -operational power
state access restriction is currently restricting controller access to the HMB (refer to Figure 454):
Figure 447: Host Memory Buffer – Command Dword 11
Bits Description
31:04 Reserved
03 Cleared to Zero (CTZ): This bit shall be cleared to ‘0’
02
Host Memory Non-operational Access Restriction Enable (HMNARE): If the HMBR bit is set to ‘1’
in the Controller Attributes (CTRATT) field in the Identify Controller data structure and:
• this bit is set to ‘1’, then HMB non-operational power state access restriction shall be enabled
(i.e., the controller shall not access the HMB after a non-operational power state is configured
by the host (refer to section 5.1.25.1.2) until the controller is in an operational power state
except for access required to process Admin commands and background operations initiated
by Admin commands); and
• this bit is cleared to ‘0’, then HMB non -operational power state access restriction shall be
disabled (i.e., the controller may access the HMB while the controller is in any non-operational
power state).
If this bit set to ‘1’ and the HMBR bit is cleared to ‘0’, then the controller shall abort the command with
a status code of Invalid Field in Command.
If this bit is cleared to ‘0’ and the HMBR bit is cleared to ‘0’, then this bit has no effect.
01
Memory Return (MR): If this bit is set to ‘1’, then the host is returning memory previously allocated to
the controller for use as the host memory buffer (HMB). That memory may have been in use for the
HMB prior to a reset or entering the Runtime D3 state (e.g., prior to the HMB being disabled). A returned
host memory buffer shall have the exact same size, descriptor list address, descriptor list contents, and
host memory buffer contents as last seen by the controller before the host memory buffer was disabled
(i.e., a Set Features command with the EHM bit cleared to '0' was processed). If this bit is cleared to
‘0’, then the host is allocating host memory resources with undefined content.
