# Sanitize Operation Status

The Sanitize Status log is the persistent observation surface for sanitize progress and the most recent sanitize outcome. It is required when the controller reports Sanitize support, survives resets and power cycles, and contains valid data whenever `CSTS.RDY=1`. [PDF p. 302](../_source/pages/page-302.md)

## Mental model

```text
       never started
             |
             v
 [sanitize processing] ---> [media verification] ---> [post-verify deallocate]
             |                       |                           |
             +-----------------------+---------------------------+
                                     v
                         success / failure outcome

 LID 81h observes: progress + current/recent status + erase evidence
```

This explanatory preview uses only the state names introduced by the rendered Sanitize Status table; the complete operation state machine is deferred to its normative section. [PDF pp. 302-303](../_source/pages/page-302.md)

## Contract and invariants

`SPROG` is a numerator over 65,536 while sanitize processing or post-verification deallocation is active. It reports `FFFFh` when the operation is not sanitizing or while the target is in Media Verification, so it is not a general percentage for every sanitize state. [PDF p. 302](../_source/pages/page-302.md)

| Observed field | Meaning |
|---|---|
| `SOS` | never started, sanitized, sanitizing, failed, or sanitized with unexpected deallocation |
| `MVCNCLD` | requested media verification was canceled by subsystem-composition change or specified reset conditions |
| `GDE` | no user data has been written and no PMR enabled since manufacture-before-first-sanitize or since the latest successful sanitize |
| `OPC` | completed overwrite passes; zero for a non-overwrite operation |

[PDF p. 303](../_source/pages/page-303.md)

`SOS=100b` is the narrow "Sanitized Unexpected Deallocate" result: a No-Deallocate request completed successfully but all media allocated for user data was deallocated, and the operation state machine is Idle. Values `101b` through `111b` are reserved. [PDF pp. 303-304](../_source/pages/page-303.md)

## Time estimates and state detail

| Field group | What it estimates | Sentinel behavior |
|---|---|---|
| `ETO`, `ETBE`, `ETCE` | background processing for Overwrite, Block Erase, or Crypto Erase without the special no-deallocate media-modification path | `0h`: expected complete when the command completes; `FFFFFFFFh`: no estimate |
| `ETODMM`, `ETBENMM`, `ETCENMM` | the corresponding operation including additional media modification when No-Deallocate was requested and the capability selects that behavior | `0h`: expected complete when the command completes; `FFFFFFFFh`: no estimate |
| `ETPVDS` | Post-Verification Deallocation, if entered | `FFFFFFFFh`: no estimate |

[PDF pp. 304-305](../_source/pages/page-304.md)

`SSI.SANS` exposes the current state only when the version-validity bit permits it: Idle, Restricted Processing, Restricted Failure, Unrestricted Processing, Unrestricted Failure, Media Verification, or Post-Verification Deallocation. `SSI.FAILS` records the failure state only when `SOS` reports failure; otherwise it is zero. This is an observation mapping, not a substitute for the normative transition machine in section 8.1.24.3. [PDF p. 305](../_source/pages/page-305.md)

## Relationships

- [Sanitize Operation Lifecycle](sanitize-operation-lifecycle.md) defines the command-side start gates and transitions observed by this log. [PDF pp. 387-390](../_source/pages/page-387.md)
- This log observes the Sanitize operation; it is distinct from the secure-erase option of Format NVM. [PDF p. 302](../_source/pages/page-302.md) [PDF pp. 218-220](../_source/pages/page-218.md)
- The precise correspondence between `SOS`, cancellation, progress, and operation states is completed by the state-machine section referenced by the log definition. [PDF pp. 302-303](../_source/pages/page-302.md)

## Evidence

- [Support, persistence, readiness, and progress semantics, PDF p. 302](../_source/pages/page-302.md)
- [Rendered Sanitize Status bit field and initial status values, adjacent PDF p. 303](../_source/pages/page-303.md)
- [Remaining outcome, operation parameters, and time estimates, PDF p. 304](../_source/pages/page-304.md)
- [No-deallocate estimates and state-information encoding, PDF p. 305](../_source/pages/page-305.md)
