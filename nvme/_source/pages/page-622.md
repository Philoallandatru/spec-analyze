---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 622
headings: ["8.2.6 Virtualization Enhancements"]
---

# Source page 622

NVM Express® Base Specification, Revision 2.1

600
4. enter either the EPF Complete Port Enabled state (refer to Figure 668) or the EPF Complete Port
Disabled state (refer to Figure 667); and
5. set the PLA variable, if supported, to Deasserted (refer to Figure 661).
Entry to the EPF Processing Port Disabled state or the EPF Processing Port Enabled state shall cause the
controller to discard commands that were fetched from a submission queue prior to entry to the state, and
to discard commands that were received out-of-band on a Management Endpoint prior to entry to the state.
The controller may implement the EPF Processing Port Disabled state or the EPF Processing Port Enabled
state. The controller shall not implement both states. All controllers in the domain shall implement the same
state.
The Emergency Power Fail Recovery Time (EPFRT) field and the Emergency Power Fail Recovery Time
Scale (EPFRTS) field (refer to Figure 313) indicate the time that the controller requires to complete recovery
during the first initialization following successful completion of Emergency Power Fail Processing. It is
implementation specific whether the controller begins recovery before or after t he host sets the CC.EN bit
to ‘1’. Recovery may or may not be complete when the controller sets the CSTS.RDY bit to ‘1’.
If Emergency Power Fail Processing does not complete successfully (e.g., main power was lost before
completion of processing), then the recovery time may exceed the Emergency Power Fail Recovery Time.
 Virtualization Enhancements
Virtualized environments may use an NVM subsystem with multiple controllers to provide virtual or physical
hosts direct I/O access. The NVM subsystem is composed of primary controller(s) and secondary
controller(s), where the secondary controller(s) depend on primary controller(s) for dynamically assigned
resources. A host may issue the Identify comman d to a primary controller specifying the Secondary
Controller List to discover the secondary controllers associated with that primary controller. All secondary
controllers shall be part of the same domain as the primary controller with which they are assoc iated.
Controller resources may be assigned or removed from a controller using the Virtualization Management
command (refer to section 5.2.6) issued to a primary controller. The following types of controller resources
are defined:
• Virtual Queue Resource (VQ Resource): a type of controller resource that manages one
Submission Queue (SQ) and one Completion Queue (CQ) (refer to section 8.2.6.1); and
• Virtual Interrupt Resource (VI Resource): a type of controller resource that manages one interrupt
vector (refer to section 8.2.6.2).
Flexible Resources are controller resources that may be assigned to the primary controller or one of its
secondary controllers. The Virtualization Management command is used to provision the Flexible
Resources between a primary controller and one of its se condary controller(s). A primary controller’s
allocation of Flexible Resources may be modified using the Virtualization Management command and the
change takes effect after any Controller Level Reset other than a Controller Reset. A secondary controller
only supports having Flexible Resources assigned or removed when in the Offline state.
Private Resources are controller resources that are permanently assigned to a primary or secondary
controller. These resources are not supported by the Virtualization Management command.
The primary controller is allowed to have a mix of Private and Flexible Resources for a particular controller
resource type. If there is a mix, then the Private Resources occupy the lower contiguous range of resource
identifiers starting with 0. Secondary controllers shall have all Private or all Flexible Resources for a
particular resource type. Controller resources assigned to a secondary controller always occupy a
contiguous range of identifiers with no gaps, starting with 0. If a particular controller r esource type is
supported as indicated in the Controller Resource Types field of the Primary Controller Capabilities
Structure, then all secondary controllers shall have that controller resource type assigned as a Flexible
Resource. Figure 670 shows the controller resource allocation model for a controller resource type that is
assignable as a Flexible Resource.
