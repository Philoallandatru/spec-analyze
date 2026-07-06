# Host Memory Buffer

Host Memory Buffer (`HMB`) is host memory allocated exclusively for controller use and described by a physically contiguous descriptor list. The Set Features completion is the ownership handoff boundary. [PDF pp. 429-431](../_source/pages/page-429.md)

## Mental model

```text
host allocates regions + descriptor list
             |
             | Set Features EHM=1
             v
[controller owns HMB; host must not write]
             |
             | Set Features EHM=0 completes
             v
[controller stops access; host may modify/reclaim]
```

This explanatory ownership sequence preserves the completion boundaries and exclusive-access rule. [PDF pp. 429-430](../_source/pages/page-429.md)

## Enable, disable, and return

Enabling an already enabled HMB is a command-sequence error; disabling an already disabled HMB succeeds without action. Before completing disable, the controller should retrieve any required data, and after completion it shall not access the HMB until re-enabled. [PDF pp. 429-430](../_source/pages/page-429.md)

`MR=1` returns the exact memory previously allocated before reset/Runtime D3: size, descriptor-list address/content, and buffer content must match the last controller-visible state. `MR=0` supplies memory with undefined contents. [PDF pp. 430-431](../_source/pages/page-430.md)

## Non-operational power-state restriction

When supported, the host may restrict HMB access after a host-requested non-operational power-state transition, except for Admin-command work and Admin-initiated background operations. This restriction is independent of permissive non-operational power mode and does not alter advertised entry latency; autonomous transitions instead require access to be minimized rather than immediately forbidden. [PDF p. 430](../_source/pages/page-430.md)

## Descriptor contract

The host supplies HMB size in controller memory-page units, a 16-byte-aligned physical address of a contiguous descriptor list, and a non-zero entry count. Each 16-byte descriptor names a page-aligned buffer address and a contiguous length in controller memory-page units; a zero length makes that descriptor ignored. [PDF pp. 431-432](../_source/pages/page-431.md)

```text
contiguous descriptor list
|-- entry 0: BADD (page aligned) + BSIZE (page units)
|-- entry 1: BADD (page aligned) + BSIZE (page units)
`-- entry n: BADD (page aligned) + BSIZE (page units)
```

This explanatory hierarchy preserves the 16-byte entry size and page-unit addressing of the rendered descriptor layout. [PDF pp. 431-432](../_source/pages/page-431.md)

Get Features returns `EHM`, restriction-enabled (`HMNARE`), and restriction-currently-active (`HMNAR`) in CQE Dword 0. Its data buffer is a 4 KiB attributes structure that reports the current size, descriptor-list address, and valid-entry count; these are observational values rather than a new ownership handoff. [PDF p. 432](../_source/pages/page-432.md)

## Relationships

- [Identify Command Model](identify-command-model.md) advertises preferred/minimum size, descriptor limits, and non-operational access restriction support. [PDF pp. 333-337](../_source/pages/page-333.md)
- [Common Controller Features](common-controller-features.md) defines host-requested and autonomous power transitions interacting with HMB access. [PDF pp. 400-405](../_source/pages/page-400.md)

## Evidence

- [HMB ownership and enable/disable rules, PDF p. 429](../_source/pages/page-429.md)
- [Non-operational restriction and memory-return semantics, PDF p. 430](../_source/pages/page-430.md)
- [Enable, size, descriptor-list address/count, adjacent PDF p. 431](../_source/pages/page-431.md)
- [Descriptor entries and Get Features attributes, PDF p. 432](../_source/pages/page-432.md)
