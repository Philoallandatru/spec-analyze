# Common Controller Features

Common controller Features apply across transport models and configure recurring policy surfaces such as arbitration, controller power, thermal thresholds, and volatile write caching. [PDF pp. 395-398](../_source/pages/page-395.md)

## Mental model

```text
Set Features completion
       |
       +-- Arbitration ------> future queue fetch policy
       +-- Power Management -> requested controller power state
       +-- Temperature ------> threshold event / hysteresis policy
       `-- Volatile Cache ---> write persistence boundary
```

This explanatory policy map groups the first common Feature definitions by downstream behavior. Commands submitted after successful completion use the new setting; already executing commands may or may not. [PDF pp. 395-398](../_source/pages/page-395.md)

## Feature contracts

| Feature | Control | Key invariant |
|---|---|---|
| Arbitration (`01h`) | high/medium/low zero-based weights and power-of-two fetch burst | `AB=111b` means no burst limit |
| Power Management (`02h`) | supported power-state index plus workload hint | successful Set completion places or is transitioning the controller to that state; autonomous transitions may continue |
| Temperature Threshold (`04h`) | sensor, over/under type, Kelvin threshold, optional hysteresis | Composite over-threshold is mandatory; all implemented sensors have both threshold types |
| Volatile Write Cache (`06h`) | cache enable | when present but disabled, namespace write data must be persistent |

[PDF pp. 396-398](../_source/pages/page-396.md)

## Temperature hysteresis

```text
Over-temperature:  start at T >= threshold
                   end   at T < threshold - hysteresis

Under-temperature: start at T <= threshold
                   end   at T > threshold + hysteresis
```

This explanatory threshold diagram follows the specified strict/non-strict comparisons. While active, the SMART Critical Warning temperature bit is set; recovery clears it, stops warning-temperature time accumulation, and triggers the configured hysteresis-recovery event. Unsupported sensors or hysteresis above `TMPTHMH` are invalid. [PDF pp. 397-398](../_source/pages/page-397.md)

## Relationships

- [Feature Values and Scope](feature-values-and-scope.md) defines the shared Set/Get, save, UUID, scope, and reset semantics. [PDF pp. 181-183, 392-395](../_source/pages/page-181.md)
- [Command Ordering and Arbitration](command-ordering-and-arbitration.md), [Power State Descriptors](power-state-descriptors.md), and [Error and Health Logs](error-and-health-logs.md) define the mechanisms controlled or observed by these Features. [PDF pp. 396-398](../_source/pages/page-396.md)

## Autonomous and non-operational power policy

Autonomous Power State Transition (`0Ch`) uses a physically contiguous 256-byte table with one 64-bit entry for each possible power state. A non-zero idle timer selects a non-operational destination after continuous idleness; zero disables autonomous transition for that source state. Unsupported-state entries are zero. [PDF pp. 400-401](../_source/pages/page-400.md)

| `APSTE` | `NOPPME` | Entry path | Background work in non-operational state |
|---|---|---|---|
| 1 | 1 | host or idle timer | allowed to exceed state power up to last operational-state limit |
| 0 | 1 | host only | allowed |
| 1 | 0 | host or idle timer | not allowed |
| 0 | 0 | host only | not allowed |

The permissive mode may trade strict power compliance for background maintenance. Without it, thermal mechanisms may be unavailable and resume performance may remain degraded until deferred work finishes. [PDF pp. 401, 404-405](../_source/pages/page-401.md)

## Timestamp and thermal management

Timestamp (`0Eh`) is a 48-bit millisecond counter. Its origin reports reset initialization versus host initialization, while `SYNC=1` warns that vendor-specific intervals were not counted. Saved timestamps may appear to move backward after reset and are unsuitable for security. [PDF pp. 402-403](../_source/pages/page-402.md)

Host Controlled Thermal Management (`10h`) defines a lower-impact threshold `TMT1` and heavier-action threshold `TMT2`; each may be disabled with zero, both must lie in the Identify-reported range, and when both are non-zero `TMT1 < TMT2`. [PDF pp. 403-404](../_source/pages/page-403.md)

Read Recovery Level Config (`12h`) changes recovery policy without changing namespace data. Its scope is one NVM Set when NVM Sets exist, otherwise the whole subsystem and its NVM Set selector is ignored. [PDF p. 405](../_source/pages/page-405.md)

## Edge cases

Software Progress Marker (`80h`) is a persistent saturating pre-boot load count: pre-boot software increments it after successful initialization unless already 255, and the OS clears it after successful OS initialization. [PDF p. 422](../_source/pages/page-422.md)

Spinup Control (`1Ah`) persistently selects host-controlled initial spinup for rotational-media Endurance Groups and is invalid when the subsystem contains none. Power Loss Signaling Config (`1Bh`) selects disabled, Emergency Power Fail, or Forced Quiescence mode within Identify-advertised support; it cannot be changed during Forced Quiescence processing. [PDF pp. 411-412](../_source/pages/page-411.md)

- In a multi-controller Domain, conflicting requested power states produce an unspecified Domain power state. [PDF pp. 394-395](../_source/pages/page-394.md)
- A cache guaranteed to persist through power loss is non-volatile and therefore outside the Volatile Write Cache Feature. If no volatile cache exists, both Set and Get fail with `Invalid Field in Command`. [PDF p. 398](../_source/pages/page-398.md)

## Evidence

- [Common Feature update boundary and changeability, PDF p. 395](../_source/pages/page-395.md)
- [Arbitration and Power Management fields, PDF p. 396](../_source/pages/page-396.md)
- [Temperature thresholds and hysteresis transitions, PDF p. 397](../_source/pages/page-397.md)
- [Threshold selectors and Volatile Write Cache contract, PDF p. 398](../_source/pages/page-398.md)
- [AER configuration boundary and APST table/interactions, PDF pp. 399-401](../_source/pages/page-399.md)
- [Timestamp origin/continuity and Keep Alive boundary, PDF pp. 402-403](../_source/pages/page-402.md)
- [Host thermal, non-operational power, and read recovery policy, PDF pp. 404-405](../_source/pages/page-404.md)
