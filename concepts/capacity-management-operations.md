# Capacity Management Operations

Capacity Management configures Endurance Groups and NVM Sets either by selecting a predefined capacity configuration or by explicitly creating and deleting capacity entities. [PDF pp. 203-206](../_source/pages/page-203.md)

## Mental model

```text
fixed management                         variable management
Capacity Configuration ID               requested 64-bit byte capacity
          |                                         |
          v                                         v
create configured Endurance Groups      create Endurance Group in Domain
          |                                         |
          v                                         v
create configured NVM Sets              create NVM Set in Endurance Group

delete parent -> contained namespaces / child NVM Sets are deleted
```

This explanatory comparison preserves the two modes and destructive containment semantics. Deleting an Endurance Group also deletes its NVM Sets and namespaces; deleting an NVM Set deletes its namespaces. [PDF pp. 203-205](../_source/pages/page-203.md)

## Operation contract

| Operation | `ELID` interpretation | Capacity fields |
|---|---|---|
| select configuration | Capacity Configuration Identifier; `0h` clears configuration | reserved |
| create Endurance Group | Domain Identifier; `0h` lets controller choose | requested 64-bit bytes |
| delete Endurance Group | existing Endurance Group Identifier | reserved |
| create NVM Set | parent Endurance Group Identifier; `0h` lets controller choose | requested 64-bit bytes |
| delete NVM Set | existing NVM Set Identifier | reserved |

The table condenses rendered Figures 156-158. CDW10 contains `ELID` and the operation, while CDW11 `CAPL` and CDW12 `CAPU` form the low and high halves of the requested capacity only for create operations. [PDF pp. 203-204](../_source/pages/page-203.md)

## Fixed configuration lifecycle

```text
[no selected configuration]
        | select valid descriptor
        v
create every described Endurance Group, then each described NVM Set
        |
        +-- select same ID --> success, no change
        |
        `-- select 0h --> delete namespaces -> NVM Sets -> Endurance Groups
                         -> clear Selected Configuration
```

This explanatory state sequence follows the ordered actions on PDF page 204. Selecting a different valid configuration while one is already selected is rejected without changing any Media Unit configuration; an unknown identifier is rejected the same way. [PDF p. 204](../_source/pages/page-204.md)

## Variable allocation and failure

Creating an Endurance Group selects Media Units when Media Units are reported, otherwise it allocates capacity from the Domain. Creating an NVM Set similarly selects Media Units or capacity from its Endurance Group. [PDF p. 205](../_source/pages/page-205.md)

| Failure | Result |
|---|---|
| no free nonzero entity identifier | Identifier Unavailable |
| requested bytes exceed applicable maximum/free capacity | Insufficient Capacity; Error Information may report the largest creatable size |
| delete uses zero/nonexistent identifier | Invalid Field in Command |
| NVM Set operation when NVM Sets unsupported | Invalid Field in Command |

On successful create, CQE Dword 0 returns the created Endurance Group or NVM Set identifier in bits 15:0. Other operations reserve that field. [PDF pp. 205-206](../_source/pages/page-205.md)

## Relationships

- Capacity Management mutates the same Domain -> Endurance Group -> NVM Set -> Namespace allocation hierarchy described by the NVM Capacity Model. [PDF pp. 141-146, 203-205](../_source/pages/page-141.md)
- Fixed selection consumes Supported Capacity Configuration descriptors and updates the Media Unit Status selected-configuration value. [PDF p. 204](../_source/pages/page-204.md)
- Insufficient-capacity completion connects command status to the Error Information log's command-specific capacity value. [PDF pp. 205-206](../_source/pages/page-205.md)

## Evidence

- [Rendered Figures 156-157, operation and capacity-low fields, PDF p. 203](../_source/pages/page-203.md)
- [Rendered Figure 158 and fixed configuration sequence, PDF p. 204](../_source/pages/page-204.md)
- [Endurance Group and NVM Set allocation/failure rules, PDF p. 205](../_source/pages/page-205.md)
- [Rendered Figures 159-160, status and created identifier, PDF p. 206](../_source/pages/page-206.md)
