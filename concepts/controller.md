# Controller

A controller is the interface between a host and an NVM subsystem. Every controller executes commands from Submission Queues, posts completions to Completion Queues, and implements one Admin Submission Queue and one Admin Completion Queue. [PDF p. 29](../_source/pages/page-029.md)

## Mental model

```text
Host software
    |
    | commands / completions / data
    v
+---------------- Controller ----------------+
| properties | queues | command processing   |
+---------------------------------------------+
    |
    v
NVM subsystem resources and namespaces
```

## Contract and invariants

All controllers in one NVM subsystem use the same allocation model. A static controller preserves Controller ID and saved Feature state across Controller Level Reset and, for message-based controllers, across associations. A dynamic controller is allocated on demand and preserves no state from an earlier association. [PDF p. 58](../_source/pages/page-058.md)

```text
                         Controller allocation model
                    +----------------+----------------+
                    |                                 |
              [Static controller]              [Dynamic controller]
              state survives reset             allocated on demand
              Fabrics state may survive         no prior-association state
              across associations

Memory-based ------> static only
Message-based -----> static or dynamic (Discovery: dynamic only)
```

This explanatory classification preserves the transport constraints and the subsystem-wide model invariant. [PDF p. 58](../_source/pages/page-058.md)

| Controller type | Purpose | I/O queues / namespace access |
|---|---|---|
| I/O | Accesses user data through one or more supported I/O Command Sets; may manage | Supports I/O queues; namespaces may be attached |
| Discovery | Discovers NVM or Discovery subsystems over Fabrics | No I/O queues, I/O commands, or exposed namespaces |
| Administrative | Manages an NVM subsystem | No I/O queues, namespace attachments, or user-data I/O |

Every type still implements exactly one Admin Submission Queue and one Admin Completion Queue; `CNTRLTYPE` in Identify Controller reports the functional type. [PDF p. 59](../_source/pages/page-059.md)

An I/O controller may simultaneously support multiple I/O Command Sets, and support may differ between controllers in the same subsystem. An Administrative controller may support I/O-Command-Set-specific Admin commands, but cannot execute I/O commands because it has no I/O Queues. [PDF pp. 60-61](../_source/pages/page-060.md)

Administrative controllers are optional in a subsystem and may manage namespaces, virtualization, health, enclosure functions, subsystem reset, or subsystem shutdown. They can also exist in a subsystem with no non-volatile medium or namespaces. [PDF pp. 61-62](../_source/pages/page-061.md)

The generic term *controller* may stand for any applicable type when context determines the type. [PDF pp. 24, 28-30](../_source/pages/page-024.md) [PDF p. 28](../_source/pages/page-028.md) [PDF pp. 29-30](../_source/pages/page-029.md)

A dynamic controller is allocated on demand and does not preserve state such as Feature settings from prior associations. An emulated controller is software-defined and may or may not have a physical NVMe controller underneath it. [PDF pp. 30, 58](../_source/pages/page-030.md)

A static controller pre-exists with a specific Controller ID and preserves state such as Feature settings across associations. Primary and secondary describe a different axis: a secondary controller depends on a primary controller for management of some controller resources. [PDF pp. 34-35](../_source/pages/page-034.md)

| Axis | Alternatives |
|---|---|
| Transport behavior | memory-based / message-based |
| Allocation lifetime | static / dynamic |
| Functional role | I/O / Discovery / Administrative |
| Resource management | primary / secondary |

These labels are composable classifications, not one flat mutually exclusive type list. [PDF pp. 28, 30, 32, 34-35](../_source/pages/page-028.md) [PDF p. 30](../_source/pages/page-030.md) [PDF p. 32](../_source/pages/page-032.md) [PDF pp. 34-35](../_source/pages/page-034.md)

## Evidence

- [Controller models and association rules, PDF p. 58](../_source/pages/page-058.md)
- [Controller type taxonomy and common Admin queues, PDF p. 59](../_source/pages/page-059.md)
- [I/O controller capabilities, PDF p. 60](../_source/pages/page-060.md)
- [Administrative controller constraints and uses, PDF pp. 61-62](../_source/pages/page-061.md)
- [Controller definition and common queue behavior, PDF p. 29](../_source/pages/page-029.md)
- [Controller type definitions, PDF pp. 28-30](../_source/pages/page-028.md)
- [Transport, allocation, and resource-management classifications, PDF pp. 32, 34-35](../_source/pages/page-032.md)
