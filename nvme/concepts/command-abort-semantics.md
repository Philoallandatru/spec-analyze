# Command Abort Semantics

Abort requests cancellation of one previously submitted command identified by Submission Queue ID and Command ID. Successful processing of Abort does not guarantee that the target command was aborted. [PDF pp. 194-195](../_source/pages/page-194.md)

## Mental model

```text
Host                 Controller                         target command
 |-- Abort(SQID,CID) --->|                                    |
 |                       +-- immediate abort possible? -------+
 |                       |        yes: stop later effects      |
 |                       |        no: may defer or do nothing  |
 |<-- Abort CQE (IANP) --|                                    |
 |<-- target CQE: Command Abort Requested, if aborted --------|
```

This explanatory sequence preserves the two independent observations: `IANP` reports whether an immediate abort occurred before the Abort CQE, while the target command's own CQE tells the host whether a deferred abort later occurred. [PDF pp. 194-195](../_source/pages/page-194.md)

## Immediate versus deferred abort

| Mode | Ordering/effect contract |
|---|---|
| immediate, target CQE first | target posts Command Abort Requested before Abort CQE |
| immediate, no later effects guaranteed | Abort CQE and target CQE may be posted in either order |
| deferred | target may be aborted after Abort CQE; target CQE carries Command Abort Requested |
| no abort | target completes normally or was not found/already completed |

An immediate abort is legal only if, after the Abort CQE is posted, the target causes no further host-memory access and no change to media, namespace, NVM Set, Endurance Group, controller, Domain, or subsystem state. An initiated but incomplete data transfer therefore prevents immediate abort when the target CQE has not already been posted. [PDF pp. 194-195](../_source/pages/page-194.md)

## Command and completion fields

```text
Abort CDW10: 31                 16 15                  0
             +-------------------+---------------------+
             | target CID        | target SQID         |
             +-------------------+---------------------+

Abort CQE DW0 bit 0: IANP = 0 immediate abort performed
                     IANP = 1 immediate abort not performed
```

These reconstructed layouts follow rendered Figures 143-144. Concurrent Abort execution is limited by Identify Controller `ACL`; excess requests may complete with Abort Command Limit Exceeded. [PDF pp. 194-195](../_source/pages/page-194.md)

For bulk cancellation, the host should prefer Cancel when supported or delete/recreate the I/O Submission Queue instead of exceeding the Abort limit with many individual requests. [PDF p. 194](../_source/pages/page-194.md)

## Relationships

- Abort targets a command by the same `SQID` plus `CID` pair used for CQE correlation. [PDF pp. 160-161, 194](../_source/pages/page-160.md)
- Queue deletion is the fallback for many outstanding commands and reconnects abort behavior to Queue Level Reset. [PDF pp. 109, 141, 194](../_source/pages/page-109.md)
- Immediate abort is an effects barrier, not merely an earlier completion ordering. [PDF pp. 194-195](../_source/pages/page-194.md)

## Evidence

- [Rendered Figure 143, Abort CDW10, PDF p. 194](../_source/pages/page-194.md)
- [Immediate-abort effects barrier and rendered Figures 144-145, PDF p. 195](../_source/pages/page-195.md)
