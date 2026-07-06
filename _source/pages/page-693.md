---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 693
headings: ["9.6.3.1 Idempotent Commands", "9.6.3.2 Command Retry Categories and Requirements", "9.6.3.2.1 Unrestricted Retry Commands"]
---

# Source page 693

NVM Express® Base Specification, Revision 2.1

671
state. Section 9.6.3.2 defines these categories and describes requirements and restrictions on retrying
outstanding commands in each category.
 Idempotent Commands
Controller processing of an idempotent command produces the same end state on the NVM subsystem
and returns the same results if that command is resubmitted one or more times with no intervening
commands. All commands tend to modify some ancillary state on the controller (e.g., tracking statistics);
these ancillary changes to state do not prevent a command from being considered idempotent. The results
of the command include the status code (excluding transient status codes or error conditions, e.g., due to
a loss of communication), any data returned to the host and any NVM subsystem changes to user data or
state (e.g., reservation state, feature contents).
For example, a read command addressed to a specific location (e.g., LBA) in a namespace is an idempotent
command. The read command addressed to a valid location in a namespace returns the same data with a
successful completion status code if that command i s submitted repeatedly. Similarly, a write command
addressed to a valid location in a namespace writes the same data to that location if submitted repeatedly.
This command is also an idempotent command.
On the other hand, a Namespace Management command (refer to section 5.1.21) that creates a
namespace is not idempotent (i.e., is a non -idempotent command), as repeating the Namespace
Management command creates additional namespaces with different namespace identifiers. Similarly, a
Reservation Register command that unregisters a host (refer to section 7.6) is also not idempotent because
repeating the command attempts to unregister a host that is no longer registered and returns an error status
code.
 Command Retry Categories and Requirements
For command retry purposes, an outstanding command is in one of three categories:
• Unrestricted Retry : The outstanding command is an idempotent command (refer to section
9.6.3.1) that is able to be retried without restrictions because that outstanding command has no
effect on user data or NVM subsystem state (e.g., an NVM Command Set Read command). Refer
to section 9.6.3.2.1.
• Delayed Retry: The outstanding command is an idempotent command for which any retry and/or
reporting of the result of that retry is required to be delayed until no further controller processing is
possible of that outstanding command because the outstanding command mod ifies user data or
NVM subsystem state (e.g., an NVM Command Set Write command, except as described in section
9.6.3.2.2). Refer to section 9.6.3.2.2.
• State-Dependent Retry : The outstanding command is not an idempotent command (e.g., a
Namespace Management command that creates a namespace) or is an idempotent command that
is able to affect behavior of other hosts. The procedures for recovery from such an outstanding
command depend on the extent, if any, to which that outstanding command has been processed
by the controller. Refer to section 9.6.3.2.3.
Sections 9.6.3.2.1, 9.6.3.2.2, and 9.6.3.2.3 define each command retry category and describe host
requirements and restrictions that prevent retry of outstanding commands from corrupting user data or
making unintended changes to NVM subsystem state.
9.6.3.2.1 Unrestricted Retry Commands
An outstanding command is an Unrestricted Retry command if that command:
a) is an idempotent command (refer to section 9.6.3.1); and
b) does not change more than ancillary state in the NVM subsystem (e.g., statistics such as the value
of the Data Units Read field in the SMART / Health Information log page (refer to Figure 206)).
For an Unrestricted Retry command:
• the Controller Capability Change (CCC) bit;
• the Namespace Inventory Change (NIC) bit;
