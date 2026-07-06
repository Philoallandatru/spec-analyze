# Controller Virtualization Resources

Controller virtualization resources are the flexible and private Virtual Queue (`VQ`) and Virtual Interrupt (`VI`) capacity managed by a primary controller for itself and its associated secondary controllers. [PDF pp. 366-368](../_source/pages/page-366.md)

## Mental model

```text
Primary controller resource pool
|-- VQ flexible total ----+-- assigned to secondary controllers
|                         `-- allocated to primary
|-- VQ private total -------- primary only
|-- VI flexible total ----+-- assigned to secondary controllers
|                         `-- allocated to primary
`-- VI private total -------- primary only
```

This explanatory allocation view preserves the separate VQ/VI pools and flexible/private ownership classes; exact byte positions remain in Figure 330. [PDF pp. 366-367](../_source/pages/page-366.md)

## Contract and invariants

| View | Meaning |
|---|---|
| Resource support | If the primary supports VQ or VI resources, every associated secondary controller supports that resource type. |
| Flexible total / assigned | Pool capacity across the primary and its secondaries, and the portion assigned to secondaries. |
| Allocated to primary / private total | Flexible allocation currently retained by the primary, plus primary-only capacity. |
| Secondary maximum / preferred granularity | Per-secondary ceiling and allocation unit that minimizes wasted implementation resources. |

The flexible allocation to the primary may change after an applicable Controller Level Reset when Virtualization Management programmed a new value; its default is implementation-specific. [PDF p. 367](../_source/pages/page-367.md)

## Secondary Controller directory

`CNS=15h` returns up to 127 secondary controllers associated with the primary processing Identify, beginning with controller identifiers greater than or equal to `CNTID`. Offline controllers remain present, including SR-IOV VFs disabled by configuration. [PDF pp. 367-368](../_source/pages/page-367.md)

Each entry binds the secondary and primary Controller IDs to online/offline state, optional SR-IOV VF number, and current flexible VQ/VI assignments. A non-VF secondary reports VF number zero. [PDF p. 368](../_source/pages/page-368.md)

## Virtualization Management lifecycle

```text
[secondary online] -- Offline --> [secondary offline, flexible VQ/VI removed]
                                          |
                                          +-- Assign VQ/VI resources
                                          `-- Online (only when configured and primary enabled)
```

This explanatory state machine preserves the offline-before-assignment rule and the idempotent Online/Offline requests. [PDF pp. 446-447](../_source/pages/page-446.md)

Only a capable primary controller issues Virtualization Management. It can persistently change its own flexible allocation for the next applicable Controller Level Reset, or take an associated secondary offline, assign VQ/VI flexible resources while offline, and bring it online. Requests for nonexistent, private, or currently unavailable resource ranges are invalid. [PDF pp. 446-447](../_source/pages/page-446.md)

The resource type distinguishes VQ from VI, while `NR` is the count to allocate or assign. Primary allocation must name the processing primary; secondary actions must name one of its associated secondaries. Assignment requires Offline state, and Online requires appropriate configuration plus an enabled primary controller. [PDF p. 447](../_source/pages/page-447.md)

For Primary Controller Flexible Allocation and Secondary Controller Assign, completion dword 0 reports the number of resources actually modified (`NRM`); it may be smaller or larger than the requested `NR`. Distinct command-specific failures separate invalid controller identity, invalid controller state, invalid requested count, and unavailable or invalid resource identifiers. [PDF pp. 447-448](../_source/pages/page-447.md)

## Relationships

- [Controller](controller.md) defines primary/secondary as one composable controller-classification axis. [PDF pp. 34-35](../_source/pages/page-034.md)
- [Identify Command Model](identify-command-model.md) supplies the `CNS` selection envelope for the primary capability and secondary list structures. [PDF pp. 317-320](../_source/pages/page-317.md)

## Evidence

- [Primary Controller Capabilities opening fields, PDF p. 366](../_source/pages/page-366.md)
- [VQ/VI resource pools and Secondary Controller List contract, PDF p. 367](../_source/pages/page-367.md)
- [Secondary Controller entry layout, PDF p. 368](../_source/pages/page-368.md)
- [Virtualization Management resource gates and state actions, PDF pp. 446-447](../_source/pages/page-446.md)
- [Virtualization Management completion result and status taxonomy, PDF pp. 447-448](../_source/pages/page-447.md)
