# Data Representation

Revision 2.1 defines conventions for encoded counts, units, strings, numeric notation, and byte ordering. Values are 1-based unless a field is explicitly described as 0's based; a 0's-based encoded value `n` represents the quantity `n + 1` and cannot represent zero. [PDF p. 25](../_source/pages/page-025.md)

## Mental model

Explanatory little-endian dword view, reconstructed from Figure 3:

```text
bit 31                         bit 0
+--------+--------+--------+--------+
| byte 3 | byte 2 | byte 1 | byte 0 |
+--------+--------+--------+--------+
   MSB                         LSB
```

Unless a structure says otherwise, multi-byte data is little-endian; a word is two bytes, a dword is four bytes, and a qword is four words. [PDF p. 27, Figure 3](../_source/pages/page-027.md)

## Numeric and unit conventions

| Form | Convention | Example |
|---|---|---|
| Hexadecimal | lower-case `h` suffix; underscore each eight digits when longer | `80h`, `1E_DEADBEEFh` |
| Binary | lower-case `b` suffix; underscore each four digits when longer | `10b`, `1000_0101b` |
| Decimal | digits with neither suffix; period decimal separator | `175`, `1.5` |
| Decimal units | `k`, `M`, `G`, ... use powers of 10 | `k = 10^3` |
| Binary units | `Ki`, `Mi`, `Gi`, ... use powers of 2 | `Ki = 2^10` |

The complete decimal/binary unit table extends through `Y = 10^24` and `Yi = 2^80`. [PDF pp. 25-26, Figure 2](../_source/pages/page-025.md) [PDF p. 26](../_source/pages/page-026.md)

## String rules

ASCII strings use code values `20h` through `7Eh`; UTF-8 strings use `20h` through `7Eh`, `80h` through `BFh`, and `C2h` through `F4h`. Both are left justified and, unless null-terminated, padded on the right with spaces. Padding after a null-terminated string should use null bytes. [PDF pp. 25-26](../_source/pages/page-025.md) [PDF p. 26](../_source/pages/page-026.md)

An admin label applies one of those string forms to identify a host, NVM subsystem, or namespace, for example by physical location or DNS name. [PDF pp. 27-28](../_source/pages/page-027.md) [PDF p. 28](../_source/pages/page-028.md)

## Evidence

- [Encoded counts, units, and ASCII rules, PDF p. 25](../_source/pages/page-025.md)
- [UTF-8, padding, and numeric notation, PDF p. 26](../_source/pages/page-026.md)
- [Byte/word/dword layout and admin labels, PDF p. 27](../_source/pages/page-027.md)
