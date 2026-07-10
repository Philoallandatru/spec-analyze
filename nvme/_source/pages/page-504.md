---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 504
headings: ["8.1.2.2 Host ANA Inaccessible Operation", "8.1.2.3 Host ANA Persistent Loss Operation", "8.1.2.4 Host ANA Change Notice Operation"]
---

# Source page 504

NVM Express® Base Specification, Revision 2.1

482
reports the ANA Optimized state for a namespace are found, then the host may use all of those controllers
to send commands.
If there are no controllers that report the ANA Optimized state for a namespace, then the host should select
a controller that reports ANA Non -Optimized State (refer to section 8.1.1.5) for that namespace and send
the command to that controller. If more than one controller that reports ANA Non -Optimized state for a
namespace are found, then the host may use all of those controllers to send commands.
If multiple controllers are being used, then the algorithm for determining which controller to use next is
outside the scope of this specification (e.g., the host may select a simple round robin algorithm, a queue
depth weighted algorithm, a transfer length weighted algorithm, or any other algorithm).
If there are no controllers that report the ANA Optimized state for a namespace and there are no controllers
that report the ANA Non-Optimized state for that namespace, then the host should examine controllers that
report the ANA Inaccessible state as described in section 8.1.2.2.
 Host ANA Inaccessible Operation
If the ANA log page reports an ANA state of ANA Inaccessible State for an ANA Group or a command
returns a status code of Asymmetric Access Inaccessible, then the host should:
• not use that controller to send commands to any namespace in that ANA Group; and
• select a different controller for sending commands to all namespaces in that ANA Group.
If there are no controllers that report the ANA Optimized state for a namespace and there are no controllers
that report the ANA Non -Optimized state, then a transition may be occurring that also impacts controllers
that are reporting the ANA Inaccessible s tate. As a result, the host should use the methods described for
Host ANA Transition operation (refer to section 8.1.2.5) to determine if the controller reporting ANA
Inaccessible state transitions during the ANATT time interval to an ANA state that enables commands to
be processed by that controller.
 Host ANA Persistent Loss Operation
If the ANA log page reports an ANA state of ANA Persistent Loss State for an ANA Group or a command
returns a status code of Asymmetric Access Persistent Loss, then the host should not use that controller to
send commands to any namespace in that ANA Group , and select a different controller for sending
commands to any namespace in that ANA Group. If the controller supports the Namespace Management
capability (refer to section 8.1.15), then the namespaces in an ANA Group reporting this state should be
detached.
 Host ANA Change Notice Operation
If the ANA log page reports an ANA state of ANA Change State for an ANA Group or a command returns
a status code of Asymmetric Access Transition, then the host should temporarily not use that controller to
send commands to any namespace in that ANA Group. If only controllers reporting ANA Inaccessible State
are available, then the host should follow these procedures to determine which controller to use. To use a
controller, the host may:
a) if Asymmetric Namespace Access Change Notices are enabled (refer to section 5.1.25.1.5) on the
controller, wait for an Asymmetric Namespace Access Change Notice from that controller. Upon
receipt of that notice, the host should examine the ANA log page to determine the new ANA state
and resume sending commands based on the new ANA state. Such notice should occur within the
ANATT time (refer to Figure 312); or
b) delay and retry the command during the ANATT time interval. The host should not immediately
retry, but rather, divide the ANATT time into equal intervals for command retry purposes (e.g., if
ANATT is 30, perform 3 retries at 10  s intervals, or 10 retrie s at 3  s intervals). During or upon
completion of the ANATT time interval, the new ANA state of the ANA Group should be known
(e.g., one of the command retries returned a different status that indicates completion of the
transition to a new ANA state). If the retried command did not complete without error, the ANA log
