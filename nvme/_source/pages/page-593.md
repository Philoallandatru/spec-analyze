---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 593
headings: []
---

# Source page 593

NVM Express® Base Specification, Revision 2.1

571
If a Sanitize command is completed with any status code other than Successful Completion, then the
controller shall not start the sanitize operation and shall not update the Sanitize Status log page. The
controller shall ignore Critical Warning(s) in the SMART / Health Information log page (e.g., read only mode)
and shall attempt to complete the sanitize operation requested. Refer to section 5 for further information
about restrictions on Admin commands during the processing of a Sanitize command.
Following a successful sanitize operation, the values of user data (including protection information (PI), if
any, and non-PI metadata, if any) that result from an audit (refer to section 1.5.11) of the sanitization target
are defined in the I/O command set specifications.
The Sanitize Status log page (refer to section 5.1.12.1.33) indicates estimated times for sanitize operations
and a consistent snapshot of information about the most recently started sanitize operation, including
whether a sanitize operation is in progress, the sanitize operation parameters and the status of the most
recent sanitize operation. The controller shall report that a sanitize operation is in progress if:
• sanitize processing is in progress (including additional media modification, if required);
• the sanitization target is in the Media Verification state; or
• the sanitization target is in the Post-Verification Deallocation state.
If a sanitize operation is not in progress, then the Global Data Erased (GDE) bit in the log page indicates
whether the sanitization target may contain any user data (i.e., whether the sanitization target has been
written to since the most recent successful sanitize operation).
The Sanitize Status log page shall be:
• initialized before any controller in the NVM subsystem is ready as described in sections 3.5.3 and
3.5.4; and
• updated when any state transition occurs (refer to section 8.1.24.3).
The Sanitize Status log page is updated periodically during a sanitize operation to make progress
information available to hosts.
During a sanitize operation, the host may periodically examine the Sanitize Status log page to check for
progress, however, the host should limit this polling (e.g., to at most once every several minutes) to avoid
interfering with the progress of the sanitize operation itself.
The Sanitize Progress (SPROG) field in the Sanitize Status log page indicates progress during states that
may take long times to complete (i.e., the Restricted Processing state, the Unrestricted Processing state,
and the Post-Verification Deallocation state). The SPROG field is cleared to 0h upon entry to any of those
states, and while in any of those states is updated as described in Figure 291. The SPROG field shall not
be modified under any conditions not explicitly permitted by this specification.
A sanitize operation completes when the sanitization target enters any of the following states (refer to
section 8.1.24.3):
• the Idle state;
• the Restricted Failure state; or
• the Unrestricted Failure state.
Upon completion of a sanitize operation or upon entry to the Media Verification state, the host should read
the Sanitize Status log page with the Retain Asynchronous Event bit cleared to ‘0’ (which clears the
asynchronous event, if one was generated).
If a sanitize operation fails (i.e., the sanitization target enters the Restricted Failure state or the Unrestricted
Failure state), all controllers in the NVM subsystem shall abort any command not allowed during a sanitize
operation with a status code of Sanitize Failed (refer to section 8.1.24.3) until a subsequent sanitize
operation is started or successful recovery from the failed sanitize operation occurs. A subsequent
successful sanitize operation or the Exit Failure Mode action may be used to recover from a failed sanitize
operation. Refer to section 5.1.22 for recovery details.
If the Sanitize command is supported, then all controllers in the NVM subsystem shall:
