# Common Command Format

The Common Command Format is the 64-byte Submission Queue Entry shared by Admin commands and I/O commands across the defined I/O Command Sets. It fixes common identity, namespace, metadata, data-pointer, and command-specific regions. [PDF pp. 155-158](../_source/pages/page-155.md)

## Mental model

```text
byte  0       4       8      16      24              40              64
      +-------+-------+-------+-------+---------------+---------------+
      | CDW0  | NSID  |CDW2/3 | MPTR  | DPTR          | CDW10..CDW15  |
      +-------+-------+-------+-------+---------------+---------------+
       opcode   scope          metadata PRP1+PRP2 or SGL1  command-specific
```

This reconstructed byte map is a compact projection of Figure 92; exact pointer meaning depends on `CDW0.PSDT`. [PDF pp. 156-158](../_source/pages/page-156.md)

## Command Dword 0

| Bits | Field | Contract |
|---|---|---|
| 31:16 | `CID` | unique together with the Submission Queue identifier; `FFFFh` should be avoided |
| 15:14 | `PSDT` | `00b` PRPs; `01b` SGLs with MPTR buffer address; `10b` SGLs with one-descriptor MPTR segment; `11b` reserved |
| 09:08 | `FUSE` | normal, first fused command, second fused command, or reserved |
| 07:02 | `OPC.FN` | command function |
| 01:00 | `OPC.DTD` | none, host-to-controller, controller-to-host, or bidirectional transfer |

The table condenses rendered Figure 91. NVMe over PCIe Admin commands use PRPs and must not use SGLs; NVMe over Fabrics Admin and I/O commands use SGLs with `PSDT=01b`, subject to transport-supported values. [PDF pp. 155-156](../_source/pages/page-155.md)

## Namespace and pointer contract

`NSID` is zero when unused. `FFFFFFFFh` is a command-dependent broadcast only where supported; an inactive NSID normally yields Invalid Field in Command, an invalid NSID yields Invalid Namespace or Format, and a nonzero value in a command that does not use NSID yields Invalid Field in Command. [PDF p. 156](../_source/pages/page-156.md)

| `PSDT` | `MPTR` bytes 16:23 | `DPTR` bytes 24:39 |
|---|---|---|
| `00b` | dword-aligned contiguous metadata buffer | `PRP1` plus reserved/second-page/PRP-list `PRP2` according to page-boundary crossings |
| `01b` | contiguous metadata buffer with Identify Controller alignment | first SGL segment (`SGL1`) |
| `10b` | qword-aligned segment containing exactly one metadata SGL descriptor | first SGL segment (`SGL1`) |

For the PRP form, `PRP2` is reserved when no page boundary is crossed, identifies the second page when exactly one boundary is crossed, and points to a PRP List when more than one boundary is crossed. For SGL forms, a data-block descriptor may describe the entire transfer; otherwise the first descriptor links a Segment or Last Segment. [PDF pp. 157-158](../_source/pages/page-157.md)

## Format boundaries

Bytes 40:63 are command-specific dwords 10 through 15. Optional vendor-specific commands may instead use dwords 10 and 11 for transfer-length fields while retaining the common pointers. Future I/O Command Sets may define an alternate command size or format, so this layout is common rather than universal. [PDF pp. 156, 158-159](../_source/pages/page-156.md)

The adjacent Fabrics Command Common SQE is a distinct 64-byte layout: byte 4 carries `FCTYPE`, bytes 5:23 are reserved, bytes 24:39 contain a Transport or Keyed SGL descriptor for the complete transfer, and bytes 40:63 are Fabrics-command-type-specific. `FCTYPE` combines a six-bit function with a two-bit transfer direction. [PDF pp. 159-160](../_source/pages/page-159.md)

```text
byte  0       4   5                 24              40              64
      +-------+---+-----------------+---------------+---------------+
      | CDW0  |TYP| reserved        | SGL1          | FCTS          |
      +-------+---+-----------------+---------------+---------------+
       OPC=7Fh FCTYPE                whole transfer   type-specific
```

This explanatory map is reconstructed from rendered Figures 94-95. Fabrics commands have no fused form, use opcode `7Fh`, and use SGLs for data transfer; `PSDT=10b` explicitly marks a transfer, while `00b` is the no-transfer value but remains accepted with SGLs for a transferring command. [PDF pp. 159-160](../_source/pages/page-159.md)

## Relationships

- `CID` plus Submission Queue identifier correlates a command with its completion. [PDF p. 155](../_source/pages/page-155.md)
- `FUSE` embeds the fused-operation position into the common SQE. [PDF p. 155](../_source/pages/page-155.md)
- `PSDT` selects both metadata and data-pointer interpretation and is constrained by the transport. [PDF pp. 155-158](../_source/pages/page-155.md)

## Evidence

- [Rendered Figure 91, Command Dword 0, PDF pp. 155-156](../_source/pages/page-155.md)
- [Rendered Figure 92, Common Command Format, PDF pp. 156-158](../_source/pages/page-156.md)
- [Rendered Figures 93-94 boundary, PDF p. 159](../_source/pages/page-159.md)
- [Rendered Figures 94-95, Fabrics SQE and CDW0, PDF pp. 159-160](../_source/pages/page-159.md)
