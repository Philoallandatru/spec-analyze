# Association and Command Lifecycle

An association is the exclusive communication relationship between one host and one controller; it encompasses that controller's Admin Queue and all its I/O Queues. Within it, commands move through submission, readiness, processing, and completion. [PDF pp. 28-29](../_source/pages/page-028.md)

## Mental model

```text
Host H
`-- exclusive Association with Controller C
    |-- Admin Queue pair (identifier 0)
    `-- I/O Queue pair(s)

Host builds command
        |
        +-- memory-based: SQ Tail Doorbell advances past command slot
        `-- message-based: host adds command capsule to SQ
                             |
                         [submitted]
                             |
                  transferred and ready
                             v
                         [candidate]
                             |
                         processed
                             v
            CQE status updated and CQE posted
                             v
                         [completed]
```

[PDF pp. 28-29](../_source/pages/page-028.md)

## Contract and invariants

For Fabrics, connecting the Admin Queue establishes the association. Subsequent I/O Queue connections must use the same subsystem port, transport type/address, Host NQN, Subsystem NQN, Controller ID, and compatible Host Identifier. A controller has at most one association, and only that host may connect its I/O Queues. [PDF p. 58](../_source/pages/page-058.md)

```text
[unassociated]
      |
      | Connect Admin Queue
      v
[associated with exactly one host]
      |-- same host may Connect I/O Queues
      |
      +-- shutdown / Controller Level Reset / Admin connection loss --> [unassociated]
      `-- I/O connection loss without individual deletion support ----> [unassociated]
```

This explanatory lifecycle includes every association-termination condition listed in the architecture. There is no explicit NVMe command that terminates the association; Disconnect deletes an I/O Queue. [PDF pp. 58-59](../_source/pages/page-058.md)

The host requests dynamic allocation by using Controller ID `FFFFh` in Fabrics Connect. Static allocation is expected to be durable but may be reclaimed while unused for implementation-specific reasons. [PDF p. 59](../_source/pages/page-059.md)

Admin Queue Connect creates both the queue pair and host/controller association. A dynamic subsystem requires `CNTLID=FFFFh` and returns any available controller; a static subsystem may accept a specific controller or `FFFEh` for any available controller, but rejects `FFFFh`. Successful allocation never returns sentinel values `FFF0h` through `FFFFh`. [PDF pp. 473-474](../_source/pages/page-473.md)

Before any I/O Queue Connect, the Admin association must exist and the controller must be enabled. Each I/O connection must match the Admin Queue's Host NQN, Subsystem NQN, Controller ID, and, when non-zero, Host Identifier; using a duplicate Queue ID is a sequence error. A different port/transport/address may route to another subsystem or fail before NVMe receives the command. [PDF p. 474](../_source/pages/page-474.md)

The Admin Connect sets the current Host Identifier and Keep Alive Timeout. The I/O Connect may additionally associate its SQ with an advertised NVM Set; its identity fields otherwise inherit the established association contract. [PDF pp. 476-477](../_source/pages/page-476.md)

Completion is not merely the end of execution: processing must be finished, status must be updated in the Completion Queue Entry, and that entry must be posted to the associated Completion Queue. [PDF p. 29](../_source/pages/page-029.md)

In NVMe over Fabrics, a capsule is the exchange unit containing a command or response and may additionally contain data and SGLs. [PDF p. 29](../_source/pages/page-029.md)

For message-based controllers, the transport connection exists before Connect. A successful Admin Queue Connect creates the association; subsequent I/O Queue Connect commands must preserve the transport and host/subsystem identity established by that Admin Queue. If Connect requires in-band authentication, the created queue accepts only Authentication Send/Receive until authentication succeeds. [PDF pp. 114-115](../_source/pages/page-114.md)

Individual I/O Queue deletion is not automatically association deletion. When both endpoints support it, Disconnect affects only its I/O queue pair; otherwise queue deletion or transport-connection termination ends the association. Transport loss with outstanding commands requires communication-loss recovery rather than blind resubmission. [PDF pp. 115-117](../_source/pages/page-115.md)

An idempotent command has the same subsystem end state and returns the same result when resubmitted one or more times with no intervening commands. The no-intervening-command condition is part of the definition and is essential when reasoning about retry after communication loss. [PDF p. 31](../_source/pages/page-031.md)

## Evidence

- [Association and candidate-command definitions, PDF p. 28](../_source/pages/page-028.md)
- [Capsule, submission, and completion definitions, PDF p. 29](../_source/pages/page-029.md)
- [Idempotent command definition, PDF p. 31](../_source/pages/page-031.md)
- [Fabrics association establishment and queue-connection identity, PDF p. 58](../_source/pages/page-058.md)
- [Association termination and controller allocation requests, PDF p. 59](../_source/pages/page-059.md)
- [Message-based Connect identity, authentication, and I/O queue deletion scope, PDF pp. 114-117](../_source/pages/page-114.md)
- [Connect controller allocation and identity validation, PDF pp. 473-477](../_source/pages/page-473.md)
