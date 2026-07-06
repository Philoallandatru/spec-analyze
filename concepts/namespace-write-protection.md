# Namespace Write Protection

Namespace Write Protection is a host-configurable namespace state that gates writes independently of Feature saveability. [PDF pp. 425-426](../_source/pages/page-425.md)

## Mental model

```text
[No Write Protect] --> [Write Protect]
        |                    |
        +--> [Until Power Cycle] --power cycle--> implementation state
        `--> [Permanent Write Protect] (irreversible by this Feature)
```

This explanatory state summary preserves the two terminal Feature restrictions; the complete state machine is defined later in section 8.1.16. [PDF pp. 425-426](../_source/pages/page-425.md)

## Contract and invariants

The Feature is namespace-specific and non-saveable, but has no independent default: reset and power-cycle results derive from the namespace's prior state, except the until-power-cycle state. Attempts to leave the until-power-cycle or permanent state fail with Feature Not Changeable. Entering until-power-cycle also requires Identify support and is prohibited in multi-Domain subsystems. [PDF pp. 425-426](../_source/pages/page-425.md)

Before entering any protected state, the controller commits the namespace's volatile write data and metadata to non-volatile media. [PDF p. 426](../_source/pages/page-426.md)

## Evidence

- [Feature states and change restrictions, PDF p. 425](../_source/pages/page-425.md)
- [Multi-Domain restriction and cache commit barrier, PDF p. 426](../_source/pages/page-426.md)
