---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 277
headings: []
---

# Source page 277

NVM Express® Base Specification, Revision 2.1

255
If the Selected Configuration field is cleared to 0h, then the following fields in each Media Unit Status
Descriptor shall be cleared to 0h:
a) Endurance Group Identifier (ENDGID);
b) NVM Set Identifier (NVMSETID);
c) Capacity Adjustment Factor; and
d) Number of Channels (MUCS).
Channel Identifier values begin with 0h and increase sequentially. If the NVM subsystem supports multiple
domains, then Channel Identifier values are unique within the specified domain. If the NVM subsystem does
not support multiple domains, then Channel Identifier values are unique within the NVM subsystem.
In the Media Unit Status Descriptor, Channel Identifiers are listed in ascending order by value, and each
Channel Identifier shall appear only once.
Figure 255: Media Unit Status Descriptor
Bytes Description
Header
01:00 Media Unit Identifier (MUID): This field indicates the identifier of the Media Unit.
03:02
Domain Identifier  (DID): This field indicates the identifier of the domain that contains this
Media Unit. Refer to section 3.2.5.3. A value of 0h indicates that a Domain Identifier is not
reported. If multiple domains are not supported, then this field shall be cleared to 0h.
05:04
Endurance Group Identifier (ENDGID): This field indicates the identifier of the Endurance
Group that contains this Media Unit. Refer to section 3.2.3. The value shall be less than or
equal to the value of the Endurance Group Identifier Maximum field (refer to Figure 312). A
value of 0h indicates that this Media Unit is not part of an Endurance Group.
07:06
NVM Set Identifier (NVMSETID):  This field indicates the identifier of the NVM Set that
contains this Media Unit. Refer to section 3.2.2. This field shall indicate a value less than or
equal to the value of the NVM Set Identifier Maximum field (refer to Figure 312).
If the controller does not support NVM Sets, then this field shall be cleared to 0h.
09:08
Capacity Adjustment Factor (CAF): This field indicates the capacity adjustment factor (refer
to section 8.1.4.1) for this Endurance Group.
A value of FFFFh indicates that value and all higher values.
A value of 0h indicates that the Capacity Adjustment Factor is not reported.
All Media Unit Status Descriptors which indicate the same Endurance Group Identifier shall
indicate the same value in their Capacity Adjustment Factor fields.
10
Available Spare (AVSP): Contains a normalized percentage (0 to 100%) of the remaining
spare capacity available for the Media Unit. The relationship between this value and the value
in the Available Spare field in the Endurance Group Information log page (refer to section
5.1.12.1.10) is outside the scope of this specification.
11
Percentage Used  (PUSED): Contains a vendor specific estimate of the percentage of life
used for the Media Unit based on the actual usage and the manufacturer’s prediction of NVM
life. A value of 100 indicates that the estimated endurance of the NVM in the Media Unit has
been consumed, but may not indicate an NVM failure. The value is allowed to exceed 100.
Percentages greater than 254 shall be represented as 255. This value shall be updated once
per power-on hour when the controller is not in a sleep state.
Refer to the JEDEC JESD218B-02 standard for SSD device life and endurance measurement
techniques.
The relationship between this value and the value in the Percentage Used field in the
Endurance Group Information log page is outside the scope of this specification.
12
Number of Channels ( MUCS): This field indicates the number of Channels attached to this
Media Unit. If this field is cleared to 0h, then no Channel Identifiers are reported for this Media
Unit.
13
Channel Identifiers Offset (CIO):  This field indicates the offset of the Channel 0 Identifier
field from the beginning of the Media Unit Status Descriptor. This field shall be a non -zero
value and a multiple of 16.
CIO-1:14 Reserved
