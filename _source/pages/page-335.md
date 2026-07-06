---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 335
headings: []
---

# Source page 335

NVM Express® Base Specification, Revision 2.1

313
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
317:316 O O R
Extended Device Self -test Time (EDSTT):  If the Device Self -test command is
supported, then this field indicates the nominal amount of time in one minute units that
the controller takes to complete an extended device self -test operation when in power
state 0. If the Device Self-test command is not supported, then this field is reserved.
318 O O R
Device Self -test Options (DSTO) : This field indicates the optional Device Self -test
command or operation behaviors supported by the controller or NVM subsystem.
 Bits Description
7:2 Reserved
1
Host-Initiated Refresh Support (HIRS):  If this bit is set to ‘1’, then the
controller support s the Host -Initiated Refresh capability (refer to section
8.1.11). If clear ed to ‘0’, then the controller does not support the Host -
Initiated Refresh capability.
This bit shall be cleared to ‘0’ if the controller does not support the Device
Self-test command (i.e., the Device Self -test Supported (DSTS) bit in the
OACS field is cleared to ‘0’).
If the controller does not support the Device Self -test command (i.e., the
DSTS bit in the OACS field is cleared to ‘0’), then This bit shall be cleared to
‘0.
0
Single Device Self-test Operation (SDSO): If this bit is set to ‘1’, then the
NVM subsystem supports only one device self -test operation in progress at
a time. If this bit is  cleared to ‘0’, then the NVM subsystem supports one
device self-test operation per controller at a time.

319 M M R
Firmware Update Granularity (FWUG): This field indicates the granularity and
alignment requirement of the firmware image being updated by the Firmware Image
Download command (refer to section 5.1.9). If the values specified in the NUMD field or
the OFST field in the Firmware Image Download command do not conform to this
granularity and alignment requirement, then the firmware update may abort with a status
code of Invalid Field in Command. For the broadest interoperability with host software, it
is recommended that the controller set this value to the lowest value possible.
The value is reported in 4  KiB units (e.g., 1h corresponds to 4  KiB, 2h corresponds to
8 KiB). A value of 0h indicates that no information on granularity is provided. A value of
FFh indicates there is no restriction (i.e., any granularity and alignment in dwords is
allowed).
321:320 M M O
Keep Alive Support (KAS): This field indicates the granularity of the KATO field in 100
millisecond units (refer to section 5.1.25.1.8). If this field is cleared to 0h, then the Keep
Alive Timer feature is not supported. The Keep Alive Timer feature is used by the Keep
Alive capability as described in section 3.9.
323:322 O O R
Host Controlled Thermal Management Attributes (HCTMA): This field indicates the
attributes of the host controlled thermal management feature. Refer to section 8.1.17.5.
 Bits Description
15:1 Reserved
0
Host Controlled Thermal Management Support (HCTMS): Iif this bit is set
to ‘1’, then the controller supports host controlled thermal management. If
this bit is cleared to ‘0’, then the controller does not support host controlled
thermal management. If this bit is set to ‘1’, then the controller shall support
the Set Features command and Get Features command with the Feature
Identifier field set to 10h.

325:324 O O R
Minimum Thermal Management Temperature (MNTMT):  This field indicates the
minimum temperature, in Kelvins, that the host may request in the Thermal Management
Temperature 1 field and Thermal Management Temperature 2 field of a Set Features
command with the Feature Identifier field set to 10h. A value of 0h indicates that the
controller does not report this fi eld or the host controlled thermal management feature
(refer to section 8.1.17.5) is not supported.
