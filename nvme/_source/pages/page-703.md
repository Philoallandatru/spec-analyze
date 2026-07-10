---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 703
headings: ["B.6.2. Non-Idempotent Command"]
---

# Source page 703

NVM Express® Base Specification, Revision 2.1

681
software until the host is able to determine that no further controller processing of that command and any
retry of that command is possible (refer to section 9.6.2).
Figure 759: Write after Write

B.6.2. Non-Idempotent Command
In the example shown in Figure 760, the host loses communication with Controller 1 and does not receive
a response from Controller 1 for a Namespace Management command that creates a namespace (refer to
section 5.1.21). The host ensures that no further controller processing of that command is possible (refer
to section 9.6.2), and then retries that command on Controller 2, which creates a second namespace .
This example shows why higher -level software (e.g., an associated application, filesystem or database)
should take steps to determine that a retry of a non -idempotent command does not cause unintended
changes to NVM subsystem state (e.g., number of namespaces).
Host detects a loss
of communication.
Host


Write A at Location X
Controller 1
 Controller 2
Retry: Write A at Location X
Write B at Location X
Write A on Controller 1
takes effect.
Controller processing command.
Location X is A not B!
Success
Success
Unsafe report to
higher-level
software for
completion of
Write A.
Safe report to
higher-level
software for
completion of
Write A.
Host determines that Controller
1 has stopped processing that
command.
