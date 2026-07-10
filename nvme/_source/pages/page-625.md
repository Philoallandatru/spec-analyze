---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 625
headings: ["8.2.6.3 Secondary Controller States and Resource Configuration"]
---

# Source page 625

NVM Express® Base Specification, Revision 2.1

603
The secondary controller is assigned VI Resources using the Virtualization Management command. The
number of VI Resources assigned is discoverable in the Secondary Controller List entry for the associated
secondary controller.
While a primary controller and/or its associated secondary controllers may concurrently support multiple
types of interrupt vectors (e.g., MSI and MSI -X), all the controllers’ VI Resources shall contain interrupt
resources for interrupt vectors of the same  type. In this revision, MSI -X is the only supported type of VI
Resource.
For a secondary controller that supports VI Resources with MSI -X vectors, if at least one VI Resource is
assigned to that controller, MSIXCAP.MXC.TS (refer to the MSI -X Capability section of the NVMe over
PCIe Transport Specification) indicates the number of VI Resources assigned to the controller. Since
MSIXCAP.MXC.TS is read -only, the value shall only be updated when the secondary controller is in the
Offline state. MSI -X Table Entries on the secondary controller for newly assigned VI Resources shall be
reset to default values.
If a secondary controller that supports VI Resources has no assigned VI Resources, then that controller
remains in the Offline state. A secondary controller is not able to transition to the Online state until a VI
Resource for interrupt vector 0 has been assigned to that controller. For a secondary controller that supports
VI Resources with MSI -X vectors, if no VI Resources are assigned to that controller, then
MSIXCAP.MXC.TS is reserved.
A primary controller that supports VI Resources shall have at least one interrupt that is a Private Resource.
Interrupt vector 0 is always assigned to the primary controller. A primary controller may be allocated VI
Resources using the Primary Controller F lexible Allocation action of the Virtualization Management
command. The VI resources allocated take effect after a Controller Level Reset and are persistent across
power cycles and resets. The number of VI Resources currently allocated is discoverable in t he Primary
Controller Capabilities Structure. For a primary controller that supports VI Resources with MSI -X vectors,
MSIXCAP.MXC.TS indicates an MSI-X Table size equal to the total number of Private Resources and the
Flexible Resources currently allocated following a Controller Level Reset.
When an I/O CQ is created, the controller supports mapping that I/O CQ to any valid interrupt vector,
regardless of whether they have the same resource identifier, as long as the I/O CQ and the interrupt vector
are attached to the same controller.
 Secondary Controller States and Resource Configuration
A secondary controller shall be in one of the following states:
• Online: The secondary controller may be in use by a host. Required resources have been
assigned. The secondary controller may be enabled in this state (CC.EN may be set to ‘1’ and
CSTS.RDY may then transition to ‘1’); or
• Offline: The secondary controller may not be used by a host. CSTS.CFS shall be set to ‘1’.
Controller properties other than CSTS are undefined in this state.
The host may request a transition to the Online or Offline state using the Virtualization Management
command. When a secondary controller transitions from the Online state to the Offline state all Flexible
Resources are removed from the secondary controller.
To ensure that the host accurately detects capabilities of the secondary controller, the host should complete
the following procedure to bring a secondary controller Online:
1. Use the Virtualization Management command to set the secondary controller to the Offline state;
2. Use the Virtualization Management command to assign VQ resources and VI resources;
3. Perform a Controller Level Reset. If the secondary controller is a VF, then this should be a VF
Function Level Reset (refer to the NVM Express NVMe over PCIe Transport Specification); and
4. Use the Virtualization Management command to set the secondary controller to the Online state.
If VI Resources are supported, then following this process ensures the MSI -X Table size indicated by
MSIXCAP.MXC.TS is updated to reflect the appropriate number of VI Resources before the transition to
the Online state.
