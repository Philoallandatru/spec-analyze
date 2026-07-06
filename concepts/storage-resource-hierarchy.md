# Storage Resource Hierarchy

The NVM storage model organizes resources using NVM subsystems, Domains, Endurance Groups, NVM Sets, Reclaim Groups, Reclaim Units, Media Units, and namespaces. These describe containment, endurance management, allocation, and placement at different layers. [PDF pp. 46-51](../_source/pages/page-046.md)

## Mental model

```text
NVM Subsystem
`-- Domain
    `-- Endurance Group
        |-- Media Unit(s)
        `-- organization (either branch)
            |-- NVM Set(s)
            |   `-- Namespace (exactly one NVM Set)
            `-- Reclaim Group(s)
                `-- Reclaim Unit(s)
                    `-- Namespace spans one or more RUs in the EG
```

The diagram is an explanatory reconstruction of the containment rules and Figures 11-15, checked against rendered PDF pages 47-51. An Endurance Group contains either one or more NVM Sets or one or more Reclaim Groups; the branches are alternative organizations, not simultaneous containment for one Endurance Group. [PDF pp. 46-51](../_source/pages/page-046.md)

## Contract and invariants

| Relationship | Cardinality or rule |
|---|---|
| Domain -> NVM Subsystem | Each Domain belongs to one subsystem |
| Endurance Group -> Domain | Each Endurance Group belongs to one Domain |
| NVM Set -> Endurance Group | Each NVM Set belongs to one Endurance Group |
| Namespace in NVM Set organization | Each namespace belongs to one NVM Set |
| Reclaim Group -> Endurance Group | Each Reclaim Group belongs to one Endurance Group |
| Namespace in Reclaim Group organization | Belongs to one Endurance Group and occupies one or more Reclaim Units among that group's Reclaim Groups |

[PDF p. 46](../_source/pages/page-046.md)

Support for Endurance Groups, multiple NVM Sets, and Reclaim Groups is optional. Vendors may ship a configuration, and customers may configure or reconfigure it; namespace creation and deletion are typical changes. A subsystem without multiple NVM Sets or multiple Endurance Groups need not report those entities. [PDF p. 51](../_source/pages/page-051.md)

In the Reclaim Group organization, each host write directs which Reclaim Group receives the namespace's user data. One namespace may therefore span multiple Reclaim Groups while remaining inside one Endurance Group. [PDF pp. 48-51](../_source/pages/page-048.md)

| Entity | Boundary expressed by the definition |
|---|---|
| Domain | Smallest indivisible unit sharing state such as power state and capacity information |
| Endurance Group | Portion of subsystem non-volatile storage whose endurance is managed as a group |
| Media Unit | Component of underlying media; Endurance Groups are composed of Media Units |
| Exported NVM resources | Exported NVM Subsystems, Exported Namespaces, and Exported Ports used for remote access |

These definitions separate shared-state, endurance-management, and remote-export concerns rather than treating them as interchangeable containment labels. [PDF p. 30](../_source/pages/page-030.md)

Placement indirection is namespace-scoped at the host-facing edge: a Placement Handle maps to an Endurance Group-scoped Reclaim Unit Handle, and that handle references one Reclaim Unit in each Reclaim Group. A Placement Identifier combines a Reclaim Group Identifier with a Placement Handle to select the referenced Reclaim Unit. [PDF pp. 32, 34](../_source/pages/page-032.md) [PDF p. 34](../_source/pages/page-034.md)

When Flexible Data Placement is enabled, an Endurance Group has `P` Reclaim Groups and `N` Reclaim Unit Handles. Each handle references one Reclaim Unit in every Reclaim Group, and a Reclaim Unit is referenced by at most one handle at a time. After a referenced unit is filled, the controller retargets that handle to an empty Reclaim Unit; reuse of a physical unit may therefore involve the same or a different handle in a later erase/use cycle. [PDF pp. 103-104](../_source/pages/page-103.md)

```text
Reclaim Unit Handle h
   |-- Reclaim Group 0 -> one writable Reclaim Unit
   |-- Reclaim Group 1 -> one writable Reclaim Unit
   `-- Reclaim Group P-1 -> one writable Reclaim Unit
                              |
                         full / retired
                              v
                    retarget to empty unit
```

This explanatory reconstruction preserves the cross-group indirection shown in Figure 70; it was checked against the rendered local PDF page 104. [PDF pp. 103-104](../_source/pages/page-103.md)

## Evidence

- [Entity list and containment rules, PDF p. 46](../_source/pages/page-046.md)
- [Simple and complex hierarchy examples, PDF pp. 47-51](../_source/pages/page-047.md)
- [Domain, Endurance Group, and exported-resource definitions, PDF p. 30](../_source/pages/page-030.md)
- [Media Unit and placement definitions, PDF pp. 32, 34](../_source/pages/page-032.md)
- [FDP Reclaim Group, Reclaim Unit Handle, and retargeting rules, PDF pp. 103-104](../_source/pages/page-103.md)
