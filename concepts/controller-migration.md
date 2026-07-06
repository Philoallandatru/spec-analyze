# Controller Migration

Controller migration coordinates a stable read of one controller's state with suspension, segmented state transfer, queue reconstruction, and resumption. Migration Receive retrieves the image; Migration Send provides mandatory Suspend, Resume, and Set Controller State operations. [PDF pp. 372-383](../_source/pages/page-372.md)

## Mental model

```text
[running]
    | Suspend Notification (optional preparation; no state change)
    | Suspend
    v
[suspending: stop fetch, drain fetched commands except AERs]
    |
    v
[suspended] -- Get Controller State --> [stable state image]
    |                                      |
    | Set Controller State: 01 -> 00* -> 10
    |                       or 11          |
    |<------- verify + commit + recreate SQ/CQ state
    |
    `-- Resume --> [fetching and processing commands]
```

This explanatory state machine is reconstructed from the Suspend, Resume, segmented-transfer, and commit rules. A Controller Level Reset on the controller processing Suspend also releases the suspended condition. [PDF pp. 376-378](../_source/pages/page-376.md) [PDF p. 383](../_source/pages/page-383.md)

## Command roles

| Operation | Purpose | Key precondition or result |
|---|---|---|
| Migration Receive: Get Controller State | read a selected slice of standard and/or vendor state | suspension plus reset/property stability makes the image internally consistent |
| Migration Send: Suspend Notification | announce a future real suspend | does not itself suspend the target |
| Migration Send: Suspend | stop new command fetch and drain already fetched commands except AERs | repeated same-type suspend is not an error |
| Migration Send: Set Controller State | transfer and commit an image | target must be suspended, enabled, or an offline secondary controller |
| Migration Send: Resume | restart command fetch and processing | target must be suspended and received state must already be verified and committed |

The command directory and operation selectors are defined on PDF pp. 375-377; state-transfer preconditions and resume sequencing continue on pp. 377-383. [PDF pp. 375-383](../_source/pages/page-375.md)

## Receive-side snapshot contract

Migration Receive currently defines `SEL=0h` for Get Controller State. A 64-bit byte offset selects the returned slice and must be dword-aligned and within the available data; `NUMDL` is zero-based. Non-zero standard-version and vendor-UUID indices must name entries advertised by Supported Controller State Formats; zero omits that component and makes its returned size zero. [PDF pp. 372-374](../_source/pages/page-372.md)

If the target remains suspended for the entire Get Controller State operation, it has stopped processing commands. The returned structure is internally consistent provided no Controller Level Reset occurs and the host does not modify the target's properties during processing. Completion Dword 0 reports whether suspension held for the whole operation, rather than merely sampling the final state. [PDF pp. 373-374](../_source/pages/page-373.md)

## Suspension contract

Real suspension (`STYPE=1h`) makes the target stop fetching commands, finish commands already fetched except outstanding AERs, and then stop initiating transport transactions. The optional `DUDMQ` bit deletes an associated User Data Migration Queue on successful suspension; otherwise the queue may receive a later vendor-timed suspended marker after the drain completes. [PDF p. 376](../_source/pages/page-376.md)

While suspended, property accesses still behave normally except that setting `CC.EN` from zero to one has implementation-specific readiness behavior. CMB and PMR continue to operate normally, although the host should avoid changing properties, CMB, or PMR. [PDF p. 377](../_source/pages/page-377.md)

## Segmented state transfer

```text
single command:       [11 only + full image] ---------> verify/commit

multiple commands:    [01 first] -> [00 middle]* -> [10 last]
                           ^             completion required between commands
                           `-- a new 01 discards any unfinished prior sequence
```

This explanatory sequence preserves the exact `SEQIND` meanings. Starting without `01b`, or resuming before the final state has been verified and committed, is a Command Sequence Error. `10b` may carry zero dwords; `11b` carries the entire state in one command. [PDF pp. 378-379](../_source/pages/page-378.md) [PDF p. 377](../_source/pages/page-377.md)

Set Controller State uses a dword-aligned 64-bit offset and a dword count. Its standard-version and vendor-UUID indices select the interpretation of the corresponding image components; a zero index requires the related size to be zero, and both indices cannot be zero together. [PDF pp. 379-380](../_source/pages/page-379.md) [PDF pp. 382-383](../_source/pages/page-382.md)

## State image and queue reconstruction

```text
Controller State
|-- 48-byte header: version, suspended attribute, standard/vendor sizes
|-- NVMe Controller State (optional)
|   |-- header: SQ count, CQ count
|   |-- 24-byte SQ states, ascending QID
|   `-- 24-byte CQ states, ascending QID
`-- Vendor Specific Data (optional)
```

This hierarchy summarizes Figures 358-361. The standard queue image records creation parameters plus live SQ/CQ head and tail pointers; CQ state also carries slot-0 phase, interrupt enable/vector, and physical-contiguity attributes. [PDF pp. 380-382](../_source/pages/page-380.md)

If standard state is supplied, no I/O SQ or CQ may already exist on the target. Successful processing of the last segment verifies and commits the image, creates the specified queues, and restores their recorded state. If any segment fails, the host should retransmit the entire image so partial state is overwritten. [PDF p. 380](../_source/pages/page-380.md) [PDF p. 383](../_source/pages/page-383.md)

## Relationships

- [Identify Command Model](identify-command-model.md) publishes the state-version and vendor-UUID directories consumed by both migration commands. [PDF pp. 365-366](../_source/pages/page-365.md)
- [Controller Data Queues](controller-data-queues.md) defines the User Data Migration Queue used to track user-data modifications outside the controller-state image. [PDF pp. 207-209](../_source/pages/page-207.md)
- [Migration Change Tracking](migration-change-tracking.md) records host-memory or namespace user-data modifications while state is moved; suspension plus an empty final report closes the host-memory-change window. [PDF pp. 434-438](../_source/pages/page-434.md)
- [Queue Pair](queue-pair.md) supplies the queue identities, geometry, pointers, phase, and interrupt attributes reconstructed during commit. [PDF pp. 381-383](../_source/pages/page-381.md)
- [Controller Enable, Shutdown, and Reset](controller-enable-shutdown-reset.md) defines the Controller Level Reset that releases suspension and the `CC.EN` behavior referenced by migration. [PDF pp. 376-377](../_source/pages/page-376.md)

## Edge cases

- A target for Set Controller State must be suspended, enabled, or an offline secondary controller; otherwise the command fails with Invalid Controller Identifier. [PDF p. 377](../_source/pages/page-377.md)
- Resume fails with Controller Not Suspended if its target is running, and with Command Sequence Error when received state is not yet verified and committed. [PDF p. 377](../_source/pages/page-377.md)
- An image offset beyond the state size, a non-dword-aligned offset, an unadvertised format index, or an index/size mismatch is invalid. [PDF pp. 379-383](../_source/pages/page-379.md)
- Migration Send also reports Invalid Controller Data Queue and Not Enough Resources where applicable. [PDF p. 383](../_source/pages/page-383.md)

## Evidence

- [Migration Receive envelope and snapshot consistency, PDF pp. 372-374](../_source/pages/page-372.md)
- [Migration Send directory and Suspend, PDF pp. 375-376](../_source/pages/page-375.md)
- [Suspended behavior, Resume, and Set preconditions, PDF p. 377](../_source/pages/page-377.md)
- [Transfer sequence and command fields, PDF pp. 378-379](../_source/pages/page-378.md)
- [Controller State and queue-state layouts, PDF pp. 380-382](../_source/pages/page-380.md)
- [Commit behavior and command status, adjacent PDF p. 383](../_source/pages/page-383.md)
