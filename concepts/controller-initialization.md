# Controller Initialization

Controller initialization is the transport-specific bootstrap that establishes the Admin path, selects supported controller settings, enables the controller, discovers its command-set and namespace configuration, creates I/O queues, and optionally arms asynchronous events. [PDF pp. 124-127](../_source/pages/page-124.md)

## Mental model

```text
Memory-based (PCIe)                 Message-based (Fabrics)
wait RDY=0                          establish transport connection
configure AQA / ASQ / ACQ           Connect -> Admin queue + association
select CSS / AMS / MPS              authenticate if AUTHREQ != 0
set EN=1; wait RDY=1                Property Get/Set CAP, CC; EN=1; wait RDY=1
          \                         /
           +--> Identify controller and I/O Command Sets
                enumerate active namespaces
                negotiate/create I/O queues
                enable events + submit AERs
```

This explanatory comparison combines the two normative sequences and the rendered Figure 82 queue-creation ladder. [PDF pp. 124-126](../_source/pages/page-124.md)

## Common capability-driven sequence

The host derives `CC.CSS` from `CAP.CSS`: select no-I/O-command-set mode with `111b`, an I/O Command Set Combination with `110b`, or the NVM Command Set with `000b` when its capability conditions apply. It also chooses a supported arbitration mechanism and memory page size before setting `CC.EN=1`, then waits for `CSTS.RDY=1`. [PDF pp. 124-126](../_source/pages/page-124.md)

After readiness, Identify Controller precedes command-set-specific discovery. If combinations are supported, the host identifies them and selects a profile. For each enabled I/O Command Set it enumerates active NSIDs and reads the applicable command-set-specific and independent Identify structures. It then determines queue resources, creates Completion Queues before Submission Queues for memory-based transports or uses Fabrics Connect for queue pairs, and may configure asynchronous events at any point after readiness. [PDF pp. 125-127](../_source/pages/page-125.md)

## Transport-specific bootstrap

| Stage | Memory-based controller | Message-based controller |
|---|---|---|
| Admin path | Program `AQA`, `ASQ`, and `ACQ` while disabled | Establish connection; Connect creates Admin pair and association |
| Properties | Binding-defined register access | Fabrics Property Get and Property Set |
| Authentication | Not part of this sequence | Required before normal initialization when `AUTHREQ != 0` |
| I/O queues | Number of Queues, interrupt setup, Create CQ then Create SQ | Number of Queues plus `CAP.MQES`, then Connect each I/O pair |

Figure 82 shows that a successful Connect creates the Admin or I/O queue before any required in-band authentication exchange. The association may be removed if the host does not set `CC.EN=1` within two minutes of establishing it. [PDF pp. 125-127](../_source/pages/page-125.md)

## Discovery controller initialization

```text
Connect received
  |-- change notifications + non-zero KATO
  |      -> controller may adjust KATO, pre-allocates AER/AEN resources
  |      -> successful Connect -> host submits AERs
  `-- otherwise
         -> fixed KATO (or Connect error for unsupported notification request)
         -> host completes connection without AERs
```

This explanatory reduction of Figure 83 preserves its decisive branches: notification support and a non-zero Keep Alive Timeout determine whether the controller reserves asynchronous-event resources. After a successful Connect, the host authenticates if required, reads capabilities, writes `CC` including `EN=1`, waits for `RDY=1`, identifies applicable Controller structures, and then reads the Discovery Log Page. [PDF p. 128](../_source/pages/page-128.md)

## Evidence

- [Memory-based initialization start, PDF p. 124](../_source/pages/page-124.md)
- [Memory-based completion and Fabrics bootstrap, PDF p. 125](../_source/pages/page-125.md)
- [Rendered Figure 82 and Fabrics initialization start, PDF p. 126](../_source/pages/page-126.md)
- [Fabrics initialization continuation and timeout, PDF p. 127](../_source/pages/page-127.md)
- [Rendered Figure 83 and Discovery initialization sequence, PDF p. 128](../_source/pages/page-128.md)
