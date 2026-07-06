# Security Protocol Exchange

Security Send and Security Receive transport protocol-defined security commands, parameters, status, and returned data between a host and controller. NVMe supplies the command envelope; the selected Security Protocol defines request/response association and payload meaning. [PDF pp. 390-392](../_source/pages/page-390.md)

## Mental model

```text
Host                         Controller / security protocol
 |-- Security Send(payload) -------------------->|
 |                                                | protocol action
 |<-- Security Receive(status / result) ----------|
       association and retention are protocol-defined
```

This explanatory exchange preserves the asymmetric directions and protocol-owned association. Receive data may be lost on communication loss or Controller Level Reset. [PDF pp. 390-392](../_source/pages/page-390.md)

## Command envelope

| Field | Send | Receive |
|---|---|---|
| `SECP` | selects Security Protocol Out semantics | selects Security Protocol In semantics |
| `SPSP` | 16-bit protocol-specific selector | same selector namespace |
| `NSSF` | NVMe-specific only for protocol `EAh` | same |
| length | transfer length | allocation length |
| data direction | host to controller | controller to host |

Reserved or unsupported protocol values fail with `Invalid Field in Command`. For protocols other than `EAh`, the NVMe Security Specific field is reserved. [PDF pp. 391-392](../_source/pages/page-391.md)

## Discovery and NVMe protocol

Security Protocol `00h` is a standalone Receive discovery request that returns the controller's supported security protocols and is not associated with a prior Send. Security Protocol `EAh` is assigned to NVMe interface use; `SPSP=0001h` selects Replay Protected Memory Block and `NSSF` selects an RPMB target. [PDF p. 391](../_source/pages/page-391.md)

## Relationships

- [Identify Command Model](identify-command-model.md) advertises RPMB target count and shared capabilities; non-zero target count requires Security Send/Receive support. [PDF pp. 333-334](../_source/pages/page-333.md)
- Protocol-specific details remain in the cited SPC/ACS specifications and are not redefined by the Base Specification. [PDF pp. 390-392](../_source/pages/page-390.md)

## Evidence

- [Security Receive purpose and retention boundary, PDF p. 390](../_source/pages/page-390.md)
- [Receive fields, protocol discovery, NVMe EAh/RPMB selection, PDF p. 391](../_source/pages/page-391.md)
- [Security Send fields and response relationship, PDF p. 392](../_source/pages/page-392.md)
