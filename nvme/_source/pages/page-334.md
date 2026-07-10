---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 334
headings: []
---

# Source page 334

NVM Express® Base Specification, Revision 2.1

312
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
275:272 O O R
Host Memory Buffer Preferred Size (HMPRE):  This field indicates the preferred size
that the host is requested to allocate for the Host Memory Buffer feature in 4  KiB units.
This value shall be greater than or equal to the Host Memory Buffer Minimum Size. If this
field is non-zero, then the Host Memory Buffer feature is supported. If this field is cleared
to 0h, then the Host Memory Buffer feature is not supported.
If the Host Managed Live Migration Support (HMLMS) bit is set to ‘1’ in one or more
controllers in the NVM subsystem; then this field shall be cleared to 0h.
279:276 O O R
Host Memory Buffer Minimum Size (HMMIN):  This field indicates the minimum size
that the host is requested to allocate for the Host Memory Buffer feature in 4 KiB units. If
this field is cleared to 0h, then the host is requested to allocate any amount of host
memory possible up to the HMPRE value.
295:280 O O R
Total NVM Capacity (TNVMCAP): This field indicates the total NVM capacity that is
accessible by the controller . The value is in bytes. This field shall be supported if the
Namespace Management capability (refer to section 8.1.15) is supported  or if the
Capacity Management capability (refer to section 8.1.4) is supported.
Refer to section 3.8.
311:296 O O R
Unallocated NVM Capacity (UNVMCAP): This field indicates the unallocated NVM
capacity that is accessible by the controller . The value is in bytes. This field shall be
supported if the Namespace Management capability (refer to section 8.1.15) is supported
or if the Capacity Management capability (refer to section 8.1.4) is supported.
Refer to section 3.8.
315:312 O O R
Replay Protected Memory Block Support (RPMBS): This field indicates if the controller
supports one or more Replay Protected Memory Blocks (RPMBs) and the capabilities.
Refer to section 8.1.21.
Bits Description
31:24
Access Size (ASZE): If the Number of RPMB Units field is non-zero, then
this field indicates the maximum number of 512B units of data that may be
read or written per RPMB access by Security Send or Security Receive
commands for the controller. This is a 0’s based value. A value of 0h
indicates support for one unit of 512B of data.
If the Number of RPMB Units field is 0h, then this field should be ignored
by the host.
23:16
Total Size (TSZE): If the Number of RPMB Units field is non-zero, then this
field indicates the number of 128 KiB units of data in each RPMB supported
in the controller. This is a 0’s based value. A value of 0h indicates support
for one unit of 128 KiB of data.
If the Number of RPMB Units field is 0h, then this field should be ignored
by the host.
15:06 Reserved
05:03
Authentication Method (AUTHM): This field indicates the authentication
method used to access all RPMBs in the controller. The values for this field
are:
Value Definition
000b HMAC SHA-256 (refer to RFC 6234)
001b to 111b Reserved

02:00
Number of RPMB Units  (NRPMBU): This field indicates the number of
RPMB targets the controller supports. All RPMB targets supported shall
have the same capabilities as defined in the RPMBS field. A value of 0h
indicates the controller does not support Replay Protected Memory Blocks.
If this value is non-zero, then the controller shall support the Security Send
and Security Receive commands.
