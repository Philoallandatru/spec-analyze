# Error and Health Logs

The Error Information log is a controller-global, recency-ordered history of extended command and non-command errors. The SMART / Health Information log is a lifetime-oriented health snapshot available at controller scope and optionally requested per namespace, although Revision 2.1 defines identical content for those two SMART views. [PDF pp. 229-230](../_source/pages/page-229.md)

## Mental model

```text
command error / async error                 controller and media state
             |                                         |
             v                                         v
   +---------------------+                    +---------------------+
   | Error Information   |                    | SMART / Health      |
   | newest entry first  |                    | current warnings +  |
   | 64-byte records     |                    | lifetime counters   |
   +----------+----------+                    +----------+----------+
              \-------------- Get Log Page -------------/
```

This explanatory comparison separates event history from health state; it is not a reconstruction of a numbered figure. [PDF pp. 229-231](../_source/pages/page-229.md)

## Error Information contract

`M=1` in a command completion indicates that extended status is available in the Error Information log; an Error asynchronous event may also point to it. The requested number of 64-byte records returns the most recent records first. Capacity is `ELPE`-defined; when full, the controller should insert the new record and discard the oldest. [PDF p. 229](../_source/pages/page-229.md)

```text
Error Information entry (64 bytes)
+--------+------+------+------+-----+------+-----+-----+--------+
| ECNT   | SQID | CID  | STS  | PEL | NSID | CSI | OPC | detail |
|  0..7  | 8..9 |10..11|12..13|14..15|24..27| 30 | 31  | rest   |
+--------+------+------+------+-----+------+-----+-----+--------+
 correlation       status + bad parameter       command identity
```

This compact explanatory projection of Figure 205 intentionally omits reserved bytes and specialized detail fields; it was checked against rendered PDF pages 229-230. `SQID=CID=FFFFh` and `PEL=FFFFh` identify an error not tied to a particular command. [PDF pp. 229-230](../_source/pages/page-229.md)

| Field group | Meaning |
|---|---|
| `ECNT` | Persistent unique error counter starting at 1, wrapping from all ones to 1; zero marks an invalid/missing slot. |
| `SQID`, `CID` | Correlate a command, or use `FFFFh` when no command applies. |
| `STS`, `PEL` | Preserve completion status and identify the least-significant erroneous byte/bit when applicable. |
| NSID and command detail | Associate namespace, I/O-command-set data, opcode/CSI, command-specific data, transport, and optional vendor log. |
| `LPVER` | Revision 2.1 defines the entry version as `1h`. |

These fields are the semantic groups of the rendered cross-page structure, not a substitute for its exact byte layout. [PDF pp. 229-230](../_source/pages/page-229.md)

The controller should remove Error Information entries on power cycle and Controller Level Reset, even though `ECNT` itself is retained across power-off conditions. [PDF p. 229](../_source/pages/page-229.md)

## SMART / Health contract

SMART / Health data is retained across power cycles unless a field says otherwise. `NSID=0h` or `FFFFFFFFh` requests controller data; hosts seeking compatibility with Revision 1.4 and earlier should use `FFFFFFFFh`. If per-namespace SMART is unsupported, any other NSID fails with `Invalid Field in Command`. [PDF p. 230](../_source/pages/page-230.md)

| Health signal | Interpretation |
|---|---|
| Critical Warning | Current bitmask for spare, temperature, reliability, all-media read-only, volatile-backup failure, and PMR read-only/unreliable conditions. |
| Composite Temperature | Implementation-defined Kelvin value representing the controller and associated namespaces, not necessarily a physical sensor location. |
| Available Spare / Threshold | Normalized remaining spare and its asynchronous-event threshold. |
| Percentage Used | Vendor estimate of consumed NVM life; 100 is endurance consumed, not necessarily failure, and values may exceed 100 up to saturation at 255. |

The warning and early health fields above are the portion of Figure 206 that begins in this batch and continues on later pages. They were checked against rendered PDF page 231; later counters are intentionally deferred to the batch that owns them. [PDF p. 231](../_source/pages/page-231.md)

Critical-warning bits represent state when Get Log Page is processed and may differ from the state at the time of a related asynchronous notification. Configured warning classes may generate AER completions. [PDF p. 231](../_source/pages/page-231.md)

## Health accounting and thermal history

```text
instantaneous state              accumulated history
warning bits + temperature  -->  data units / host commands
spare + percentage used          busy time / power cycles / hours
sensor temperatures              unsafe shutdowns / errors / thermal time
```

This explanatory split distinguishes a sampled condition from counters retained across power cycles; it is not a byte-layout reconstruction. [PDF pp. 231-234](../_source/pages/page-231.md)

| Counter family | Interpretation |
|---|---|
| Data Units Read/Written | Count thousands of 512-byte units, rounded up; metadata is excluded. |
| Host Read/Write Commands | Count completed read/write commands requested by the host. |
| Controller Busy Time | Minutes during which the controller is busy processing I/O commands. |
| Power history | Power cycles, power-on hours, and unsafe shutdowns; unsafe shutdown scope follows the shutdown-notification rules. |
| Error history | Media/data-integrity errors and the lifetime number of Error Information log entries. |
| Thermal history | Minutes above warning/critical composite thresholds plus up to eight temperature sensors and thermal-management transition counts/time. |

The SMART counters are large lifetime-style accumulators; sensor values are Kelvin, and an unsupported temperature sensor reports zero. [PDF pp. 232-234](../_source/pages/page-232.md)

## Relationships

- [Completion Queue Entry and Status](completion-queue-entry-and-status.md) supplies the `M` hint and status copied into an Error Information record. [PDF p. 229](../_source/pages/page-229.md)
- [Asynchronous Event Reporting](asynchronous-event-reporting.md) points hosts to error or health detail and controls which SMART warnings notify. [PDF pp. 229, 231](../_source/pages/page-229.md)
- [Log Page Retrieval](log-page-retrieval.md) defines length, offset, event retention, and scope-selection mechanics shared by both logs. [PDF pp. 223-227](../_source/pages/page-223.md)

## Edge cases

- `ECNT=0` denotes an invalid entry, including lost entries or unused capacity; it is not a valid first error identifier. [PDF p. 229](../_source/pages/page-229.md)
- `CSI`, `OPC`, and `SQID` together identify the command and command set only when the Error Information log version makes CSI/OPC valid. [PDF p. 230](../_source/pages/page-230.md)
- Namespace-specific SMART requests in Revision 2.1 do not produce namespace-specific field values, even when the request form is supported. [PDF p. 230](../_source/pages/page-230.md)

## Evidence

- Error history behavior and first half of Figure 205: [PDF p. 229](../_source/pages/page-229.md)
- Figure 205 continuation and SMART scope rules: [PDF p. 230](../_source/pages/page-230.md)
- SMART warning and initial health fields: [PDF p. 231](../_source/pages/page-231.md)
- SMART lifetime counters, shutdown/error history, and thermal sensors: [PDF pp. 232-234](../_source/pages/page-232.md)
