---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 616
headings: []
---

# Source page 616

NVM Express® Base Specification, Revision 2.1

594
Figure 659: Power Loss Signaling Variables
Variable
Name Reset1 Description
PLA Deasserted
PLA Value: The PLA variable values are defined as follows:
Value Definition
Asserted-FQ The controller is performing Forced Quiescence Processing
(refer to section 8.2.5.2).
Asserted-EPF-Enabled The controller is performing Emergency Power Fail
processing and the port is enabled (refer to section 8.2.5.3).
Asserted-EPF-Disabled The controller is performing Emergency Power Fail
Processing and the port is disabled (refer to section 8.2.5.3).
Deasserted The controller is not performing Power Loss Signaling
processing.

Note:
1. Following a Controller Level Reset, the variable shall be set to this value.
Transport-specific details of the PLN variable and the PLA variable, including effects on communication
connectivity between host and controller, are described in the Power Loss Signaling Support section of the
appropriate NVMe Transport specification and in the Power Loss Signaling Interactions section of the NVM
Express Management Interface Specification.
If the controller supports Power Loss Signaling, then the controller:
• shall support the PLN variable as specified in this section;
• may support the PLA variable as specified in this section;
• shall support Forced Quiescence Processing (refer to section 8.2.5.2), Emergency Power Fail
Processing (refer to section 8.2.5.3), or both;
• if Forced Quiescence Processing is supported, shall report a non -zero value in the Forced
Quiescence Vault Time field in the Power State Descriptor of one or more of the supported power
states (refer to Figure 313);
• if Emergency Power Fail Processing is supported, shall report non -zero values in the Emergency
Power Fail Vault Time field and the Emergency Power Fail Recovery Time field in the Power State
Descriptor of one or more of the supported power states (refer to Figure 313);
• shall support reporting of whether I/O performance is degraded in the I/O Impacted (IOI) field (i.e.,
reports values 10b and 11b) in the I/O Command Set Independent Identify Namespace data
structure (refer to Figure 319); and
• shall support the Power Loss Signaling Config feature (refer to section 5.1.25.1.19).
If the PLN variable is set to Asserted, then the controller performs either Forced Quiescence or Emergency
Power Fail Processing, as determined by the setting of the Power Loss Signaling Config feature.
The controller shall ignore transitions in the PLN variable if:
a) a Controller Level Reset (refer to section 3.7.2) is in process; or
b) if the CSTS.SHST field is not cleared to 00b (i.e., the controller is in the process of shutting down
or has completed shutdown).
If the PLN variable is set to Asserted and the controller is in a power state for which:
a) the Emergency Power Fail Vault Time field is cleared to 0h;
b) the Forced Quiescence Vault Time field is cleared to 0h; or
c) the Emergency Power Fail Recovery Time field is cleared to 0h,
then the time to perform an action for which the corresponding value is cleared to 0h is vendor specific.
If the controller is in the EPF Complete Port Enabled state, the EPF Complete Port Disabled state, or the
FQ Complete state and power is lost, then during the first restoration of power following the power loss,
