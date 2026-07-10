---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 528
headings: ["8.1.8.3.1 Directive Receive", "8.1.8.3.1.1 Return Parameters (Directive Operation 01h)"]
---

# Source page 528

NVM Express® Base Specification, Revision 2.1

506
• abort that Directive Send command with a status code of Invalid Field in Command; and
• indicate the Directive Operation field in the Parameter Error Location field (refer to Figure 205) if
an entry is added to the Error Information log page due to aborting that Directive Send command.
8.1.8.3.1 Directive Receive
This section defines operations used with the Directive Receive command for the Streams Directive.
 Return Parameters (Directive Operation 01h)
The Return Parameter operation returns a data structure that specifies the features and capabilities
supported by the Streams Directive, including namespace specific values. The DSPEC field in command
Dword 11 is not used for this operation. The data struc ture returned is defined in Figure 607. If an NSID
value of FFFFFFFFh is specified, then the controller:
• returns the NVM subsystem specific values;
• may return any namespace specific values that are the same for all namespaces (e.g., SWS); and
• clears all other namespace specific fields to 0h.
Figure 607: Streams Directive – Return Parameters Data Structure
Bytes Description
NVM Subsystem Specific Fields
01:00 Max Streams Limit (MSL): This field indicates the maximum number of concurrently open streams that
the NVM subsystem supports. This field returns the same value independent of specified namespace.
03:02
NVM Subsystem Streams Available (NSSA): This field indicates the number of NVM subsystem stream
resources available. These are the stream resources that are not allocated for the exclusive use by a
host in any specific namespace. This field returns the same value independent of specified namespace.
05:04
NVM Subsystem Streams Open (NSSO): This field indicates the number of open streams in the NVM
subsystem that are not associated with a namespace for which resources were allocated using an
Allocate Resources operation. This field returns the same value independent of specified namespace.
06
NVM Subsystem Stream Capability (NSSC):  This field indicates the stream capabilities of the NVM
subsystem.
Bits Description
7:2 Reserved
1
Streams Require Non-Zero Host Identifier (SRNZID): This bit indicates whether the Host
Identifier is required to be set to a non-zero value before the Streams Directive is able to be
enabled. If this bit is cleared to ‘0’, then the Host Identifier is not required to be set to a non -
zero value before the Streams Directive is able to be enabled. If this bit is set to ‘1’, then the
Host Identifier is required to be set to a non -zero value before the Streams Directive is able
to be enabled.
0
Shared Stream Identifiers (SSID): This bit indicates whether stream identifiers may be
shared by multiple Host Identifiers, or if a stream identifier is associated with a single Host
Identifier. If this bit is cleared to ‘0’, then the stream identifier is associated with a single non-
zero Host Identifier. If this bit is set to ‘1’, then the stream identifier may be associated with
multiple non-zero Host Identifiers.

15:07 Reserved
Namespace Specific Fields
19:16
Stream Write Size (SWS): This field indicates the alignment and size of the optimal stream write as a
number for the specified namespace  where the unit  of granula rity is specified by the applicable I/O
Command Set. The size indicated should be less than or equal to Maximum Data Transfer Size (MDTS)
that is specified in units of minimum memory page size. SWS may change if the namespace is
reformatted with a different User Data Format. If the NSID value is set to FFFFFFFFh, then this field may
be cleared to 0h if a single user data size cannot be indicated.
Refer to the applicable I/O Command Set specification  for how this field is utilized to optimize
performance and endurance.
