# Namespace

A namespace is a quantity of non-volatile memory formatted into logical blocks and presented as a storage abstraction that a host accesses through one or more controllers. Namespace allocation, attachment, and sharing are distinct relationships. [PDF pp. 33, 46-57](../_source/pages/page-033.md)

## Mental model

```text
Allocated?     NVM resources ---- allocation ----> Namespace
Attached?      Controller    ---- attachment ----> Namespace
Accessible?    Host path     ---- visibility ----> Namespace
```

These questions must not be collapsed: a namespace may exist without being attached to a given controller, and shared access may involve multiple controllers or paths. [PDF pp. 52-57](../_source/pages/page-052.md)

## Evidence

Namespace Write Protection Config (`84h`) selects no protection, write protection, protection until power cycle, or permanent protection. The latter two states cannot be changed by Set; until-power-cycle also requires capability support and is prohibited in multi-Domain subsystems. Transitioning into protection flushes that namespace's volatile data and metadata to non-volatile media. [PDF pp. 425-426](../_source/pages/page-425.md)

- [Normative definition area, PDF p. 33](../_source/pages/page-033.md)
- [Storage model and sharing, PDF pp. 46-57](../_source/pages/page-046.md)
