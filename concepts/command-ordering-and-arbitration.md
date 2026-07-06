# Command Ordering and Arbitration

NVMe normally treats commands as independent: Submission Queue order does not impose processing, completion, or media ordering. Host software must impose application-required dependencies, except where the protocol defines a fused or command-set-specific atomic operation. [PDF pp. 120-121](../_source/pages/page-120.md)

## Mental model

```text
submitted commands in each SQ
          |
          v
 controller chooses candidate commands
          |
          +-- arbitration selects an SQ
          +-- Arbitration Burst limits starts before reselection
          v
 processing may overlap and complete out of submission order
          |
          `-- completion makes command state changes globally visible
```

This explanatory pipeline distinguishes selection from completion; selection order does not imply completion order. [PDF p. 121](../_source/pages/page-121.md)

## Fused and atomic operations

A fused operation is an optional two-command atomic unit. Both commands occupy adjacent entries in one Submission Queue, may wrap from its last slot to its first, execute in sequence without an intervening operation, and each receives its own completion. In a memory-based queue one Tail Doorbell update exposes both; a message-based transport delivers them in order with no intervening same-SQ capsule. [PDF pp. 119-121](../_source/pages/page-119.md)

| Failure or control action | Required result |
|---|---|
| Requested fused operation unsupported | Abort with Invalid Field in Command |
| Adjacent partner missing or malformed | Abort non-zero-`FUSE` command with Command Aborted due to Missing Fused Command |
| First command fails | Abort the second command |
| Second command fails | First completion status is defined by that fused operation |
| Host aborts pair | Submit an Abort separately for each command |

Atomic-operation definitions and further fused-operation applicability belong to the applicable I/O Command Set specification. [PDF pp. 120-121](../_source/pages/page-120.md)

## Arbitration mechanisms

```text
Round robin:       ASQ --+
                  I/O SQ +--> equal-priority RR --> processing
                  I/O SQ +

Weighted + urgent: Admin ----------------------- strict priority 1 --+
                   Urgent SQs -- RR ------------ strict priority 2 --+--> processing
                   High/Medium/Low -- RR -- WRR - strict priority 3 --+
```

This compact reconstruction of Figures 80-81 was checked against rendered PDF pages 122-123. Every controller supports round robin. Weighted round robin with urgent priority and vendor-specific arbitration are optional. Under weighted arbitration, Admin outranks Urgent, which outranks the weighted High/Medium/Low pool; excessive Urgent work may starve that pool. Round robin operates among queues at the same level, and the smaller of remaining weight credits and Arbitration Burst limits starts per queue visit. [PDF pp. 121-123](../_source/pages/page-121.md)

## Outstanding-command boundary

A command is outstanding after submission until the host receives its completion or obtains another protocol-defined proof that processing can no longer continue. Such proofs include a successful effective Abort or Cancel, controller-not-ready/shutdown-complete observations for non-Fabrics commands, deletion of the carrying memory-based SQ, successful Fabrics Disconnect of that I/O queue, or restored communication to the same controller after loss. [PDF pp. 123-124](../_source/pages/page-123.md)

## Evidence

- [Ordering and fused-operation contract, PDF pp. 120-121](../_source/pages/page-120.md)
- [Candidate selection, completion visibility, and arbitration, PDF p. 121](../_source/pages/page-121.md)
- [Figures 80-81 and arbitration classes, PDF pp. 122-123](../_source/pages/page-122.md)
- [Outstanding-command termination conditions, PDF pp. 123-124](../_source/pages/page-123.md)
