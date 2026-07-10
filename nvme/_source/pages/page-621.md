---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 621
headings: ["8.2.5.3 Emergency Power Fail Processing"]
---

# Source page 621

NVM Express® Base Specification, Revision 2.1

599
If the controller enters the FQ Processing state, then the controller shall perform the following actions in
sequence:
1. set the PLA variable, if supported, to Asserted-FQ (refer to Figure 661);
2. stop fetching commands on all submission queues;
3. perform the following actions in parallel or in any sequence:
a) process commands received out -of-band on a Management Endpoint as described in the
Power Loss Signaling Interactions section of the NVM Express Management Interface
Specification;
b) if a command was fetched prior to entering this state, then process that command as described
in this section; and
c) prepare for power loss;
4. enter the FQ Complete state (refer to Figure 664); and
5. set the PLA variable to Deasserted (refer to Figure 661).
Entry to the FQ Processing state should cause the controller to complete processing of all previously
fetched commands (e.g., to post a CQE with a status code of Successful Completion, to abort the command
and post a CQE with an appropriate status code). If a background operation is in progress (e.g., a device
self-test operation or a sanitize operation), then that background operation should be suspended until the
PLN variable is set to Deasserted and fetching and processing commands resumes.
While in the FQ Processing state, the controller shall abort a previously -fetched Set Features command
specifying the Power Loss Signaling Config feature identifier with a status code of Commands Aborted due
to Power Loss Notification.
While in the FQ Processing state, if power is lost and subsequently restored, then resumption of command
processing:
a) may take more time than resumption of command processing after a normal shutdown completes
and then power is cycled; and
b) typically takes less time than resumption of command processing after an abrupt shutdown
completes and then power is cycled.
If the controller transitions from the FQ Processing state to the PLS Not Ready state, then the controller
shall abort Forced Quiescence Processing.
Forced Quiescence Processing shall not cause a loss of communication connectivity between the host and
the controller.
If the PLN variable is set to Asserted and is then set to Deasserted without an intervening loss of main
power, Controller Level Reset, or shutdown (refer to Figure 660), then the controller resumes fetching and
processing commands.
 Emergency Power Fail Processing
This section describes the behavior of each controller in the domain when the domain is configured for
Power Loss Signaling with Emergency Power Fail (refer to section 5.1.25.1.19).
If the controller enters the EPF Processing Port Disabled state or the EPF Processing Port Enabled state
as described in Figure 663, then the controller shall perform the following actions in sequence:
1. set the PLA variable, if supported, to the value specified in Figure 661 for that state (i.e., Asserted-
EPF-Disabled or Asserted-EPF-Enabled);
2. stop fetching commands on all submission queues;
3. perform the following actions in parallel or in any sequence:
a) process port communications as described in Figure 661; and
b) prepare for power loss in a manner that may or may not allow command processing to resume
quickly in the event of power loss and then power resumption;
