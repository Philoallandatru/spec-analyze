# Controller Properties

A controller property is a dword or qword controller attribute with read, write, or read/write semantics. Properties form the transport-independent control surface; the transport binding defines how the host performs the access. [PDF p. 71](../_source/pages/page-071.md)

## Mental model

```text
Host software
    |
    | one exact-width access at the property's starting offset
    v
+---------------- controller property space ----------------+
| CAP | VS | INTMS | INTMC | CC | CSTS | ... | transport ... |
+------------------------------------------------------------+
    ^                                      ^
    | Property Get/Set                     | binding-defined access
    | message-based                        | memory-based
```

This explanatory diagram shows the common property map and the transport-dependent access paths; a single access may not span two properties. [PDF pp. 71-73](../_source/pages/page-071.md)

## Access contract

The host shall use the property's declared width and an offset at its beginning unless a transport specification says otherwise. An access touching any portion of two or more properties is unsupported. Reserved properties and reserved bits are read-only and return `0h`; hosts write reserved bits and properties as zero, but software should not rely on an unsupported access returning zero. [PDF pp. 71, 73](../_source/pages/page-071.md) [PDF p. 73](../_source/pages/page-073.md)

Message-based controllers expose reads through Property Get and writes through Property Set. Memory-based access is defined by the applicable transport binding. The common map below `1000h` is followed by transport-specific and optional vendor-specific regions: memory-based layouts begin transport-specific content at `1000h`, while message-based layouts reserve `1000h`-`12FFh` for Fabrics and begin vendor-specific space at `1300h`. [PDF pp. 71-73](../_source/pages/page-071.md)

| Access notation | Meaning |
|---|---|
| `RO` | read only |
| `RW` | read/write |
| `RWC` | write `1` to clear |
| `RWS` | write `1` to set |
| `Impl Spec` / `HwInit` | implementation-chosen behavior / configuration-dependent default |

Field references use `Property.Field`, with optional sub-fields such as `CAP.CRMS.CRWMS`; array fields include an element offset. [PDF p. 73](../_source/pages/page-073.md)

## Capability and configuration handshake

`CAP` reports constraints that configuration must obey: ready modes, memory-page-size range, command-set support, queue limits, arbitration mechanisms, and transport-relevant queue behavior. `CC` selects operating values within those limits. Before setting `CC.EN` to `1`, the host shall choose valid `CC.AMS`, `CC.MPS`, and `CC.CSS` values; queue entry sizes must be initialized before creating I/O queues. [PDF pp. 74-79](../_source/pages/page-074.md)

```text
read CAP constraints
        |
        v
choose CC.AMS / CC.MPS / CC.CSS / queue entry sizes
        |
        v
set CC.EN=1  ---> controller ready transition uses CAP/CRTO timeout contract
```

This explanatory sequence connects capability discovery to legal controller enablement; `CAP.TO` is in 500 ms units and is capped at `FFh`, while controllers supporting `CRTO` may report longer enable timeouts there. [PDF pp. 76, 78-79](../_source/pages/page-076.md) [PDF pp. 78-79](../_source/pages/page-078.md)

### CAP bit-field map

```text
63 62 61 60:59 58 57 56 55:52 51:48 47:46 45 44:37 36 35:32 31:24 23:19 18:17 16 15:0
+----+--+-----+--+--+--+-----+-----+-----+--+-----+--+-----+-----+-----+-----+--+----+
| R  |NS|CRMS |NS|CM|PM|MPSMX|MPSMN| CPS |BP| CSS |NR|DSTRD| TO  |  R  | AMS |CQ|MQES|
|    |SE|     |SS|BS|RS|     |     |     |S |     |S |     |     |     |     |R |    |
+----+--+-----+--+--+--+-----+-----+-----+--+-----+--+-----+-----+-----+-----+--+----+
```

This compact map is explanatory; Figure 36 supplies the exact access, reset, dependency, and controller-type rules. [PDF pp. 74-77](../_source/pages/page-074.md)

| Encoded field | Interpretation |
|---|---|
| `MPSMIN`, `MPSMAX` | host page size = `2^(12 + value)` bytes |
| `DSTRD` | doorbell stride = `2^(2 + value)` bytes; `0h` means 4 bytes |
| `TO` | worst-case ready transition in 500 ms units; `FFh` means 127.5 seconds |
| `MQES` | 0's-based maximum individual queue size; `1h` represents two entries |

[PDF pp. 75-77](../_source/pages/page-075.md)

`CAP.MQES` is 0's-based with a minimum encoded value of `1h` (two entries). For PCIe it limits host-created I/O Submission and Completion Queues; for Fabrics it limits only host-created I/O Submission Queues. `CAP.CQR` requires physically contiguous queues when set and is always set for message-based transports. [PDF p. 77](../_source/pages/page-077.md)

## Version and interrupt properties

`VS` encodes major, minor, and tertiary specification version components; Revision 2.1 is encoded as `MJR=2h`, `MNR=1h`, `TER=0h`. Lettered revisions share the listed numeric version. [PDF pp. 77-78](../_source/pages/page-077.md)

`INTMS` and `INTMC` are paired bit-significant views of the controller interrupt mask state: writing `1` to `INTMS` masks a vector, while writing `1` to `INTMC` unmasks it; writing `0` has no effect. Both read back current mask state. They apply to pin-based interrupts and MSI, and must not be accessed while configured for MSI-X, which uses its own mask table. [PDF p. 78](../_source/pages/page-078.md)

## Fabrics Property access

For message-based transports, Fabrics Property Get/Set carries the same property surface through command capsules. Get and Set select a 4-byte or 8-byte access at one property offset; Get returns the value in response bytes 0-7, while Set embeds the update value in command bytes 48-55. Invalid properties, offsets, or access sizes fail as Invalid Field in Command. [PDF pp. 479-481](../_source/pages/page-479.md)

```text
memory-based host -- MMIO/property access ------> controller property
message-based host -- Property Get/Set capsule -> same property surface
```

This explanatory transport mapping distinguishes access mechanism from property semantics. [PDF pp. 71-73, 479-481](../_source/pages/page-071.md)

## Evidence

- [Property map and access rules, PDF pp. 71-73](../_source/pages/page-071.md)
- [CAP capability fields, PDF pp. 74-77](../_source/pages/page-074.md)
- [VS, INTMS, INTMC, and start of CC, PDF pp. 77-78](../_source/pages/page-077.md)
- [CC enablement and queue-size preconditions, PDF pp. 78-79](../_source/pages/page-078.md)
