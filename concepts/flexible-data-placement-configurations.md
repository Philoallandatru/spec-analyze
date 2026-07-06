# Flexible Data Placement Configurations

The FDP Configurations log is an Endurance Group-scoped catalog of static placement configurations that may currently be applied to that group. [PDF pp. 294-295](../_source/pages/page-294.md)

## Mental model

```text
Endurance Group selected by ENDGID
`-- FDP configuration descriptor (valid now?)
    |-- Reclaim Group count + Placement Identifier split
    |-- Reclaim Unit Handle count and ordered handle types
    |-- namespace / placement limits
    `-- nominal RU size + estimated handle-retarget time
```

This explanatory diagram summarizes Figures 279-281, checked against rendered local PDF pages 294-295. [PDF pp. 294-295](../_source/pages/page-294.md)

## Contract and invariants

| Constraint | Meaning |
|---|---|
| `NUMFDPC` | Zero-based number of configuration descriptors |
| `FDPCV=0` | Entry is not currently available and the host should ignore it |
| `NRG`, `NRUH` | Both are non-zero |
| `RGIF` | Non-zero when more than one Reclaim Group exists; selects MSBs of Placement Identifier used as RGID |
| `MAXPIDS` | Zero-based maximum and less than `NRG * NRUH` |
| `NNS` | Namespace limit, bounded by `MNAN` when non-zero, otherwise by `NN` |
| Handle list | Ordered by Reclaim Unit Handle Identifier |

[PDF pp. 294-295](../_source/pages/page-294.md)

Creating an Endurance Group or enabling FDP in one may change which catalog entries are valid. Descriptor size, vendor-specific size, and zero padding make descriptors variable length while maintaining the next eight-byte boundary. [PDF pp. 294-295](../_source/pages/page-294.md)

`FDPVWC=1` means the configuration has a volatile write cache and forces Identify Controller `VWCP=1`, including compatibility cases where no general controller cache exists for hosts unaware of FDP. [PDF p. 294](../_source/pages/page-294.md)

The nominal Reclaim Unit size applies to each unit within each Reclaim Group. `ERUTL` bounds how long a handle may reference a unit before controller retargeting is allowed; zero means no estimate is reported. [PDF p. 295](../_source/pages/page-295.md)

The `Initially Isolated` handle type initially separates data written through different handles within the same Reclaim Group, but vendor-specific movement such as garbage collection may later co-locate data written through handles of that same type. [PDF p. 295](../_source/pages/page-295.md)

`Persistently Isolated` is stronger: after controller-internal movement, data remains isolated to the same Reclaim Unit Handle within the Reclaim Group. The active configuration also determines whether a Placement Identifier is entirely a Placement Handle or splits its most-significant `RGIF` bits into a Reclaim Group Identifier. [PDF p. 296](../_source/pages/page-296.md)

## Runtime observation

Flexible Data Placement (`1Dh`) is a mandatory-save update selecting one valid configuration index for an Endurance Group and enabling or disabling FDP. A change may alter Identify-reported User Data Format fields, so the host should rediscover affected Identify views. [PDF p. 412](../_source/pages/page-412.md)

FDP Events (`1Eh`) enables an ordered list of event types for the Reclaim Unit Handle reached through one namespace's Placement Handle. It is invalid when FDP is disabled, rejects broadcast NSID, and applies to the shared handle across namespaces in the same Endurance Group. Get Features returns supported event types in ascending order with per-type enable state. [PDF pp. 413-414](../_source/pages/page-413.md)

```text
static configuration                runtime views
NRG + NRUH + RGIF  ----------+----> RU Handle Usage (LID 21h)
                             +----> FDP Statistics (LID 22h)
enabled event mask per RUH --+----> FDP Events (LID 23h)
                                      |-- host events
                                      `-- controller events
```

This explanatory diagram relates the configuration catalog to the three Endurance Group-scoped runtime logs; it is not a reconstruction of a numbered figure. [PDF pp. 296-299](../_source/pages/page-296.md)

| Runtime view | Stable contract |
|---|---|
| RU Handle Usage | One descriptor per configured handle: unused, explicitly host-selected, or the single controller-selected handle; at most one entry may be controller-selected. |
| FDP Statistics | Saturating 128-bit host-written, media-written, and media-erased byte counters; changing the FDP Feature clears them, while firmware update does not. |
| FDP Events | Fixed 4 KiB oldest-first event history, filtered to host or controller events and recorded only when enabled for the affected handle. |

The Usage, Statistics, and Events logs reserve command `NSID` while FDP is enabled and fail with `FDP Disabled` otherwise. Statistics aggregate every namespace that existed in the Endurance Group since FDP was last enabled. [PDF pp. 296-298](../_source/pages/page-296.md)

When the event history is full, a newly enabled event discards the oldest entry. The event count resets when FDP transitions from disabled to enabled. Ordering follows occurrence, not necessarily timestamp value, because Timestamp configuration may change across Controller Level Reset. [PDF pp. 298-299](../_source/pages/page-298.md)

Each event carries validity bits independently for Placement Identifier, NSID, and the Reclaim Group/Reclaim Unit Handle location. Host events cover early handle retarget, time-limit expiry, reset-driven retarget, and invalid placement; controller events cover media reallocation and implicit retarget. [PDF pp. 299-301](../_source/pages/page-299.md)

## Evidence

- [FDP catalog header and attributes, PDF p. 294](../_source/pages/page-294.md)
- [Configuration limits, variable layout, and handle type, PDF p. 295](../_source/pages/page-295.md)
- [Persistent isolation, Placement Identifier split, and Usage-log behavior, PDF p. 296](../_source/pages/page-296.md)
- [Usage descriptors and FDP Statistics boundary, PDF p. 297](../_source/pages/page-297.md)
- [Statistics counters and FDP Events retention, PDF p. 298](../_source/pages/page-298.md)
- [Event selector, ordering, and event-type table, PDF p. 299](../_source/pages/page-299.md)
- [Event validity fields and initial host events, PDF p. 300](../_source/pages/page-300.md)
- [Remaining host/controller event meanings, PDF p. 301](../_source/pages/page-301.md)
