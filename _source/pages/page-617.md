---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 617
headings: ["8.2.5.1 Power Loss Signaling Processing State Machine"]
---

# Source page 617

NVM Express® Base Specification, Revision 2.1

595
processing of commands may be affected while the controller performs internal recovery operations.
Examples of these effects include:
a) a namespace not being ready (i.e., the NRDY bit is cleared to ‘0’ in the NSTAT field; refer to Figure
319); and
b) commands to a namespace being processed at reduced performance, as indicated by the IOI field
in the NSTAT field (refer to Figure 319).
 Power Loss Signaling Processing State Machine
Figure 660 illustrates how transitions in the PLN variable initiate Power Loss Signaling processing by the
controller. Each circle represents a processing state.
Figure 660: Power Loss Signaling Processing State Machine

For all states, a power cycle causes a transition to the PLS Not Ready state.
The Power Loss Signaling processing state of the controller determines the value of the PLA variable and
whether communications on the port are processed (refer to Figure 661).
PLS Ready
FQ
Processing
EPF
Processing
Port
Enabled
EPF
Complete
Port
Disabled
FQ
Complete
See Note 1
See Note 3
PLN Deasserted
Power Applied
EPF
Complete
Port
Enabled
Controller Level Reset
or CSTS.SHST    00b
EPF
Processing
Port
Disabled
See Note 2
PLS Not
Ready
EPF
Processing
Complete
EPF
Processing
Complete
FQ
Processing
Complete
CSTS.SHST = 00b
Controller Level Reset
or CSTS.SHST    00b
Note 1: EPF is Enabled & PLN is Asserted & EPF Processing Port Enabled state is supported.
Note 2: EPF is Enabled & PLN is Asserted & EPF Processing Port Disabled state is supported.
Note 3: FQ is Enabled & PLN is Asserted.
