# Host Metadata and Management Addresses

Management-address Features publish system- or host-provided agent URIs, while Host Metadata Features store typed host-platform and namespace annotations for later retrieval. [PDF pp. 416-422](../_source/pages/page-416.md)

## Mental model

```text
NVM subsystem management
|-- Embedded Management Controller Address -> system/enclosure agent URI
`-- Host Management Agent Address ----------> host-software agent URI

Host Metadata (4 KiB typed descriptor set)
|-- Enhanced Controller Metadata -> repeated element types allowed
|-- Controller Metadata ----------> at most one per type; legacy compatibility
`-- Namespace Metadata -----------> controller + namespace scoped
```

This explanatory hierarchy separates connection addresses from descriptive metadata and preserves the multiplicity/scope differences among metadata Features. [PDF pp. 416-422](../_source/pages/page-416.md)

## Management addresses

Embedded Management Controller Address (`78h`) and Host Management Agent Address (`79h`) each carry a 508-byte RFC 3986 URI after four reserved bytes. The former is system-provided; the latter identifies an agent residing in host software. Both should be lockable when Command and Feature Lockdown is supported. [PDF pp. 416-417](../_source/pages/page-416.md)

## Metadata update contract

All three metadata Features are changeable and non-saveable. The host transfers a 4 KiB structure containing zero or more variable-length typed descriptors, and each Set update is atomic. Growing the value beyond 4 KiB is invalid. [PDF pp. 417-420](../_source/pages/page-417.md)

| Action | Enhanced controller | Controller / namespace |
|---|---|---|
| add or replace one type | prohibited | create if absent, replace the single existing value |
| delete all specified types | allowed and idempotent | allowed and idempotent |
| add another value of a type | allowed | prohibited |

`GDHM=1` asks the controller to generate implementation-specific default strings; that modified default survives later default reads but not Controller Level Reset. Controller and Namespace Metadata allow at most one descriptor per type, whereas Enhanced Controller Metadata allows duplicates. [PDF pp. 418-420](../_source/pages/page-418.md)

Enhanced/Controller Metadata describe operating-system, pre-boot, processor, driver, firmware, product, and failure-record information. Controller Metadata exists for older management-controller compatibility and should not be used when Enhanced Controller Metadata is also supported. Namespace Metadata is both namespace- and controller-specific and names OS/pre-boot namespace labels and qualifiers. [PDF pp. 420-422](../_source/pages/page-420.md)

## Relationships

- [Command and Feature Lockdown](command-and-feature-lockdown.md) may prohibit management-address updates. [PDF pp. 416-417](../_source/pages/page-416.md)
- [Identity, Name, and List Formats](identity-name-and-list-formats.md) defines the UTF-8 processing boundary used by textual elements. [PDF pp. 186-188](../_source/pages/page-186.md)

## Evidence

- [CDQ boundary and Embedded Management Controller Address, PDF p. 416](../_source/pages/page-416.md)
- [Host Management Agent Address and metadata family, PDF p. 417](../_source/pages/page-417.md)
- [Generated defaults, PDF p. 418](../_source/pages/page-418.md)
- [Atomic element actions and 4 KiB bound, PDF p. 419](../_source/pages/page-419.md)
- [Descriptor framing and Enhanced Controller Metadata, PDF p. 420](../_source/pages/page-420.md)
- [Controller metadata element catalog and compatibility, PDF p. 421](../_source/pages/page-421.md)
- [Namespace Metadata and adjacent Host Identifier boundary, PDF p. 422](../_source/pages/page-422.md)
