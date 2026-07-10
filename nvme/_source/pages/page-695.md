---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 695
headings: []
---

# Source page 695

NVM Express® Base Specification, Revision 2.1

673
an associated application, filesystem or database) that is able to determine the extent, if any, to which the
outstanding command has been processed and enforce ordering requirements among commands (refer to
section 3.4.1).
A host should treat an outstanding command that is a State -Dependent Retry command as having
command ordering requirements enforced by higher -level software with respect to other commands (refer
to section 3.4.1). Hence, for any such command, a host should not report completion or error status,
including errors caused by communication loss, to higher-level software (e.g., an associated application, a
filesystem or database), until the host has determined that no further controller processing of the
outstanding command and retries, if any, of that command is possible. If a host violates this
recommendation, corruption of user data or unintended changes to NVM subsystem state are possible ;
refer to Annex B.6.2 for an example where unintended changes occur to NVM subsystem state.
A host that does not adhere to the recommendations in this section for handling outstanding commands
that are State-Dependent Retry commands risks causing corruption of user data and unintended changes
to NVM subsystem state. It is strongly recommended tha t host NVMe implementations adhere to these
recommendations to avoid these outcomes.
