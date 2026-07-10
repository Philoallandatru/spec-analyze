# Feature Values and Scope

Features are controller operating parameters addressed by Feature Identifier through Get Features and Set Features. A Feature has a scope and may expose default, current, and optionally saved values. [PDF pp. 181-183](../_source/pages/page-181.md)

## Mental model

```text
manufacturer-defined default (not changeable)
                 |
                 +---- reset fallback -------------------+
                 v                                       |
             current value <---- Set Features -------- saved value
                 |                         `-- Save=1 ----^
                 `---- active controller behavior
```

This explanatory state model condenses the value relationships. When saved-value support is absent, only default and current exist; a non-saveable Feature may nevertheless define current-value persistence across resets and power cycles. [PDF p. 181](../_source/pages/page-181.md)

## Value contract

| Value | Meaning |
|---|---|
| default | manufacturer-set and immutable unless the Feature definition specifies otherwise |
| current | value actively used by the controller |
| saved | persistent restore value available only for saveable Features |

If saved-value support exists, Get Features can discover each Feature's supported capabilities. Setting a saveable Feature may update current alone or current plus saved; requesting the saved value of a Feature without one returns its default. [PDF p. 181](../_source/pages/page-181.md)

| Get Features `SEL` | Returned attribute |
|---|---|
| `000b` | current |
| `001b` | default |
| `010b` | saved, or default when saving is unsupported/no value exists |
| `011b` | supported capabilities |

This compact reconstruction follows rendered Figure 192. A supported vendor-specific Feature may additionally use CDW14 `UIDX` to select a UUID-defined namespace for its meaning. [PDF pp. 220-222](../_source/pages/page-220.md)

When `SEL=011b`, CQE Dword 0 reports whether any value is changeable, whether the Feature is namespace-scoped, and whether it is saveable. Changeability may still be value-dependent; reporting a Feature as changeable does not mean every value transition is legal. [PDF p. 223](../_source/pages/page-223.md)

## Reset and scope interaction

```text
reset covers entire subsystem
`-- every affected scope: saved -> saved value; otherwise default

reset covers subset of subsystem
|-- controller / namespace-per-controller scope: restore saved/default
`-- broader resource scopes: unchanged unless specified otherwise
```

This explanatory comparison reconstructs rendered Figures 124-125. For non-saveable, non-persistent Features, the affected controller-local scope returns to default. A reset limited to part of a multi-controller or multi-Domain subsystem does not implicitly replace current values at subsystem, Endurance Group, NVM Set, or Namespace scope. [PDF pp. 181-182](../_source/pages/page-181.md)

| Feature scope | Normal `NSID` selector rule |
|---|---|
| controller | `0h` or `FFFFFFFFh`; Set with a valid NSID fails as Feature Not Namespace Specific |
| namespace | active NSID selects one; `FFFFFFFFh` may set all attached namespaces only when the controller permits it |
| subsystem, Domain, Endurance Group, NVM Set | `NSID` should be `0h`; nonzero values are rejected under the Common Command Format rules |

For namespace-scoped Get Features, `FFFFFFFFh` is not an aggregate read and normally fails with Invalid Namespace or Format. Unsupported Feature Identifiers fail with Invalid Field in Command. [PDF pp. 182-183](../_source/pages/page-182.md)

## Coordination boundary

Multiple hosts concurrently changing a Feature whose scope is broader than one controller require host coordination; the coordination procedure is outside the Base Specification. [PDF p. 182](../_source/pages/page-182.md)

## Set Features execution contract

Set Features selects a Feature Identifier, optionally requests persistence with `SV`, and may use a Feature-specific dword or memory buffer. A PRP-backed buffer may cross at most one page boundary and may not begin with a PRP List. Vendor Features may use a UUID Index only when both the command and selected Feature support UUID selection. [PDF pp. 392-395](../_source/pages/page-392.md)

```text
Feature ID + scope selector
          |
          +-- current update
          `-- SV=1 and saveable --> saved update across power/reset
                                   otherwise: Feature Identifier Not Saveable
```

This explanatory update path combines the Set command with the value model above. The Feature directory separately reports persistence when a Feature is not saveable, memory-buffer use, and scope from controller through namespace, NVM Set, Endurance Group, Domain, subsystem, Reclaim Unit Handle, or Controller Data Queue. [PDF pp. 393-395](../_source/pages/page-393.md)

After Set completes, the host should rediscover or reinitialize capabilities affected by that Feature. Commands submitted afterward use the new setting; already executing commands may use old or new settings, so a host needing a clean boundary should drain relevant work first. For a non-changeable Feature, a different value fails; an identical value may either succeed or fail as not changeable. [PDF p. 395](../_source/pages/page-395.md)

## Relationships

Host Behavior Support (`16h`) reverses the usual capability direction: the host advertises behaviors the controller may rely on. The non-saveable, replace-not-merge structure enables Advanced Command Retry, Telemetry Data Area 4, command-set-specific LBA format extensions, dispersed-namespace identity behavior, and selected Copy descriptor formats. Until a behavior is advertised, the controller shall not use functionality that depends on it. [PDF pp. 407-409](../_source/pages/page-407.md)

- Feature scope determines both the valid selector and which state a partial reset may leave unchanged. [PDF pp. 181-183](../_source/pages/page-181.md)
- Namespace selector validity builds on allocated/active NSID rules and the Common Command Format. [PDF p. 183](../_source/pages/page-183.md)
- Individual Feature definitions may override the general reset/default rules. [PDF pp. 181-182](../_source/pages/page-181.md)
- [Common Controller Features](common-controller-features.md) groups the first common policy Features built on this envelope. [PDF pp. 395-398](../_source/pages/page-395.md)

## Evidence

- [Feature default/current/saved contract, PDF p. 181](../_source/pages/page-181.md)
- [Rendered Figures 124-125, reset scope and value restoration, PDF pp. 182](../_source/pages/page-182.md)
- [Controller, namespace, and resource-scope NSID selection, PDF pp. 182-183](../_source/pages/page-182.md)
- [Rendered Figures 191-194, Get Features selectors and identifiers, PDF pp. 220-222](../_source/pages/page-220.md)
- [Rendered Figure 195, supported-capabilities result, PDF p. 223](../_source/pages/page-223.md)
- [Set Features envelope, save bit, UUID selection, and Feature directory, PDF pp. 392-395](../_source/pages/page-392.md)
- [Update ordering and rediscovery guidance, PDF p. 395](../_source/pages/page-395.md)
