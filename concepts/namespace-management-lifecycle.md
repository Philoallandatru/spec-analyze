# Namespace Management Lifecycle

Namespace Management creates and deletes allocated namespaces, while Namespace Attachment independently attaches or detaches an existing namespace to I/O controllers. These are persistent control-plane operations rather than I/O queue operations. [PDF pp. 383-387](../_source/pages/page-383.md)

## Mental model

```text
[unallocated NSID]
       |
       | Namespace Management: Create
       v
[allocated, unattached namespace]
       |          ^
 Attach|          |Detach
       v          |
[active on selected controller(s)]
       |
       | Namespace Management: Delete
       v
[deleted + detached from every controller]
```

This explanatory lifecycle distinguishes allocation from per-controller attachment. Create does not attach; delete removes the namespace and therefore detaches it everywhere. [PDF pp. 383-386](../_source/pages/page-383.md)

## Attachment contract

Attach and detach consume a 4 KiB Controller List and persist across every reset and Virtualization Management operations that take a secondary controller offline. An attachment target must be an I/O controller that supports and has enabled the namespace's I/O Command Set. [PDF pp. 383-385](../_source/pages/page-383.md)

| Gate | Failure |
|---|---|
| Domain aggregate exceeds `MAXDNA` | Namespace Attachment Limit Exceeded |
| target controller exceeds `MAXCNA` | Namespace Attachment Limit Exceeded |
| command set unsupported / profile-disabled | I/O Command Set Not Supported / Not Enabled |
| private namespace already attached elsewhere | Namespace Is Private |
| Administrative controller or malformed list | Controller List Invalid |
| attachment would produce disallowed ANA condition | ANA Attach Failed |

Processing stops at the first failing Controller List entry; the Error Information Log records that entry's byte offset, so earlier entries may already have been processed. [PDF pp. 384-385](../_source/pages/page-384.md)

## Create and delete contract

Namespace Management support implies Namespace Attachment support and is prohibited for controllers in an Exported NVM subsystem. I/O commands continue executing while management is in progress. [PDF p. 385](../_source/pages/page-385.md)

Create uses `NSID=0`, a selected Command Set Identifier, a command-set-specific 512-byte parameter region, and optional vendor data; the controller chooses and returns the new NSID. Delete names one created namespace, while `NSID=FFFFFFFFh` deletes all namespaces and succeeds even when none exist. [PDF pp. 385-387](../_source/pages/page-385.md)

Deletion implicitly detaches the namespace from all controllers. Hosts should detach first; otherwise attached-namespace change events are generated, and every successful deletion also produces allocated-namespace change reporting. [PDF p. 385](../_source/pages/page-385.md)

Create may fail for unsupported format, insufficient capacity, exhausted NSID capacity, unsupported thin provisioning, invalid ANA Group, or unsupported I/O Command Set. For insufficient capacity, the Error Information Log reports the additional unallocated bytes required. [PDF pp. 386-387](../_source/pages/page-386.md)

## Relationships

- [Namespace Identifiers](namespace-identifiers.md) separates allocation, attachment, and durable namespace identity. [PDF pp. 97-99](../_source/pages/page-097.md)
- [NVM Capacity Model](nvm-capacity-model.md) supplies the unallocated capacity consumed by namespace creation. [PDF pp. 140-143](../_source/pages/page-140.md)
- [Namespace Topology and Change Logs](namespace-topology-and-change-logs.md) exposes attach/detach/delete changes to hosts. [PDF pp. 288-293](../_source/pages/page-288.md)

## Evidence

- [Namespace Attachment persistence and command envelope, PDF p. 383](../_source/pages/page-383.md)
- [Attachment limits, command-set gates, and first-error behavior, PDF p. 384](../_source/pages/page-384.md)
- [Attachment statuses and Namespace Management lifecycle, PDF p. 385](../_source/pages/page-385.md)
- [Create structure and failure information, PDF p. 386](../_source/pages/page-386.md)
- [Created NSID completion value, PDF p. 387](../_source/pages/page-387.md)
