# Directive Exchange

Directive Receive and Directive Send are paired Admin commands that move Directive-Type-dependent data between a host and controller. The generic commands provide the envelope; each Directive Type defines its operations, data structures, extra dwords, and command-specific status. [PDF pp. 212-214](../_source/pages/page-212.md)

## Mental model

```text
Host                         Controller
 |-- Directive Send --------->|  host-to-controller type data
 |   DTYPE + DOPER + DSPEC    |
 |                            |
 |<-- Directive Receive ------|  controller-to-host type data
     DTYPE + DOPER + DSPEC
```

This explanatory direction diagram is based on the Receive and Send command definitions. [PDF pp. 212-214](../_source/pages/page-212.md)

## Shared command envelope

| Field | Meaning |
|---|---|
| Data Pointer | start of the transfer buffer |
| `NUMD` | 0's-based number of dwords to transfer |
| `DTYPE` | selects the Directive Type |
| `DOPER` | selects an operation within that type |
| `DSPEC` | Directive-Type-dependent value |
| CDW12-13 | optional, as defined by the selected type and operation |

Receive and Send use the same field partition in CDW10-11; only transfer direction differs. This table condenses rendered Figures 175-180. [PDF pp. 212-213](../_source/pages/page-212.md)

## Receive length behavior

If `NUMD` requests less than the returned structure, Receive transfers only that prefix. If it requests more, the controller transfers the entire structure and no padding or additional data. [PDF p. 212](../_source/pages/page-212.md)

## Relationships and edge cases

- Directive-specific semantics are intentionally deferred to the Extended Capabilities definition for the chosen type; the generic Admin command does not assign meaning to `DOPER`, `DSPEC`, or CDW12-13 by itself. [PDF pp. 212-214](../_source/pages/page-212.md)
- Completion always returns through the Admin Completion Queue, and possible command-specific status values depend on the Directive Type. [PDF pp. 213-214](../_source/pages/page-213.md)

## Evidence

- [Rendered Receive buffer and length behavior, PDF p. 212](../_source/pages/page-212.md)
- [Rendered Receive/Send common field layouts, PDF p. 213](../_source/pages/page-213.md)
- [Send completion boundary, PDF p. 214](../_source/pages/page-214.md)
