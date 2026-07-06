# Queue Pair

A queue pair consists of a Submission Queue used by the host to submit commands and an associated Completion Queue used by the controller to post completions. Admin queues handle management/control; I/O queues carry I/O command sets. [PDF pp. 41-43](../_source/pages/page-041.md)

## Mental model

```text
Host                 Submission Queue       Controller       Completion Queue
 |  write command ---------->|                  |                    |
 |  notify / ring doorbell ->|----------------->|                    |
 |                           |     execute      |                    |
 |                           |                  |---- completion --->|
 |<-------------------------- interrupt or poll ---------------------|
```

For memory-based transports queues reside in memory. Queue mapping is not universally one-to-one: I/O Submission Queues may share an I/O Completion Queue, while the Admin pair is one-to-one. In NVMe over Fabrics, every I/O Submission Queue maps one-to-one to an I/O Completion Queue. [PDF pp. 41-43](../_source/pages/page-041.md)

```text
Memory-based I/O                 NVMe over Fabrics I/O
SQ A --+                         SQ A ------------ CQ A
       +--> CQ X                 SQ B ------------ CQ B
SQ B --+                         (no n:1 I/O mapping)
```

This explanatory comparison is reconstructed from the rendered queue-pair examples and the Fabrics differences list. [PDF pp. 41-43](../_source/pages/page-041.md)

The Admin Queue is specifically the Submission Queue and Completion Queue with identifier 0. Its Submission Queue is uniquely associated with its Completion Queue. [PDF p. 28](../_source/pages/page-028.md)

| Event | Memory-based model | Message-based model |
|---|---|---|
| Submission | Tail Doorbell write advances past the occupied SQ slot | Host adds a command capsule to a Submission Queue |
| Completion | Controller finishes processing, updates the CQ entry status, and posts it to the associated CQ | Same protocol-level completion condition |

These submission definitions distinguish placement of a command from the point at which the protocol considers it submitted. [PDF p. 29](../_source/pages/page-029.md)

An I/O command is defined by where it is submitted: an I/O Submission Queue. An I/O Completion Queue may be associated with one or more I/O Submission Queues, preserving the many-to-one mapping shown above. [PDF p. 32](../_source/pages/page-032.md)

For memory-based queues, the SQ and CQ are fixed-slot circular buffers. The host advances the SQ Tail doorbell after adding commands and advances the CQ Head after consuming completions; a CQE Phase Tag lets the host detect newly posted entries without a register read. The controller fetches SQEs in order but may execute their commands out of order. [PDF pp. 42-43](../_source/pages/page-042.md)

NVMe over Fabrics does not support CQ flow control, so the host must reserve completion capacity before submission. SQ flow control may be disabled by agreement, in which case the host must also ensure SQ capacity. [PDF p. 43](../_source/pages/page-043.md)

Except for fused operations, a controller may process commands within or across Submission Queues in any order, and media commitment need not follow submission order. Queue priority belongs to the Submission Queue, not to an individual command. Completions are also unordered; the CQE's Submission Queue Identifier and Command Identifier provide correlation. [PDF pp. 59-60](../_source/pages/page-059.md)

The host creates both members of every I/O queue pair before use. Completion notification is transport-specific, while the protocol-level correlation remains the SQID/CID pair. [PDF p. 60](../_source/pages/page-060.md)

## Memory-based queue lifecycle and flow control

Before creating any I/O queue, the host uses Number of Queues (`07h`) to request separate SQ and CQ counts. Counts are zero-based, the controller may allocate fewer or more than requested, and the allocation remains fixed until reset. Issuing Set after any I/O SQ or CQ exists is a Command Sequence Error. [PDF pp. 427-428](../_source/pages/page-427.md)

Interrupt Coalescing (`08h`) applies only to I/O queues and recommends signaling when either a 100-microsecond time bound or completion-count threshold is met. Zero in either field effectively permits immediate interrupts. Interrupt Vector Configuration (`09h`) disables coalescing per already-associated I/O interrupt vector; Admin CQ interrupts are never coalesced. [PDF pp. 428-429](../_source/pages/page-428.md)

```text
configure Admin queues -> negotiate queue count -> create CQ -> create associated SQ(s)
        use circular Head/Tail pointers and doorbells       |
delete CQ <- delete every associated SQ <-------------------'
```

Completion Queues must exist before their associated Submission Queues and must outlive them. The host uses SQ Tail and CQ Head doorbells; the controller reports consumed SQ slots through `SQHD`, while a changed Phase Tag marks a newly posted CQE. A full CQ blocks further posts to that CQ and may stall its associated SQs, but other SQs must continue. [PDF pp. 107-108](../_source/pages/page-107.md)

| Circular-buffer condition | Pointer relation | Consequence |
|---|---|---|
| Empty | Head equals Tail | consumer has no entry |
| Full | Head equals Tail plus one, with wrap | one slot remains unused; producer must stop |

The one-unused-slot rule distinguishes Full from Empty without another state bit. These conditions were checked against rendered Figures 73-74 on PDF pages 109-110. [PDF pp. 109-110](../_source/pages/page-109.md)

Invalid doorbell values trigger an Invalid Doorbell Write Value asynchronous event when a request is outstanding; the affected queue is then deleted and recreated. Modifying an SQE after submission but before consumption, or a CQE after posting but before host consumption, is undefined. [PDF p. 108](../_source/pages/page-108.md)

Bulk abort uses Multiple Command Cancel or deletion/recreation of the I/O Submission Queue. If communication is lost before all completions or successful queue deletion, retry/recovery must follow the communication-loss rules to avoid interaction with outstanding commands. [PDF p. 109](../_source/pages/page-109.md)

For Fabrics, each queue is a unidirectional capsule channel and the Connect command creates Submission and Completion Queues as a pair. Transport delivery replaces memory doorbells, and transport-specific low-level congestion/reliability flow control is outside the Base Specification's end-to-end contract. [PDF p. 110](../_source/pages/page-110.md)

## Message-based queue lifecycle and capacity

```text
[transport connection established]
              |
       Connect on the queue being created
              v
[queue pair created] -- AUTHREQ != 0 --> [authentication only]
              |                                  |
              | AUTHREQ = 0                      | authentication succeeds
              `---------------------+------------'
                                    v
                    Admin: Fabrics only while CC.EN=0
                    Admin: Fabrics + Admin while RDY=1
                    I/O: I/O commands while controller enabled
                                    |
                   Disconnect on that I/O SQ (last SQ command)
                                    v
                  [final Disconnect response] -> [queue deleted]
```

This explanatory state sequence was checked against rendered PDF pages 114-117. Connect is sent exactly once over a pre-established transport connection and creates both queues only on success. A duplicate Queue ID is a Command Sequence Error. The message-based model therefore does not use `AQA`, `ASQ`, `ACQ`, or the memory-model Create/Delete I/O Queue Admin commands. [PDF pp. 114-115](../_source/pages/page-114.md)

Individual I/O Queue deletion is isolated only when both endpoints advertise support. Otherwise, deleting an I/O Queue or losing its transport connection terminates the association. A successful Disconnect is the last command processed on that SQ and its response is the last CQE consumed for that pair; commands before it may complete or abort, but none may be processed after its completion is posted. [PDF pp. 115-117](../_source/pages/page-115.md)

The Connect command negotiates SQ flow control per queue pair. It is disabled only when the host sets `DISSQFC=1` and the controller returns `SQHD=FFFFh`; otherwise it remains enabled and later response capsules carry valid SQ head progress, subject to the defined update optimization. [PDF pp. 474-475](../_source/pages/page-474.md)

Connect also declares connecting entity, individual-I/O-queue-deletion support, optional I/O priority, Admin KATO, and optional I/O SQ-to-NVM-Set association. Queue ID zero creates the Admin pair; IDs 1 through 65,534 create I/O pairs, with zero-based SQ size. [PDF pp. 475-476](../_source/pages/page-475.md)

Disconnect is valid only on the I/O Queue it deletes; Admin submission is Invalid Queue Type. It does not itself delete the transport connection, which may be released only after all queues using it are gone. Its response is the final CQE and the controller performs no further command work on that pair; host submissions after Disconnect are undefined. [PDF pp. 478-479](../_source/pages/page-478.md)

| Message-based capacity rule | Required behavior |
|---|---|
| SQ flow control enabled | Controller reports consumed slots via `SQHD`; full means Head is one beyond Tail, so usable capacity is size minus one |
| SQ flow control disabled | Host keeps outstanding commands below SQ size; `SQHD` is zero/reserved after Connect |
| CQ capacity | No CQ Head/Tail flow control; host sizes for all possible outstanding responses |

With enabled SQ flow control, submitting to a full SQ is fatal: the controller stops command processing, sets `CSTS.CFS`, terminates the transport connection, and ends the association. The same fatal outcome applies when disabled flow control permits outstanding commands to reach or exceed SQ size. A too-small message-based CQ instead has undefined results when a response reaches a full CQ. [PDF pp. 117-118](../_source/pages/page-117.md)

The transport may omit `SQHD` only from ordinary successful responses, never Connect or Disconnect responses. Responses without an update may arrive in any order, but transmitted `SQHD` values must progress in order and be sent periodically to prevent SQ starvation. [PDF pp. 118-119](../_source/pages/page-118.md)

## Queue attributes and transport guarantees

| Attribute | Contract |
|---|---|
| Size | 16-bit 0's-based slot count; minimum two slots; one slot is unusable under Head/Tail pointer semantics |
| I/O queue identifier | `1` through `65535`; Admin queues use identifier `0` |
| Priority | Urgent, High, Medium, or Low only has effect with weighted round robin plus urgent priority |
| Coordination | Admin actions may affect multiple I/O queue pairs, so host software coordinates them with I/O-owning threads |

I/O queues may contain at most 65,536 slots subject to `CAP.MQES`; Admin queues are capped at 4,096 slots, with the message-based Admin SQ further bounded by the Discovery Log Page Entry `ASQSZ`. `MAXCMD` describes how many commands a controller processes at once per I/O Queue and can guide CQ sizing, but is not the queue's slot count. [PDF pp. 119-120](../_source/pages/page-119.md)

```text
ordinary capsules: transport may reorder reliably delivered capsules
fused pair:         First -> Second, with no same-SQ capsule inserted between
SQHD-bearing CQEs:  update 1 -> update 2 -> update 3 (delivered in order)
transport failure:  delete affected I/O queue/connection, or end association
```

This explanatory transport contract separates reliable delivery from ordering. A transport may report detected errors through command status and the Error Information log, but unrecoverable loss must cause queue/connection deletion or association termination. Altering a response capsule after controller submission and before host delivery is undefined. [PDF p. 119](../_source/pages/page-119.md)

## Queue-level reset

```text
quiesce affected SQ(s)
        |
memory: Delete SQ(s) -> Delete CQ -> Create CQ -> Create SQ(s)
fabrics: Disconnect I/O pair -> Connect again with nonzero QID
        |
        `-> queue attributes may change
```

This explanatory sequence was checked against rendered PDF page 141. A queue-level reset is deletion followed by recreation, not a distinct wire operation. For a memory-based shared CQ, every associated SQ should be deleted before the CQ and recreated after it; an SQ without its corresponding CQ has undefined behavior. The host should make the affected queues idle first because deletion aborts pending commands and those aborts may or may not produce CQEs. [PDF p. 141](../_source/pages/page-141.md)

## Evidence

Number of Queues (`07h`) is an initialization-only negotiation before any I/O queues exist. Requested and allocated SQ/CQ counts are zero-based and may differ because of availability or allocation units; allocation remains fixed until reset. [PDF pp. 427-428](../_source/pages/page-427.md)

Interrupt Coalescing (`08h`) applies only to I/O queues and combines a 100-microsecond time recommendation with a zero-based completion threshold; either zero effectively disables aggregation. Interrupt Vector Configuration (`09h`) may disable coalescing per vector only after that vector is associated with an I/O CQ; the Admin CQ never uses coalescing. [PDF pp. 428-429](../_source/pages/page-428.md)

- [Command-set and queue overview, PDF p. 41](../_source/pages/page-041.md)
- [Admin Queue definition, PDF p. 28](../_source/pages/page-028.md)
- [Submission and completion definitions, PDF p. 29](../_source/pages/page-029.md)
- [I/O queue and command definitions, PDF p. 32](../_source/pages/page-032.md)
- [Circular queues, identifiers, and Phase Tag, PDF pp. 42-43](../_source/pages/page-042.md)
- [Fabrics queue mapping and flow-control differences, PDF p. 43](../_source/pages/page-043.md)
- [Processing, priority, completion ordering, and I/O queue creation, PDF pp. 59-60](../_source/pages/page-059.md)
- [Memory queue setup, pointer use, posting, and flow control, PDF pp. 107-108](../_source/pages/page-107.md)
- [Abort and circular-buffer Empty/Full conditions, PDF pp. 109-110](../_source/pages/page-109.md)
- [Fabrics unidirectional queue model, PDF p. 110](../_source/pages/page-110.md)
- [Fabrics queue creation, authentication state, and deletion, PDF pp. 114-117](../_source/pages/page-114.md)
- [Fabrics SQ flow control and CQ capacity, PDF pp. 117-119](../_source/pages/page-117.md)
- [Connect flow-control and queue attributes, PDF pp. 474-477](../_source/pages/page-474.md)
- [Disconnect final-response barrier, PDF pp. 478-479](../_source/pages/page-478.md)
- [Transport guarantees and queue attributes, PDF pp. 119-120](../_source/pages/page-119.md)
- [Queue-level reset for memory- and message-based transports, PDF p. 141](../_source/pages/page-141.md)
For PCIe, I/O Completion Queues are created before the Submission Queues that reference them. Both use zero-based `QSIZE`, non-zero queue identifiers bounded by the Number of Queues Feature, page-aligned PRP storage, and stable PRP Lists until successful deletion or controller reset. `CAP.CQR` may prohibit non-contiguous queues; CMB placement adds the `CMBLOC.CQPDS` gate. [PDF pp. 441-443](../_source/pages/page-441.md)

An I/O SQ selects its CQ, arbitration priority, contiguity, and optional NVM Set association. A non-zero NVM Set association must name an advertised NVM Set when the capability is supported, and the host should keep namespaces from other NVM Sets off that SQ. [PDF p. 443](../_source/pages/page-443.md)

```text
Create CQ(qid, storage, interrupt)
        |
        `-- Create SQ(qid, CQID, priority, optional NVM Set)
                    |
Delete SQ: drain/abort prior commands; completion closes CQE production
        |
        `-- Delete CQ only after every associated SQ is gone
```

This explanatory lifecycle preserves the dependency and deletion barriers of the rendered command tables. [PDF pp. 441-445](../_source/pages/page-441.md)

Successful SQ deletion explicitly completes commands that already received CQEs and implicitly completes all remaining commands as Command Aborted due to SQ Deletion; after the delete completion, no further CQE may be posted for that SQ. A CQ cannot be deleted while any SQ still references it. Admin queues cannot be deleted by these commands. [PDF pp. 444-445](../_source/pages/page-444.md)

### Shadow Doorbell and EventIdx buffers

Doorbell Buffer Config supplies two single-page, page-aligned host-memory mirrors for emulated controllers: the host updates Shadow Doorbell entries and the para-virtualized controller updates EventIdx entries. Entries alternate SQ tail and CQ head by queue identifier at stride `4 << CAP.DSTRD`; the page must cover identifiers through `max(NSQA, NCQA)`. [PDF pp. 445-446](../_source/pages/page-445.md)

This configuration is controller-scoped, supports neither metadata nor SGLs, and is lost on Controller Level Reset. Invalid buffer addresses fail the command. [PDF pp. 445-446](../_source/pages/page-445.md)
