# Command Sets

NVMe groups commands into the Admin Command Set, one or more I/O Command Sets, and the Fabrics Command Set. The queue on which a command may be submitted and the controller state in which it is processed depend on its set. [PDF p. 41](../_source/pages/page-041.md)

## Mental model

```text
NVMe Command Sets
|-- Admin Command Set --------> Admin Submission Queue
|-- I/O Command Sets ---------> I/O Submission Queues
|   |-- NVM
|   |-- Zoned Namespace
|   `-- Key Value (examples)
`-- Fabrics Command Set ------> Admin SQ; some commands may use I/O SQ
```

This is a text reconstruction of Figure 5, checked against rendered PDF page 41. [PDF p. 41](../_source/pages/page-041.md)

Fabrics commands differ from Admin and I/O commands in that a controller processes them regardless of whether the controller is enabled. [PDF p. 41](../_source/pages/page-041.md)

## Namespace binding

```text
create namespace
      |
      +-- choose exactly one I/O Command Set
                    |
                    `-- fixed for namespace lifetime

controller supports + enables that set
      |
      `-- namespace may be attached; commands use that set's interpretation
```

This explanatory lifecycle diagram was checked against rendered PDF page 52. Every namespace is associated with exactly one I/O Command Set at creation, and the association does not change during that namespace's lifetime. A namespace may attach only to a controller on which its I/O Command Set is both supported and enabled. [PDF p. 52](../_source/pages/page-052.md)

The I/O Command Set determines the namespace's data model and command interpretation: the NVM Command Set represents data as logical blocks, while the Key Value Command Set represents it as key-value pairs. A controller may support multiple I/O Command Sets. [PDF p. 52](../_source/pages/page-052.md)

## Evidence

When `CC.CSS=110b` selects all supported I/O Command Sets, I/O Command Set Profile (`19h`) selects one advertised combination index. A zero-valued combination or a combination excluding a command set used by an attached namespace is rejected; successful completion is the transition point. With any other `CC.CSS`, the Feature has no effect and Set succeeds without changing selection. [PDF pp. 410-411](../_source/pages/page-410.md)

- [Types and routing of command sets, PDF p. 41](../_source/pages/page-041.md)
- [Namespace-to-I/O-Command-Set binding, PDF p. 52](../_source/pages/page-052.md)
