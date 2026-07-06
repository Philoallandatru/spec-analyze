---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 554
headings: ["8.1.16.1 Namespace Write Protection – Theory of Operation"]
---

# Source page 554

NVM Express® Base Specification, Revision 2.1

532
Figure 621: Namespace Write Protection State Definitions
M/O State Definition
Persistent Across
Power
Cycles
Controller
Level Resets
M Write Protect The namespace is write protected. Yes Yes
O Write Protect Until
Power Cycle
The namespace is write protected until
the next power cycle. No Yes
O Permanent Write
Protect
The namespace is permanently write
protected. Yes Yes
Notes:
M – If the Namespace Write Protection capability is supported, then support of this state is mandatory.
O – If the Namespace Write Protection capability is supported, then support of this state is optional.
The Write Protect Until Power Cycle state shall not be used in multi -domain NVM subsystems because
clearing that state requires simultaneous power cycle of the namespace and all controllers to which that
namespace is attached. The result of a command that attempts to use that state in a multi -domain NVM
subsystem is specified in section 5.1.25.1.31.
Figure 622 defines the transition between write protection states. All state transitions are based on Set
Features commands unless otherwise specified. The initial state of a namespace at the time of its creation
is the No Write Protect state.
Figure 622: Namespace Write Protection State Machine Model

The Write Protect Until Power Cycle and Permanent Write Protect states are subject to the controls defined
in the Write Protection Control field (refer to Figure 633), which determines whether the controller processes
or aborts Set Features commands which cause a transition into either of these two states (refer to section
8.1.21).
The results of using Namespace Write Protection in combination with an external write protection system
(e.g., TCG Storage Interface Interactions Specification) are outside the scope of this specification.
 Namespace Write Protection – Theory of Operation
If Namespace Write Protection is supported by the controller, then the controller shall:
• indicate the level of support for Namespace Write Protection capabilities in the Namespace Write
Protection Capabilities (NWPC) field in the Identify Controller data structure by:
o setting the No Write Protect and Write Protect Support bit to ‘1’ in the NWPC field;
o setting the Write Protect Until Power Cycle Support bit to ‘1’ in the NWPC field, if the Write
Protect Until Power Cycle state is supported; and
