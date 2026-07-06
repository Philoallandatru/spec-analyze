# Completion Queue Entry and Status

A Completion Queue Entry (CQE) correlates a completed command, returns queue-consumption information, and reports its outcome. Admin and I/O commands share a common layout of at least 16 bytes; Fabrics responses use a separate 16-byte common layout. [PDF pp. 160-161](../_source/pages/page-160.md)

## Mental model

```text
submitted command                         posted completion
SQID + CID ------------------------------> SQID + CID
SQ tail ---- controller consumes SQEs ---> SQHD releases slots
command processing ----------------------> STATUS + optional error log
CQ wrap ---------------------------------> Phase Tag marks a new CQE
```

This explanatory flow joins the correlation, flow-control, and visibility roles of Figures 96-100. `SQID` is reserved for Fabrics, and the Phase Tag is reserved there because its queue transport does not use the memory circular-CQ mechanism. [PDF pp. 160-162](../_source/pages/page-160.md)

## Common layouts

| Region | Admin and I/O CQE | Fabrics CQE |
|---|---|---|
| bytes 0:7 | command-specific DW0-DW1 | Fabrics response type specific |
| bytes 8:11 | `SQID` + `SQHD` | `SQHD` + reserved |
| bytes 12:15 | `CID` + Phase Tag + Status | `CID` + reserved bit + Status |

In the memory-style common CQE, `SQID` plus `CID` uniquely identifies the command. `SQHD` is a snapshot taken when the CQE is created, so the controller may have consumed more SQEs before the host reads it. If a CQE is written in pieces, the controller updates the Phase Tag in the final write. [PDF pp. 160-161](../_source/pages/page-160.md)

For Fabrics, `SQHD` is reserved when Submission Queue flow control is disabled. The common status encoding is shifted into bits 15:1 of the final two-byte Status Info field, leaving bit 0 reserved. [PDF p. 161](../_source/pages/page-161.md)

## Status contract

```text
31      30    29:28      27:25        24:17
+-------+-----+-----------+------------+-----------+
| DNR   | M   | CRD       | SCT        | SC        |
+-------+-----+-----------+------------+-----------+
 retry?  log?  delay slot   code family  detail
```

This reconstructed bit field follows rendered Figure 100. A zero Status means successful completion without fatal or non-fatal error. Otherwise `SCT` selects the family and `SC` the detailed condition; if several errors apply and no precedence is specified, the vendor chooses which applicable status to return. [PDF pp. 161-163](../_source/pages/page-161.md)

| Field | Host interpretation |
|---|---|
| `DNR=1` | the same command is expected to fail if resubmitted to any controller in the subsystem |
| `M=1` | additional command status information is available in the Error Information log |
| `CRD` | when retry is allowed and ACRE is enabled, select immediate retry or controller delay time 1, 2, or 3 |
| `SCT` | Generic, Command Specific, Media/Data Integrity, Path Related, or Vendor Specific family |

`DNR=0` only means retry may succeed; it is not a guarantee. A nonzero `CRD` is advisory: the host should wait for the selected delay, but retrying early is not itself an error. `CRD` is reserved when `DNR=1` or Advanced Command Retry is disabled. [PDF p. 162](../_source/pages/page-162.md)

## Code-space organization

Within each status-code family, `SC=00h..7Fh` applies to Admin or multiple command sets, `80h..BFh` is I/O-Command-Set-specific, and `C0h..FFh` is vendor-specific. The Generic family therefore groups protocol-wide outcomes rather than defining a separate concept page for every code. [PDF pp. 163-166](../_source/pages/page-163.md)

| Generic outcome class | Representative conditions |
|---|---|
| command/sequence validation | invalid opcode or field, CID conflict, missing fused command, invalid namespace |
| transfer description | data-transfer error, invalid SGL form/length/type/offset, invalid PRP offset |
| controller/lifecycle gate | internal error, power-loss abort, SQ deletion, keep-alive expiry, sanitize state, media-not-ready |
| resource/access gate | CMB use, write protection, lockdown, capacity, reservation, placement-handle validation |
| retry-significant | Command Interrupted requires `DNR=0`; Format In Progress also requires `DNR=0` |

These classes summarize the reviewed cross-page Generic Command Status table without replacing its normative per-code definitions. [PDF pp. 163-166](../_source/pages/page-163.md)

Command Specific status binds a detailed error to one or more opcodes. Its common `00h..6Fh` range covers queue management, firmware, Features, namespace management/attachment, virtualization, sanitize, discovery/zoning, exported-subsystem management, and controller-data-queue operations; `70h..7Fh` is Directive specific, `80h..BFh` selects the I/O-command-specific table, and `C0h..FFh` is vendor specific. [PDF pp. 166-169](../_source/pages/page-166.md)

| Command-specific branch | What the host learns |
|---|---|
| common Admin/management | which command contract or lifecycle gate rejected the operation |
| I/O Command Set specific | an outcome whose meaning belongs to the selected I/O Command Set |
| Fabrics command specific | Connect/Disconnect format, controller availability, discovery restart, authentication, or transport-specific failure |

The Fabrics branch includes an explicit `Authentication Required` gate for queues that have not completed in-band authentication and reserves `B0h..BFh` for the applicable Transport binding. [PDF pp. 168-169](../_source/pages/page-168.md)

Media and Data Integrity status identifies media commitment/recovery failures, end-to-end protection-tag checks, access denial, or command-set-specific media conditions. Path Related status instead describes the particular host-controller or controller-resource path; in a multipath environment the host should generally try another path when one exists, while ANA Transition is retried after the transition completes and Persistent Loss/Inaccessible should not be retried through the same controller. [PDF pp. 169-171](../_source/pages/page-169.md)

## Phase Tag lifecycle

```text
host initializes every CQ slot P=0
              |
controller first posts slot 0 with P=1
              |
controller advances Tail through the ring
              |
Tail wraps to slot 0 -> posted Phase value toggles to P=0
              |
each later wrap toggles the expected posted Phase again
```

This explanatory state sequence condenses rendered Figure 108. The host recognizes a new memory-based CQE by the Phase value expected for its current Head position; the controller writes the Phase Tag last when constructing a CQE in multiple writes. [PDF pp. 160, 171-173](../_source/pages/page-171.md)

Admin CQ slots are initialized before `CC.EN` becomes one; I/O CQ slots are initialized before Create I/O Completion Queue is submitted. Host consumption advances Head but does not rewrite the Phase bits, so an empty slot may retain an old Phase value. [PDF pp. 171-173](../_source/pages/page-171.md)

## Relationships

- `SQHD` connects completion processing to Submission Queue capacity and may release multiple consumed entries at once. [PDF pp. 160-161](../_source/pages/page-160.md)
- The Phase Tag belongs to circular Completion Queue visibility, while `CID`/`SQID` provide command correlation; its expected value toggles only when the controller's posting position wraps. [PDF pp. 160-161, 171-173](../_source/pages/page-160.md)
- `M` links a CQE to Error Information log retrieval; `DNR` and `CRD` inform, but do not by themselves define, a safe higher-level retry policy. [PDF p. 162](../_source/pages/page-162.md)

## Evidence

- [Rendered Figures 96-99, common CQE layouts, PDF pp. 160-161](../_source/pages/page-160.md)
- [Rendered Figures 100-101, Status field and families, PDF pp. 162-163](../_source/pages/page-162.md)
- [Rendered Figure 102, Generic status values, PDF pp. 163-166](../_source/pages/page-163.md)
- [Rendered Figures 103-105, Command Specific status branches, PDF pp. 166-169](../_source/pages/page-166.md)
- [Rendered Figures 106-107, media/data and path-related status, PDF pp. 169-171](../_source/pages/page-169.md)
- [Rendered Figure 108, Phase Tag transitions, PDF pp. 171-173](../_source/pages/page-171.md)
