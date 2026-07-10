---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 195
headings: ["5.1.1.2 Command Completion", "5.1.2 Asynchronous Event Request command"]
---

# Source page 195

NVM Express® Base Specification, Revision 2.1

173
host memory access, command processing does not modify: NVM media, NVM Set state, Endurance
Group state, namespace state, controller state, domain state, or NVM subsystem state) subsequent to the
posting of the CQE for the command that requested the abort.  If the controller is not able to ensure there
are no effects of the command to abort subsequent to the posting of the CQE for the command that
requested the abort, then the controller shall not perform an immediate abort on that command.
For example, if a command:
• requests a data transfer;
• the data transfer has been initiated and not completed; and
• the CQE for that command was not posted prior to the posting of the CQE for the command that
requested the abort,
then the controller is prohibited from performing an immediate abort.
Performing an immediate abort effects the information returned in the CQE for the command that requested
the abort as described in section 5.1.1.2 for an Abort command and in section 7.1.1 for a Cancel command.
 Command Completion
Upon completion of the Abort command, the controller posts a completion queue entry to the Admin
Completion Queue indicating the status for the Abort  command. Dword 0 of the completion queue entry
indicates information about the command to abort as shown in Figure 144.
Figure 144: Abort Command – Completion Queue Entry Dword 0
Bits Description
31:01 Reserved
00
Immediate Abort Not Performed (IANP): Indicates whether or not an immediate abort was performed.
If this bit is set to ‘1’, then an immediate abort was not performed (i.e., the command to abort was not
aborted prior to posting the CQE for the Abort command) for any reason (e.g., the immediate abort
requirements were not met, the requested command to abort was not f ound). If this bit is cleared to ‘0’,
then an immediate abort was performed (i.e., the command to abort was aborted prior to posting the
CQE for the Abort command).
If the Immediate Abort Not Performed bit in Dword 0 (refer to Figure 144) is set to ‘1’ in the completion
queue entry for the Abort command and a deferred abort is performed, then the controller shall set the
status code for the command to abort to Command Abort Requested. The host should examine the status
in the completion queue entry of the command to abort to determine if the command was aborted.
Command specific status code values associated with the Abort command are defined in Figure 145.
Figure 145: Abort – Command Specific Status Values
Value Description
3h Abort Command Limit Exceeded: The number of concurrently outstanding Abort commands has exceeded
the limit indicated in the Identify Controller data structure.
 Asynchronous Event Request command
Asynchronous events are used to notify a host of status, error, and health information as these events
occur. To enable asynchronous events to be reported by the controller, a host needs to submit one or more
Asynchronous Event Request commands to the controller. The controller specifies an event to the host by
completing an Asynchronous Event Request command.  A host should expect that the controller may not
execute the command immediately; the command should be  completed when there is an event to be
reported.
The Asynchronous Event Request command is submitted by a host to enable the reporting of asynchronous
events from the controller.  This command has no timeout.  The controller posts a completion queue entry
for this command when there is an asynchronous event to report to the host.  If Asynchronous Event
