---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 694
headings: ["9.6.3.2.2 Delayed Retry Commands", "9.6.3.2.3 State-Dependent Retry Commands"]
---

# Source page 694

NVM Express® Base Specification, Revision 2.1

672
• the Namespace Capability Change (NCC) bit; and
• the Logical Block Content Change (LBCC) bit
are all cleared to ‘0’ in the Commands Supported and Effects data structure (refer to Figure 210) in the
Commands Supported and Effects log page (refer to section 5.1.12.1.6). If any of these four bits is set to
‘1’, then that command is not an Unrestricted Retry command.
A host may retry any outstanding command that is an Unrestricted Retry command immediately after
communication loss without determining whether further controller processing of that outstanding command
is possible.
For recovery purposes, a host may treat any outstanding command that is an Unrestricted Retry command
as if that command were a Delayed Retry command or a State-Dependent Retry command.
9.6.3.2.2 Delayed Retry Commands
An outstanding command is a Delayed Retry command if that command:
a) is an idempotent command (refer to section 9.6.3.1); and
b) changes user data or NVM subsystem state (e.g., Read Recovery Level (refer to section 8.1.20) or
Reservation state (refer to section 8.1.22)),
unless the changes to user data or NVM subsystem state are able to affect the behavior of any other host
(e.g., refer to the example in Annex B.6.4). As explained further in Annex B.6.4, use of individual Delayed
Retry commands (e.g., an NVM Command Set Write command) that are not part of a fused operation to
affect the behavior of other hosts is strongly discouraged.
A host should treat an outstanding command that is a Delayed Retry command as having command
ordering requirements with respect to other commands where those command ordering requirements are
enforced by higher-level software (refer to section 3.4.1). Hence, for any such command, a host should not
report completion or error status, including errors caused by communication loss, to higher -level software
(e.g., an associated application, filesystem or database) until the host has determined that no fur ther
controller processing of that command and retries, if any, of that command is possible. If a host violates
this recommendation, corruption of user data and unintended changes to NVM subsystem state are
possible; refer to Annex B.6.1 for an example where user data is corrupted.
A host is able to comply with this “should not report” recommendation by delaying submission of a retry of
any outstanding command that is a Delayed Retry command until no further controller processing is
possible of the original outstanding command and an y previously submitted retries of that command. This
avoids host delays in reporting completion of any command upon receiving the CQE for that command .
A host may treat any outstanding command that is a Delayed Retry command as if that command were a
State-Dependent Retry command.
A host that does not adhere to the recommendations in this section for handling outstanding commands
that are Delayed Retry commands risks causing corruption of user data. It is strongly recommended that
host NVMe implementations adhere to these recommendations to avoid data corruption.
9.6.3.2.3 State-Dependent Retry Commands
An outstanding command is a State-Dependent Retry command if the command changes user data or NVM
subsystem state, and the command:
a) is not an idempotent command (refer to section 9.6.3.1); or
b) changes user data or NVM subsystem state in a way that is able to affect the behavior of other
hosts.
A host should not retry an outstanding command that is a State -Dependent Retry command without first
determining that command retry is the appropriate recovery action. This is because retrying such a
command may have different results than the original com mand, duplicate the results of the original
command, or affect the behavior of other hosts in a different manner than the original command. In general,
determination of the appropriate recovery action is only able to be performed by higher-level software (e.g.,
