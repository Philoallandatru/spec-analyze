# NVM Sets and Endurance Groups

An NVM Set is a logically, and potentially physically, separate collection of NVM from which namespaces are created. An Endurance Group is the boundary across which endurance is managed; it may contain one NVM Set or coordinate endurance across several NVM Sets. [PDF pp. 99-102](../_source/pages/page-099.md)

## Containment and inheritance

```text
Domain
`-- Endurance Group Y          endurance managed together
    |-- NVM Set A
    |   |-- Namespace A1       inherits Set A attributes
    |   |-- Namespace A2
    |   `-- unallocated NVM
    `-- NVM Set B
        |-- Namespace B1
        `-- unallocated NVM

Domain
`-- Endurance Group Z
    `-- NVM Set C              endurance local to this set
```

A formatted namespace is wholly contained in exactly one NVM Set and inherits attributes such as optimal write size. Every NVM Set belongs to exactly one Endurance Group, and every Endurance Group belongs to exactly one Domain. [PDF pp. 99-102](../_source/pages/page-099.md)

## Identifiers

| Identifier | Width | Scope | Zero value |
|---|---:|---|---|
| NVM Set Identifier | 16 bits | unique within NVM subsystem | reserved / invalid |
| Endurance Group Identifier | 16 bits | unique within NVM subsystem | reserved / invalid |

Unless a command says otherwise, supplying zero where one of these identifiers is required causes `Invalid Field in Command`. [PDF p. 101](../_source/pages/page-101.md)

## Capacity view

```text
NVM Set total capacity
+-----------------------------------------------+
| namespaces containing formatted storage      |
+-------------------------------+---------------+
| allocated                     | unallocated   |
+-------------------------------+---------------+
```

The NVM Set List reports identifier, optimal write size, total capacity, unallocated capacity, and associated Endurance Group. [PDF pp. 100-101](../_source/pages/page-100.md)

## NVM-Set-aware control paths

Identify, Capacity Management, Namespace Management, selected Features, Connect, Create I/O Submission Queue, and selected log pages carry or expose an NVM Set Identifier. If NVM Sets are unsupported, identifier fields are cleared to zero and Endurance Groups may still define endurance boundaries directly. [PDF pp. 100-102](../_source/pages/page-100.md)

## Endurance events

Endurance Group Event Configuration (`18h`) selects which non-reserved critical-warning bits cause the specified Endurance Group to enter the aggregate event log. `ENDGID=0` disables the warning mask for the operation; nonexistent groups or reserved warning bits are invalid. Enabling a notice while its condition is already true sends the event. [PDF p. 410](../_source/pages/page-410.md)

```text
Set Features per Endurance Group
        | configure selected critical events
        v
Endurance Group event occurs
        |
        +--> entry in Event Aggregate log
        `--> optional Asynchronous Event notice to host
```

The Endurance Group Information log describes group state, while the aggregate log identifies groups with outstanding events. [PDF pp. 102-103](../_source/pages/page-102.md)

The Endurance Group Information log is a 512-byte lifetime view selected by the 16-bit Endurance Group Identifier in the Get Log Page Log Specific Identifier. It covers capacity that is already assigned to NVM Sets as well as unallocated capacity still owned by the group. [PDF p. 246](../_source/pages/page-246.md)

```text
Endurance Group health
  |-- current: spare / threshold / life used / warning bits
  |-- lifetime: host reads+writes / media writes / integrity errors
  `-- capacity: total / unallocated / endurance estimate
```

Warnings cover group-wide read-only state, degraded reliability, and spare below threshold; if the same warning applies to every Endurance Group, SMART/Health reflects the corresponding subsystem warning. The log also identifies rotational-media groups and their containing Domain. [PDF p. 247](../_source/pages/page-247.md)

Read/write and endurance values use rounded-up billions of bytes. Host data-unit counters exclude controller-internal traffic, while Media Units Written includes controller writes such as garbage collection; comparing host writes with media writes therefore exposes write amplification at group scope. Zero means a particular optional estimate/counter is not reported. [PDF pp. 247-248](../_source/pages/page-247.md)

The Endurance Group Event Aggregate is an ascending list of group identifiers with pending enabled events. Adding a group generates an asynchronous event; a successful `RAE=0` read of that group's Endurance Group Information log acknowledges the detail and removes the aggregate entry. Reads beyond the identifier-bounded list return zeroes. [PDF p. 275](../_source/pages/page-275.md)

## Evidence

- [NVM Set containment and command-set binding, PDF pp. 99-100](../_source/pages/page-099.md)
- [NVM Set attributes/requirements and Endurance Group model, PDF p. 101](../_source/pages/page-101.md)
- [Endurance Group requirements and event configuration, PDF p. 102](../_source/pages/page-102.md)
- [Endurance Group Information log selection and lifetime scope, PDF p. 246](../_source/pages/page-246.md)
- [Rendered Endurance Group warning, endurance, activity, error, and capacity fields, PDF pp. 247-248](../_source/pages/page-247.md)
- [Endurance Group event aggregation and acknowledgment, PDF p. 275](../_source/pages/page-275.md)
