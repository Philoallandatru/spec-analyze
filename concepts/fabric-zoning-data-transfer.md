# Fabric Zoning Data Transfer

Fabric Zoning commands let a Direct Discovery Controller (DDC) locate and exchange a Zoning data structure held by a Centralized Discovery Controller (CDC). Lookup maps operation-specific lookup data to a Zoning Data Key; Receive and Send then move the structure in dword-sized fragments. [PDF pp. 457-460](../_source/pages/page-457.md)

## Mental model

```text
DDC lookup data -- FZL --> CDC Zoning database
                              |
                              `-- Zoning Data Key (ZDK)
                                      |
                 +--------------------+--------------------+
                 |                                         |
        FZR: CDC -> DDC fragments                  FZS: DDC -> CDC fragments
        offset + length; CQE LF                    offset + length + command LF
```

This explanatory flow separates key lookup from fragmented transfer and preserves the different placement of the last-fragment signal. [PDF pp. 457-460](../_source/pages/page-457.md)

## Contract and invariants

| Operation | Selector context | Fragment termination |
|---|---|---|
| Lookup (`FZL`) | lookup data in the command buffer | returns `ZDK` in CQE Dword 0 |
| Receive (`FZR`) | DDC uses `ZDK`; CDC uses current Transaction ID | controller returns `LF` in CQE Dword 0 |
| Send (`FZS`) | DDC uses `ZDK`; CDC uses current Transaction ID | sender sets `LF` in command Dword 12 |

The `ZDKC` bit disambiguates a Zoning Data Key from a Transaction ID. A CDC accepts the key context and a DDC accepts the transaction context; the opposite combinations fail with Invalid Field in Command. `NUMD` must be non-zero. [PDF pp. 458-459](../_source/pages/page-458.md)

## Fragment lifecycle

```text
[select ZDK / Transaction ID]
             |
             v
 [offset, non-zero dword count] -- transfer --> [LF = 0: continue]
             ^                                      |
             `--------------------------------------'
                                                    |
                                            [LF = 1: complete]
```

This explanatory state sequence applies to both directions; Receive reports `LF` in the completion while Send carries it in the command. [PDF pp. 458-459](../_source/pages/page-458.md)

Offsets are dword aligned. A controller may reject low offset bits that are not zero; if it does not reject them, it behaves as though they were zero. Receive additionally rejects an offset beyond the requested structure size. [PDF pp. 458-459](../_source/pages/page-458.md)

## Errors and relationships

Lookup, Receive, and Send share command-specific outcomes for a locked or missing structure, disabled Fabric Zoning, and a DDC that is not permitted to access the ZoneGroup. [PDF pp. 457-460](../_source/pages/page-457.md)

Fabric Zoning is a fabric-wide access-control configuration mechanism. It is separate from the Allowed Host List maintained by an [Exported NVM Subsystem](exported-and-underlying-resources.md). [PDF p. 31](../_source/pages/page-031.md)

## Evidence

- [Lookup and returned Zoning Data Key, PDF p. 457](../_source/pages/page-457.md)
- [Receive fields and completion status, PDF p. 458](../_source/pages/page-458.md)
- [Receive termination and Send fields, PDF p. 459](../_source/pages/page-459.md)
- [Send completion status, PDF p. 460](../_source/pages/page-460.md)
