# Transport Models

NVMe defines memory-based and message-based models for communication between a host and an NVM subsystem. A transport determines how commands, responses, and data cross that boundary. [PDF pp. 39-46](../_source/pages/page-039.md)

## Mental model

```text
Memory-based (e.g. PCIe)          Message-based (e.g. Fabrics)

Host memory + MMIO                Host                 Subsystem
   SQ / CQ / data                  | command capsule --> |
        |                          | <-- response capsule|
        v                          | <------ data ------> |
   Controller
```

Message-based transports divide further into message-only and message/memory subtypes according to whether explicit memory operations participate in transfer. [PDF p. 39](../_source/pages/page-039.md)

```text
NVMe architecture, queues, command sets, properties
                         |
transport-specific specialization + binding services
                         |
                    NVMe Transport
                         |
             fabric protocol / physical fabric
```

This explanatory layering keeps the transport-independent NVMe interface above the binding and the native fabric below it; native fabric services are outside the NVMe specification family. [PDF pp. 43-44](../_source/pages/page-043.md)

The Base Specification deliberately spans both models; each Transport specification binds the NVMe protocol, including controller properties, to a concrete transport such as PCIe, RDMA, or TCP. [PDF p. 23](../_source/pages/page-023.md)

In NVMe over Fabrics, the information-exchange unit is a capsule. A capsule contains a command or response and may also contain command/response data and SGLs. [PDF p. 29](../_source/pages/page-029.md)

The NVMe Transport layer provides reliable delivery above the fabric and is independent of the fabric's physical interconnect and low-level protocol. An NVM subsystem port is one protocol interface and may aggregate one or more physical fabric interfaces; its NQN and 16-bit Port ID identify it. One port may support multiple transports, but a connected controller is bound to one subsystem port. [PDF pp. 39, 44](../_source/pages/page-039.md) [PDF p. 44](../_source/pages/page-044.md)

| Model | Commands/responses | Data path |
|---|---|---|
| Memory-based | Explicit memory reads/writes | Explicit memory reads/writes |
| Message-only | Capsules/messages | Capsules/messages only |
| Message/memory | Capsules | Capsules or explicit memory operations |

The two message-based subtypes share capsule-based command and response transfer and differ in how data may move. [PDF pp. 39-40](../_source/pages/page-039.md)

For message-based queues, the Transport delivers command capsules and reports arrival in a binding-specific way. Admin capsule sizes are fixed by the binding; Identify Controller reports the I/O command and response capsule sizes. Both Admin and I/O queues support SGL-based transfers, and the binding defines which SGL types may select in-capsule data or explicit memory transactions. Unused, non-reserved capsule bytes are undefined and must not be interpreted. [PDF p. 110](../_source/pages/page-110.md)

## Fabrics capsules and data placement

```text
Command capsule (host -> controller)
byte 0                 63 64                              N-1
+------------------------+----------------------------------+
| 64-byte SQE            | data OR additional SGLs          |
+------------------------+----------------------------------+

Response capsule (subsystem -> host)
byte 0         15 16                                     N-1
+----------------+------------------------------------------+
| 16-byte CQE    | optional data                            |
+----------------+------------------------------------------+
```

This explanatory layout is reconstructed from Figures 75-76. A command capsule never mixes data and additional SGLs after the SQE. The SQE's Command Identifier is unique among outstanding commands on that queue; the response CQE corresponds to an earlier command capsule. [PDF p. 111](../_source/pages/page-111.md)

| Data placement selected by SGL | Capsule consequence |
|---|---|
| Memory address | Data moves through transport-supported memory transactions |
| Offset | Data is carried in the command or response capsule |
| Additional descriptor segment | Descriptors are contiguous immediately after the SQE |

The SQE contains one SGL entry. Additional in-capsule SGLs begin at byte 64; in-capsule command data begins at byte offset `64 + ICDOFF * 16`, leaving any intervening bytes undefined. Descriptor count/space limits come from Identify Controller, and violations are aborted with Invalid Number of SGL Descriptors; an invalid SGL offset returns SGL Offset Invalid. These layouts and limits were checked against rendered PDF pages 111-114. [PDF pp. 111-114](../_source/pages/page-111.md)

## Evidence

- [Transport taxonomy, PDF pp. 39-46](../_source/pages/page-039.md)
- [Transport binding's place in the specification family, PDF p. 23](../_source/pages/page-023.md)
- [Capsule definition, PDF p. 29](../_source/pages/page-029.md)
- [Transport layer and port definitions, PDF pp. 33-34](../_source/pages/page-033.md)
- [Fabrics layering and port binding, PDF pp. 43-44](../_source/pages/page-043.md)
- [Message queue delivery, capsule sizing, and SGL transfer contract, PDF p. 110](../_source/pages/page-110.md)
- [Command/response capsule layouts and data-placement rules, PDF pp. 111-114](../_source/pages/page-111.md)
