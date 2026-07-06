# Command and Feature Lockdown

Command and Feature Lockdown is an optional subsystem capability that can prohibit selected Admin, management-interface, PCIe-command, or Set Features operations. Its log separates what is lockable from what is currently prohibited on the in-band Admin Queue and out-of-band Management Endpoint paths. [PDF pp. 284-286](../_source/pages/page-284.md)

## Mental model

```text
                    Contents selector
              +----------+-----------+
              | lockable | prohibited|
              +----------+-----+-----+
                               | Admin Queue
Scope selector                 ` Management Endpoint
  |-- Admin opcodes
  |-- Feature IDs (+ optional vendor UUID)
  |-- Management Interface opcodes
  `-- PCIe command opcodes
```

This explanatory matrix was checked against rendered Figures 265–266. Scope chooses the identifier namespace; Contents chooses capability versus the currently enforced path-specific state. [PDF pp. 285-286](../_source/pages/page-285.md)

## Query contract

| Selector | Values |
|---|---|
| Contents | `00b` lockable; `01b` currently prohibited on Admin Queue; `10b` currently prohibited on Management Endpoint |
| Scope | `0h` Admin opcode; `2h` Feature ID; `3h` Management Interface opcode; `4h` PCIe opcode |
| UUID | considered only for Feature scope, where it may select vendor-specific Set Features lockdown information |

The returned 512-byte page echoes the selected Contents and Scope, gives a byte length, and lists coded identifiers in ascending numeric order. A zero length means the list is empty. Requesting Management Endpoint prohibition state when the subsystem has no Management Endpoint fails with `Invalid Field in Command`. [PDF pp. 285-286](../_source/pages/page-285.md)

## Relationships

- The effects catalogs describe support and possible impact; Lockdown independently describes whether supported operations may be or currently are prohibited. [PDF pp. 281-286](../_source/pages/page-281.md)
- The separate Lockdown command controls the capability; this log is its read-side inventory. [PDF p. 284](../_source/pages/page-284.md)

## Enforcement command

The Lockdown command names an opcode or Feature Identifier, chooses the identifier namespace with Scope, selects Admin Queue, Management Endpoint, or both with Interface, and then prohibits or allows execution. Feature scope may also carry a UUID Index when both Lockdown and that vendor Feature support UUID selection. [PDF pp. 370-371](../_source/pages/page-370.md)

| Condition | Result |
|---|---|
| target is not lockable | `Prohibition of Command Execution Not Supported` |
| Management Endpoint selected but absent | `Invalid Field in Command` |
| PCIe command scope combined with Admin Queue path | `Invalid Field in Command` |
| requested state already holds | successful idempotent request, not an error |

After successful prohibition, all applicable controllers and Management Endpoints in the subsystem enforce the selected rule. [PDF pp. 370-372](../_source/pages/page-370.md)

## Evidence

- [Capability purpose and UUID-qualified Feature scope, PDF p. 284](../_source/pages/page-284.md)
- [Rendered Log Specific Parameter selector, PDF p. 285](../_source/pages/page-285.md)
- [Rendered returned list layout and ordering, PDF p. 286](../_source/pages/page-286.md)
- [Lockdown command selectors and subsystem enforcement, PDF pp. 370-371](../_source/pages/page-370.md)
- [Lockdown command status, PDF p. 372](../_source/pages/page-372.md)
