# Migration Change Tracking

Migration change tracking is the paired Track Send/Track Receive control plane for recording either namespace user-data changes in a User Data Migration Queue or host-memory writes caused by a selected controller. [PDF pp. 433-438](../_source/pages/page-433.md)

## Mental model

```text
management controller                       target controller
Track Send: start ranges ------------------> command processing writes host memory
        |                                                   |
        `------ tracking state records changed ranges <-----'
                                |
Track Receive <-----------------+  returns descriptors; successful read consumes them
        |
        `-- MTR=1: repeat receive
            MTR=0,SUSP=1: complete while target remains suspended
```

This explanatory sequence preserves the separation between the controller processing tracking commands and the controller whose writes are observed, plus the destructive-read and suspension termination rules. [PDF pp. 434-438](../_source/pages/page-434.md)

## Two tracking modes

| Track Send operation | Observed change | Destination / retrieval |
|---|---|---|
| Log User Data Changes (`SEL=0h`) | user-data modifications to namespaces attached to the controller associated with a selected User Data Migration Queue | controller posts I/O-command-set-defined entries to that CDQ |
| Track Memory Changes (`SEL=1h`) | writes into selected host-memory ranges caused by commands processed by a selected controller | Track Receive `SEL=0h` returns changed-range descriptors |

The first mode is started or stopped by `LACT` and identifies a controller-local `CDQID`; the second is started or stopped by `TACT` and identifies a different target `CNTLID`. [PDF pp. 436-438](../_source/pages/page-436.md)

## Host-memory tracking contract

Starting host-memory tracking supplies one or more aligned ranges, a requested granularity encoded as `2^n * 4 KiB`, and a descriptor count. The requested granularity must lie within the Identify Controller minimum/maximum bounds, while controller-local and subsystem-wide descriptor totals must stay within their advertised limits. If the optional length-limit rule is active, each requested length must be a power of two. [PDF pp. 437-438](../_source/pages/page-437.md)

The processing controller cannot track itself, cannot start tracking a suspended target, and rejects a target already tracked by any controller in the subsystem. Clearing `TACT` stops tracking and causes the data buffer to be ignored. [PDF p. 438](../_source/pages/page-438.md)

The request structure uses version zero, a one-based descriptor count, and 12-byte descriptors whose page-granularity-aligned address and length identify host-memory addresses used by the target controller's PRPs or SGLs. Misalignment or a range outside that target's associated host memory is invalid. [PDF p. 439](../_source/pages/page-439.md)

If the controller cannot satisfy the requested count or granularity, it returns Not Enough Resources together with the granularity and descriptor count it can support. Later reconfiguration that changes an accepted range into non-host memory makes subsequent tracking behavior undefined; MSI/MSI-X register ranges should not be requested because reporting those writes is optional. [PDF pp. 439-440](../_source/pages/page-439.md)

## Receive and consumption semantics

Track Receive selects Tracked Memory Changes and uses a zero-based requested dword count. A successful receive removes the changes actually reported; new reports cover changes since tracking began or since the previous successful report while tracking remained enabled. High-frequency changes to one address may be delayed to permit fair reporting of other addresses. [PDF pp. 433-434](../_source/pages/page-433.md)

The returned header carries target controller identity, reported granularity, descriptor count, `MTR`, and whether the target was suspended while the command was processed. `MTR=1` requires another receive. `MTR=0` means no more changes were available at processing time; together with `SUSP=1`, it establishes that no more tracked changes can appear until the target resumes. [PDF pp. 434-435](../_source/pages/page-434.md)

Each returned 16-byte descriptor gives a granularity-aligned start address and a length in reported-granularity units. A zero descriptor count requires `MTR=0`. [PDF p. 435](../_source/pages/page-435.md)

## User-data logging gates

Starting User Data Migration Queue logging fails if the selected queue does not exist on the processing controller, if any controller is already logging for the queue-associated target, or if that target is suspended. The selected I/O Command Set owns the posted-entry format and additional posting requirements. [PDF pp. 436-437](../_source/pages/page-436.md)

## Relationships

- [Controller Migration](controller-migration.md) supplies the suspension boundary that makes a final `MTR=0` report conclusive. [PDF pp. 434-435](../_source/pages/page-434.md)
- [Controller Data Queues](controller-data-queues.md) owns User Data Migration Queue creation, memory lifetime, and controller-local identifiers. [PDF pp. 436-437](../_source/pages/page-436.md)
- [Identify Command Model](identify-command-model.md) advertises memory-range granularity and descriptor capacity limits consumed when tracking starts. [PDF pp. 363-364, 437-438](../_source/pages/page-363.md)

## Evidence

- [Track Receive envelope, PDF p. 433](../_source/pages/page-433.md)
- [Destructive reporting and suspension interpretation, PDF pp. 434-435](../_source/pages/page-434.md)
- [Track Send operations and User Data Migration Queue logging, PDF pp. 436-437](../_source/pages/page-436.md)
- [Host-memory tracking resource and state gates, PDF pp. 437-438](../_source/pages/page-437.md)
- [Request descriptors, negotiated fallback, and address stability, PDF pp. 439-440](../_source/pages/page-439.md)
