# Namespace Identifiers

A Namespace Identifier (`NSID`) is a controller-facing handle for a namespace, not the namespace's durable identity. Its validity is subsystem-wide, while allocation is a subsystem relationship and activity is evaluated separately for each controller. [PDF pp. 97-99](../_source/pages/page-097.md)

## Three classification axes

```text
Numeric space
  0h ---------------- invalid
  1h..NN ------------ valid range
  NN+1..FFFFFFFEh --- invalid
  FFFFFFFFh --------- broadcast (not an ordinary valid NSID)

Within valid range, subsystem view:
  allocated   -> refers to an existing namespace
  unallocated -> refers to no namespace

For each controller, attachment view:
  active   -> namespace is attached to this controller
  inactive -> namespace is not attached here
```

An unallocated NSID is inactive for every controller. One allocated NSID may be active on some controllers and inactive on others. [PDF pp. 97-98](../_source/pages/page-097.md)

## Command error semantics

| Command specifies | Default result |
|---|---|
| inactive NSID | `Invalid Field in Command` |
| invalid NSID | `Invalid Namespace or Format` |
| `FFFFFFFFh` | Broadcast to all namespaces where the command permits it |

[PDF p. 97](../_source/pages/page-097.md)

## Discovery and pagination

```text
Active for one controller:
  Identify CNS=0h per NSID, or CNS=2h lists of up to 1024

Allocated in subsystem:
  Identify CNS=10h lists of up to 1024

repeat list request with continuation until complete
```

The maximum valid NSID (`NN`) may be much larger than the maximum number simultaneously allocated (`MNAN`). [PDF pp. 98-99](../_source/pages/page-098.md)

## Handle versus durable identity

NSIDs may change across power-off conditions. To recognize the same namespace across paths or time, hosts use UUID, NGUID, or EUI64 where available; `UIDREUSE` reports reuse characteristics for NGUID/EUI64. [PDF p. 99](../_source/pages/page-099.md)

When Namespace Management, ANA Reporting, or NVM Sets are supported, each NSID denotes the same physical namespace across controllers. Without those capabilities, shared-namespace NSIDs remain subsystem-unique, while private-namespace NSIDs need not be. [PDF p. 98](../_source/pages/page-098.md)

## Queue relationship

A namespace is not inherently bound to one Submission Queue. Host software may choose such a relationship, but the controller supports every attached namespace from every I/O Submission Queue. [PDF p. 99](../_source/pages/page-099.md)

## Changed-attached namespace notification

```text
since last successful read
        |
        v
[changed NSID 1] ... [changed NSID 1024]
        or, on overflow
[FFFFFFFFh] [0] ... [0]
```

The Changed Attached Namespace List reports namespaces whose Identify data changed, that became attached or detached, or that were deleted since the log was last read. It holds at most 1,024 NSIDs; more changes collapse to the overflow sentinel `FFFFFFFFh` in the first entry with the remainder zero-filled, so the host must re-enumerate rather than treat the list as complete. [PDF p. 235](../_source/pages/page-235.md)

## Evidence

- [Validity, allocation, attachment, and errors, PDF pp. 97-98](../_source/pages/page-097.md)
- [Uniqueness and enumeration, PDF pp. 98-99](../_source/pages/page-098.md)
- [Persistent identity and queue independence, PDF p. 99](../_source/pages/page-099.md)
- [Changed Attached Namespace List and overflow behavior, PDF p. 235](../_source/pages/page-235.md)
