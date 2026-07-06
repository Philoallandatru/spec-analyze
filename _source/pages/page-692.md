---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 692
headings: ["9.6.2 End of Controller Processing of Outstanding Commands", "9.6.3 Command Retry After Communication Loss"]
---

# Source page 692

NVM Express® Base Specification, Revision 2.1

670
 End of Controller Processing of Outstanding Commands
This section describes how a host determines that no further controller processing of an outstanding
command is possible after a loss of communication happens. At the time when a host detects a
communication loss with a controller, the outstanding commands, if any, are commands for which the host
is unable to receive a CQE as a result of the communication loss (refer to section 3.4.5).
Some commands (e.g., Sanitize) initiate background operations. These background operations are able to
continue after a host loss of communication with the controller that started the background operation. After
such a loss of communication, additional measures (e.g., commands submitted to a different controller) are
necessary for the host to track progress and completion of such a background operation .
A host that is unable to communicate with a controller should perform the following steps in order to
determine that no further controller processing of outstanding commands is able to occur :
1. For message -based transports, terminate the association and the associated transport
connections. This step is skipped for memory-based transports.
2. Wait for sufficient time to ensure that the controller has detected a loss of communication using at
least one of the following:
a. If the controller uses Command Based Keep Alive (refer to section 3.9.3.1), wait at least
until 2 * KATT (refer to section 3.9) from the time the host submitted the most recent Keep
Alive Command to the controller;
b. If the controller uses Traffic Based Keep Alive (refer to section 3.9.4.1), wait at least until
3 * KATT from the time the host submitted the most recent command to the controller; or
c. Receive a transport-specific notification for determining that the controller has terminated
an NVMe Transport connection or detected a loss of communication (e.g., a fabric
notification or a PCIe surprise link down error notification for a PCIe link that directly
connects a host to an NVM subsystem (e.g., an SSD)).
3. Wait for additional sufficient time to ensure that the controller has stopped processing commands
using one of the following:
a. If the CQT field (refer to Figure 312) is non-zero, wait for the amount of time indicated in
the CQT field to elapse; or
b. If the CQT field is cleared to 0h, wait for an implementation specific amount of time (e.g.,
10 seconds). The host should allow this value to be administratively configured .
The specification of the times to wait to ensure that the controller has detected a Keep Alive Timeout
described in this section (i.e., 2 * KATT and 3 * KATT) assumes that the transport delays any command by
at most one KATT. Once the last command is fetch ed by the controller, the controller is required to detect
a Keep Alive Timeout after at most a further 1 * KATT for Command Based Keep Alive and at most 2 *
KATT for Traffic Based Keep Alive (refer to Figure 90). The sum of the two delays (i.e., the transport delay
and the delay to detect the Keep Alive timeout) is 2 * KATT for Command Based Keep Alive and 3 * KATT
for Traffic Based Keep Alive.
 Command Retry After Communication Loss
If the host loses communication with a controller, then the host is unable to receive a completion (CQE) for
any outstanding command (refer to section 3.4.5) that has been submitted to that controller. If the host is
able to use another controller to access the same NVM subsystem or re -establish communication with the
original controller, the host may be able to use that controller to recover from the communi cation loss by
retrying outstanding commands. It is strongly recommended that any host retry of any outstanding
commands after communication loss follow the procedures and requirements in this section in order to
avoid possible corruption of user data and unintended changes to NVM subsystem state (e.g., Reservation
state (refer to section 8.1.22)).
For command retry purposes, every outstanding command falls into one of three command retry categories,
Unrestricted Retry, Delayed Retry, or State -Dependent Retry, based on whether the command is
idempotent (refer to section 9.6.3.1), and whether the command modifies user data or NVM subsystem
