# Namespace Access Models

Namespace terminology distinguishes allocation, controller attachment, subsystem scope, and the command-selected namespace. These are separate dimensions of access. [PDF pp. 30, 33-35](../_source/pages/page-030.md)

## Mental model

```text
private namespace   --> attached to at most one controller at a time
shared namespace    --> concurrently attached within one NVM subsystem
dispersed namespace --> concurrently accessed across 2+ participating subsystems

one host -- 2+ independent paths --> one namespace   = multi-path I/O
2+ hosts -- different controllers --> shared namespace = namespace sharing

SQE.NSID --identifies--> specified namespace
Controller --uses NSID--> namespace access
```

[PDF pp. 30, 33-35](../_source/pages/page-030.md) [PDF p. 33](../_source/pages/page-033.md) [PDF pp. 34-35](../_source/pages/page-034.md)

The access-pattern portion of this explanatory diagram is checked against rendered PDF page 54. Both multi-path I/O and namespace sharing require at least two controllers in the subsystem. Concurrent access by multiple hosts requires host coordination, but the coordination procedure is outside the Base Specification's scope. [PDF p. 54](../_source/pages/page-054.md)

## Contract and invariants

| Term | Boundary |
|---|---|
| Private namespace | No more than one attached controller at a time |
| Shared namespace | Two or more concurrent controller attachments in one subsystem |
| Dispersed namespace | Concurrent access through controllers in two or more participating subsystems |
| Specified namespace | Namespace selected by the NSID field of a command |

An NSID is controller-relative: it is the identifier a controller uses to provide access to a namespace and is also the SQE field carrying that identifier. [PDF p. 33](../_source/pages/page-033.md)

Multi-path I/O and namespace sharing are different axes: the former is multiple independent paths from one host to one namespace; the latter is multiple hosts reaching one common namespace through different controllers. [PDF p. 54](../_source/pages/page-054.md)

For every controller that can access one shared namespace, the NSID is identical and Identify Namespace returns the same namespace data. A globally unique namespace identifier, rather than an NSID alone, lets software recognize multiple paths to that namespace. [PDF p. 55](../_source/pages/page-055.md)

```text
Controller A -- NSID 2 --+
                          +--> one shared namespace (global identifier G)
Controller B -- NSID 2 --+

same namespace  => same NSID through every attached controller
same identity   => same Identify Namespace content + global identifier
```

This explanatory mapping is reconstructed from Figures 20-21 and their accompanying requirements. [PDF pp. 55-56](../_source/pages/page-055.md)

Controllers may operate on a shared namespace concurrently. Atomicity is that of the controller receiving each command and need not match across controllers; host software or the application must enforce ordering between commands sent through different controllers. [PDF pp. 55-56](../_source/pages/page-055.md)

Independent ports isolate port reset effects: a reset affects the controller on that port, not another controller, the shared namespace, or the other controller's operations. SR-IOV is another realization of the same access model, with a controller per Physical or Virtual Function and a common namespace shared across them. [PDF pp. 56-57](../_source/pages/page-056.md)

Asymmetric controller behavior means namespace access characteristics may depend on subsystem internals or on which controller is used. Such a subsystem may expose Asymmetric Namespace Access Reporting. [PDF p. 57](../_source/pages/page-057.md)

## ANA discovery log

```text
ANA log header: global change count
  `-- ANA Group descriptors (ascending AGID)
        |-- group-local change count
        |-- one ANA state for this controller
        `-- attached NSIDs (ascending, optional with RGO)
```

The ANA log contains only groups having namespaces attached to the controller that processes Get Log Page. `RGO=1` returns group descriptors without NSID lists. Each descriptor reports Optimized, Non-Optimized, Inaccessible, Persistent Loss, or Change state for all listed namespaces as accessed through that controller. [PDF pp. 250-252](../_source/pages/page-250.md)

Because descriptors are variable length, hosts should use index offsets when the log advertises them: index zero addresses the header, one the first descriptor, and so on. For a segmented read, the host re-reads the header and accepts the snapshot only when its global change count still matches. [PDF pp. 250-251](../_source/pages/page-250.md)

The global change count starts at zero after Controller Level Reset and wraps to zero; an optional per-group count starts at one and wraps to one. A per-group value of zero means the count is not reported, so the host must compare descriptor content directly. [PDF pp. 251-252](../_source/pages/page-251.md)

## Relationships

A participating NVM subsystem is one containing controllers that provide access to a dispersed namespace. This does not turn the dispersed namespace into separate per-subsystem namespaces. [PDF pp. 30, 33](../_source/pages/page-030.md) [PDF p. 33](../_source/pages/page-033.md)

## Evidence

- [Dispersed namespace, PDF p. 30](../_source/pages/page-030.md)
- [Namespace and NSID, PDF p. 33](../_source/pages/page-033.md)
- [Private, shared, and specified namespace, PDF pp. 34-35](../_source/pages/page-034.md)
- [Multi-path I/O and namespace sharing overview, PDF p. 54](../_source/pages/page-054.md)
- [Shared-namespace identity, NSID, and concurrency, PDF pp. 55-56](../_source/pages/page-055.md)
- [Independent ports, SR-IOV, and asymmetric behavior, PDF pp. 56-57](../_source/pages/page-056.md)
## ANA groups and controller-relative states

ANA reports the relationship between one controller and an ANA Group, not a global property of a namespace. Every attached namespace on a supporting controller belongs to one valid ANA Group; all namespaces in that group share state transitions and must lie in the same Domain. A dispersed namespace belongs to an ANA Group in every participating subsystem, but its group identifier need not match across subsystems. [PDF pp. 497-500](../_source/pages/page-497.md)

```text
                 internal/path reconfiguration
 [Optimized] <------------------------------> [Non-Optimized]
      |                                             |
      +----------------> [Change] <-----------------+
                            |
                            v
                     [Inaccessible] ---- recovery ----> accessible state
                            |
                            `----> [Persistent Loss]  (terminal)
```

This explanatory state graph preserves that Change may be invisible, Inaccessible may recover, and Persistent Loss is terminal; the specification does not require every arrow shown as a direct transition. [PDF pp. 500-502](../_source/pages/page-500.md)

| ANA state | Access contract |
|---|---|
| Optimized | all supported operations work with optimized access; mandatory reportable state |
| Non-Optimized | all supported operations work, potentially slower or less efficiently |
| Inaccessible | user data unavailable; most commands abort, but future recovery is possible |
| Persistent Loss | user data persistently unavailable; this controller/group relationship never transitions again |
| Change | transition in progress; most commands abort with Asymmetric Access Transition |

Inaccessible and Persistent Loss may clear namespace capacity fields in Identify because accurate values are unavailable; the host should use a path reporting Optimized or Non-Optimized. For Inaccessible or Change, the host may retry another accessible path or retry the same path for at least `ANATT` seconds as appropriate. [PDF pp. 500-502](../_source/pages/page-500.md)

## ANA notifications and command effects

When enabled, a controller reports an ANA Change Notice after an ANA Group ID change, a failed transition, or entry into Optimized, Non-Optimized, Inaccessible, or Persistent Loss, except when state entry results from namespace attachment. [PDF p. 502](../_source/pages/page-502.md)

ANA does not blanket-disable every Admin command. Namespace-independent, non-I/O-command-set-specific Admin operations generally remain available, but Get/Set Features, Get Log Page, and Identify have explicit exceptions: reservation/I/O-specific Features may be unavailable, selected logs may be unavailable, Identify capacity may be zeroed, saving Feature values may be prohibited, and broadcast Set Features may fail if any attached namespace is in an adverse state. [PDF pp. 502-503](../_source/pages/page-502.md)
