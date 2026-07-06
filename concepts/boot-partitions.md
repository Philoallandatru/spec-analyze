# Boot Partitions

Boot Partitions are an optional pair of equal-sized controller-resident partitions. The host selects a partition range, provides a page-aligned destination buffer, and observes a status field while the controller copies the requested data. [PDF pp. 88-89](../_source/pages/page-088.md)

## Read flow

```text
1. BPMBL.BMBBA <- page-aligned host destination buffer
2. BPRSEL <- { BPID, offset in 4 KiB, size in 4 KiB }
                         |
                         v
3. BPINFO.BRS: 00 no request
                -> 01 in progress
                -> 10 completed successfully
                -> 11 error (destination contents undefined)
```

`BPINFO.ABPID` reports the active partition. `BPINFO.BPSZ` reports each partition's size in 128 KiB units, and both partitions have the same size. [PDF p. 88](../_source/pages/page-088.md)

## Bounds and alignment

```text
Boot Partition
+------------------------------------------------+
|<-- offset * 4 KiB -->|<-- size * 4 KiB -->|    |
+------------------------------------------------+

require: offset + size <= partition size
destination BPMBL address: 4 KiB/page aligned (low 12 bits reserved)
```

An out-of-range request transfers no data and reports an error through `BPINFO.BRS`. [PDF pp. 88-89](../_source/pages/page-088.md)

## Read-only log path

```text
Get Log Page, LID=15h, BPID
              |
              v
+-------------- 16-byte header --------------+------------------+
| LID | BPINFO: active ID + size (128 KiB)   | partition bytes  |
+---------------------------------------------+------------------+
```

The Boot Partition log provides a second, read-only path to the partition accessible through `BPRSEL`. `CDW10.BPID` selects partition 0 or 1; the returned header repeats the active partition and common partition size, followed by exactly `BPSZ * 128 KiB` of selected partition data. Reading the log does not change `BPINFO`, `BPRSEL`, or `BPMBL`. This layout was checked against rendered Figures 267–268. [PDF pp. 286-287](../_source/pages/page-286.md)

## Write-protection state

```text
power cycle -> [Write Locked]
                 | Set per partition
                 +--> [Write Unlocked]
                 +--> [Write Locked Until Power Cycle] --power cycle--> [Write Locked]
                 `--> [RPMB controlled] (Feature cannot override)
```

This explanatory state summary applies independently to both Boot Partitions. Set value `000b` means no requested change and is never returned by Get; `100b` is Get-only and reports RPMB control. Locked-until-power-cycle and RPMB-controlled states reject Feature attempts to change them. [PDF pp. 426-427](../_source/pages/page-426.md)

## Evidence

Boot Partition Write Protection Config (`85h`) independently requests no change, unlock, lock, or lock-until-power-cycle for each partition; both default to locked after power cycle. RPMB control is observable but cannot be directly selected or changed through this Feature, and lock-until-power-cycle cannot be changed before the next cycle. [PDF pp. 426-427](../_source/pages/page-426.md)

- [Boot Partition information and read selection, PDF p. 88](../_source/pages/page-088.md)
- [Boot Partition memory buffer, PDF p. 89](../_source/pages/page-089.md)
- [Rendered Boot Partition log selector and layout, PDF pp. 286-287](../_source/pages/page-286.md)
