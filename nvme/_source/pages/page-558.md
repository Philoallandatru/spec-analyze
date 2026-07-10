---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 558
headings: ["8.1.17.2 Autonomous Power State Transitions", "8.1.17.3 NVM Subsystem Workloads"]
---

# Source page 558

NVM Express® Base Specification, Revision 2.1

536
• CMB accesses, if any;
• processing of Admin commands and processing background operations , if any, initiated by that
command (e.g., Device Self -test command (refer to section 5.1.5), Sanitize command (refer to
section 5.1.22)); and
• additional transport-specific accesses as defined in the applicable NVMe Transport binding
specification.
For the operations listed in the preceding paragraph, the controller:
• may exceed the power advertised by the non-operational power state;
• shall logically remain in the current non-operational power state unless an I/O command is received
or if an explicit transition is requested by a Set Features command with the Power Management
Feature Identifier; and
• shall not exceed the maximum power advertised for the most recent operational power state.
HMB non -operational power state access restriction (refer to section  5.1.25.2.4) does not prohibit the
controller from accessing the HMB while processing Admin commands and while performing host-initiated
background operations initiated by Admin commands. HMB non-operational power state access restriction
does prohibit the controller from accessing the HMB in order to perform controller -initiated activity (i.e. ,
activity not directly associated with a command).
Execution of  controller-initiated background operations may exceed the power advertised by the non -
operational power state , if the Non-Operational Power State Permissive Mode is supported and enabled
(refer to section 5.1.25.1.10).
No I/O commands are processed by the controller while in a non-operational power state. The host should
wait until there are no pending I/O commands prior to issuing a Set Features command to change the
current power state of the device to a non-operational power state and not submit new I/O commands until
the Set Features command completes. Issuing an I/O command in parallel may result in the controller being
in an unexpected power state.
When in a non -operational power state, regardless of whether autonomous power state transitions are
enabled, the controller shall autonomously transition back to the most recent operational power state to
process an I/O command.
 Autonomous Power State Transitions
The controller may support autonomous power state transitions, as indicated in the Identify Controller data
structure in Figure 312. Autonomous power state transitions provide a mechanism for the host to configure
the controller to automatically transition between power states on certain conditions without software
intervention.
The entry condition to transition to the Idle Transition Power State is that the controller has been in idle for
a continuous period of time exceeding the Idle Time Prior to Transition time specified. The controller is idle
when there are no commands outstanding to any I/O Submission Queue. If a controller has an operation in
process (e.g., device self -test operation) that would cause controller power to exceed that advertised for
the proposed non -operational power state, then the controller should not auto nomously transition to that
state.
The power state to transition to shall be a non-operational power state (a non-operational power state may
autonomously transition to another non-operational power state). If an operational power state is specified
by a Set Features command specifying the Autonomous Power State Transitions feature (i.e., the Feature
Identifier field set to 0Ch (refer to section 5.1.25.1.6), then the controller should abort the command with a
status code of Invalid Field in Command. Refer to section 8.1.17.1 for more details.
 NVM Subsystem Workloads
The workload values described in this section may specify a workload hint in the Power Management
Feature (refer to section 5.1.25.1.2) to inform the NVM subsystem or indicate the conditions for the active
power level.
