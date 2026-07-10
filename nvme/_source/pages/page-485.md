---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 485
headings: ["7.2 Flush command", "7.2.1 Command Completion"]
---

# Source page 485

NVM Express® Base Specification, Revision 2.1

463
Figure 559: Cancel – Completion Queue Entry Dword 0
Bits Description
31:16
Commands Eligible for Deferred Abort (CEDA): This field indicates the number of commands that match
the specified criteria (refer to Figure 556 and Figure 557) on which a deferred abort may be performed.  A
value of 0h indicates that no commands are eligible for a deferred abort or that the controller does not
support deferred aborts. A value of FFFFh indicates that FFFFh or more commands are eligible for a
deferred abort.
If the Cancel Action (refer to Figure 557) specified a Single Command Cancel and an immediate abort was
performed on the specified command, then this field shall be cleared to 0h.
15:00
Commands Aborted  (CMDA): This field indicates the number of commands on which the controller
performed an immediate abort as a result of processing this Cancel command. A value of 0h indicates that
the controller did not perform an immediate abort on any commands as a result of p rocessing this Cancel
command.
If the Cancel Action (refer to Figure 557) specified a Single Command Cancel and an immediate abort was
performed on the specified command, then this field shall be set to 1h.
7.2 Flush command
The Flush command is used to request that the contents of volatile write cache be made non-volatile.
If a volatile write cache is enabled (refer to section 5.1.25.1.4), then the Flush command shall commit data
and metadata associated with the specified namespace(s) to non-volatile storage media. The flush applies
to all commands for the specified namespace(s) completed by the controller prior to the submission of the
Flush command. The controller may also flush additional data and/or metadata from any namespace.
If the Flush Behavior (FB) field is set to 11b in the VWC field in the Identify Controller data structure (refer
to Figure 312) and the specified NSID is FFFFFFFFh, then the Flush command applies to all namespaces
attached to the controller processing the Flush command. If the FB field is set to 10b and the specified
NSID is FFFFFFFFh, then the controller aborts the command with a status code of Invalid Namespace or
Format. If the FB field is cleared to 00b, then the controller behavior if the specified NSID is FFFFFFFFh is
not indicated. Controllers compliant with NVM Express Base Specification, Revision 1.4 and later shall not
set the FB field to the value of 00b.
If a namespace exists in an Endurance Group that has:
• Flexible Data Placement (refer to section 8.1.10) enabled; and
• the Volatile Write Cache Not Present (VWCNP) bit set to ‘1’ in the I/O Command Set Independent
Identify Namespace data structure (refer to Figure 319),
then, even though the Volatile Write Cache field in the Identify Controller data structure (refer to Figure 312)
indicates the presence of a volatile write cache in the controller, this namespace does not have a volatile
write cache present.
A host may use the Volatile Write Cache Not Present (VWCNP) bit in the I/O Command Set Independent
Identify Namespace data structure to determine if a Volatile Write Cache is not present for a namespace.
If a volatile write cache is not present or not enabled, then Flush commands shall have no effect and:
a) shall complete successfully if a sanitize operation is not in progress; and
b) may complete successfully if a sanitize operation is in progress.
All command specific fields are reserved.
 Command Completion
Upon completion of the Flush command, the controller posts a completion queue entry to the associated
I/O Completion Queue.
