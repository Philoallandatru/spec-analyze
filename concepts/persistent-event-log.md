# Persistent Event Log

The Persistent Event Log is an NVM-subsystem-global history of significant events not tied to a particular command. Events survive power cycles and resets, are generally returned newest first, and are read through a frozen reporting context so a multi-command transfer can be checked for consistency. [PDF pp. 252-255](../_source/pages/page-252.md)

## Retention and pressure

```text
event occurrence -> persistent store -> reporting context snapshot -> host reads
                         |                         |
                         | future events stay     ` generation check
                         | outside snapshot
                         ` vendor suppression/deletion under pressure
```

The controller logs every supported occurrence unless a repeated event exceeds a vendor frequency threshold. At size, count, or category limits, deletion policy is vendor-specific; events affecting multiple controllers should be logged once by one vendor-selected controller. Sanitize may remove or alter events to prevent user-data inference. [PDF pp. 252-253](../_source/pages/page-252.md)

## Reporting-context lifecycle

```text
no context -- establish+read --> context exists -- read slices --> release
     ^                              |                        |
     `------ reset / timeout -------'                        ` no data returned
```

| Action | Contract |
|---|---|
| Read | Requires an existing context; otherwise Command Sequence Error. |
| Establish and read | Freezes length and event set, then returns data at the requested offset; fails if a context already exists. |
| Release | Drops a context if present, returns no data, and is idempotent. |
| Establish/check header | Always returns the 512-byte header and reports whether a context already existed. |

The action table summarizes rendered Figure 225. A context normally lasts until explicit release, NVM Subsystem Reset, Controller Level Reset, or a vendor timeout long enough for retrieval. Events occurring while it exists are logged but excluded from that snapshot. [PDF pp. 253-255](../_source/pages/page-253.md)

For a segmented read, the host reads the header generation after establishing the context and again after the complete transfer. A mismatch means the context may have been lost and the assembled log may be invalid, so the host re-reads it. [PDF p. 254](../_source/pages/page-254.md)

## Snapshot header and ownership

The 512-byte log header reports event count, total byte length, log/header revisions, context-establishment time, power history, controller identity, subsystem NQN, and a generation number that advances only when a newly established context returns different data. Reporting Context Information says whether a context already existed and whether it was established through an NVM subsystem port or Management Endpoint, including that port identifier. [PDF pp. 256-257](../_source/pages/page-256.md)

```text
512-byte header
+ identity + capture time + generation + context owner + support bitmap +
event 0 | event 1 | ... | event N
```

The Supported Events Bitmap maps bit number directly to Event Type. Base event families cover health snapshots, firmware, timestamp/reset, hardware errors, namespace changes, format/sanitize, Feature changes, telemetry, thermal excursions, media verification, vendor-specific, and TCG-defined events. [PDF pp. 258, 260](../_source/pages/page-258.md)

## Self-describing event framing

```text
+ event type/revision + header length + controller/time/port +
+ vendor-info length + event length +
| optional vendor information | type-specific event data |
```

Each entry is independently parseable: `EHL+3` gives total header size, and `EL` covers optional vendor information plus type-specific event data. The header identifies the creating controller, occurrence timestamp, and port association (`subsystem port`, `Management Endpoint`, or `no port`). Event Type Revision versions the type-specific data independently of the overall log revision. [PDF pp. 258-259](../_source/pages/page-258.md)

Revision 2.1 controllers supporting this log advertise `ECRH`, which couples support for the 512-byte establish/check-header action with the Generation Number consistency protocol. [PDF p. 260](../_source/pages/page-260.md)

## Foundational event records

| Event | Recorded evidence |
|---|---|
| SMART / Health snapshot | Complete 512-byte SMART/Health image; created at least every 24 power-on hours for every controller, or every primary controller under virtualization management. |
| Firmware Commit | Old/new revisions, action, slot, completion SCT/SC, and optional vendor result. |
| Timestamp change | Previous timestamp plus milliseconds since Controller Level Reset; current timestamp is in the common event header. |
| Power-on or reset | Effective firmware plus one reset descriptor per controller, or per primary controller under virtualization management. |

These mandatory/conditional records preserve enough cross-controller data to reconstruct firmware and reset history. A reset descriptor reports controller ID, firmware activation success/failure, whether Format NVM was active, controller power-cycle count, power-on milliseconds, and controller timestamp. [PDF pp. 260-262](../_source/pages/page-260.md)

## Hardware-error evidence

Hardware-error events use a two-byte code followed by code-specific optional evidence. Categories include PCIe correctable/non-fatal/fatal errors and link changes, subsystem or Endurance Group critical warnings, unexpected power loss, fatal controller state, media/data-integrity completion status, and controller-ready timeout. Unavailable optional information is cleared to zero unless a code says otherwise. [PDF pp. 263-266](../_source/pages/page-263.md)

```text
hardware error code
  |-- PCIe error ------> device/AER status, masks, header and prefix logs
  |-- warning ---------> warning bitmap (+ Endurance Group ID)
  |-- power loss ------> lifetime count + out-of-band cause flag
  |-- media error -----> captured CQE
  `-- ready timeout ---> CNR | admin-media-not-ready | namespace-not-ready | CRIME
```

The additional structures preserve state at event time. PCIe AER fields exist only when AER is supported; the ready-timeout byte distinguishes command-processing readiness from attached-namespace and Admin-command media readiness and records the selected media-independent ready mode. [PDF pp. 265-266](../_source/pages/page-265.md)

## Namespace and destructive-operation journal

The Change Namespace event preserves successful Namespace Management inputs and resulting identifiers: size/capacity, format/protection/sharing attributes, ANA Group, NVM Set, Endurance Group, and created/deleted NSID. A delete-all event uses `FFFFFFFFh` and leaves per-namespace attributes reserved. [PDF pp. 266-268](../_source/pages/page-266.md)

| Paired event | Start evidence | Completion evidence |
|---|---|---|
| Format NVM | Validated NSID, controller format attributes, and command Dword 10 before media modification | smallest observed remaining progress, incomplete/error flags, CQE status, and vendor completion information |
| Sanitize | capabilities plus command Dwords 10–11 when the target enters a processing state | sanitize progress/status plus vendor completion information when the target reaches idle or a failure state |

The paired records distinguish accepted intent from the eventual result, including partial Format modification and sanitize failure. [PDF pp. 268-269](../_source/pages/page-268.md)

## Feature-change journal

A Set Feature event is required when a supported-to-log Feature changes through a successful Set Features command and may be recorded for a successful no-op setting. Logging eligibility varies by Feature and controller type; some combinations are prohibited or not recommended. The variable event payload identifies how many consecutive command dwords from Dword 10, optional completion Dword 0, and memory-buffer bytes were retained. [PDF pp. 270-271](../_source/pages/page-270.md)

## Diagnostic and extension events

| Event | Persistent evidence |
|---|---|
| Telemetry created | Copy of the first 512-byte Host- or Controller-Initiated telemetry header. |
| Thermal excursion | Threshold transition class plus absolute Kelvin delta from the crossed threshold. |
| Sanitize media verification | Header-only marker when sanitize enters Media Verification. |
| Vendor-specific | One or more typed descriptors carrying a stable event code, UUID index, and name/string/binary/signed-integer data. |
| TCG-defined | Event revision and payload reserved to TCG. |

Thermal records distinguish high warning/critical and throttling transitions, return-to-normal/stopped-throttling transitions, and low-temperature threshold crossings; repeated threshold reports remain subject to vendor frequency suppression. [PDF pp. 272-275](../_source/pages/page-272.md)

```text
vendor event
`-- descriptor(s)
    |-- code + UUID index
    `-- typed data: name | ASCII | binary | signed integer
```

When Get Log Page specifies a UUID index, returned vendor events include both events for that UUID and events defined by the subsystem manufacturer. If an event-name descriptor exists, it is first and its name remains stable for the same vendor event code. [PDF pp. 273-275](../_source/pages/page-273.md)

## Hardware-error records

An NVM Subsystem Hardware Error event uses Event Type `05h` and Event Type Revision `02h`. The subsystem records every detected error from its vendor-supported set unless the general repeated-event suppression rule applies; unavailable optional-source fields are zeroed. The event carries a hardware-error code plus code-specific additional information when that information exists. [PDF p. 263](../_source/pages/page-263.md)

| Initial code family | Additional-information rule |
|---|---|
| PCIe correctable, uncorrectable non-fatal, or uncorrectable fatal error | Uses the PCIe error-information format referenced by the code table. |
| PCIe Link Status change | Captures the PCI Express Link Status register at the event. |
| PCIe Link Not Active | Carries no additional hardware-error information. |

This is a compact reconstruction of the beginning of Figure 236; later code values continue beyond this batch. [PDF p. 263](../_source/pages/page-263.md)

## Relationships

- [Log Page Retrieval](log-page-retrieval.md) provides byte offsets and transfer sizing used to read the frozen snapshot. [PDF pp. 223-225, 253-255](../_source/pages/page-223.md)
- [Controller Enable, Shutdown, and Reset](controller-enable-shutdown-reset.md) defines the reset boundaries that may release a reporting context even though stored events persist. [PDF pp. 135-141, 252-253](../_source/pages/page-135.md)

## Evidence

- Global persistence and ANA boundary: [PDF p. 252](../_source/pages/page-252.md)
- Ordering, suppression, capacity pressure, and context retention: [PDF p. 253](../_source/pages/page-253.md)
- Generation consistency protocol: [PDF p. 254](../_source/pages/page-254.md)
- Rendered Action field and reporting-context operations: [PDF p. 255](../_source/pages/page-255.md)
- Rendered header, context ownership, supported-event bitmap, and generic event framing: [PDF pp. 256-260](../_source/pages/page-256.md)
- SMART snapshot, firmware, timestamp, and reset event formats: [PDF pp. 260-262](../_source/pages/page-260.md)
- Hardware-error codes and code-specific evidence: [PDF pp. 263-266](../_source/pages/page-263.md)
- Namespace, Format, Sanitize, and Set Feature journal events: [PDF pp. 266-271](../_source/pages/page-266.md)
- Telemetry, thermal, media-verification, vendor, and TCG event families: [PDF pp. 272-275](../_source/pages/page-272.md)
- Rendered NVM Subsystem Hardware Error framing and initial code families: [PDF p. 263](../_source/pages/page-263.md)
