---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 128
headings: ["3.5.3 Controller Ready Modes During Initialization"]
---

# Source page 128

NVM Express® Base Specification, Revision 2.1

106
Figure 83: Discovery Controller Initialization process flow
Discovery Controller receives
Connect Command
1
 from
Host
Controller uses provided KATO
but MAY use an alternate
KATO
2
Controller pre -allocates
resources for AERs and AENs
3
Connect Command completes
with successful status
4
Host sends AER Commands
Controller uses Fixed KATO
2 Controller returns a Connect
Command error Controller uses Fixed KATO
2
Host completes connection to
controller
4
Host does not send AER
Commands
Connect
Command with
non-zero KATO
2
Connect
Command with
non-zero KATO
2
Discovery
Controller
supports
Discovery change
notifications
Y N
Y NNY

Notes:
1. Refer to section 6.3.
2. Refer to the Keep Alive command in section 5.1.14.
3. Refer to the Asynchronous Event Request command in section 5.1.2.
4. Refer to the following steps in this section.
After the Connect command completes with a status of Successful Completion, the host performs the
following steps:
1. NVMe authentication is performed if required (refer to section 8.3.4.2);
2. The host determines the controller’s capabilities by reading the Controller Capabilities property;
3. The host configures the controller’s settings by writing the Controller Configuration property,
including setting CC.EN to ‘1’ to enable command processing;
4. The host waits for the controller to indicate that the controller is ready to process commands. The
controller is ready to process commands when CSTS.RDY is set to ‘1’ in the Controller Status
property; and
5. The host determines the features and capabilities of the controller by issuing an Identify command,
specifying each applicable Controller data structure.
After initializing the Discovery controller, the host reads the Discovery log page. Refer to section 5.1.12.3.1.
 Controller Ready Modes During Initialization
There are two controller ready modes:
• Controller Ready With Media: By the time the controller becomes ready (i.e., by the time that
CSTS.RDY transitions from ‘0’ to ‘1’) after the controller is enabled (i.e., CC.EN transitions from ‘0’
to ‘1’), then:
a) the controller shall be able to process all commands without error as described in section
3.5.4.1; and
