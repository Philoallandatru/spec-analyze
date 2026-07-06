# Identity, Name, and List Formats

NVMe uses several identifier families for vendor attribution, human-readable device information, globally unique namespace identity, subsystem/host naming, and compact ordered lists. Their byte order and uniqueness scope are part of the format contract. [PDF pp. 183-189](../_source/pages/page-183.md)

## Mental model

```text
global identity
|-- NVM subsystem: NQN
|-- namespace: EUI64 and/or NGUID and/or UUID
`-- controller: subsystem NQN + subsystem-local CNTLID

presentation / attribution
|-- VID, SSVID, IEEE OUI
`-- Serial Number, Model Number

enumeration
|-- Controller List: ascending CNTLID values
`-- Namespace List: ascending NSID values
```

This explanatory hierarchy separates durable identity from handles and display/authority fields. `CNTLID` alone is only subsystem-unique; combining it with the subsystem identifier yields a globally unique controller identity. [PDF pp. 183-188](../_source/pages/page-183.md)

## Encoding families

| Identifier | Width/form | Byte/string order | Purpose |
|---|---|---|---|
| `VID`, `SSVID` | 16 bits each | little endian | PCI-assigned vendor attribution |
| Serial Number, Model Number | fixed ASCII fields | character order; space padded | vendor-assigned controller/subsystem description |
| IEEE OUI Identifier | 24 bits | little endian | Identify Controller authority value |
| `EUI64` | 8 octets | big endian | globally unique namespace identifier |
| `NGUID` | 16 octets | big endian | OUI plus vendor/extension namespace identity |
| UUID | 16 octets | RFC 9562 order | globally unique namespace identifier or UUID-based NQN component |

The compact table summarizes rendered Figures 126-136. The three-byte OUI Identifier is little endian, but the OUI embedded in EUI64 and NGUID participates in their big-endian octet strings; treating those representations as interchangeable reverses bytes. [PDF pp. 183-186](../_source/pages/page-183.md)

UUID versions may coexist. UUIDv8 uniqueness is implementation-defined, so all implementations visible to one administrative-tool domain are jointly responsible for generating non-colliding values when UUIDv8 is used. [PDF p. 186](../_source/pages/page-186.md)

## Namespace durable identity

A newly created namespace exposes at least one globally unique identifier through nonzero `EUI64`, nonzero `NGUID`, or a Namespace Identification Descriptor containing a UUID. If both EUI64 and NGUID are zero, a valid Namespace UUID is mandatory. [PDF pp. 188-189](../_source/pages/page-188.md)

`UIDREUSE=0` permits a deleted namespace's nonzero EUI64/NGUID value to be reused for a later namespace; `UIDREUSE=1` prohibits that reuse. This durable identity is distinct from the NSID command handle, which is only subsystem-scoped and controller-relative in attachment state. [PDF p. 189](../_source/pages/page-189.md)

## NVMe Qualified Names

```text
domain-authority form
nqn.yyyy-mm.<reversed-domain>:<authority-defined suffix>

UUID form
nqn.2014-08.org.nvmexpress:uuid:<RFC-9562 UUID text>
```

This explanatory grammar condenses the two supported NQN forms. An NQN is a null-terminated UTF-8 string of at most 223 bytes and is permanent for the lifetime of the host or NVM subsystem. In the domain form, the date must fall within the authority's ownership interval and the reversed domain must not be `org.nvmexpress`. [PDF pp. 187-188](../_source/pages/page-187.md)

For pre-1.2.1 controllers without a subsystem NQN, host software may construct the legacy 223-byte form from the standard prefix, ASCII VID/SSVID, Serial Number, Model Number, and space padding. The rendered Figure 139 layout is retained as source evidence rather than copied as a second normative modern NQN format. [PDF p. 188](../_source/pages/page-188.md)

## Ordered list structures

| List | Header | Entry width/order | Empty/unused representation |
|---|---|---|---|
| Controller List | 16-bit `NUMCIDS`, maximum 2,047 | 16-bit ascending controller identifiers | count zero; unused entries zero |
| Namespace List | none | 32-bit ascending namespace identifiers | first/remaining unused entries zero |

This compact reconstruction follows rendered Figures 137-138. List order is numeric ascending; zero is padding/absence, not a valid listed controller or namespace identity. [PDF pp. 186-187](../_source/pages/page-186.md)

## UTF-8 processing boundary

```text
configuration entry                         configuration entry
into Host                                   into NVM subsystem
    | optional text processing                  | optional text processing
    v                                           v
 Host ---------------- raw NVMe bytes ---------------- NVM subsystem
      receive: no normalization / locale logic
```

This explanatory boundary reconstructs Figure 140. NVMe participants store and compare protocol-carried UTF-8 strings as binary strings: normalization, locale-sensitive comparison, or UTF-8 validation may occur at a local configuration-entry boundary but should not be applied when a string is received through NVMe. [PDF pp. 189-190](../_source/pages/page-189.md)

## Relationships

Host Identifier (`81h`) registers the host identity used to associate controllers and reservation rights. Zero means no association with any other controller and does not persist reservation identity across Controller Level Reset; Fabrics requires the 128-bit form, while a subsystem supporting both sizes rejects inconsistent non-zero sizes across controllers. A non-zero current value cannot be overwritten directly. [PDF pp. 422-423](../_source/pages/page-422.md)

For PCIe the Feature is optional, defaults to zero, and may support either width; for Fabrics it is mandatory and always 128 bits. Neither transport has a saved Host Identifier value. [PDF p. 424](../_source/pages/page-424.md)

Namespace Admin Label (`1Fh`) is a mandatory-save, null-terminated 256-byte UTF-8 human label. Saved and current values are identical, Set requires `SV=1`, external management changes are visible to Get Features, and successful Sanitize resets both values to the all-null default. [PDF pp. 414-415](../_source/pages/page-414.md)

- NQN is the globally unique subsystem identifier and also identifies hosts for identification/authentication. [PDF pp. 187-188](../_source/pages/page-187.md)
- Namespace EUI64, NGUID, and UUID provide durable global identity, while NSID selects a namespace in commands. [PDF pp. 188-189](../_source/pages/page-188.md)
- Controller global identity is composite: subsystem identifier plus `CNTLID`. [PDF p. 188](../_source/pages/page-188.md)

## Evidence

PCIe makes Host Identifier optional and permits 64-bit and/or 128-bit forms, defaulting to zero with no saved value. Fabrics makes the Feature mandatory and requires 128 bits. Streams/reservations created while the ID is zero stay associated with the zero identity even after a later non-zero registration. [PDF pp. 423-424](../_source/pages/page-423.md)

- [Rendered Figures 126-136, identifier byte layouts, PDF pp. 183-186](../_source/pages/page-183.md)
- [Rendered Figures 137-138, Controller and Namespace Lists, PDF pp. 186-187](../_source/pages/page-186.md)
- [NQN formats and constraints, PDF pp. 187-188](../_source/pages/page-187.md)
- [Rendered Figure 139 and identifier uniqueness scopes, PDF p. 188](../_source/pages/page-188.md)
- [Namespace identifier requirements and UTF-8 processing, PDF p. 189](../_source/pages/page-189.md)
- [Rendered Figure 140, UTF-8 entry/receive boundary, PDF p. 190](../_source/pages/page-190.md)
