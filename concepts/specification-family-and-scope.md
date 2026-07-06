# Specification Family and Scope

The Base Specification defines the transport-independent protocol by which host software communicates with an NVM subsystem. I/O Command Set specifications extend that base with commands and related structures, while Transport specifications bind the protocol and controller properties to a transport. NVMe-MI and the Boot Specification address management and boot use respectively. [PDF p. 23](../_source/pages/page-023.md)

## Mental model

Explanatory applicability map, reconstructed from Figure 1:

```text
                 I/O Command Set specifications
                              |
NVMe-MI -------- Base Specification -------- Boot Specification
                              |
                   Transport specifications
```

The lines mean applicability, not hierarchy, protocol layering, or system architecture. [PDF p. 23, Figure 1](../_source/pages/page-023.md)

## Contract and invariants

| Area | Base Specification boundary |
|---|---|
| In scope | Controller properties, commands, and common aspects of supported I/O Command Sets |
| Transport binding | Implemented by a controller through a specific NVMe Transport |
| Usage model | Not prescribed: SSD, main memory, cache, backup, and redundancy are outside scope |
| Implementation | Media management, wear leveling, erase policy, and cache algorithms are abstracted |

These boundaries keep the interface implementation-agnostic: physical devices, gateways, and software implementations may all conform to the same protocol contract. [PDF p. 24](../_source/pages/page-024.md)

## Requirement language

| Keyword | Meaning in this revision |
|---|---|
| mandatory / shall | Required for conformance and interoperability |
| may | Permitted choice without preference |
| optional | Not required, but conforming behavior is required if implemented |
| should | Strongly preferred choice |
| obsolete | Removed functionality from an earlier revision |

Reserved storage and coded values are not ordinary extension points: reserved fields are cleared unless a future extension says otherwise; a reserved coded value received in a defined command field is an error; writing one to a controller property field has undefined results. [PDF pp. 24-25](../_source/pages/page-024.md) [PDF p. 25](../_source/pages/page-025.md)

## Evidence

- [Specification family and Figure 1, PDF p. 23](../_source/pages/page-023.md)
- [Scope, exclusions, and initial keywords, PDF p. 24](../_source/pages/page-024.md)
- [Requirement and reserved-value semantics, PDF p. 25](../_source/pages/page-025.md)
