# Log Page Retrieval

Get Log Page is the Admin command for reading typed controller, namespace, Domain, Endurance Group, or NVM-subsystem information. The selected Log Page Identifier (LID) determines both the returned structure and its scope; unsupported LIDs fail with `Invalid Log Page`, subject to the command-specific exception referenced by the specification. [PDF p. 223](../_source/pages/page-223.md) [PDF pp. 226-227](../_source/pages/page-226.md)

## Mental model

```text
Host                         Controller interface                 Log object
 |                                   |                               |
 |-- Get Log Page ------------------>|                               |
 |   LID + scope selector            |-- select supported LID ------>|
 |   transfer length                 |-- choose byte/index start --->|
 |   byte offset or index            |<-- requested dword range -----|
 |<---------------- data buffer -----|                               |
 |   successful RAE=0 may clear the corresponding async event       |
```

This is an explanatory command flow, not a reconstruction of a numbered figure. [PDF pp. 223-225](../_source/pages/page-223.md)

## Contract and invariants

| Concern | Contract |
|---|---|
| Transfer size | `NUMDU:NUMDL` is a 0's-based dword count; requesting beyond the end returns the complete log plus undefined trailing dwords. |
| Event retention | Successful completion with `RAE=0` clears the corresponding asynchronous event; `RAE=1` retains it, and unsuccessful completion always retains it. |
| Byte offset | `OT=0`; `LPOU:LPOL` is a byte offset and is treated as dword aligned. An offset beyond the log fails with `Invalid Field in Command`. |
| Index offset | `OT=1`; `LPOU:LPOL` is an entry index and is legal only when the selected LID advertises `IOS=1`. |
| Command-set selection | `CDW14.CSI` is used only by LIDs that support it and when `CC.CSS=110b`; otherwise it is ignored. |
| Vendor UUID | `CDW14.UIDX` selects UUID-qualified information only where that log supports UUID selection. |

The command uses the common Data Pointer plus CDW10 through CDW14; all other command-specific fields are reserved. The table summarizes the rendered command layouts rather than replacing their exact bit definitions. [PDF pp. 223-225](../_source/pages/page-223.md)

## Supported-log discovery

The Supported Log Pages log (`LID 00h`) contains 256 four-byte LID Supported and Effects entries, one for every possible LID on the interface where the command was submitted. Different Admin Queue or management-endpoint instances may expose different sets. [PDF pp. 227-228](../_source/pages/page-227.md)

```text
Supported Log Pages (1024 bytes)
+---------+---------+-----+-----------+
| LID 00h | LID 01h | ... | LID FFh   |
+---------+---------+-----+-----------+
    each 32-bit entry:
    31             16 15          2 1   0
   +----------------+--------------+---+---+
   | LID-specific   |   reserved   |IOS|SUP|
   +----------------+--------------+---+---+
```

This explanatory layout is reconstructed from Figures 203 and 204 and was checked against rendered PDF page 228. If `SUP=0`, the host should ignore the rest of that entry. `IOS` must be zero when extended Get Log Page data is unsupported. [PDF p. 228](../_source/pages/page-228.md)

For controllers with I/O Queues, supported logs may depend on the I/O Command Set selected in `CC.CSS`, or on `CDW14.CSI` when `CC.CSS=110b`. UUID selection may further qualify the reported set. [PDF p. 227](../_source/pages/page-227.md)

## Scope and selectors

Scope belongs to the log definition: subsystem-scoped data is global to the subsystem, Domain-scoped data to the Domain, controller-scoped data to the processing controller, and namespace-scoped data to the specified namespace. An individual field may explicitly define a narrower or different scope. [PDF pp. 226-227](../_source/pages/page-226.md)

For controller- or subsystem-scoped logs, an NSID other than `0h` or `FFFFFFFFh` is invalid. Multi-scope logs resolve their effective scope using Domain support or the supplied NSID according to their own definition. [PDF p. 227](../_source/pages/page-227.md)

## Relationships

- [Asynchronous Event Reporting](asynchronous-event-reporting.md) uses log retrieval both to obtain event detail and, normally with `RAE=0`, to clear the event mask. [PDF p. 224](../_source/pages/page-224.md)
- [Feature Values and Scope](feature-values-and-scope.md) configures event classes and other behavior whose state is later observed through logs. [PDF pp. 223-224](../_source/pages/page-223.md)
- [Error and Health Logs](error-and-health-logs.md) are concrete examples of controller/global and controller-or-namespace scope. [PDF pp. 229-230](../_source/pages/page-229.md)

## Edge cases

- A byte offset with low bits set may be rejected, but a controller that accepts it must behave as though bits 1:0 were zero. [PDF p. 225](../_source/pages/page-225.md)
- An index beyond the number of structures, or use of index mode when `IOS=0`, fails with `Invalid Field in Command`. [PDF p. 225](../_source/pages/page-225.md)
- Log support is interface-instance-specific; discovery on one interface must not be assumed to describe another. [PDF p. 227](../_source/pages/page-227.md)

## Evidence

- Command behavior and completion: [PDF p. 223](../_source/pages/page-223.md)
- Data pointer, length, RAE, LID, and log-specific selectors: [PDF p. 224](../_source/pages/page-224.md)
- Offset, offset type, CSI, and UUID index: [PDF p. 225](../_source/pages/page-225.md)
- LID registry and scopes: [PDF pp. 226-227](../_source/pages/page-226.md)
- Supported Log Pages structure: [PDF pp. 227-228](../_source/pages/page-227.md)
