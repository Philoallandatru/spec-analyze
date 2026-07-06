---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 426
headings: ["5.1.25.1.32 Boot Partition Write Protection Config (Feature Identifier 85h)"]
---

# Source page 426

NVM Express® Base Specification, Revision 2.1

404
in the CTRATT field of the Identify Controller data structure (refer to Figure 312)), then the controller shall
abort the command with a status code of Feature Not Changeable.
If a Set Features command change s the namespace to a write protected state, then the controller shall
commit all volatile write cache data and metadata associated with the specified namespace to non -volatile
storage media as part of transitioning to the write protected state.
5.1.25.1.32 Boot Partition Write Protection Config (Feature Identifier 85h)
This Feature is used by the host to configure Boot Partition write protection states and to determine the
write protection state of both Boot Partitions supported by a controller. Refer to section 8.1.3.3.1 for
definition and behaviors of the Boot Partition write protection states and state transitions. The settings are
specified in Command Dword 11.
This Feature is not saveable (refer to Figure 195). The default value of both the Boot Partition 0 Write
Protection State field and the Boot Partition 1 Write Protection State field after a power cycle is Write
Locked.
If a Get Features command is submitted for this Feature, the attributes specified in Figure 442 are returned
in Dword 0 of the completion queue entry for that command. A controller shall not return a value of 000b
for either the Boot Partition 0 Write Protection State field or the Boot Partition 1 Write Protection State field
as a result of the Get Features command for the Boot Partition Write Protection Config feature.
If a Set Features command is submitted for this Feature with either the Boot Partition 0 Write Protection
State field or the Boot Partition 1 Write Protection State field cleared to 000b, then the controller shall not
change the Boot Partition write protection state for that Boot Partition as part of the Set Features command
completion.
If the Boot Partition Write Protection Enable bit is set to ‘1’ in the RPMB Device Configuration Block data
structure (refer to section 8.1.21), then the controller shall return a value of 100b for both the Boot Partition
0 Write Protection State field and the Boot Partition 1 Write Protection State field as a result of the Get
Features command for the Boot Partition Write Protection Config feature.
If a Set Features command is submitted for this Feature with either the Boot Partition 0 Write Protection
State field or the Boot Partition 1 Write Protection State field set to 100b, then the controller shall abort the
command with a status code of Invalid Field in Command.
Figure 442: Boot Partition Write Protection Config - Command Dword 11
Bits Description
31:06 Reserved
05:03
Boot Partition 1 Write Protection State (BP1WPS): This field specifies the write protection state of
Boot Partition 1.
The default value of this field is Write Locked.
Value Definition
000b1 Change in state not requested
001b Write Unlocked
010b Write Locked
011b Write Locked Until Power Cycle
100b2 Write Protection controlled by RPMB
101b to 111b Reserved
