# Controller Data Queues

A Controller Data Queue (CDQ) is a queue in host memory into which the controller posts a defined data type. The Controller Data Queue Admin command creates and deletes these queues; the Base-defined queue type is the User Data Migration Queue. [PDF pp. 206-210](../_source/pages/page-206.md)

## Mental model

```text
Host allocates host-memory pages
        |
        +-- contiguous: PRP1 -> one buffer
        `-- non-contiguous: PRP1 -> stable PRP List -> pages
        |
Create CDQ(queue type, size, controller)
        v
Controller ---- posts queue-type data ----> CDQ entries
        |
Delete CDQ(CDQID) or controller reset
        `-> host may release/modify mapping
```

This explanatory lifecycle combines rendered Figures 161-168. A PRP List used by a CDQ must remain at the same host-memory location and unchanged until deletion or reset; modifying it earlier is undefined. [PDF pp. 207-209](../_source/pages/page-207.md)

## Create contract

| Input | Meaning |
|---|---|
| `SEL=0h` | create CDQ; `SEL=1h` deletes one; vendor operations occupy `C0h..FFh` |
| `PRP1` | page-aligned host-memory buffer or PRP List; offset is zero |
| `PC` | contiguous buffer versus PRP List |
| `CDQSIZE` | queue size in dwords; multiple of queue-type entry size |
| `QT` | queue data type; Base value `0h` is User Data Migration Queue |

This compact reconstruction follows rendered Figures 161-165. CDQ memory must be host memory, not CMB or PMR. Per-command and subsystem-wide memory-range limits, plus per-controller and subsystem queue-count limits, gate creation. [PDF pp. 207-208](../_source/pages/page-207.md)

## User Data Migration Queue

A User Data Migration Queue logs user data modified while the specified controller is being migrated. Only one such queue may be associated with the same target controller, and queue-count exhaustion produces Not Enough Resources. The applicable I/O Command Set defines entry requirements and format. [PDF p. 209](../_source/pages/page-209.md)

```text
Create UDMQ(CNTLID) -> CQE.CDQID unique within processing controller
                              |
                              v
Delete CDQ(CDQID) ------------+
```

This explanatory identifier flow reflects Figures 166-168. `CDQID` is controller-local; deleting an identifier that does not exist on the processing controller fails with Invalid Controller Data Queue. [PDF pp. 209-210](../_source/pages/page-209.md)

## Relationships

Controller Data Queue Feature (`21h`) advances the host-owned head pointer and optionally arms a one-shot tail-slot trigger. A successful Set clears a pending tail event; when the trigger fires, its enable bit clears. Because the old trigger slot may be posted before reconfiguration completes, the host checks the queue after completion and rearms or disables as needed. [PDF pp. 415-416](../_source/pages/page-415.md)

Head movement must identify the current pointer when empty or a controller-posted entry when non-empty; the trigger must name a valid posted-entry slot. Invalid CDQ IDs have a dedicated error. [PDF p. 416](../_source/pages/page-416.md)

- One-shot AERs can announce a CDQ tail-pointer post or full condition and carry the `CDQID`. [PDF p. 202](../_source/pages/page-202.md)
- CDQ memory uses the common PRP/PRP List model but adds lifetime stability and host-memory-only constraints. [PDF pp. 173-174, 207-208](../_source/pages/page-173.md)
- Controller migration consumes User Data Migration Queue contents; entry semantics belong to the selected I/O Command Set. [PDF p. 209](../_source/pages/page-209.md)
- [Migration Change Tracking](migration-change-tracking.md) starts and stops User Data Migration Queue logging; it rejects missing queues, duplicate logging for the same target, and a target already suspended. [PDF pp. 436-437](../_source/pages/page-436.md)

## Evidence

- [Rendered Figure 161 and CDQ creation limits, PDF p. 207](../_source/pages/page-207.md)
- [Rendered Figures 162-165, create layouts, PDF p. 208](../_source/pages/page-208.md)
- [Rendered Figures 166-167, UDMQ and deletion, PDF p. 209](../_source/pages/page-209.md)
- [Rendered Figures 168-169, completion identifier and status, PDF p. 210](../_source/pages/page-210.md)
