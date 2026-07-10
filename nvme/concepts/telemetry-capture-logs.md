# Telemetry Capture Logs

NVMe telemetry logs preserve implementation-defined internal controller or NVM-subsystem state behind a standardized 512-byte header and 512-byte data-block envelope. Host-Initiated telemetry captures on request; Controller-Initiated telemetry exposes a controller-triggered saved capture. [PDF pp. 241-246](../_source/pages/page-241.md)

## Shared block model

```text
byte 0                                                byte 511
+--------------------------- standardized header -----------+
block 1        block 2                     block n
+-------------+-------------+ ... +-------------------------+
| all data areas begin at block 1; each area reports its    |
| cumulative last-block boundary                            |
+-----------------------------------------------------------+
```

The areas overlap as progressively larger prefixes rather than forming disjoint consecutive regions. Last-block values therefore do not decrease from Area 1 through the highest supported area. Transfers and log offsets must be multiples of 512 bytes. [PDF pp. 241-245](../_source/pages/page-241.md)

## Host-initiated capture

Setting `CTHID` on Get Log Page captures current internal state. The resulting data remains stable until a later requested capture, Firmware Commit, or power-on reset. If the log advertises `MCDAS`, the host may use `MCDA` to cap creation at Area 1, Areas 1–2, Areas 1–3, or Areas 1–4; otherwise that request field is ignored. [PDF pp. 241-244](../_source/pages/page-241.md)

```text
Get Log Page LID=07h
  CTHID=1 ----> capture + generation++
  MCDA --------> optional maximum area
                  |
                  v
header: vendor OUI | area ends | scope | generation | reason
```

The header also mirrors controller-initiated availability and generation so one read can correlate both capture streams. Scope is Controller or NVM subsystem in Revision 2.1 implementations, and the eight-bit generation number wraps from `FFh` to `00h`. [PDF p. 243](../_source/pages/page-243.md)

## Controller-initiated capture

Controller-Initiated Areas 1–3 persist across all resets; Area 4 may persist across Controller Level Reset. The availability flag and generation number persist across power states/resets. When availability is one, it is cleared only after a successful in-band Get Log Page with `RAE=0`; out-of-band Management Endpoint visibility may exist regardless of that flag. [PDF pp. 244-246](../_source/pages/page-244.md)

| Difference | Host-Initiated | Controller-Initiated |
|---|---|---|
| Capture trigger | Host sets `CTHID` | Vendor-specific controller condition |
| Stability/retention | Until recapture, Firmware Commit, or power-on reset | Areas 1–3 across all resets; Area 4 may cross Controller Level Reset |
| Event acknowledgment | No availability-clearing rule | Successful in-band read with `RAE=0` clears availability |
| LID | `07h` | `08h` |

[PDF pp. 241-246](../_source/pages/page-241.md)

## Area 4 negotiation

`DA4S` says Area 4 exists, while Host Behavior Support `ETDAS` says the host can consume the extended Area 4 boundary. With both set, the variable log size follows Area 4; otherwise host-visible sizing stops at Area 3 even when the controller supports Area 4. Data beyond the applicable final boundary is undefined. [PDF pp. 242, 244-245](../_source/pages/page-242.md)

## Relationships

- [Log Page Retrieval](log-page-retrieval.md) supplies LID, offset, `RAE`, and LID-specific discovery mechanics. [PDF pp. 223-228, 241-246](../_source/pages/page-223.md)
- [Firmware Update Lifecycle](firmware-update-lifecycle.md) may invalidate a Host-Initiated capture through Firmware Commit. [PDF p. 242](../_source/pages/page-242.md)
- [Asynchronous Event Reporting](asynchronous-event-reporting.md) directs the host to Controller-Initiated telemetry and uses `RAE=0` retrieval as acknowledgment. [PDF pp. 198, 246](../_source/pages/page-198.md)

## Evidence

- Host capture trigger and Self-test boundary: [PDF p. 241](../_source/pages/page-241.md)
- Host capture retention, block sizing, and Area 4 negotiation: [PDF p. 242](../_source/pages/page-242.md)
- Host header, scope, generation, and reason: [PDF pp. 243-244](../_source/pages/page-243.md)
- Controller capture retention, header, availability, and generation: [PDF pp. 244-246](../_source/pages/page-244.md)
