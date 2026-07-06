# Command Effects and Support

The Commands Supported and Effects log is a controller-published catalog that tells host software whether each Admin or selected I/O opcode is supported and what subsystem state that command may change. The 4,096-byte log contains one 32-bit descriptor for each Admin opcode and one for each I/O opcode in the selected I/O Command Set. [PDF p. 236](../_source/pages/page-236.md)

## Mental model

```text
opcode -> supported? -> possible effects -> host coordination
                         | scope             | pause conflicts
                         | capability change | wait for completion
                         ` data change       ` re-enumerate / re-Identify
```

This explanatory flow shows why the log is operational metadata rather than merely an opcode list. [PDF pp. 236-238](../_source/pages/page-236.md)

## Descriptor semantics

| Group | Meaning |
|---|---|
| `CSUPP` | The opcode is supported; when clear, every other descriptor field is zero. |
| Scope | Possible impact on namespace, controller, NVM Set, Endurance Group, Domain, or whole NVM subsystem; multiple scope bits may be set. |
| `USS` | The command supports UUID selection. |
| `CSE` / `CSER` | Submission/execution recommendations serialize commands that could conflict at namespace or global scope; a supported nonzero relaxation supersedes `CSE`. |
| Capability effects | The command may change controller capability, namespace inventory, one namespace's capability, or logical-block content. |

The compact table summarizes rendered Figure 210 without reproducing reserved bits. The controller describes possible overall effects, including optional command behavior, so the host should treat set flags conservatively. [PDF pp. 237-238](../_source/pages/page-237.md)

## Host use

After a command that may change a capability or namespace inventory, host software may need to pause use of affected namespaces, wait for command completion, and re-enumerate or reissue Identify. When a namespace is attached to multiple controllers, hosts coordinate submissions across those controllers to satisfy the descriptor's execution recommendation. [PDF p. 236](../_source/pages/page-236.md)

The I/O half of the log is interpreted for the command set selected by `CC.CSS`, except that the Command Set Identifier in Command Dword 14 selects the command set when `CC.CSS=110b`. [PDF p. 236](../_source/pages/page-236.md)

## Feature and management-command effects

```text
interface instance + selected command set / UUID
                |
                +--> FID 00h..FFh -> supported + one scope + effects
                `--> MI opcode 00h..FFh -> supported + one scope + effects
```

The Feature Identifiers Supported and Effects log is interface-instance-specific: an Admin Queue, PCIe VDM endpoint, or 2-Wire endpoint may expose a different FID set. For I/O controllers, the selected or profiled I/O Command Set also filters support; when UUID selection is advertised, `UIDX` further selects the reported FID namespace. [PDF p. 281](../_source/pages/page-281.md)

| Catalog | Per-entry support | Scope rule | Common change flags |
|---|---|---|---|
| Feature Identifier | `FSUPP`; unsupported means all other bits zero | zero means unreported; otherwise exactly one of namespace/controller/NVM Set/Endurance Group/Domain/subsystem/CDQ | user data, one namespace capability, namespace inventory, controller capability |
| NVMe-MI opcode | `CSUPP`; unsupported means all other bits zero | zero means unreported; otherwise exactly one of namespace/controller/NVM Set/Endurance Group/Domain/subsystem | user data, one namespace capability, namespace inventory, controller capability |

This compact comparison was checked against rendered Figures 261–264. The Feature catalog additionally reports whether Get/Set Features accepts UUID selection; both catalogs describe the overall possible effect, including optional behavior, rather than the effect of one particular invocation. [PDF pp. 282-284](../_source/pages/page-282.md)

## Relationships

- [Command Ordering and Arbitration](command-ordering-and-arbitration.md) selects work for execution; command-effects metadata tells the host where additional serialization is advisable. [PDF pp. 120-123, 237-238](../_source/pages/page-120.md)
- [Namespace Identifiers](namespace-identifiers.md) supplies the namespace inventory that a command may invalidate or require the host to rediscover. [PDF pp. 235-238](../_source/pages/page-235.md)

## Evidence

- Log layout, command-set selection, and host coordination: [PDF p. 236](../_source/pages/page-236.md)
- Scope and command-submission fields: [PDF p. 237](../_source/pages/page-237.md)
- Relaxation and capability/data-change flags: [PDF p. 238](../_source/pages/page-238.md)
- Interface-specific Feature support and effects: [PDF pp. 281-283](../_source/pages/page-281.md)
- NVMe-MI command support, scope, and effects: [PDF pp. 283-284](../_source/pages/page-283.md)
