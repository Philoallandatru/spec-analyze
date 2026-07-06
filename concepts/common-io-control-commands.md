# Common I/O Control Commands

Common I/O control commands are I/O-queue commands defined across I/O Command Sets for cancellation, volatile-cache commitment, and management-plane exchange. They are submitted only after the controller is ready and the I/O queues exist. [PDF pp. 482-486](../_source/pages/page-482.md)

## Mental model

```text
I/O Submission Queue
|-- Cancel ----------------> selected fetched commands -> immediate/deferred abort
|-- Flush -----------------> completed writes -> non-volatile media barrier
`-- I/O Management Receive -> controller management state -> host buffer
```

This explanatory grouping preserves the distinct control effects while showing their common I/O-queue execution plane. [PDF pp. 482-486](../_source/pages/page-482.md)

## Cancel selection and timing

Cancel targets commands on the same I/O SQ on which it is submitted. Single-command mode selects CID plus one NSID or all NSIDs; multiple-command mode requires `CID=FFFFh` and selects every other command on that SQ for one NSID or all NSIDs. The encoded SQID must match the submitting SQ. [PDF pp. 482-484](../_source/pages/page-482.md)

The controller applies cancellation to matching commands fetched before Cancel processing. It may include commands fetched during processing but need not affect commands not yet fetched. Each selected command may be aborted immediately before the Cancel CQE or deferred without ordering guarantees; the target command's eventual CQE reports Command Abort Requested when an abort occurs. [PDF pp. 483-484](../_source/pages/page-483.md)

| Cancel result field | Meaning |
|---|---|
| `CMDA` | commands immediately aborted by this Cancel |
| `CEDA` | matching commands eligible for a later deferred abort; saturates at `FFFFh` |

The Cancel CQE does not prove deferred outcomes. When immediate-abort count is zero, or for multiple-command cancellation, the host examines each target command's CQE. [PDF pp. 484-485](../_source/pages/page-484.md)

## Flush persistence barrier

Flush commits data and metadata for commands completed before its submission from volatile write cache to non-volatile media; the controller may commit additional namespace data. Broadcast NSID behavior depends on Identify Controller `FB`, and an FDP namespace may independently report no volatile cache through `VWCNP`. [PDF p. 485](../_source/pages/page-485.md)

If no volatile cache is present or enabled, Flush has no effect and normally completes successfully. During Sanitize it may still complete successfully. [PDF p. 485](../_source/pages/page-485.md)

## I/O Management Receive

I/O Management Receive selects an operation and a zero-based dword length, allowing prefix reads but never transferring beyond the defined structure. The Base-defined Reclaim Unit Handle Status operation returns one I/O-Command-Set-defined descriptor per accessible Placement Handle/Reclaim Group relationship. [PDF pp. 486-487](../_source/pages/page-486.md)

The status is sampled descriptor by descriptor and may omit effects of outstanding commands. The operation requires one concrete namespace and FDP enabled in its Endurance Group; reads past the result structure return zeros. [PDF pp. 486-487](../_source/pages/page-486.md)

## I/O Management Send

The Base-defined Reclaim Unit Handle Update operation takes a zero-based count of Placement Identifiers and retargets each referenced handle toward an empty Reclaim Unit. A unit already containing user data must be replaced; an already-empty unit may also be replaced. [PDF pp. 487-489](../_source/pages/page-487.md)

The count is bounded by both `MAXPIDS` and `NRG * NRUH`; each Placement Identifier must name a valid Reclaim Group and Placement Handle. Failure is not atomic: earlier identifiers may or may not already be updated. A concurrent write using one listed identifier may land in the unit referenced before processing or at command completion. [PDF pp. 488-489](../_source/pages/page-488.md)

## Relationships

- [Command Abort Semantics](command-abort-semantics.md) defines the immediate/deferred effect barriers reused by Cancel. [PDF pp. 194-195, 483-485](../_source/pages/page-194.md)
- [Common Controller Features](common-controller-features.md) configures the volatile write cache whose completed writes Flush commits. [PDF pp. 398-400, 485](../_source/pages/page-398.md)
- [Flexible Data Placement Configurations](flexible-data-placement-configurations.md) supplies the Reclaim Unit Handle model observed through I/O Management Receive. [PDF pp. 294-301, 486-487](../_source/pages/page-294.md)

## Evidence

- [Common I/O opcode matrix and Cancel scope, PDF p. 482](../_source/pages/page-482.md)
- [Cancel fetched-command and abort-timing rules, PDF pp. 483-485](../_source/pages/page-483.md)
- [Flush persistence and namespace capability rules, PDF p. 485](../_source/pages/page-485.md)
- [I/O Management Receive and Reclaim Unit Handle Status, PDF pp. 486-487](../_source/pages/page-486.md)
- [I/O Management Send and Reclaim Unit Handle Update, PDF pp. 487-489](../_source/pages/page-487.md)
