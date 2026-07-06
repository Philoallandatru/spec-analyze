# Predictable Latency Mode

Predictable Latency Mode describes each supported NVM Set as alternating between a Deterministic Window (`DTWIN`) with bounded work/time expectations and a Non-Deterministic Window (`NDWIN`) used to prepare for a later deterministic interval. Its log plane reports the current window, warning/transition events, static bounds, and live remaining estimates. [PDF pp. 248-250](../_source/pages/page-248.md)

## Window model

```text
              reads / writes / time budget exhausted
[DTWIN] ----------------------------------------------> [NDWIN]
   ^                                                       |
   |     minimum preparation interval                      |
   `-------------------------------------------------------'
```

This explanatory state model captures the logged windows; detailed transition rules are defined later in the specification. [PDF pp. 249-250](../_source/pages/page-249.md)

| Reported group | Meaning |
|---|---|
| Window status | Disabled, Deterministic Window, or Non-Deterministic Window. |
| Events | Read/write/time warnings and autonomous DTWIN-to-NDWIN transitions caused by excursion or exceeded values. |
| Static values | Typical DTWIN read/write budget, maximum DTWIN time, and high/low minimum NDWIN time. |
| Reliable estimates | Remaining DTWIN reads, writes, and milliseconds under current activity/conditions. |

Static typical/maximum/minimum values are lifetime worst-case parameters; estimates decrement or evolve with host activity and operating conditions. Reads are counted as 4 KiB random reads, while writes use the NVM Set's Optimal Write Size. [PDF pp. 249-250](../_source/pages/page-249.md)

## Event discovery and acknowledgment

```text
event occurs -> ordered Aggregate list contains NVMSETID
                    |
                    v
read per-set log with RAE=0 -> clear event bits + remove aggregate entry
```

The aggregate log is an ascending list of non-zero NVM Set Identifiers with pending enabled events. A newly added entry generates an asynchronous event. Reading beyond the list returns zeroes, and successful `RAE=0` retrieval of the corresponding per-set log clears that set's events and aggregate entry. [PDF pp. 249-250](../_source/pages/page-249.md)

## Feature configuration

Predictable Latency Mode Config (`13h`) enables one NVM Set and configures event thresholds. Successful enablement completes only after required background preparation and leaves the NVM Set in the Non-Deterministic Window. Its event structure separately enables excursion/exceeded-value and read/write/time warning events; warnings trigger when the corresponding estimate falls below its threshold. [PDF pp. 405-406](../_source/pages/page-405.md)

Predictable Latency Mode Window (`14h`) selects DTWIN or NDWIN for every namespace in the NVM Set. It is invalid while the mode is disabled; completion is the transition boundary, and entering DTWIN may delay completion until the minimum NDWIN residence time has elapsed. [PDF pp. 406-407](../_source/pages/page-406.md)

## Relationships

- [NVM Sets and Endurance Groups](nvm-sets-and-endurance-groups.md) supplies the resource and identifier selected by the per-set log. [PDF pp. 99-103, 248](../_source/pages/page-099.md)
- [Asynchronous Event Reporting](asynchronous-event-reporting.md) supplies notification and acknowledgment behavior for newly pending set events. [PDF pp. 195-202, 249-250](../_source/pages/page-195.md)

## Evidence

- Per-set selector and Endurance Group boundary: [PDF p. 248](../_source/pages/page-248.md)
- Window/event fields and static/reliable values: [PDF pp. 249-250](../_source/pages/page-249.md)
- Ordered aggregate list and per-set clearing: [PDF p. 250](../_source/pages/page-250.md)
- Mode enablement, event thresholds, and window selection: [PDF pp. 405-407](../_source/pages/page-405.md)
