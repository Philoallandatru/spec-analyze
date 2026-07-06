---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 525
headings: []
---

# Source page 525

NVM Express® Base Specification, Revision 2.1

503
to and in multiples of the Stream Granularity Size. This provides optimal performance and endurance of the
media.
Stream resources are the resources in the NVM subsystem that are necessary to track operations
associated with a specified stream identifier. There are a maximum number of stream resources that are
available in an NVM subsystem as indicated by the Max Stream Limit (MSL) field in the Return Parameters
data structure (refer to Figure 607).
Available NVM subsystem stream resources are stream resources that are not allocated for exclusive use
in any namespace. Available NVM subsystem stream resources are reported in the NVM Subsystem
Streams Available (NSSA) field (refer to Figure 607) and may be used by any host in any namespace that:
• has the Streams Directive enabled;
• has not been allocated exclusive stream resources by that host if the Shared Stream Identifiers
(SSID) bit is cleared to ‘0’ in the NSSC field; and
• has not been allocated exclusive stream resources by any host if the SSID bit is set to ‘1’.
Each time stream resources are allocated for exclusive use in a specified namespace, the available NVM
subsystem stream resources reported in the NSSA field are reduced.
For a given namespace:
a) a host allocates stream resources to that namespace for the exclusive use of that host(s) by issuing
the Allocate Resources operation;
b) other hosts may concurrently allocate stream resources to that namespace for their exclusive use;
and
c) hosts which have not allocated stream resources to that namespace may use available NVM
subsystem stream resources for access to that namespace.
If an NVM subsystem has streams resources allocated to a host with a Host Identifier value of 0h and the
Host Identifier is subsequently changed to a non -zero value, then those streams resources remain
associated with the Host with a Host Identifier value of 0h and are not associated with the host with the
non-zero Host Identifier.
The Directive operations that shall be supported if the Streams Directive is supported are listed in Figure
604. The Directive Specific field in a command is referred to as the Stream Identifier when the Directive
Type field is set to the Streams Directive.
Figure 604: Streams – Directive Operations
Directive
Command
Directive Operation
Name
Directive Operation
Value Reference
Directive Receive
Return Parameters 01h 8.1.8.3.1.1
Get Status 02h 8.1.8.3.1.2
Allocate Resources 03h 8.1.8.3.1.3
Reserved All other values
Directive Send
Release Identifier 01h 8.1.8.3.2.1
Release Resources 02h 8.1.8.3.2.2
Reserved All other values
Stream identifiers are assigned by the host and may be in the range 0001h to FFFFh. The host may specify
a sparse set of stream identifiers (i.e., there is no requirement for the host to use Stream Identifiers in any
particular order).
The host may access a namespace through multiple controllers in the NVM subsystem. The controllers in
an NVM subsystem indicate in the SSID bit (refer to Figure 607) if a stream identifier is unique based on
the Host Identifier (i.e., the same stream identifier used to access the same namespace by a host that has
registered a different Host Identifier is referencing a different stream), or if a stream identifier may be used
by multiple Host Identifiers (i.e., the same stream identifier used to access the same namespace by a host
