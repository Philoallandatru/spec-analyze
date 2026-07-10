---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 165
headings: []
---

# Source page 165

NVM Express® Base Specification, Revision 2.1

143
Figure 102: Status Code – Generic Command Status Values
Value Description I/O Command
Set(s)1
1Dh Sanitize In Progress: The requested function (e.g., command) is prohibited while a
sanitize operation is in progress. Refer to section 8.1.24.3.
1Eh SGL Data Block Granularity Invalid: See the applicable I/O Command Set
specification for the description. NVM, ZNS
1Fh
Command Not Supported for Queue in CMB: The implementation does not support
submission of the command to a Submission Queue in the Controller Memory Buffer
or command completion to a Completion Queue in the Controller Memory Buffer.
Note: NVM Express revision 1.3 and later use this status code only for Sanitize
commands.

20h
Namespace is Write Protected: The command is prohibited while the namespace is
write protected as a result of a change in the namespace write protection state as
defined by the Namespace Write Protection State Machine (refer to Figure 622).

21h
Command Interrupted: Command processing was interrupted and the controller is
unable to successfully complete the command. The host should retry the command.
If this status code is returned, then the controller shall clear the Do Not Retry bit to ‘0’
in the Status field of the CQE (refer to Figure 100). The controller shall not return this
status code unless the host has set the Advanced Command Retry Enable (ACRE)
field to 1h in the Host Behavior Support feature (refer to section 5.1.25.1.14).

22h
Transient Transport Error: A transient transport error was detected. If the command
is retried on the same controller, the command is likely to succeed. A command that
fails with a transient transport error four or more times should be treated as a persistent
transport error that is not likely to succeed if retried on the same controller.

23h
Command Prohibited by Command and Feature Lockdown: The command was
aborted due to command execution being prohibited by the Command and Feature
Lockdown (refer to section 8.1.5).

24h
Admin Command Media Not Ready: The Admin command requires access to media
and the media is not ready. The Do Not Retry bit indicates whether re -issuing the
command at a later time may succeed. This status code shall only be returned:
a) for Admin commands; and
b) if the controller is in Controller Ready Independent of Media mode (i.e.,
CC.CRIME bit is set to ‘1’).
This status code shall not be returned with the Do Not Retry bit cleared to ‘0’ after the
amount of time indicated by the Controller Ready With Media Timeout
(CRTO.CRWMT) field after the controller is enabled (i.e., CC.EN transitions from ‘0’ to
‘1’).
Refer to Figure 84 for the list of Admin commands permitted to return this status code.

25h
Invalid Key Tag:  The command was aborted due to an invalid KEYTAG field value
(refer to Figure 620) as:
a) the value of the specified KEYTAG field is greater than the Maximum Key Tag
(MAXKT) field in the I/O Command Set Independent Identify Namespace data
structure (refer to Figure 319); or
b) defined by the appropriate security specification (e.g., TCG Storage Interface
Interactions specification).

26h
Host Dispersed Namespace Support Not Enabled:  The command is prohibited
while the Host Dispersed Namespace Support (HDISNS) field is not set to 1h in the
Host Behavior Support feature (refer to Figure 407).

27h Host Identifier Not Initialized
28h
Incorrect Key: The command was aborted due to the key associated w ith the
KEYTAG field being incorrect.
The specific conditions under which a key is considered incorrect are defined by the
appropriate security specification (e.g., TCG Storage Interface Interactions
specification).
