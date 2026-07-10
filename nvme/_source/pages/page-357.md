---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 357
headings: []
---

# Source page 357

NVM Express® Base Specification, Revision 2.1

335
the NSID field is set to FFFFFFFFh, then the controller may abort the command with a status code of Invalid
Namespace or Format.
Figure 319: Identify – I/O Command Set Independent Identify Namespace Data Structure
Bytes O/M1 Description Reported2
00 M
Common Namespace Features (NSFEAT): This field defines features of the
namespace.
Bits Description
7:6 Reserved
5
Volatile Write Cache Not Present (VWCNP): This bit indicates that a
volatile write cache is not present for this namespace.
If this bit is set to ‘1’, then there is no volatile write cache present for this
namespace and the host should ignore the Volatile Write Cache
Present bit in the Identify Controller data structure (refer to Figure 312)
for this namespace.
If this bit is cleared to ‘0’, then the presence of a volatile write cache for
this namespace is defined in the Volatile Write Cache bit in the Identify
Controller data structure (refer to Figure 312).
4
Rotational Media (RMEDIA): If this bit is set to ‘1’, then the namespace
stores data on rotational media (refer to section 8.1.23). If this bit is
cleared to ‘0’, then the namespace does not store data on rotational
media.
3
UID Reuse (UIDREUSE):  If this bit is set to ‘1’ , then the value in the
NGUID field for this namespace, if non -zero, is never reused by the
controller and that the value in the EUI64 field for this namespace, if
non-zero, is never reused by the controller. If this bit is cleared to ‘0’,
then the NGUID value m ay be reused and the EUI64 value may be
reused by the controller for a new namespace created after this
namespace is deleted. This bit shall be cleared to ‘0’ if both NGUID and
EUI64 fields are cleared to 0h. Refer to section 4.7.1.
2:0 Reserved

No
01 O
Namespace Multi-path I/O and Namespace Sharing Capabilities (NMIC): This
field specifies multi -path I/O and namespace sharing capabilities of the
namespace.
Bits Description
7:2 Reserved
1
Dispersed Namespace (DISNS):  If this bit is set to ‘1’, then the
namespace is a dispersed namespace (refer to section 8.1.9). If this bit
is set to ‘1’, then the namespace shall be a shared namespace (i.e.,
the Shared Namespace (SHRNS) bit shall also be set to ‘1’). If this bit
is cleared to ‘0’, then the namespace is not a dispersed namespace.
0
Shared Namespace (SHRNS):  If this bit is set to ‘1’, then the
namespace may be attached to two or more controllers in the NVM
subsystem concurrently (i.e., may be a shared namespace). If this bit
is cleared to ‘0’, then the namespace is a private namespace and is
able to be attached to only one controller at a time.

Yes
