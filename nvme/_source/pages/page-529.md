---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 529
headings: ["8.1.8.3.1.2 Get Status (Directive Operation 02h)"]
---

# Source page 529

NVM Express® Base Specification, Revision 2.1

507
Figure 607: Streams Directive – Return Parameters Data Structure
Bytes Description
21:20
Stream Granularity Size (SGS):  This field indicates the stream granularity size for the specified
namespace in Stream Write Size (SWS) units. If the NSID value is set to FFFFFFFFh, then this field may
be cleared to 0h.
Refer to the applicable I/O Command Set specification  for how this field is utilized to optimize
performance and endurance.
Namespace and Host Identifier Specific Fields
23:22
Namespace Streams Allocated (NSA): This field indicates the number of stream resources allocated
for exclusive use of the specified namespace.
If the SSID bit is cleared to ‘0’ in the NSSC field, then those exclusive stream resources are shared by
the controller processing the Return Parameters operation and by all other controllers that share the
same non-zero Host Identifier, and are attached to the specified namespace.
If the SSID bit is set to ‘1’, then those exclusive stream resources are shared by all controllers that are
associated with any non-zero Host Identifier and are attached to this namespace.
If this value is non-zero, then the namespace may have up to NSA number of concurrently open streams.
If this field is cleared to 0h, then no stream resources are currently allocated to this namespace and the
namespace may have up to NSSA number of concurrently open streams.
25:24
Namespace Streams Open (NSO) : This field indicates the number of open streams in the specified
namespace.
If the SSID bit is cleared to ‘0’, then this field indicates the number of streams that were opened by the
controller processing the Return Parameters operation and by all other controllers that share the same
non-zero Host Identifier, and are attached to this namespace.
If the SSID bit is set to ‘1’ , then this field indicates the number of streams that were opened by the
controller processing the Return Parameters operation and all other controllers that are associated with
any non-zero Host Identifier and are attached to this namespace.
Note: It is not possible for a host to retrieve the number of open streams using resources allocated to the
specified namespace by other hosts.
31:26 Reserved
 Get Status (Directive Operation 02h)
The Get Status operation returns information about the status of currently open streams for the specified
namespace and the host issuing the Get Status operation. The DSPEC field in command Dword 11 is not
used for this operation.
If the SSID bit  is cleared to ‘0’  in the NSSC field , then the information returned describes only those
resources for the specified namespace that are associated with hosts that are registered with the same
non-zero Host Identifier value as the host issuing the Get Status operation. If the SSID bit is set to ‘1’, then
the information returned describes the resources for the specified namespace that are associated with
hosts that are registered with any non-zero Host Identifier.
If an NSID value of FFFFFFFFh is specified, then the controller shall return information about the status of
currently open streams in the NVM subsystem that use resources which are not allocated for the exclusive
use of a particular namespace. If a stream identifier value being returned is in use by different namespaces,
then that stream identifier shall be returned only once.
Stream Identifier 1 (i.e., returned at offset 03:02) contains the value of the open stream of lowest numerical
value. Each subsequent field contains the value of the next numerically greater stream identifier of an open
stream.
The data structure returned is defined in Figure 608. All fields are specific to the specified namespace if the
NSID value was not set to FFFFFFFFh.
