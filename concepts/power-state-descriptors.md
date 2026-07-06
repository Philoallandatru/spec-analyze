# Power State Descriptors

Identify Controller exposes up to 32 power-state descriptors. Each descriptor combines whether I/O is operational, entry/exit latency, relative read/write performance, idle/active/maximum power, and optional power-loss processing times so a host can trade power against service and recovery latency. [PDF pp. 350-353](../_source/pages/page-350.md)

## Mental model

```text
host chooses power state
       |
       +-- operational? -> I/O may continue
       +-- entry/exit latency
       +-- relative read/write throughput + latency
       +-- idle / active / sustained-maximum power
       `-- forced-quiesce / emergency-power-fail timing
```

This is an explanatory grouping of Figure 313 rather than a bit-layout reproduction. [PDF pp. 350-353](../_source/pages/page-350.md)

## Power and performance contract

| Descriptor group | Interpretation |
|---|---|
| `NOPS` | Clear means I/O commands are processed; set marks a non-operational state. |
| Entry/exit latency | Maximum transition times in microseconds; zero means unreported. |
| Relative performance | Lower latency numbers mean lower latency; lower throughput numbers mean higher throughput. Values are ranks within the supported-state count. |
| Idle power | Typical 30-second power after ten seconds of strict idle, excluding commands, property access, background work, sanitize, and self-test. |
| Active power | Largest ten-second average under the stated workload. |
| Maximum power | Sustained maximum power guideline for host policy. |

Power values pair a magnitude with their own scale. Zero magnitude or “not reported” scale means the host lacks that estimate; these fields are guidance and may be supplemented by platform/form-factor requirements. [PDF pp. 351-353](../_source/pages/page-351.md)

## Power-loss timing

Forced-quiescence vault time, emergency-power-fail vault time, and emergency-power-fail recovery time are reported only when the corresponding Power Loss Signaling mode is supported. Each is `value × scale`, where the scale ranges from one microsecond through one million seconds; zero value means unreported and forces its scale to zero. [PDF pp. 350-353](../_source/pages/page-350.md)

## Relationships

- [Controller Initialization](controller-initialization.md) consumes entry/exit and recovery timing when returning a controller to service. [PDF pp. 124-134, 351-353](../_source/pages/page-124.md)
- [Identify Command Model](identify-command-model.md) locates the descriptor array after the controller/Fabrics capability region. [PDF p. 350](../_source/pages/page-350.md)

## Evidence

- Descriptor array and time-scale fields: [PDF pp. 350-351](../_source/pages/page-350.md)
- Active/idle power, relative performance, transition latency, and operational flag: [PDF p. 352](../_source/pages/page-352.md)
- Maximum power and time-scale value table: [PDF p. 353](../_source/pages/page-353.md)
