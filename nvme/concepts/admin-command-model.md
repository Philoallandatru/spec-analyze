# Admin Command Model

The Admin Command Set defines commands submitted on the Admin Submission Queue. It combines commands common to every transport with memory-model and message-model command groups, using the common SQE/CQE envelopes plus command-specific dwords. [PDF pp. 191-192](../_source/pages/page-191.md)

## Mental model

```text
Admin Submission Queue
|-- common to all transports: Identify, Features, logs, firmware, namespace, ...
|-- memory-based only: Create/Delete I/O Queues, Doorbell Buffer, virtualization
`-- message-based only: discovery, zoning, exported-resource management
          |
          `-- opcode selects command; DTD selects transfer direction
```

This explanatory taxonomy condenses rendered Figure 141. Fabrics commands share opcode `7Fh` and distinguish their operation inside the Fabrics command format; unlisted Admin opcodes are reserved. [PDF pp. 191-192](../_source/pages/page-191.md)

## Opcode and selector contract

| Aspect | Contract |
|---|---|
| opcode | combines the six-bit function with the two-bit Data Transfer Direction |
| NSID unused | command clears the field to `0h` |
| NSID used | `FFFFFFFFh` is generally supported unless the command's rule or table note narrows it |
| vendor range | combined opcodes `C0h..FFh` |
| transport partition | common commands in 5.1, memory-specific in 5.2, message-specific in 5.3 |

The exact opcode matrix and its per-command exceptions remain in rendered Figure 141 rather than being duplicated as dozens of field pages. Exported-resource management commands in that table are prohibited for controllers in subsystems using only a memory-based transport model. [PDF pp. 191-192](../_source/pages/page-191.md)

Admin processing should remain independent of I/O queue congestion; for example, a full I/O Completion Queue should not stall Delete I/O Submission Queue. [PDF p. 191](../_source/pages/page-191.md)

## Format and sanitize gate

```text
[ordinary Admin command]
        |
        +-- namespace under Format? -- not allowlisted --> may abort: Format In Progress
        |
        `-- sanitize active? -------- not allowlisted --> prohibited / command-specific gate

[Format NVM request]
        `-- conflicting non-allowlisted Admin work --> may abort: Command Sequence Error
```

This explanatory gate summarizes rendered Figure 142. The allowlist keeps recovery/control paths such as Abort, AER, queue deletion/creation, selected logs, Identify, Keep Alive, and selected Fabrics operations available, subject to the table's extra restrictions. Vendor commands are allowed only when they neither affect nor retrieve user data. [PDF pp. 192-193](../_source/pages/page-192.md)

## Relationships

- The common SQE defines opcode, NSID, pointers, and command-specific dwords; the Admin section assigns meanings to the command-specific region. [PDF pp. 155-159, 191](../_source/pages/page-155.md)
- Controller type support remains governed by the separate command support matrix, not merely by an opcode being assigned. [PDF pp. 65-67, 191](../_source/pages/page-065.md)
- Format and sanitize operations temporarily narrow which Admin commands may execute. [PDF pp. 192-193](../_source/pages/page-192.md)

## Evidence

- [Rendered Figure 141, Admin opcode matrix, PDF pp. 191-192](../_source/pages/page-191.md)
- [Rendered Figure 142, Format/Sanitize Admin allowlist, PDF pp. 192-193](../_source/pages/page-192.md)
