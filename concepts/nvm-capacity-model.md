# NVM Capacity Model

The capacity model separates logical capacity allocation from physical Media Unit organization. A subsystem may report capacity at subsystem, Domain, Endurance Group, NVM Set, formatted-namespace, and Media Unit levels; support for any particular reporting level is optional. [PDF pp. 141-142](../_source/pages/page-141.md)

## Mental model

```text
NVM Subsystem capacity
`-- Domain capacity
    `-- Endurance Group capacity       <- each Media Unit belongs to exactly one
        `-- NVM Set capacity           <- each Media Unit belongs to exactly one if supported
            |-- formatted Namespace(s) <- each wholly inside one NVM Set
            `-- unallocated capacity

Channel(s) <---- data path ----> Media Unit(s)
   1..*                              1..*
```

This explanatory reconstruction was checked against rendered PDF pages 141-143 and the simple organization in Figure 86. Capacity containment and Channel connectivity are different relationships: capacity is allocated down the hierarchy, while Channels connect to Media Units for data transfer. [PDF pp. 141-143](../_source/pages/page-141.md)

## Allocation contract

| Source capacity or resource | Allocated to | Required containment |
|---|---|---|
| Domain capacity | one or more Endurance Groups | each Endurance Group remains in one Domain |
| Endurance Group capacity | one or more NVM Sets | each NVM Set remains in one Endurance Group |
| NVM Set capacity | one or more formatted namespaces | each namespace remains in one NVM Set |
| Media Unit | Endurance Group; also NVM Set when supported | exactly one target at each supported level |

Allocation need not consume all capacity at any level. Capacity Management creates and allocates Endurance Groups/NVM Sets and their Media Units; Namespace Management allocates formatted namespace capacity. If NVM Sets are unsupported, every `NVMSETID` field is zero; if Endurance Groups are unsupported, every `ENDGID` field is zero. [PDF p. 142](../_source/pages/page-142.md)

## Media organization and performance

```text
Simple maximum-bandwidth example (Figure 86)

one Endurance Group / one NVM Set
  |-- Channel 0 -> MU MU MU MU
  |-- Channel 1 -> MU MU MU MU
  |-- Channel 2 -> MU MU MU MU
  `-- Channel 3 -> MU MU MU MU

one operation may access Media Units across all Channels
```

This compact reconstruction of Figure 86 was checked against rendered PDF page 143. The example puts all 16 Media Units in one Endurance Group and one NVM Set to maximize bandwidth. [PDF pp. 142-143](../_source/pages/page-142.md)

| Organization (rendered figure) | Media Unit grouping | Intended tradeoff |
|---|---|---|
| Simple (Figure 86) | one Endurance Group and one NVM Set span four Channels | maximum aggregate bandwidth |
| Vertical (Figure 87) | each Channel's four Media Units form a separate Endurance Group and NVM Set | four isolated sets, each bounded by one Channel's bandwidth |
| Horizontal dual-NAND (Figure 88) | each of four Channels contributes one SLC Media Unit and three QLC Media Units to separate groups/sets | bandwidth across all Channels for both a smaller SLC set and a larger QLC set |

The table is an explanatory compression of rendered Figures 86–88. Figure 88 also uses a Capacity Adjustment Factor of about 400 for the SLC group and 100 for the QLC group; these are example configuration values, not universal SLC/QLC ratios. [PDF pp. 143-145](../_source/pages/page-143.md)

## Capacity reporting selection

| Entity being created or deleted | Supported hierarchy | Capacity information consulted and updated |
|---|---|---|
| Endurance Group | no Domains | NVM subsystem (`TNVMCAP`, `UNVMCAP`, `MEGCAP`) |
| Endurance Group | Domains | Domain Attributes Entry |
| NVM Set | NVM Sets and Endurance Groups | Endurance Group Information log page |
| Formatted namespace | neither Endurance Groups nor Domains | NVM subsystem |
| Formatted namespace | Domains but no Endurance Groups | Domain Attributes Entry |
| Formatted namespace | Endurance Groups but no NVM Sets | Endurance Group Information log page |
| Formatted namespace | NVM Sets | NVM Set Attributes Entry |

This reconstruction of Figure 89 was checked against rendered PDF page 146. “Consulted and updated” means the host uses that level to judge available capacity and the controller changes its allocated/unallocated accounting after creation or deletion. NVM Set support implies Endurance Group support. [PDF pp. 145-146](../_source/pages/page-145.md)

## Runtime Media Unit inventory

```text
Domain-selected Media Unit Status log
  `-- Media Unit descriptor (ascending MUID)
      |-- Domain -> Endurance Group -> NVM Set assignment
      |-- capacity adjustment + spare + percentage used
      `-- Channel IDs (ascending, unique)
```

The Media Unit Status log reports every accessible Media Unit for a selected Domain, the number of accessible Channels, and the active capacity configuration. Identifier scope is Domain-local in a multi-Domain subsystem and subsystem-wide otherwise. A zero Domain selector means the controller's own Domain; inaccessible Domain information makes the log unavailable. [PDF pp. 276-278](../_source/pages/page-276.md)

If no capacity configuration is selected, each descriptor clears its Endurance Group, NVM Set, capacity adjustment, and Channel-count fields. Otherwise the descriptor associates the Media Unit with resource parents and a sorted, duplicate-free Channel list. Spare and percentage-used are Media-Unit-local measurements; their relationship to Endurance Group values is intentionally unspecified. [PDF pp. 277-278](../_source/pages/page-277.md)

## Supported configuration inventory

The Supported Capacity Configuration List is Domain-selected and ordered by unique Capacity Configuration Identifier. Each variable-length configuration contains ordered Endurance Group configuration descriptors; those in turn contain unique ordered NVM Set identifiers and Channel configuration descriptors. [PDF pp. 278-279](../_source/pages/page-278.md)

```text
Supported Capacity Configuration List
`-- Capacity Configuration (ascending unique CCID)
    `-- Endurance Group configuration (ascending unique ENDGID)
        |-- capacity, spare, adjustment, endurance estimate
        |-- NVM Set IDs (ascending unique)
        `-- Channel configuration (ascending unique CHID)
            `-- Media Unit configuration (ascending unique MUID)
```

This explanatory hierarchy was checked against rendered Figures 257–260. Descriptors at every nested list may have different lengths. An Endurance Group descriptor reports total/spare byte capacity and a rounded-up lifetime endurance estimate in billions of bytes; zero means not reported. Each Channel descriptor enumerates its attached Media Units, while Revision 2.1 requires the Media Unit descriptor's extension length to be zero. [PDF pp. 279-281](../_source/pages/page-279.md)

```text
Capacity Configuration
`-- Endurance Group configuration
    |-- capacity adjustment / total / spare / endurance estimate
    |-- ordered NVM Set IDs
    `-- Channel configuration
        `-- ordered Media Unit configurations
```

An Endurance Group configuration combines capacity/endurance estimates with the logical NVM Set list and physical Channel layout. Channel descriptors list their Media Units; each current Base-defined Media Unit Configuration Descriptor is an eight-byte extensible record whose added-length field is zero. Saturating all-ones values mean the stated value and higher, while zero means an optional estimate is not reported. [PDF pp. 279-281](../_source/pages/page-279.md)

## Evidence

- [Capacity-reporting hierarchy and optionality, PDF p. 141](../_source/pages/page-141.md)
- [Allocation, zero identifiers, Media Units, and Channels, PDF p. 142](../_source/pages/page-142.md)
- [Rendered Figure 86 and vertical-organization introduction, PDF p. 143](../_source/pages/page-143.md)
- [Rendered Figures 87-88 and their configuration descriptors, PDF pp. 144-145](../_source/pages/page-144.md)
- [Rendered Figure 89 capacity-information selection matrix, PDF p. 146](../_source/pages/page-146.md)
- [Rendered Media Unit Status log and descriptor, PDF pp. 276-278](../_source/pages/page-276.md)
- [Rendered Supported Capacity Configuration descriptor hierarchy, PDF pp. 278-281](../_source/pages/page-278.md)
- [Rendered Endurance Group, Channel, and Media Unit configuration descriptor continuation, PDF pp. 279-281](../_source/pages/page-279.md)
