# Asynchronous Event Reporting

Asynchronous Event Request (AER) is a long-lived Admin command that lets a controller report status, health, notice, I/O-specific, immediate, one-shot, or vendor events through a completion. [PDF pp. 195-202](../_source/pages/page-195.md)

## Mental model

```text
Host                         Controller                       Log Page
 |-- submit one or more AERs -->|                                |
 |                              |<-- event occurs                |
 |<-- CQE: AET + AEI + LID -----|                                |
 |                              |-- mask same event type --------|
 |-- Get Log Page, RAE=0 --------------------------------------->|
 |<-- event details ---------------------------------------------|
 |                              `-- event type unmasked/cleared  |
 |-- submit replacement AER --->|                                |
```

This explanatory lifecycle covers ordinary log-backed events. Immediate events are reported only if an AER is already outstanding; one-shot events clear when reported; those categories do not necessarily use the log-read clearing step. [PDF pp. 195-197](../_source/pages/page-195.md)

## Outstanding and pending behavior

- AER has no timeout, and multiple outstanding requests reduce reporting latency up to the controller's advertised limit. [PDF pp. 195-196](../_source/pages/page-195.md)
- Reset aborts outstanding AERs without returning CQEs. [PDF p. 196](../_source/pages/page-196.md)
- Except for immediate events, an enabled event that occurs without an outstanding AER should be retained for the next request; identical responses may coalesce, while different responses should be queued. [PDF p. 197](../_source/pages/page-197.md)
- Once an ordinary event type is reported, that type is masked until its clearing action succeeds; media-not-ready prevents reporting when the required clearing log cannot yet be returned. [PDF p. 196](../_source/pages/page-196.md)

## Completion encoding

```text
AER CQE DW0
31        24 23        16 15         8 7       3 2       0
+-----------+------------+-------------+---------+---------+
| reserved  | LID        | AEI         |reserved | AET     |
+-----------+------------+-------------+---------+---------+

AER CQE DW1: event-specific 32-bit parameter
```

This reconstructed field map follows rendered Figures 147-148. `AET` selects the event family, `AEI` selects the event within it, `LID` points to the clearing/details log when applicable, and Dword 1 carries event-specific data. [PDF pp. 197-198](../_source/pages/page-197.md)

| Event family | Retention/clearing model |
|---|---|
| Error, SMART/Health, Notice, I/O-specific, vendor | normally log-backed; read indicated log with `RAE=0` |
| Immediate | report only when an AER exists at occurrence time |
| One-Shot | report once; controller clears notification on completion |

The exact AEI values span Figures 149-155. Major notice families cover namespace attributes, firmware activation, telemetry, ANA, endurance/reachability/discovery changes; I/O-specific events include reservation and sanitize state; immediate events include subsystem shutdown; one-shot events cover Controller Data Queue tail/full notifications. [PDF pp. 198-202](../_source/pages/page-198.md)

Before clearing a persistent or transient threshold event, the host should adjust its threshold or mask if the underlying condition remains, otherwise clearing can immediately retrigger notification. [PDF p. 197](../_source/pages/page-197.md)

## Relationships

The Asynchronous Event Configuration Feature is the event-family enable mask. It independently gates Discovery/Host/AVE/Pull-Model log changes, allocated/attached namespace changes, reachability, temperature-hysteresis recovery, normal shutdown, Endurance Group and Predictable Latency aggregates, ANA, telemetry, firmware activation, and SMART warnings. Enabling an already-true persistent condition causes its event to be sent; enabling an unsupported resource-specific family is invalid. [PDF pp. 399-400](../_source/pages/page-399.md)

This Feature controls notification delivery, not whether the underlying condition or log change occurs. Some recovery events require an AER to be outstanding at occurrence time, while retained-event behavior follows the event-family rules. [PDF pp. 399-400](../_source/pages/page-399.md)

- AER completion points to Get Log Page for details and clearing, coupling notification to log retrieval. [PDF pp. 196-198](../_source/pages/page-196.md)
- Asynchronous Event Configuration selects which configurable conditions generate events. [PDF p. 196](../_source/pages/page-196.md)
- AER limit excess is reported as Command Specific status `05h`. [PDF p. 197](../_source/pages/page-197.md)

## Evidence

- [AER lifetime, masking, event families, PDF pp. 195-197](../_source/pages/page-195.md)
- [Rendered Figures 146-148, status and CQE encoding, PDF pp. 197-198](../_source/pages/page-197.md)
- [Rendered Figures 149-150, Error and SMART/Health events, PDF p. 198](../_source/pages/page-198.md)
- [Rendered Figure 151, Notice events, PDF pp. 199-201](../_source/pages/page-199.md)
- [Rendered Figures 152-155, I/O-specific, Immediate, and One-Shot events, PDF pp. 201-202](../_source/pages/page-201.md)
