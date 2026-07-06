# Controller Enable, Shutdown, and Reset

The `CC` and `CSTS` properties form a host/controller handshake for configuration, enablement, readiness, shutdown, and reset. Queue configuration is legal only in specific disabled states, and shutdown completion changes which reset is required before command execution may resume. [PDF pp. 79-85](../_source/pages/page-079.md)

## Enable/disable state machine

```text
                 configure CC + AQA/ASQ/ACQ
                            |
                            v
[Disabled] CC.EN=0, CSTS.RDY=0
     |  set CC.EN=1
     v
[Enabling] ---------------- wait CAP.TO / CRTO ----------------+
     | CSTS.RDY=1                                           timeout
     v                                                       |
[Ready] commands may be submitted                            v
     | clear CC.EN=0                                  error handling
     v
[Controller Reset / Disabling]
     | I/O queues deleted; Admin queues reset
     | CSTS.RDY=0
     `---------------------------> [Disabled]
```

The host must not submit commands after enabling until `CSTS.RDY=1`. Repeating an enable while already ready, or a disable while already not ready, has undefined results. [PDF pp. 82, 84](../_source/pages/page-082.md)

## CC configuration fields

```text
31:25   24      23:20   19:16  15:14  13:11  10:07  06:04  03:01  00
+-----+-------+-------+-------+------+-------+------+-------+------+--+
|  R  | CRIME |IOCQES |IOSQES | SHN  | AMS   | MPS  | CSS   |  R   |EN|
+-----+-------+-------+-------+------+-------+------+-------+------+--+
```

- `CRIME` chooses ready-with-media or ready-independent-of-media when supported.
- Queue entry sizes are powers of two and become unsafe to change after corresponding I/O queues exist.
- `AMS`, `MPS`, and `CSS` may be changed only while disabled and must be supported by `CAP`.
- `EN` controls command processing and initiates Controller Reset when cleared from `1` to `0`. [PDF pp. 79-82](../_source/pages/page-079.md)

## Status fields

```text
CSTS: [ ST | PP | NSSRO | SHST(2b) | CFS | RDY ]
        |    |     |        |          |     `-- command-ready handshake
        |    |     |        |          `-------- fatal status
        |    |     |        `------------------- shutdown progress
        |    |     `---------------------------- subsystem reset observed
        |    `---------------------------------- processing temporarily paused
        `--------------------------------------- controller vs subsystem shutdown
```

`SHST` is `00b` for normal operation, `01b` while shutdown is in progress, and `10b` when complete. `ST=0` means this status describes controller shutdown; `ST=1` means NVM Subsystem Shutdown. [PDF pp. 82-84](../_source/pages/page-082.md)

## Shutdown paths

```text
Normal:  CC.SHN=01b --> SHST=01b --> SHST=10b
Abrupt:  CC.SHN=10b --> SHST=01b --> SHST=10b

ST=0 after completion -> Controller Level Reset + enable
ST=1 after completion -> NVM Subsystem Reset + enable
```

Clearing `CC.EN` while issuing shutdown can cause both shutdown and Controller Reset. A subsystem shutdown in progress or complete prevents a normal enable from making the controller ready until an NVM Subsystem Reset occurs. [PDF pp. 80, 82-84](../_source/pages/page-080.md)

## NVM Subsystem Reset trigger

If `CAP.NSSRS=1`, writing `4E564D65h` (ASCII `NVMe`) to `NSSR.NSSRC` initiates an NVM Subsystem Reset; any other write has no functional effect, and reads return zero. [PDF p. 85](../_source/pages/page-085.md)

## NVM Subsystem Shutdown property

```text
write NSSD.NSSC = 4E726D6Ch ("Nrml") -> normal subsystem shutdown
write NSSD.NSSC = 41627074h ("Abpt") -> abrupt subsystem shutdown
other value                            -> no functional effect
```

The affected scope is the controller's Domain when `CAP.CPS=10b`, or the entire NVM subsystem when `CAP.CPS=11b`. Reads return zero. [PDF p. 91](../_source/pages/page-091.md)

`CRTO.CRIMT` bounds readiness for commands that do not need media in ready-independent mode; `CRTO.CRWMT` bounds full readiness including attached namespaces/media and is greater than or equal to `CRIMT`. Both use 500 ms units. [PDF p. 92](../_source/pages/page-092.md)

See [Controller Ready Modes and Timeouts](controller-ready-modes.md) for the initialization-time mapping among `CAP.CRMS`, `CC.CRIME`, `CAP.TO`, and both `CRTO` deadlines. [PDF pp. 128-131](../_source/pages/page-128.md)

## Shutdown processing contract

```text
Normal operation (SHST=00b)
       | CC.SHN=01b / 10b, or NSSD request
       v
Shutdown in progress (SHST=01b) -- may abort commands for PLN
       |
       v
Shutdown complete (SHST=10b) --> ready to power off

Controller shutdown: ST=0
Subsystem shutdown:  ST=1 (overrides controller shutdown)
```

Only one shutdown mechanism is active for a controller; an NVM Subsystem Shutdown request overrides an in-progress controller shutdown. `CC.EN/CSTS.RDY` may remain either `0/0` or `1/1` while `SHST` reports the media shutdown state, so enablement and shutdown status are related but separate axes. [PDF pp. 132-133](../_source/pages/page-132.md)

For a normal memory-based shutdown, the host stops new I/O, lets outstanding commands complete, deletes Submission Queues before Completion Queues, writes `CC.SHN=01b`, and waits for `CSTS.ST=0, SHST=10b`. Abrupt shutdown skips queue draining and uses `CC.SHN=10b`. Entry to PCIe D3 follows the normal path; disabling via `CC.EN` during shutdown is discouraged because that initiates Controller Reset. [PDF p. 133](../_source/pages/page-133.md)

For Fabrics, Property Set writes `CC.SHN`; the host may disconnect at the transport or poll `CSTS.SHST`, but the controller does not initiate that disconnect. Until Controller Level Reset or dynamic-controller removal, only Fabrics commands are processed and Keep Alive is disabled. After `CC.EN` becomes zero, the association is retained for at least two minutes. [PDF pp. 134-135](../_source/pages/page-134.md)

After controller shutdown completes, command execution resumes only through the specified reset/re-enable path, or—when `CC.EN=0`—a single write that sets `EN=1` and clears `SHN=00b`, followed by initialization. Loss of communication before all completions or shutdown-complete status requires the communication-loss precautions because outstanding commands may interact with later commands through another controller. [PDF pp. 134-135](../_source/pages/page-134.md)

## NVM Subsystem Shutdown and reset scope

```text
NSSD normal / abrupt request
           |
           v
[ST=1, SHST=01b: subsystem/domain shutdown in progress]
           | wait reported latency (0h -> at least 30 s)
           v
[ST=1, SHST=10b on every controller in affected scope]
           | affected scope is ready to power off
           v
NVM Subsystem Reset -> abort shutdown, clear ST/SHST, reset all scoped controllers
```

This explanatory state sequence was checked against rendered PDF pages 135-139. For `CAP.CPS=11b`, the affected scope is the whole single-Domain subsystem; for `CAP.CPS=10b`, it is the initiating controller's Domain. Completion may first be observed on any controller only when the whole affected scope is ready, and is then reported on all controllers in that scope. [PDF pp. 135-139](../_source/pages/page-135.md)

| Condition | Controller Level Reset behavior during subsystem shutdown |
|---|---|
| `CAP.NSSES=1` | Controller Reset still initiates CLR; a non-subsystem-reset CLR preserves `CSTS.ST/SHST` |
| `CAP.NSSES=0` | Controller Reset is disabled while shutdown is reported; a transport-specific CLR after completion may clear `ST/SHST` |

Portable host behavior is to follow every NVM Subsystem Shutdown with an NVM Subsystem Reset. That reset aborts an in-progress shutdown, clears `ST/SHST` throughout its scope, initiates Controller Level Reset on every scoped controller, disables their PMRs, and performs transport-specific reset actions. A single-Domain reset covers the subsystem; in a multi-Domain subsystem it may cover one Domain or the whole subsystem. [PDF pp. 135-140](../_source/pages/page-135.md)

## Reset levels

| Reset level | Initiation | Principal effects |
|---|---|---|
| NVM Subsystem Reset | power application, supported `NSSR.NSSRC` write, management method, or vendor event | resets subsystem/Domain scope, initiates CLR on all scoped controllers, disables scoped PMRs |
| Controller Level Reset | subsystem reset, `CC.EN: 1 -> 0`, or transport-specific reset | stops commands, deletes all I/O queues, returns controller idle with `RDY=0`, resets properties/state subject to defined exceptions |
| Queue Level Reset | delete then recreate the I/O queue or pair | changes queue attributes without resetting the controller |

For a Controller Reset on a memory-based controller, Admin Queue and PMR properties plus `CMBMSC` survive; a Function Level Reset preserves only `CMBMSC`; message-based controllers have no CLR property exceptions. After CLR, the host restores transport/property state, enables and waits for readiness, reconfigures with Admin commands, and recreates I/O queues. [PDF pp. 139-141](../_source/pages/page-139.md)

## Admin Queue bootstrap

```text
while CC.EN=0:
  AQA <- ACQ size | ASQ size        (0's based, 2..4096 entries)
  ASQ <- page-aligned physical base
  ACQ <- page-aligned physical base (interrupt vector 0)
  CC  <- valid entry sizes / modes / EN=1
  wait CSTS.RDY=1
```

The Admin queues are physically contiguous and have Queue Identifier zero. `AQA`, `ASQ`, and `ACQ` survive Controller Reset and may be modified only while disabled. [PDF pp. 82, 85-86](../_source/pages/page-082.md)

## Evidence

- [Controller Configuration fields, PDF pp. 79-82](../_source/pages/page-079.md)
- [Controller Status fields, PDF pp. 82-84](../_source/pages/page-082.md)
- [NVM Subsystem Reset and Admin Queue attributes, PDF p. 85](../_source/pages/page-085.md)
- [Admin Completion Queue and CMB location start, PDF p. 86](../_source/pages/page-086.md)
- [NVM Subsystem Shutdown and Controller Ready Timeouts, PDF pp. 91-92](../_source/pages/page-091.md)
- [Shutdown interaction matrix, PDF pp. 132-133](../_source/pages/page-132.md)
- [Memory-based shutdown and restart rules, PDF pp. 133-134](../_source/pages/page-133.md)
- [Fabrics shutdown and association lifetime, PDF pp. 134-135](../_source/pages/page-134.md)
- [Subsystem/Domain shutdown scope and completion, PDF pp. 135-138](../_source/pages/page-135.md)
- [NVM Subsystem and Controller Level Reset contracts, PDF pp. 139-141](../_source/pages/page-139.md)
