# Domains and Divisions

A Domain is the smallest indivisible NVM-subsystem unit that shares state such as power state and capacity information. In a multi-Domain subsystem, Domain boundaries are communication, fault, and management boundaries; the Domains must cooperate to operate as one subsystem. [PDF pp. 104-105](../_source/pages/page-104.md)

## Mental model

```text
                     one NVM Subsystem
+----------------------+   +----------------------+
| Domain 1             |   | Domain 2             |
| controllers: 0..*    |<->| controllers: 0..*    |
| Endurance Groups:0..*|   | Endurance Groups:0..*|
+----------------------+   +----------------------+
           communication / fault boundary
                         X  division
```

This is an explanatory projection of Figures 71 and 72, checked against rendered local PDF pages 105-106. A Domain may contain controllers, storage, both, or neither; a namespace may remain shared across Domain boundaries. [PDF pp. 105-106](../_source/pages/page-105.md)

## Contract and invariants

| Condition | Required representation or behavior |
|---|---|
| Multiple Domains supported | All controllers set `CTRATT.MDS`; controller and supported Endurance Group Domain Identifiers are non-zero |
| Single-Domain subsystem | Domain Identifier fields are zero |
| Domain identity | Identifier is unique within the NVM subsystem |
| Multiple Domains | Asymmetric Namespace Access Reporting is supported |

[PDF pp. 104-105](../_source/pages/page-104.md)

A division is an event or action that disrupts inter-Domain communication. It may make controller knowledge partial, change namespace access, and affect subsystem-scoped operations such as capacity reporting, sanitize, format, and SMART information. Domain Identifiers let host software infer which controllers or Endurance Groups may share a failure boundary. [PDF pp. 104, 106-107](../_source/pages/page-104.md)

## Reservation recovery state

```text
[Divided]
    |
    | communication resumes
    v
[Synchronize persistent reservation state]
    | success                     | cannot synchronize
    v                             +--> [ANA Persistent Loss]
[Accessible]                      `--> [Controller stops, CFS=1]
    ^
    |
[ANA Inaccessible while unsynchronized]
```

This explanatory state machine was checked against rendered local PDF page 106. When reservation state is not synchronized after a division, the namespace's ANA Group remains ANA Inaccessible until synchronization; permanent failure leads to ANA Persistent Loss or the controller stopping command processing and setting `CSTS.CFS`. [PDF p. 106](../_source/pages/page-106.md)

## Evidence

- [Domain and division semantics, PDF p. 104](../_source/pages/page-104.md)
- [Composition, identifiers, and topology example, PDF p. 105](../_source/pages/page-105.md)
- [Reservation synchronization and identifier use, PDF pp. 106-107](../_source/pages/page-106.md)
