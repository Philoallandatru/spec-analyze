# Keep Alive

Keep Alive is a per-controller watchdog for detecting communication failure between a host and controller. `KATT` is the controller's current Keep Alive Timeout Time; the transport binding defines allowed bounds and whether the feature must be enabled. A controller advertising non-zero `KAS` supports the timer and the Keep Alive command. [PDF pp. 146-147](../_source/pages/page-146.md)

## Mental model

```text
inactive -- enable + ready + no shutdown + KATO>0 --> active
   ^                                                   |
   |                                                   | activity rule depends on mode
   |                                                   v
   +---- disable / not-ready / shutdown -------- [watch interval]
                                                       |
                                        no qualifying activity by deadline
                                                       v
                                                 [timeout cleanup]
```

This explanatory state view combines the activation predicates with the two watchdog modes; it is not a numbered specification figure. [PDF pp. 147-149](../_source/pages/page-147.md)

## Configuration and activation

A host sets `KATO` in Fabrics Connect or with Set Features. A required timer cannot be disabled with zero, and a value above a transport maximum is rejected with Keep Alive Timeout Invalid without changing the controller value. A non-zero value below the applicable minimum is raised to that minimum, then rounded up to the `KAS` granularity; Get Features returns the effective value. [PDF p. 147](../_source/pages/page-147.md)

The Set Features encoding expresses `KATO` in milliseconds. Transports not requiring the timer default to zero; transports such as RDMA and TCP default to 120,000 ms, rounded up to `KAS`. The applicable transport binding owns any additional requirements. [PDF p. 403](../_source/pages/page-403.md)

The controller timer is active only while `CC.EN=1`, `CSTS.RDY=1`, `CC.SHN=00b`, `CSTS.SHST=00b`, and effective `KATO` is non-zero. Activation initializes it to `KATT`; while active, the host should preserve Admin Submission Queue space for a Keep Alive command. [PDF p. 147](../_source/pages/page-147.md)

## Watchdog modes

| Dimension | Command Based (`TBKAS=0`) | Traffic Based (`TBKAS=1`) |
|---|---|---|
| Controller evidence of liveness | successful Keep Alive completion | any Admin or I/O command fetched during an interval |
| Controller deadline | expire after `KATT` since last start/restart | inspect successive `KATT` intervals |
| Host cadence guidance | send at `KATT/2` | check at `KATT/4`; ordinary completed traffic may suppress a Keep Alive |
| Worst case after last fetched ordinary command | approximately `KATT` under the command-only rule | up to `2 * KATT` |

Command Based Keep Alive restarts the controller timer after a successful Keep Alive completion or successful non-zero KATO Set Features completion. The host may use command-based behavior regardless of the controller mode and should treat a missing Keep Alive completion by `KATT` after submission as a host-detected timeout. [PDF pp. 147-149](../_source/pages/page-147.md)

Traffic Based Keep Alive is usable by the host only when the controller also uses it. At each controller interval boundary, any fetched Admin or I/O command prevents timeout and starts another interval; otherwise timeout occurs. Because a command can be fetched just after one check, Figure 90 shows detection may wait until the following check, up to `2 * KATT`. [PDF pp. 149-150](../_source/pages/page-149.md)

```text
check 1       last command       check 2                check 3
   |--------------- KATT ------------|------- KATT ---------|
                       fetched        traffic seen:          no traffic:
                                      continue               timeout
```

This compact timeline was reconstructed from rendered Figure 90; it preserves the two adjacent `KATT` windows that create the worst case. [PDF p. 150](../_source/pages/page-150.md)

## Timeout cleanup

Within `CQT` after a controller-detected timeout, the controller records an Error Information Log entry with Keep Alive Timeout Expired, stops command processing, and sets `CSTS.CFS=1`. A message-based controller additionally terminates the association's transport connections and breaks the association, after which a new Admin Queue Connect may form another association. [PDF p. 151](../_source/pages/page-151.md)

A host that detects timeout while commands remain outstanding should follow the communication-loss recovery rules before submitting dependent work through another controller, because the earlier commands may still interact with later commands. [PDF pp. 148, 150-151](../_source/pages/page-148.md)

## Relationships

- Keep Alive runs on a controller and, in command mode, uses its Admin Queue. [PDF pp. 146-147](../_source/pages/page-146.md)
- Timer activation is coupled to controller enable/readiness and suppressed during shutdown. [PDF p. 147](../_source/pages/page-147.md)
- Controller-detected timeout becomes a fatal-status and error-recovery event; for Fabrics it also destroys the association. [PDF p. 151](../_source/pages/page-151.md)

## Evidence

- [Capability, transport policy, configuration, and activation, PDF pp. 146-147](../_source/pages/page-146.md)
- [Command Based controller and host behavior, PDF pp. 147-149](../_source/pages/page-147.md)
- [Traffic Based behavior and rendered Figure 90, PDF pp. 149-150](../_source/pages/page-149.md)
- [Timeout cleanup, PDF p. 151](../_source/pages/page-151.md)
