# Fabrics Discovery and Authentication

NVMe over Fabrics separates finding accessible subsystem paths from authorizing command traffic on a connected controller. A Discovery Service exposes only Discovery controllers and tells a host which accessible NVM subsystems and paths are available. [PDF p. 45](../_source/pages/page-045.md)

## Mental model

```text
bootstrap information (implementation-specific)
                    |
                    v
          [Discovery Service]
                    |
         subsystem paths + security need
                    |
                    v
            Connect to controller
                    |
          +---------+----------+
          | secure channel?    | establish before any command
          | in-band auth?      | only auth commands on new queue
          +---------+----------+
                    |
                    v
             normal command traffic
```

This explanatory sequence combines the discovery and authentication gates defined on PDF pages 45-46. [PDF pp. 45-46](../_source/pages/page-045.md)

## Contract and invariants

| Gate | Signal | Command restriction |
|---|---|---|
| Fabric secure channel | Discovery Service says it is required | Controller accepts no Fabrics, Admin, or I/O command until the channel exists |
| NVMe in-band authentication | Connect response says it is required | On the queue created by Connect, only authentication commands are accepted until authentication completes |

An NVM subsystem may require either gate or both. [PDF p. 46](../_source/pages/page-046.md)

The initial information used to reach a Discovery Service is implementation-specific; the specification gives host configuration, hypervisor/OS properties, or other mechanisms as examples. Discovery can enumerate accessible subsystems, multiple paths, and statically configured controllers, and may support persistent Discovery-controller connections and asynchronous notifications. [PDF p. 45](../_source/pages/page-045.md)

A Discovery controller is always message-based and dynamic. It implements discovery-related features only: no I/O Queues, I/O commands, or namespaces. A Discovery subsystem that advertises a unique Discovery Service NQN must accept both that NQN and the well-known Discovery Service NQN in Connect; otherwise it accepts the well-known NQN. [PDF pp. 58, 62](../_source/pages/page-058.md)

Authentication Send transfers security-protocol commands or parameters to the controller; Authentication Receive retrieves protocol-defined status/data associated with one or more prior Sends. The selected SPC-5 security protocol defines association and payload interpretation, while NVMe supplies the FCTYPE, SGL, protocol selectors, and transfer/allocation lengths. Reserved protocol values fail as Invalid Parameter. [PDF pp. 471-473](../_source/pages/page-471.md)

Authentication Receive data is ephemeral across communication loss and Controller Level Reset. Both authentication commands may be issued on Admin or I/O Queues, unlike Property commands, and their direction bits distinguish host-to-controller Send from controller-to-host Receive. [PDF pp. 471-473](../_source/pages/page-471.md)

On successful Connect, `AUTHREQ` reports whether an authentication transaction is required or authentication followed by secure-channel establishment is required. A failed Connect uses Connect-specific status and parameter-location information rather than Invalid Field in Command or an Error Information Log entry. [PDF pp. 477-478](../_source/pages/page-477.md)

## Discovery results and path selection

```text
Discovery Log Page
|-- entry: subsystem X, Port A ---- independent failure domain A
|-- entry: subsystem X, Port B ---- independent failure domain B
`-- entry: referral -------------> another Discovery Service

single association  -> choose one applicable entry
multiple associations -> prefer different Port IDs for path independence
```

Entries have no required ordering and may be filtered by Host NQN. Multiple entries for one subsystem may represent multiple fabric paths or multiple static controllers sharing a path; different Port IDs identify transport connections independent with respect to port hardware failures. [PDF p. 63](../_source/pages/page-063.md)

The Discovery log (`LID 70h`) is persistent and exists only on Discovery controllers. A request may return the host/requester-specific subset, port-local records, or—when supported and requested—all registered subsystem ports without `HOSTNQN` filtering. Only a Direct Discovery Controller may support the port-local-only option; a controller may require authentication or external configuration before advertising all-subsystem access. [PDF pp. 306-307](../_source/pages/page-306.md)

```text
Supported capability       Get Log Page request       Returned-page flag
EXTDLPES ----------------> EXTDLPE ------------------> EXTEND
PLEOS (DDC only) --------> PLEO --------------------> PORTLCL
ALLSUBES ----------------> ALLSUBE ------------------> ALLSUB
```

This explanatory negotiation diagram preserves the three capability/request/result relationships. [PDF pp. 306-308](../_source/pages/page-306.md)

## Consistent discovery snapshots

```text
read header + entries in increasing offset order
                     |
                     v
          re-read Generation Counter
             /                 \
        unchanged             changed
           |                     |
      process data       discard and restart
```

A one-command read is atomic. A multi-command read should use the Generation Counter protocol above; the controller may instead return `Discover Restart` when contents change. When index offsets are supported, index `0` addresses the header and index `1` begins at Entry 0. [PDF p. 307](../_source/pages/page-307.md)

## Entry semantics and layout

| Record signal | Meaning |
|---|---|
| `SUBTYPE=01h` | referral to another Discovery Service |
| `SUBTYPE=02h` | NVM subsystem whose controllers may expose namespaces |
| `SUBTYPE=03h` | another access path to the current Discovery Service |
| `DUPRETINFO=1` | current-service entries return duplicate discovery information, so one path may stand in for the group |

Referral processing deeper than eight levels is not required. `DUPRETINFO` is valid only for current-service entries and may also let one enabled change notification represent the duplicate-return group. [PDF p. 307](../_source/pages/page-307.md)

The fixed 1,024-byte entry carries transport/address family, subsystem type, transport security requirements, Port ID, Controller ID, maximum Admin SQ size, flags, service identifier, subsystem NQN, transport address, and transport-specific address data. Dynamic-model entries use `CNTLID=FFFFh`; static entries may name a controller or use `FFFEh` to request allocation while retaining the returned ID for reconnection. [PDF pp. 308-310](../_source/pages/page-308.md)

An Extended Discovery Log Page Entry preserves that 1,024-byte header, then adds total entry length, attribute count, and a variable extended-attribute list. The length fields, rather than a fixed stride, delimit successive attributes. [PDF p. 311](../_source/pages/page-311.md)

## Discovery information management

```text
entity header (Host / DDC / CDC)
        |
        +-- Register: 1..N entries; matching key replaces, new key adds
        +-- De-register: 1..N entry keys
        `-- Update: exactly 2 entries, identify old key + atomic replacement
                         |
                         v
                 CDC or DDC registration records
```

This explanatory task model preserves entry cardinality and the atomic two-entry update rule; exact command and data layouts remain in Figures 495-499. [PDF pp. 449-455](../_source/pages/page-449.md)

Discovery Information Management registers, de-registers, or updates host or NVM-subsystem discovery records. Host and CDC submissions contain host extended entries; DDC submissions contain basic or extended NVM-subsystem entries and alone may mark them port-local. The header's UUID-form NQN identifies the submitting entity, while the selected entry key is either transport-address based (required for host records) or port-ID based (also allowed for subsystem records). [PDF pp. 449-453](../_source/pages/page-449.md)

| Task | Required entry count | Effect |
|---|---:|---|
| Register | one or more | replace a matching key or add a new record |
| De-register | one or more | remove matching registration records |
| Update | exactly two | first entry identifies the record; second atomically replaces it |

Insufficient registration capacity fails the whole register command. Entity-type/field conflicts fail as Invalid Discovery Information. Host extended entries require at least one Host Identifier attribute; subsystem extended entries may have none. [PDF pp. 450-455](../_source/pages/page-450.md)

An extended entry retains the 1,024-byte discovery prefix, then uses `TEL` and `NUMEXAT` to frame a variable attribute list. A null first byte in `TRADDR` selects the connection's remote IP address, but such null-address registration or de-registration is limited to one entry per command. [PDF pp. 454-455](../_source/pages/page-454.md)

| Extended attribute | Applicability | Length rule |
|---|---|---|
| Host Identifier | mandatory for host entries; prohibited for subsystem entries | exactly 16 bytes |
| Admin label ASCII | optional | 4 to 256 bytes |
| Admin label UTF-8 | optional | 4 to 256 bytes |
| Vendor specific | optional | type-defined |

Every extended-attribute value length is non-zero and a multiple of four; unused bytes are zero. Host-entry-only routing fields such as subsystem type, Port ID, Controller ID, and Admin SQ size are cleared and ignored, while common transport, NQN, address, total-length, and attribute-list fields remain active for both entry types. [PDF pp. 455-456](../_source/pages/page-455.md)

## Discovery inventories beyond subsystem paths

```text
Discovery controller
|-- LID 70h: subsystem paths / referrals
|-- LID 71h: registered hosts
|-- LID 72h: authentication verification entities (AVEs)
`-- LID 73h: pull-model DDC request to a CDC
```

This explanatory taxonomy groups the related discovery-plane inventories and request channel. [PDF pp. 312-317](../_source/pages/page-312.md)

The Host Discovery log is supported only by Discovery controllers and inventories hosts that registered discovery information. It may return the requester-specific subset or, when `ALLHOSTES` is advertised and `ALLHOSTE` is requested, all registered hosts without `HOSTNQN` filtering; the returned `ALLHOST` flag states that result scope. Authentication or out-of-band configuration may gate advertisement of all-host access. [PDF pp. 312-315](../_source/pages/page-312.md)

Each Host Discovery entry is variable length and must contain at least one extended attribute carrying a Host Identifier. Its fixed prefix identifies the host NQN and transport address, while `NCC` reports whether that host is connected to the CDC; on a non-CDC the bit is zero and ignored. Multi-command Host Discovery retrieval uses the same read-in-order, re-read-generation, discard-on-change protocol as the subsystem Discovery log. [PDF pp. 313-315](../_source/pages/page-313.md)

| Log | Entry framing | Essential payload |
|---|---|---|
| Host Discovery (`71h`) | 1,024-byte header, then `TEL`-delimited entries | Host NQN, transport address, mandatory Host Identifier attribute |
| AVE Discovery (`72h`) | 1,024-byte header, then `TEL`-delimited entries | AVE NQN plus zero or more 20-byte IPv4/IPv6 TCP transport records |
| Pull Model DDC Request (`73h`) | `ORI`, total length, operation-specific bytes | GAZ, AAZ, RAZ, or Discovery Log Page Request |

This compact layout summary was checked against rendered Figures 299-304; exact byte layouts remain in the cited source. [PDF pp. 314-317](../_source/pages/page-314.md)

An explicit persistent Discovery connection is requested with a non-zero Keep Alive Timer. Such a controller must support Keep Alive, Asynchronous Event Request, and Discovery Log change notification; a Discovery controller never supports Disconnect. [PDF pp. 63-65](../_source/pages/page-063.md)

## Discovery Controller ID sentinels

| Controller ID | Meaning in discovery/Connect |
|---|---|
| `FFF0h`-`FFFCh` | Reserved; invalid in Connect |
| `FFFDh` | Registered dispersed-namespace controller is in another participating subsystem; invalid in Connect |
| `FFFEh` | Allocate any available static controller |
| `FFFFh` | Allocate any available dynamic controller |

For a static allocation obtained through `FFFEh`, the host should remember the actual returned Controller ID and reuse it for future associations to that controller. [PDF p. 64](../_source/pages/page-064.md)

## Change notification

```text
Discovery/Host Discovery log changes
              |
              v
Asynchronous Event: Notice (2h)
  |-- LID 70h + information F0h -> Discovery log changed
  `-- LID 71h + information F1h -> Host Discovery log changed
```

Notifications are sent only to entities that requested the corresponding asynchronous event type. [PDF pp. 64-65](../_source/pages/page-064.md)

## Pull-model discovery-log delivery

```text
pull-model DDC requests log from CDC
              |
CDC -- SDLP(status, LID, offset, dword count, data) --> DDC
              |                                         |
              `---- if CQE.LPUR=1, register one resend --'
                         registration consumed by next SDLP
```

This explanatory exchange preserves sender/receiver roles, segmented offset framing, and the one-shot update registration. [PDF pp. 469-470](../_source/pages/page-469.md)

Send Discovery Log Page carries the same bytes that Get Log Page would return, together with transferred LID, log-specific parameter, 64-bit offset, dword count, and a requested-log status. Only Discovery (`70h`), Host Discovery (`71h`), and AVE Discovery (`72h`) logs are allowed; other LIDs carry no data and report Not Allowed. [PDF pp. 469-470](../_source/pages/page-469.md)

The pull-model DDC may set `LPUR` in its completion to request a resend when that CDC log changes. The registration lasts only until the CDC issues the subsequent SDLP to that DDC, so continued updates require re-registration. [PDF p. 470](../_source/pages/page-470.md)

## Relationships

A controller is associated with exactly one host at a time, while multiple hosts may reach different controllers through the same NVM subsystem port. Discovery therefore identifies paths to controllers and subsystems; it does not make one controller a multi-host association. [PDF pp. 44-45](../_source/pages/page-044.md)

## Evidence

- [Discovery Service behavior, PDF p. 45](../_source/pages/page-045.md)
- [Authentication gates, PDF p. 46](../_source/pages/page-046.md)
- [Discovery controller model and NQN connection rules, PDF pp. 58, 62](../_source/pages/page-058.md)
- [Discovery entries, persistent connections, and path selection, PDF p. 63](../_source/pages/page-063.md)
- [Controller ID sentinels and change notifications, PDF pp. 64-65](../_source/pages/page-064.md)
- [Discovery-log capability and filtering negotiation, PDF pp. 306-308](../_source/pages/page-306.md)
- [Fixed Discovery entry fields and security/path semantics, PDF pp. 308-310](../_source/pages/page-308.md)
- [Extended Discovery entry framing, adjacent PDF p. 311](../_source/pages/page-311.md)
- [Host, AVE, and pull-model DDC discovery logs, PDF pp. 312-317](../_source/pages/page-312.md)
- [Discovery Information Management tasks, keys, and data framing, PDF pp. 449-455](../_source/pages/page-449.md)
- [Extended attribute types, lengths, and entry applicability, PDF pp. 455-456](../_source/pages/page-455.md)
- [Send Discovery Log Page framing and update registration, PDF pp. 469-470](../_source/pages/page-469.md)
- [Fabrics command matrix and Authentication Send/Receive, PDF pp. 471-473](../_source/pages/page-471.md)
- [Connect authentication/security requirements, PDF pp. 477-478](../_source/pages/page-477.md)
