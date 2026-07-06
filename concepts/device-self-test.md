# Device Self-test

Device Self-test is an Admin operation that starts a short, extended, Host-Initiated Refresh, or vendor-specific test, or aborts an operation already in progress. Host-Initiated Refresh is itself a device self-test operation. [PDF pp. 210-212](../_source/pages/page-210.md)

## Mental model

```text
                         start short / extended / refresh
[no test in progress] ------------------------------------> [test in progress]
        ^                                                           |
        |                                                           | new start
        | success, log unchanged                                    `--> reject
        |                                                           |
        `----------------------- abort command ----------------------'
                                  |
                                  `-> abort operation -> write newest result
                                      -> clear current result -> success
```

This explanatory state machine reconstructs the non-vendor actions in rendered Figure 173. [PDF pp. 211-212](../_source/pages/page-211.md)

## Scope and selectors

| `NSID` | Included scope |
|---|---|
| `00000000h` | controller only; no namespace |
| `00000001h`-`FFFFFFFEh` | one valid, active namespace |
| `FFFFFFFFh` | all attached namespaces accessible when the operation starts |

For Host-Initiated Refresh, the controller ignores `NSID`. The table condenses rendered Figure 170. [PDF p. 210](../_source/pages/page-210.md)

The `STC` action code distinguishes short (`1h`), extended (`2h`), refresh (`3h`), vendor-specific (`Eh`), and abort (`Fh`). Command Dword 15 is vendor-specific only with `STC=Eh`; otherwise it is reserved. [PDF p. 211](../_source/pages/page-211.md)

## State and completion contract

With no operation running, a valid short, extended, or refresh command updates the current Device Self-test Status result, starts the operation, and then completes successfully. An abort with no operation running succeeds without modifying the log. [PDF pp. 211-212](../_source/pages/page-211.md)

With an operation running, a new standard start is rejected as Device Self-test in Progress. Abort performs an ordered transition: abort the operation, create the newest-result log entry, clear the current result, then complete successfully. Vendor-specific behavior remains vendor-specific. [PDF pp. 211-212](../_source/pages/page-211.md)

The in-progress scope is controller-local when `SDSO=0` and subsystem-wide when `SDSO=1`. Completion is posted after the applicable state actions. [PDF p. 212](../_source/pages/page-212.md)

## Status and result log

```text
current operation + percent complete
                 |
                 v
completion/abort -> push newest result at slot 1
                    shift older results toward slot 20
```

The Device Self-test log exposes the current operation (`none`, `short`, or `extended` for Base-defined values), a percentage-complete field that is meaningful only while a test is active, and the twenty newest completed or aborted results in newest-first order. [PDF pp. 238-239](../_source/pages/page-238.md)

When fewer than twenty results exist, unused entries use status `Fh` and clear the remaining result fields. A controller completing or aborting a test creates the new result before clearing the current-operation field, preserving the transition observed by the host. [PDF pp. 238-239](../_source/pages/page-238.md)

Each 28-byte result records the requested test code and its outcome: success, explicit abort, reset, namespace removal, Format, execution failure, failed segment, unknown abort, or sanitize. Validity bits independently qualify the optional namespace, failing LBA, Status Code Type, and Status Code fields; a first-failing segment number is meaningful only for the outcome that explicitly identifies it. [PDF pp. 240-241](../_source/pages/page-240.md)

| Result evidence | Use |
|---|---|
| Power-on hours | Timestamp-like age at completion/abort, excluding low-power-state time. |
| Namespace + failing LBA | Locate the failure only when their validity bits are set. |
| SCT + SC | Preserve completion-style diagnostic status only when separately marked valid. |
| Vendor-specific | Carry implementation-defined detail without changing Base-defined validity rules. |

[PDF pp. 240-241](../_source/pages/page-240.md)

## Relationships

- Operation status and history are exposed through the Device Self-test log page. [PDF pp. 211-212](../_source/pages/page-211.md)
- Namespace validity and controller-relative attachment affect ordinary self-tests, while Host-Initiated Refresh deliberately ignores the namespace selector. [PDF p. 210](../_source/pages/page-210.md)

## Evidence

- [Rendered namespace scope and command boundary, PDF p. 210](../_source/pages/page-210.md)
- [Rendered action codes and first half of processing matrix, PDF p. 211](../_source/pages/page-211.md)
- [Rendered processing-matrix continuation, scope note, and completion status, PDF p. 212](../_source/pages/page-212.md)
- [Rendered Device Self-test log header and newest-first result list, PDF pp. 238-239](../_source/pages/page-238.md)
- [Rendered Self-test Result structure, outcomes, and validity-gated diagnostics, PDF pp. 240-241](../_source/pages/page-240.md)
