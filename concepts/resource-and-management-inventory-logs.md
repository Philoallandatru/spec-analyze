# Resource and Management Inventory Logs

Several Get Log Page results expose inventories at a resource boundary rather than command history: rotational-media lifetime mechanics at Endurance Group scope, and management endpoints at NVM-subsystem scope. [PDF pp. 287-289](../_source/pages/page-287.md)

## Mental model

```text
Endurance Group selected by ENDGID
`-- rotational inventory: actuators, RPM, spinup/load success and failure counts

NVM subsystem
`-- up to 8 management-address descriptors
    `-- type + null-terminated UTF-8 URI
```

This explanatory diagram summarizes Figures 269, 271, and 272, checked against rendered local PDF pages 287-289. [PDF pp. 287-289](../_source/pages/page-287.md)

## Contract and invariants

| Log | Scope/selector | Stable contract |
|---|---|---|
| Rotational Media Information (`16h`) | Endurance Group via Log Specific Identifier | Retained across power cycles and resets; reports actuator count, Power State 0 RPM, and saturating 32-bit lifetime event counts |
| Management Address List (`18h`) | NVM subsystem | Zero to eight fixed-size descriptors; scanning stops at descriptor 7 or the first `MAT=FFh` terminator |

[PDF pp. 287-289](../_source/pages/page-287.md)

The rotational counters distinguish successful and failed transitions. Spinup concerns controller power-state movement from non-operational to operational; load concerns an actuator's corresponding state transition. Each counter increments while below `FFFFFFFFh`, so saturation is not wraparound. A subsystem with no rotational-media Endurance Group should not support this log. [PDF pp. 287-288](../_source/pages/page-287.md)

Management address type `1h` names a subsystem management agent and `2h` names a fabric interface manager. The address is a null-terminated UTF-8 URI. `MAT=0h` reserves the address field, while `MAT=FFh` terminates meaningful descriptors. [PDF p. 289](../_source/pages/page-289.md)

## Relationships

Rotational information complements the Endurance Group health and capacity view, while management addresses provide an out-of-band discovery path rather than an NVMe queue or association. [PDF pp. 287-289](../_source/pages/page-287.md)

## Evidence

- [Rotational-media selector and first fields, PDF p. 287](../_source/pages/page-287.md)
- [Rotational lifetime counters and dispersed-log boundary, PDF p. 288](../_source/pages/page-288.md)
- [Management address list and descriptor, PDF p. 289](../_source/pages/page-289.md)
