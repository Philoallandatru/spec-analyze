---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 527
headings: []
---

# Source page 527

NVM Express® Base Specification, Revision 2.1

505
The value of Namespace Streams Allocated (NSA) indicates how many resources for individual stream
identifiers have been allocated for exclusive use for the specified namespace by the associated controllers.
This indicates the maximum number of stream identifiers that may be open at any given time in the specified
namespace by the associated controllers. To request a different number of resources than are currently
allocated for exclusive use by the associated controllers for a specific namespace, all currently allocated
resources are first required to be released using the Release Resources operation. There is no mechanism
to incrementally increase or decrease the number of allocated resources for a given namespace.
Streams are opened by the controller when the host issues a Write command that specifies a stream
identifier that is not currently open. While a stream is open the controller maintains context for that stream
(e.g., buffers for associated data). The host may determine the streams that are open using the Get Status
operation.
For a namespace that has a non-zero value of Namespace Streams Allocated (NSA), if the host submits a
Write command specifying a stream identifier not currently in use and stream resources are exhausted,
then an arbitrary stream identifier for that namespa ce is released by the controller to free the stream
resources associated with that stream identifier for the new stream. The host may ensure the number of
open streams does not exceed the allocated stream resources for the namespace by explicitly releasing
stream identifiers as necessary using the Release Identifier operation.
For a namespace that has zero namespace stream resources allocated, if the host submits an I/O command
specifying a stream identifier not currently in use and:
• NVM subsystem streams available are exhausted, then an arbitrary stream identifier for an arbitrary
namespace that is using NVM subsystem stream resources is released by the NVM subsystem to
free the stream resources associated with that stream identifier for the new stream; or
• all NVM subsystem stream resources have been allocated for exclusive use for specific
namespaces, then the Write command is treated as a normal Write command that does not specify
a stream identifier.
The host determines parameters associated with stream resources using the Return Parameters operation.
The host may get a list of open stream identifiers using the Get Status operation.
If the Streams Directive becomes disabled for use by a host within a namespace, then all stream resources
and stream identifiers shall be released for that host for the affected namespace. If the host issues a Format
NVM command, then all stream identifiers for all open streams for affected namespaces shall be released.
If the host deletes a namespace, then all stream resources and all stream identifiers for that namespace
shall be released. If the write protection state of a namespace changes such that the namespace becomes
write protected (refer to section 8.1.16), then the controller shall release all stream resources and stream
identifiers for that namespace.
Streams Directive defines the command specific status values specified in Figure 606.
Figure 606: Streams Directive – Command Specific Status Values
Value Definition
7Fh Stream Resource Allocation Failed: The controller was not able to allocate stream resources for
exclusive use of the specified namespace and no NVM subsystem stream resources are available.
The Streams Directive is not allowed to be enabled in a ny namespace that is contained in an Endurance
Group with Flexible Data Placement enabled . If the specified namespace is contained in an Endurance
Group that has Flexible Data Placement enabled, then the controller shall not enable the Streams Directive.
If:
• a Directive Send command specifies the Streams Directive in the DTYPE field and the Directive
Operation field is set to 01h (i.e., Enable Directive); and
• the namespace specified by the NSID field  is contained in an Endurance Group that has Flexible
Data Placement enabled,
then the controller shall:
