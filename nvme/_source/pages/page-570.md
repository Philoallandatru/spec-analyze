---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 570
headings: []
---

# Source page 570

NVM Express® Base Specification, Revision 2.1

548
programmed) (refer to Figure 635). Once the key is programmed, the RPMB Operation Result Operation
Status field shall not be set to 07h.
An Authenticated Data Write to the RPMB Device Configuration Block data structure that attempts to set
the Boot Partition Write Protection Enabled bit when RPMB Boot Partition Write Protection is not supported
results in an RPMB Operation Result Operation Status of 05h (i.e., Write failure in the RPMB Operation
Status field) (refer to Figure 635).
An Authenticated Data Write to the RPMB Device Configuration Block data structure that attempts to set
the Boot Partition Write Protection Enabled bit when either Boot Partition is in the Write Locked Until Power
Cycle state (refer to section 5.1.25.1.32 and section 8.1.3.3.1) results in an RPMB Operation Result
Operation Status of 05h (i.e., Write failure in the RPMB Operation Status field) (refer to Figure 635).
An Authenticated Data Write to the RPMB Device Configuration Block data structure that attempt s to
change either the Boot Partition 0 Write Locked bit or the Boot Partition 1 Write Locked bit  when the Boot
Partition Write Protection Enabled bit is cleared to ‘0’ results in an RPMB Operation Result Operation Status
field set to 05h (i.e., Write failure) (refer to Figure 635).
If Set Features Boot Partition Write Protection is supported, then an Authenticated Data Write to the RPMB
Device Configuration Block data structure that successfully enables RPMB Boot Partition Write Protection
shall also result in the controller changing  the Boot Partition 0 Write Protection State and Boot Partition 1
Write Protection State values in the Boot Partition Write Protection Config feature to a value of 100b to
indicate that Boot Partition write protection is controlled by RPMB.
The controller may support multiple RPMB targets. RPMB targets are not contained within a namespace.
Controllers in the NVM subsystem may share the same RPMB targets. Security Send and Security Receive
commands for RPMB do not use the namespace ID field; N SID shall be cleared to 0h. Each RPMB target
operates independently – there may be requests outstanding to multiple RPMB targets at once (where the
requests may be interleaved between RPMB targets). In order to guarantee ordering the host should issue
and wait for completion for one Security Send or Security Receive command at a time. Each RPMB target
requires individual authentication and key programming. Each RPMB target may have its own unique
Authentication Key.
The message types defined in Figure 634 are used by the host to communicate with an RPMB target.
Request Message Types are sent from the host to the controller using Security Send commands. Response
Message Types are retrieved by the host from the controller using Security Receive commands.
Figure 633 defines the RPMB Device Configuration Block data structure – the non-volatile contents stored
within the controller for RPMB target 0.
Figure 633: RPMB Device Configuration Block Data Structure
Bytes Type Description
00 RW
Boot Partition Protection Enable (BPPEE): This field specifies whether Boot Partition Protection
is enabled.
Bits Description
7:1 Reserved
0
Boot Partition Write Protection Enabled  (BPPED): If this bit is set to ‘1’, then
RPMB Boot Partition Write Protection is enabled. If this bit is cleared to ‘0’, then
RPMB Boot Partition Write Protection is disabled or not supported. Once enabled,
the controller shall prevent disabling RPMB Boot Partition Write Protection.

01 RW
Boot Partition Protection State (BPLS): This field specifies the current Boot Partition protection
state when RPMB Boot Partition Write Protection is enabled. This field shall be cleared to 0h unless
RPMB Boot Partition Write Protection is enabled. Refer to section 8.1.3.3.
Bits Description
7:2 Reserved
1
Boot Partition 1 Write Locked (BPP1L): If this bit is set to ‘1’, then Boot Partition 1
(i.e., the BPID bit in Figure 181 is set to 1) is write locked. If this bit is cleared to ‘0’,
then the Boot Partition 1 is write unlocked.
