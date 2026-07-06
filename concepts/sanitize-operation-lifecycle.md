# Sanitize Operation Lifecycle

Sanitize starts or recovers a subsystem-scoped background erase operation using Block Erase, Crypto Erase, or Overwrite, with optional media verification and post-verification deallocation. It is prohibited in Exported NVM Subsystems. [PDF pp. 387-390](../_source/pages/page-387.md)

## Mental model

```text
                         successful processing + EMVS
[Idle] -- start erase ------------------------------> [Media Verification]
   |             |                                         |
   |             `-- no verification ----------------> [Idle]
   |                                                       |
   `-- Exit Failure Mode                                   | Exit Verification
                                                           v
                                            [Post-Verification Deallocation]
                                                           |
                                                           v
                                                        [Idle]
```

This explanatory preview reconstructs only the transitions explicitly selected by the Sanitize command. The complete restricted/unrestricted failure state machine remains in section 8.1.24 and will be integrated when that source range is processed. [PDF pp. 387-390](../_source/pages/page-387.md)

## Command and background-operation boundary

Sanitize command completion does not mean erase completion. A successful command starts the background operation and updates the Sanitize Status log before posting its CQE. Any unsuccessful command starts nothing, changes neither the log nor user data, and leaves the prior state intact. [PDF pp. 389-390](../_source/pages/page-389.md)

| Action/control | Contract |
|---|---|
| Block/Crypto/Overwrite | starts the selected background sanitize processing |
| `EMVS` | after successful Block or Crypto Erase with normal deallocation, enters Media Verification |
| Exit Media Verification | starts no new sanitize; transitions to deallocation only when already verifying |
| `NDAS` | requests retained allocation unless the controller inhibits it |
| `AUSE` | chooses restricted versus unrestricted completion mode |
| overwrite parameters | pass count (`0` means 16), optional pattern inversion, and 32-bit pattern |

Media Verification is invalid for Overwrite, with `NDAS=1`, or when the capability is absent. Exit Media Verification is invalid unless the target is currently in that state. [PDF pp. 388-390](../_source/pages/page-388.md)

## Start gates and serialization

The command is rejected when a PMR is enabled, any namespace is write-protected, firmware activation requiring reset is pending, any subsystem controller is suspended, the divided multi-Domain subsystem cannot start the operation, or unsupported CMB queue placement is used. New firmware activation is prohibited during an active sanitize. [PDF pp. 387-390](../_source/pages/page-387.md)

`SANICAP` gates supported erase actions, no-deallocate behavior, and verification support. Unsupported actions or illegal option combinations fail as invalid fields rather than silently degrading to another erase method. [PDF pp. 387-388](../_source/pages/page-387.md)

## Relationships

Sanitize Config (`17h`) chooses how a controller whose no-deallocate behavior is inhibited handles `NDAS=1`: error mode rejects the command, while warning mode permits processing and records `Sanitized Unexpected Deallocate` after successful completion. The subsystem-scoped setting has no effect when no-deallocate is not inhibited. [PDF p. 409](../_source/pages/page-409.md)

- [Sanitize Operation Status](sanitize-operation-status.md) is the persistent progress/outcome observation surface for this background lifecycle. [PDF pp. 302-305](../_source/pages/page-302.md)
- [Firmware Update Lifecycle](firmware-update-lifecycle.md), [Controller Migration](controller-migration.md), and [Controller Memory Windows](controller-memory-windows.md) define three independent serialization gates. [PDF pp. 388-390](../_source/pages/page-388.md)
- [Format NVM Lifecycle](format-nvm-lifecycle.md) defines namespace-format secure erase, which is distinct from subsystem-scoped Sanitize. [PDF pp. 218-220](../_source/pages/page-218.md)

## Evidence

- [Sanitize scope, actions, background phases, and capability gates, PDF p. 387](../_source/pages/page-387.md)
- [Media-verification transitions and serialization gates, PDF p. 388](../_source/pages/page-388.md)
- [Command-start atomicity and option fields, PDF p. 389](../_source/pages/page-389.md)
- [Action values, CQE/log ordering, and statuses, PDF p. 390](../_source/pages/page-390.md)
