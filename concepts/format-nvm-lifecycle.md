# Format NVM Lifecycle

Format NVM changes media format attributes and may perform User Data Erase or Cryptographic Erase. Successful completion prevents any previously stored user data in affected namespaces from being returned. [PDF pp. 217-220](../_source/pages/page-217.md)

## Mental model

```text
                    SES = no erase          SES = secure erase
                           |                       |
scope capability ----------+-----------------------+
                           |
           FNS selects format scope / SENS selects erase scope
                           |
             NSID selects one, attached set, or subsystem set
                           v
           [format media + optional secure erase]
                           |
                 completion only when done
```

This explanatory decision flow condenses rendered Figure 188. `FNS` is relevant only when `SES=000b`; `SENS` is relevant only for nonzero `SES`. [PDF pp. 218-220](../_source/pages/page-218.md)

## Scope contract

| Scope bit | `NSID` | Affected namespaces |
|---|---|---|
| applicable scope bit `0` | one allocated NSID | specified namespace |
| applicable scope bit `0` | `FFFFFFFFh` | all namespaces attached to the controller |
| applicable scope bit `1` | allocated NSID or `FFFFFFFFh` | every namespace in the NVM subsystem |

The applicable scope bit is `FNS` for ordinary format and `SENS` for secure erase. If `FNVMBS=1`, broadcast `NSID=FFFFFFFFh` is prohibited. When a valid broadcast scope contains no namespace, the command completes successfully without work. [PDF pp. 218-219](../_source/pages/page-218.md)

## Secure erase and format fields

| `SES` | Effect |
|---|---|
| `000b` | no secure erase |
| `001b` | User Data Erase; controller may use cryptographic erase when all affected data is encrypted |
| `010b` | Cryptographic Erase by deleting the encryption key |

The command's format index combines `LBAFU` and `LBAFL`; protection-information and metadata fields are I/O-Command-Set-specific. `LBAFU` is ignored when Host Behavior Support disables LBA Format Extension. [PDF pp. 219-220](../_source/pages/page-219.md)

## Concurrency and rejection

- Conflicting in-flight I/O may cause Format NVM to fail with Command Sequence Error; new I/O to an affected namespace during format may fail with Format in Progress. [PDF pp. 218-219](../_source/pages/page-218.md)
- A divided multi-Domain subsystem that cannot reach the selected namespaces returns an ANA inaccessible or persistent-loss path status. [PDF p. 218](../_source/pages/page-218.md)
- Disabled/unsupported format, write protection, invalid broadcast, or invalid format fields cause the corresponding Invalid Namespace or Format, Namespace Is Write Protected, Invalid Field, or Invalid Format status. [PDF pp. 219-220](../_source/pages/page-219.md)

## Relationships

- Format NVM temporarily narrows the Admin command allowlist and interacts with I/O command processing. [PDF pp. 192-193, 218-219](../_source/pages/page-192.md)
- Successful settings are reported through Identify Namespace structures. [PDF p. 219](../_source/pages/page-219.md)
- Secure erase is a Format option, distinct from the broader Sanitize operation state machine. [PDF pp. 218-220](../_source/pages/page-218.md)

## Evidence

- [Format boundary and Firmware Download completion, PDF p. 217](../_source/pages/page-217.md)
- [Rendered Figure 188, Format/secure-erase scope matrix, PDF p. 218](../_source/pages/page-218.md)
- [Scope edge cases and rendered Figure 189 start, PDF p. 219](../_source/pages/page-219.md)
- [Rendered Figure 189 continuation and Figure 190 status, PDF p. 220](../_source/pages/page-220.md)
