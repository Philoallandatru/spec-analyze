# Namespace Topology and Change Logs

Namespace topology logs describe which subsystems, Reachability Groups, and Reachability Associations participate in access, while the Changed Allocated Namespace List reports controller-visible namespace inventory changes. [PDF pp. 288-293](../_source/pages/page-288.md)

## Mental model

```text
dispersed namespace (specified NSID)
`-- participating NVM subsystem NQNs

controller-visible topology
`-- Reachability Association (shared operation characteristic)
    `-- Reachability Group(s) (shared reachability attributes)
        `-- attached namespace NSIDs

allocated namespace changes --> up to 1024 NSIDs, or overflow sentinel FFFFFFFFh
```

This explanatory hierarchy summarizes Figures 270 and 274-278, checked against rendered local PDF pages 288-293. [PDF pp. 288-293](../_source/pages/page-288.md)

## Contract and invariants

| View | Membership/ordering | Consistency rule |
|---|---|---|
| Dispersed participants (`17h`) | One 256-byte NQN per participant; entry 0 is the processing controller's subsystem | Generation counter changes when participants are added or removed |
| Reachability Groups (`1Ah`) | Only groups containing namespaces attached to the processing controller; NSIDs ascending | Re-read header after segmented retrieval and retry if global change count differs |
| Reachability Associations (`1Bh`) | Only associations containing qualifying Reachability Groups; RGIDs ascending | Same header validation protocol |
| Changed Allocated Namespace List (`1Ch`) | Up to 1,024 changed NSIDs | More than 1,024 changes yields first entry `FFFFFFFFh` and zero-filled remainder |

[PDF pp. 288-293](../_source/pages/page-288.md)

The dispersed-participant log rejects NSID `0h` and `FFFFFFFFh`, and rejects a namespace not marked dispersed. Every returned NQN matches that participant's `SUBNQN`; discovering those remote participants is outside this specification. [PDF pp. 288-289](../_source/pages/page-288.md)

`RGO=1` omits NSIDs from Reachability Group descriptors; `RAO=1` similarly omits RGIDs from Reachability Association descriptors. Where index offsets are supported, index zero starts at the log header and later indexes select descriptors. [PDF pp. 290-292](../_source/pages/page-290.md)

Global change counts start at zero after Controller Level Reset and wrap to zero. Optional descriptor-local counts start at one and wrap to one; zero means the controller does not report that local count, requiring content comparison. [PDF pp. 290-293](../_source/pages/page-290.md)

Association characteristic `01h` states reachability without a performance claim; `02h` and `03h` distinguish whether fast copy is supported between namespaces represented by member groups. [PDF p. 293](../_source/pages/page-293.md)

The changed-allocated list covers Identify Namespace data changes, attachment and detachment, creation, and deletion since the last read. [PDF p. 293](../_source/pages/page-293.md)

## Evidence

- [Dispersed participant validation and header, PDF p. 288](../_source/pages/page-288.md)
- [Participant list, PDF p. 289](../_source/pages/page-289.md)
- [Reachability Group snapshot protocol, PDF pp. 290-291](../_source/pages/page-290.md)
- [Reachability Association snapshot protocol, PDF pp. 291-293](../_source/pages/page-291.md)
- [Changed Allocated Namespace List, PDF p. 293](../_source/pages/page-293.md)
