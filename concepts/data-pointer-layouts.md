# Data Pointer Layouts

Data pointer layouts describe command data and metadata as controller-addressable memory regions. PRPs use fixed-size physical-memory-page pointers and optional page lists; SGLs use typed descriptors and segments. [PDF pp. 173-175](../_source/pages/page-173.md)

## Mental model

```text
PRP form                                  SGL form
command                                   command
|-- PRP1 -> first page + optional offset  `-- SGL1 -> data descriptor
`-- PRP2 -> second page                         or segment descriptor
          or packed PRP List                         |
                 |                                  `-> typed descriptors
                 `-> next list page (if needed)         `-> next/last segment
```

This explanatory comparison preserves the two indirection models: PRP list length is implied by the command transfer and page size, while an SGL explicitly describes lengths and typed segment links. [PDF pp. 173-175](../_source/pages/page-173.md)

## PRP entry contract

```text
63                         n+1 n                         0
+-----------------------------+--------------------------+
| Page Base Address           | offset within CC.MPS page|
+-----------------------------+--------------------------+
```

This reconstructed bit field follows rendered Figures 109-110. `CC.MPS` determines the offset width; every PRP is 64 bits and dword aligned. Only the command's first PRP entry, and a command field used as the first PRP List pointer where the command permits it, may carry a nonzero in-page offset. [PDF pp. 173-174](../_source/pages/page-173.md)

| PRP position | Offset rule | Role |
|---|---|---|
| first command PRP | may be nonzero; bits 1:0 are `00b` | begins data at an in-page address |
| second command PRP | zero | second data page or first PRP List page |
| entry inside PRP List | zero | one data page, except the chaining entry |
| next-list pointer | page aligned | points to another packed PRP List page |

If bits 1:0 are nonzero, the controller may report PRP Offset Invalid; if it does not, it behaves as though those bits were zero. A nonzero offset where page alignment is required should also produce PRP Offset Invalid. [PDF p. 174](../_source/pages/page-174.md)

## PRP List structure

```text
one memory page containing packed 64-bit entries
+-----------------------------+
| data page address 0         |
| data page address 1         |
| ...                         |
| data page address N         |  <- final data entry, or
| next PRP List page address  |  <- chain when another list page is needed
+-----------------------------+
```

A PRP List occupies one contiguous memory page, starts at entry zero, contains no duplicate of PRPs already carried by the command, and uses its last available entry to chain when another list page is required. Data pages named by adjacent entries may be physically contiguous or unrelated; the list shape is unchanged. [PDF p. 174](../_source/pages/page-174.md)

## SGL descriptors and segments

An SGL is one or more qword-aligned contiguous segments containing typed descriptors. Only a segment's last descriptor may link to another Segment or Last Segment, and a last segment contains neither kind of link. Data Block plus Bit Bucket lengths must cover the requested transfer; the controller must not transfer beyond the described total. [PDF p. 175](../_source/pages/page-175.md)

Controllers advertise supported SGL forms and whether Data Block address/length alignment is byte or dword granular. Unsupported descriptor format or illegal segment-link placement requires command abort; a dword-only controller should abort descriptors whose Address or Length has either low bit set. [PDF p. 175](../_source/pages/page-175.md)

```text
16-byte generic descriptor
+-----------------------------------------------+------+
| bytes 0:14: descriptor-type-specific content | ID   |
+-----------------------------------------------+------+
                                                  |  |
                                           type 7:4  subtype 3:0
```

This reconstructed byte map follows rendered Figures 113-116. An SGL segment is a packed array of 16-byte descriptors; descriptor type selects data, discard, segment link, keyed data, transport data, or vendor-specific interpretation, while subtype selects address, offset, or transport-specific addressing where legal. [PDF pp. 176-177](../_source/pages/page-176.md)

| Type | Role | Key invariant |
|---|---|---|
| Data Block | transfer one addressed byte range | zero length is valid; address plus length must not overflow 64 bits |
| Bit Bucket | discard part of source data for a destination buffer | has no effect for a source buffer |
| Segment | link to a following non-final segment | link length is nonzero and a multiple of 16 |
| Last Segment | link to the final segment | target segment contains no further segment link |
| Keyed Data Block | transfer an addressed range with a 32-bit key | length is encoded in 24 bits |
| Transport Data Block | delegate transfer mechanism/buffers to the binding | address bytes are reserved in the Base Specification |

The descriptor roles and field boundaries were checked against rendered Figures 117-122. Reserved or unsupported type/subtype combinations become SGL Descriptor Type errors; an all-zero descriptor is a valid zero-length Data Block descriptor. [PDF pp. 176-179](../_source/pages/page-176.md)

## SGL transfer coverage example

```text
controller logical data:  [3 KiB][4 KiB][discard 2 KiB][4 KiB] = 13 KiB
                              |      |                    |
host destination blocks:     A      B                    C   = 11 KiB

segment 0: Data(A,3 KiB) -> Segment(1)
segment 1: Data(B,4 KiB), BitBucket(2 KiB) -> LastSegment(2)
segment 2: Data(C,4 KiB)
```

This explanatory reconstruction of Figure 123 shows why transfer coverage is not identical to bytes placed in host memory. The command accesses 26 logical blocks of 512 bytes (13 KiB), while three Data Block descriptors receive 11 KiB and a destination Bit Bucket consumes the remaining 2 KiB without storing it. [PDF pp. 179-180](../_source/pages/page-179.md)

## Metadata Region boundary

The Base Specification names the Metadata Region but delegates its applicability and detailed format to the selected I/O Command Set. It is therefore kept as a boundary of this page rather than expanded into a Base-defined layout. [PDF p. 181](../_source/pages/page-181.md)

## Relationships

- `CDW0.PSDT` selects whether the Common Command Format interprets its metadata and data pointers as PRPs or SGLs. [PDF pp. 155-158](../_source/pages/page-155.md)
- PRP transfer coverage is inferred from command parameters plus `CC.MPS`; SGL coverage is carried in descriptor lengths. [PDF pp. 173-175](../_source/pages/page-173.md)
- NVMe over Fabrics commands use SGL descriptors rather than PRPs, with transport-specific descriptor forms where applicable. [PDF pp. 43, 159-160](../_source/pages/page-043.md)

## Evidence

- [Rendered Figures 109-110, PRP entry address and offset, PDF pp. 173-174](../_source/pages/page-173.md)
- [Rendered Figures 111-112, contiguous and non-contiguous PRP Lists, PDF p. 174](../_source/pages/page-174.md)
- [SGL segment and descriptor contract start, PDF p. 175](../_source/pages/page-175.md)
- [Rendered Figures 113-122, SGL descriptor layouts, PDF pp. 176-179](../_source/pages/page-176.md)
- [Rendered Figure 123, chained SGL read with Bit Bucket, PDF pp. 179-180](../_source/pages/page-179.md)
- [Metadata Region specification boundary, PDF p. 181](../_source/pages/page-181.md)
